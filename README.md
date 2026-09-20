# Under the Abstraction

A Markdown-first workspace for drafting and archiving technical articles published on Hashnode.

## Structure

- `ideas.md` tracks article ideas in the Inbox and Developing stages.
- `templates/article.md` is the starting point for each article.
- `articles/drafts/<article-slug>/index.md` contains an article in progress.
- `articles/drafts/<article-slug>/images/` contains its images.
- `articles/archive/` stores read-only snapshots of published article folders.

## Workflow

1. Copy the article template to `articles/drafts/<article-slug>/index.md` and add an `images/` directory.
2. Write and revise the article in the draft folder.
3. Publish it manually through the Hashnode UI.
4. Move the complete article folder from `articles/drafts/` to `articles/archive/` and keep it as a read-only snapshot.

## License

Code (scripts, configuration, and code snippets embedded in articles) is licensed under the MIT License, see `LICENSE-MIT`. Written article content is licensed under CC BY-NC-ND 4.0, see `LICENSE-CC-BY-NC-ND`.

## AI Usage

Some articles here were researched and drafted with AI assistance. See `AI_USAGE.md` for the full disclosure.
