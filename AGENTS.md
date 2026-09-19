# AGENTS.md

## Purpose

This repository contains drafts and archived articles for the **Under the Abstraction** technical blog.

The GitHub repository is the source of truth for article content. Hashnode is used for drafting metadata, previewing and publishing through its UI.

## Repository Structure

```text
articles/
├── drafts/
└── archive/

templates/
└── article.md

ideas.md
```

Each article uses:

```text
articles/drafts/<article-slug>/
├── index.md
└── images/
    └── .gitkeep
```

## Starting a New Article

Before creating a new article:

1. Ask the author to create an empty draft in Hashnode.
2. Ask the author to provide the Hashnode draft URL.
3. Do not begin the formal article draft until the URL is provided.
4. Create the article folder under `articles/drafts/`.
5. Copy the metadata structure from `templates/article.md`.
6. Add the supplied Hashnode URL to `hashnode_url`.

Use this metadata:

```yaml
---
title: ""
subtitle: ""
slug: ""
tags: []
hashnode_url:
---
```

Do not impose standard article headings. The structure must be designed around the specific topic.

## Writing Process

Before writing the complete article:

1. Discuss the central idea with the author.
2. Identify the intended reader and main takeaway.
3. Propose an outline.
4. Challenge unclear claims, weak reasoning and unnecessary scope.
5. Wait for agreement before expanding the outline into a full article.

Do not agree with the author merely to be agreeable. Give concise, reasoned pushback when an idea can be improved.

The author should remain involved in shaping the argument, examples and final wording.

## Research

Web search may be used to:

* Explore and validate ideas.
* Check current technical behaviour.
* Verify specifications, limitations and terminology.
* Find authoritative supporting sources.

Prefer official documentation, specifications, source repositories and primary sources.

Clearly distinguish:

* Verified facts.
* Reasonable interpretations.
* Personal opinions.
* Unverified assumptions.

Do not invent facts, benchmarks, quotations, personal experiences or production incidents.

## Writing Style

Write in a natural, technically informed human voice.

Avoid:

* Em dashes.
* Generic AI introductions.
* Excessive headings.
* Repetitive summaries.
* Artificial enthusiasm.
* Marketing language.
* SEO filler.
* Unnecessary rhetorical questions.
* Phrases such as “in today’s fast-paced world”, “let’s delve into”, “game-changing”, “unlock”, “seamlessly” and similar clichés.

Prefer:

* Direct sentences.
* Concrete examples.
* Clear opinions.
* Honest limitations.
* Natural transitions.
* Technical depth where it adds value.
* Short explanations before introducing jargon.

Preserve the author’s point of view. Do not flatten the article into generic documentation.

## Confidentiality

Never include confidential, proprietary or employer-specific information unless the author explicitly approves it.

Do not expose:

* Internal system names.
* Company architecture.
* Credentials or tokens.
* Private URLs.
* Customer information.
* Internal incidents.
* Non-public code or documentation.

When using lessons from professional experience, generalize or anonymize the details and confirm the wording with the author.

## Hashnode Workflow

Hashnode and GitHub do not automatically synchronize on the free tier.

The workflow is:

1. Write and review the article in this repository.
2. Ask the author to copy the final Markdown body into the existing Hashnode draft.
3. Do not include YAML metadata in the Hashnode article body.
4. Ask the author to review and publish through the Hashnode UI.
5. Never publish or modify Hashnode content without explicit permission.

After the author confirms publication:

1. Ask for the final public Hashnode URL if it differs from the draft URL.
2. Update `hashnode_url`.
3. Confirm that the local Markdown represents the published version.
4. Move the complete article folder from `articles/drafts/` to `articles/archive/`.
5. Do not move the article before publication is explicitly confirmed.

Archived articles are snapshots. Do not modify them unless the author asks to synchronize a later correction.

## Git Rules

Do not commit, push, amend, rebase or modify remotes unless explicitly requested.

When asked to create a commit:

* Review the relevant diff first.
* Include only changes related to the requested work.
* Use one short, clear, one-line commit message.
* Use the repository’s configured author.
* Do not add `Co-authored-by` or any AI attribution.
* Do not mention the agent in the commit message.

Example:

```text
Add MCP elicitation draft
```

Do not push unless the author explicitly requests it.

## Final Control

The author has final control over:

* Topic selection.
* Article structure.
* Technical opinions.
* Wording.
* Hashnode publishing.
* Moving articles into the archive.
* Git commits and pushes.

When uncertain, ask instead of assuming.
