# pr-assets

Storage for screenshots and images embedded in PostHog PR descriptions, so engineers and their agents can reference images from `gh`-scriptable URLs.

> **Everything in this repo is public.** GitHub renders PR images through its anonymous camo proxy, so this repo must stay public for embeds to work. Never upload customer data, secrets, tokens, or internal-only information.

## Upload

```bash
FILE=path/to/screenshot.png
KEY="$(date +%Y/%m)/$(uuidgen | tr '[:upper:]' '[:lower:]').${FILE##*.}"
SHA=$(base64 < "$FILE" | tr -d '\n' | gh api -X PUT "repos/PostHog/pr-assets/contents/$KEY" \
  -f message="add screenshot" -F content=@- --jq '.commit.sha')
echo "![screenshot](https://raw.githubusercontent.com/PostHog/pr-assets/$SHA/$KEY)"
```

Paste the printed markdown into your PR description. Any PostHog org member's `gh` auth works — no clone, no branch, one API call.

## Conventions

- Paths are `YYYY/MM/<uuid>.<ext>`. Random names avoid collisions; date dirs keep the tree browsable and prunable.
- Embed URLs are pinned to the commit SHA, so images keep rendering even if the file is later moved or deleted.
- Pipe the base64 via stdin (`-F content=@-`) as above — passing it as an argument fails for files over ~1 MB (ARG_MAX).
- Concurrent uploads can occasionally return HTTP 409; retry once.
- Images only, a few MB each. Videos and large binaries don't belong here.
