# Query Patterns

## Good Vera Queries

```sh
vera search "authentication middleware" --json | jq -r '.[] | "\(.file_path):\(.line_start)-\(.line_end) \(.symbol_type // ""):\(.symbol_name // "")\n\(.content)"'
vera search "JWT token validation" --json | jq -r '.[] | [.file_path, .line_start, .line_end, .symbol_type, .symbol_name] | @tsv'
vera search "parse_config" --json | jq -r '.[].content'
vera search "request rate limiting" --lang rust --json | jq -r '.[].file_path'
vera search "routes" --path "src/**/*.ts" --path "tests/**/*.ts" --json | jq -r '.[].content'
vera search "handler" --type function --limit 5 --json | jq -r '.[] | [.file_path, .line_start, .symbol_name] | @tsv'
vera search "token validation" --changed --json | jq -r '.[].file_path'
vera search "config loading" --base origin/main --json | jq -r '.[].content'
```

## Weak Vera Queries

Single generic words return noise:

- `vera search "code" --json`
- `vera search "utils" --json`
- `vera search "file" --json`

Fix: describe what the code *does*, not what it *is*.

## When To Use `vera references` Instead

For structural queries about call relationships, use `references` or `dead-code` instead of `search`:

```sh
vera references parse_config --json | jq -r '.[] | [.file_path, .line_start, .symbol_name] | @tsv'            # who calls parse_config?
vera references parse_config --callees --json | jq -r '.[] | [.file_path, .line_start, .symbol_name] | @tsv'  # what does parse_config call?
vera dead-code --json | jq -r '.[] | [.file_path, .line_start, .symbol_name] | @tsv'                           # functions with no callers
```

These query the call graph built during indexing (direct calls only, no dynamic dispatch).

## When To Use `vera grep` Instead of `rg`

`vera grep` searches only indexed files, so `.veraignore` and exclusion rules apply automatically. Use it when you want results scoped to the project's indexed corpus.

```sh
vera grep "fn\s+main" --json | jq -r '.[] | [.file_path, .line_start, .content] | @tsv'                       # regex over indexed files
vera grep "TODO|FIXME" -i --json | jq -r '.[].content'                                                        # case-insensitive
vera grep "queryClient|invalidateQueries" --path "frontend/src/**" --json | jq -r '.[].file_path'
vera grep "Authorization" --lang rust --type function --json | jq -r '.[].content'
vera grep "handler" --scope docs --json | jq -r '.[].content'                                                 # scoped to documentation
vera grep "use std::collections" --context 0 --json | jq -r '.[].content'                                     # no surrounding context lines
vera grep "parse" --compact --json | jq -r '.[] | [.file_path, .line_start, .content] | @tsv'                 # signatures only
```

## When To Use `rg` Instead

- File name search: `rg --files | rg "docker"`
- Counting occurrences
- Bulk find-and-replace prep
- Files outside the Vera index

## Narrowing Results

Add one filter at a time:

1. `--lang rust`: restrict to a language
2. `--path "src/auth/**"`: restrict to a path glob; repeat it to OR multiple patterns
3. `--type function`: restrict to symbol type
4. `--limit 3`: fewer, higher-confidence results
5. `--scope source`: restrict to a corpus scope (see SKILL.md for scope table)

## Multi-Query Search

`vera search` accepts multiple quoted queries and merges the results with reciprocal rank fusion:

```sh
vera search "OAuth token refresh" "JWT expiry handling" "auth middleware" --json | jq -r '.[].content'
```

Use this when one phrasing is too narrow but the task is still one coherent search.

## Git-Scoped Search

When the task is limited to modified files or a PR diff, scope the search first:

```sh
vera search "auth middleware" --changed --json | jq -r '.[].content'
vera grep "TODO|FIXME" --changed --json | jq -r '.[].content'
vera overview --base origin/main --json
```

Use:

- `--changed` for modified, staged, and untracked files
- `--since <rev>` for changes since a specific revision
- `--base <rev>` for changes since `merge-base(HEAD, <rev>)`

## Intent-Based Reranking

Add `--intent` when the raw query is short but you know the higher-level goal:

```sh
vera search "config" --intent "find where database connection strings are loaded from environment variables" --json | jq -r '.[].content'
```

Use this when the raw query is too short or ambiguous to capture what you actually need.

## Structural Search

Use `vera structural` for the common structural tasks agents hit repeatedly:

```sh
vera structural definitions parse_config --json | jq -r '.[] | [.file_path, .line_start, .symbol_name] | @tsv'
vera structural env DATABASE_URL --json | jq -r '.[] | [.file_path, .line_start, .content] | @tsv'
vera structural routes --path "src/**" --json | jq -r '.[] | [.file_path, .line_start, .content] | @tsv'
vera structural sql --json | jq -r '.[] | [.file_path, .line_start, .content] | @tsv'
vera structural impls Loader --json | jq -r '.[] | [.file_path, .line_start, .symbol_name] | @tsv'
```

Use `vera structural impls <symbol> --json | jq ...` for explicit inheritance or conformance declarations only. It does not infer implicit interface satisfaction.
