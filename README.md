# mdfm-action

Use [mdfm](https://github.com/sushichan044/mdfm) on GitHub Actions.

## Basic Usage

> [!WARNING]
> mdfm outputs newline-delimited JSON (JSONL format).
>
> When using `jq` to process the output as an array, you must use the `-s` (slurp) option to read the entire input stream into an array.

```yaml
name: Extract Markdown Frontmatter

on: [push]

jobs:
  extract:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract frontmatter
        uses: sushichan044/mdfm-action@v1
        id: mdfm
        with:
          pattern: "**/*.md"

      - name: Display results
        run: |
          echo "Processed $(echo '${{ steps.mdfm.outputs.json }}' | jq -s 'length') files"
          echo '${{ steps.mdfm.outputs.json }}' | jq -s '.[0].frontMatter'
```

## Advanced Usage with File Output

```yaml
- name: Extract blog posts
  uses: sushichan044/mdfm-action@v1
  id: blog-posts
  with:
    pattern: "content/blog/*.md"

- name: Process blog posts
  run: |
    # Filter published posts
    jq '[.[] | select(.frontMatter.published == true)]' ${{ steps.blog-posts.outputs.json-output-file }} > published-posts.json

    # Generate blog index
    jq -r '.[] | "- [\(.frontMatter.title)](\(.path))"' published-posts.json > blog-index.md
```

## Action Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `pattern` | Glob pattern to match Markdown files (e.g., `**/*.md`) | Yes | - |
| `version` | Version of mdfm to use | No | `latest` |
| `github-token` | GitHub token for downloading mdfm binary | No | `${{ github.token }}` |

## Action Outputs

| Output | Description |
|--------|-------------|
| `json` | JSON string containing extracted frontmatter and content |
| `json-output-file` | Path to a file temporary stores the JSON output |
