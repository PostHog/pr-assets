# pr-assets

Storage for screenshots and images embedded in PostHog PR descriptions, so engineers and their agents can reference images from `gh`-scriptable URLs.

> **Everything in this repo is public.** GitHub renders PR images through its anonymous camo proxy, so this repo must stay public for embeds to work. Never upload customer data, secrets, tokens, or internal-only information.

## Upload

This repo requires verified signed commits. Uploaders that use the GitHub Contents API create unsigned commits, so they are blocked by the branch rules.

```bash
FILE=path/to/screenshot.png
FILE="$(cd "$(dirname "$FILE")" && pwd)/$(basename "$FILE")"
KEY="$(date +%Y/%m)/$(uuidgen | tr '[:upper:]' '[:lower:]').${FILE##*.}"

git clone git@github.com:PostHog/pr-assets.git pr-assets-upload
cd pr-assets-upload
mkdir -p "$(dirname "$KEY")"
cp "$FILE" "$KEY"
git add "$KEY"
git commit -m "add screenshot"
git push origin main

SHA=$(git rev-parse HEAD)
echo "![screenshot](https://raw.githubusercontent.com/PostHog/pr-assets/$SHA/$KEY)"
```

Paste the printed markdown into your PR description. Any PostHog org member can push, as long as their git signing setup is working.

If an uploader fails with HTTP 409 and mentions `Commits must have verified signatures`, use the signed git workflow above. The same rule can block `gh api` and agent upload helpers.

## Conventions

- Paths are `YYYY/MM/<uuid>.<ext>`. Random names avoid collisions; date dirs keep the tree browsable and prunable.
- Embed URLs are pinned to the commit SHA, so images keep rendering even if the file is later moved or deleted.
- Concurrent pushes can occasionally race. Pull `main`, then push again.
- Images only, a few MB each. Videos and large binaries don't belong here.

## Example

This 16x16 test image was uploaded with the command above; its file was later deleted from `main`, yet the SHA-pinned URL still renders:

![example-embed](https://raw.githubusercontent.com/PostHog/pr-assets/cfdadbcab2a9d2d3e2d206a8e2bc694f65f8804d/2026/07/67d06e3d-3fe8-4ed0-b183-8bff80cd77bb.png)
