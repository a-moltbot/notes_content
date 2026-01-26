# notes_content

This repo stores **Hugo content only** (Markdown + front matter).

It is designed to work with a separate Hugo site repo (e.g. `notes_hugo` / your current `notes_clawd`) that:
- checks out this repo during CI
- copies `content/` into the Hugo site
- builds + deploys to GitHub Pages

## Structure

- `content/` — Hugo content tree (e.g. `content/posts/*.md`)

## Writing

Add / edit Markdown files under `content/`.

## Triggering deploy

On every push to `main`, this repo triggers a build in the Hugo site repo using `repository_dispatch`.

### Required secret (in THIS repo)

Create a fine-grained PAT and add it as a repo secret:
- Name: `NOTES_HUGO_DISPATCH_TOKEN`
- Permissions: access to the target Hugo site repo, sufficient to trigger workflows (repository dispatch)

Also set (either as repo Variables or edit the workflow file):
- `NOTES_HUGO_REPO` (default: `Zilong-L/notes_clawd`)

