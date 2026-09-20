---
title: "MCP Elicitation: The Second Sign-In Problem"
subtitle: ""
slug: "mcp-elicitation"
tags: ["mcp", "oauth", "authentication", "ai-agents"]
hashnode_url: https://hashnode.com/draft/6a90c4e2808ee0ab726f15b2
---
<!-- copy from here -->

A few weeks into building an MCP server, I thought the identity problem was solved. I sign in through Google, the MCP server validates the token at the door, every tool call after that runs as me. Clean.

Then one of the tools tried to save a page to Notion, and it just failed. Not an auth error I recognized, not a bug in the tool code. I was authenticated. The server knew exactly who I was. Notion had never heard of me.

That's the moment it stopped being a solved problem.

## One login isn't one identity

The setup: my MCP server sits behind Google for sign-in, so getting past the front door just means Google recognizes me. But the tool that writes to Notion needs a Notion-scoped credential, and Google signing me in has nothing to do with whether Notion will let me create a page there. These are two separate identity boundaries. My MCP server only ever solved the first one.

This isn't a corner case you hit once during setup. It's structural. Any MCP server that's a thin wrapper over a real backend will have tools that reach past the front-door identity into some other system with its own authorization rules. The 401 challenge at the server boundary tells you nothing about what happens three tool calls later.

![Google gets you past the MCP server's front door; Notion has no idea who you are yet](https://raw.githubusercontent.com/SiddhanthNB/under-the-abstraction/main/articles/drafts/mcp-elicitation/images/identity-boundaries.png)

Solid line: the boundary the front door actually solves. Dashed line: the one it doesn't, and never claimed to.

## Elicitation is the answer, technically

The MCP spec actually has a name for this now. It splits authorization into two lifecycles: MCP Authorization, the OAuth handshake between the client and the server, and External Authorization, whatever the server needs to do against a third-party resource. The mechanism for the second one is elicitation, specifically URL mode: the server hands the client a URL, the client sends the user there to sign in, and the resulting third-party token never transits back through the client. It's a deliberate security boundary, not an oversight.

Concretely, when a tool has no credential for that second identity provider, the server sends this back instead of a result:

```json
{
  "method": "elicitation/create",
  "params": {
    "mode": "url",
    "url": "https://mcp-server.example.com/connect/second-idp?state=...",
    "message": "Connect your account to continue"
  }
}
```

The client's response just means the user agreed to open that link, not that sign-in is done:

```json
{ "action": "accept" }
```

So the fix is exactly what the spec describes. The tool call fails for lack of a credential from the second identity provider, the server issues an elicitation request with a sign-in URL, the client shows it to me, I authorize, the tool call gets retried. On paper this is a solved problem.

![The full elicitation round trip: tool call fails, server asks for a sign-in URL, user authorizes with the second IdP, tool call retries](https://raw.githubusercontent.com/SiddhanthNB/under-the-abstraction/main/articles/drafts/mcp-elicitation/images/elicitation-sequence.png)

Worth being precise about that last retry step: the server isn't polling anything, and it doesn't push a notification back either. Once the user finishes signing in, nothing happens on its own, the user has to go try the tool call again themselves. Each retried tool call is then a fresh, stateless check: the server looks at whatever it has stored and either finds a usable token now or it doesn't. That's a deliberate choice, consistent with the broader move toward a stateless MCP spec: no session state to keep, no open connection to hold while waiting on an OAuth flow that might take a few seconds or might never finish at all, and any retry can land on any server instance behind a load balancer.

## Where it actually breaks

Two things get in the way.

First, client support is uneven. URL mode elicitation is recent enough that plenty of MCP clients don't render it yet, or render it inconsistently. You can build the server side correctly and still have nowhere for the prompt to land, depending on what the user happens to be running.

Second, and this is the one that actually bit me: elicitation assumes the host has a face. The spec's own architecture draws a line between host, client, and server, and it's the host that owns the UI. That's a fine assumption for a desktop app or a chat interface. It falls apart the moment your MCP host is embedded in something backend, a queued job, an orchestration service, a bot with no human watching in real time. There's no browser to open. The elicitation request has nowhere to go.

![User talks to the frontend, the frontend talks to a backend that embeds the MCP host and client, and the MCP server sits right next to that backend](https://raw.githubusercontent.com/SiddhanthNB/under-the-abstraction/main/articles/drafts/mcp-elicitation/images/headless-host.png)

The backend is the MCP host, the MCP server sits right next to it, and neither of them has a face. The elicitation request lands there and dead-ends, it has to travel all the way back out through the frontend before the user ever sees it. The spec describes the ideal case, a host with a face. Real systems are often messier than that.

When that happens, don't treat it as a dead end, treat it as a routing problem, and not a clean one. The missing credential is a structured, retryable failure, but resolving it still needs the user to eventually complete the sign-in. You haven't removed that requirement, you've only relocated where the user has to show up. That means the job has to already know which user it's running for and have a channel to reach that user: a dashboard entry, a notification, a chat thread further up the stack. Retry the tool call once the user signs in there. And if the user never shows up, the job stays failed. That has to be a designed outcome, not a case you're hoping never happens.

## Don't go looking for the second login early

One thing worth being deliberate about: resolve that second identity lazily, not eagerly. It's tempting to authenticate against every downstream system a session might touch, right at the start, so nothing fails later. Don't. Tokens acquired that way go stale before they're used, or get created for tools the session never actually calls. Trigger the elicitation only when a specific tool call needs that specific credential. That's also just what the primitive is for. Front-loading it defeats the point.

## Making it less annoying

Two ways to cut down on how often that second sign-in interrupts anything.

The first is much less exciting and doesn't depend on anyone's cooperation: once that Notion sign-in completes, cache the resulting token against that identity at the server. Every later tool call checks the cache before doing anything else:

```python
def get_notion_token(user_id):
    token = token_store.get(user_id, "notion")
    if token and not token.is_expired():
        return token
    return None  # no usable token, the tool call falls through to elicitation
```

Only go back to elicitation when there's genuinely nothing usable left, not on every call. And one thing worth being deliberate about here: save the refresh token alongside the access token, if the provider issues one. That's what lets you mint a new access token silently when the old one just aged out, instead of sending the user through elicitation for something that routine:

```python
def get_notion_token(user_id):
    token = token_store.get(user_id, "notion")
    if token and not token.is_expired():
        return token
    if token and token.refresh_token:
        token = notion_oauth.refresh(token.refresh_token)
        token_store.set(user_id, "notion", token)
        return token
    return None  # refresh token is gone too, this is when elicitation actually triggers
```

One sign-in now covers every expiry after it, right up until the refresh token itself is revoked or expires. Basic backend hygiene, and it removes most of the friction without touching either identity provider's configuration.

The second is architectural, and bigger: put a broker between the two identity providers that can exchange a token from one for a token in the other directly, using OAuth token exchange, without the user seeing a prompt at all. It's naturally a sidecar, called on demand rather than baked into the server. The catch is that it only works if the two identity providers actually trust each other for that exchange, which is a configuration and governance question between two systems you may not control, not something you can route around in code. Worth its own piece later.
