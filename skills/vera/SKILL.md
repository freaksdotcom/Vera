---
name: vera
description: Code search over the current repository. Before reading files to answer "where is X", "how does Y work", "find Z", or "what calls W", run Vera with `--json`; use `jq` when filtering, limiting, or formatting the result. Use `vera grep` for exact strings and regex, `vera structural` for definitions, routes, and env reads. Do not read multiple files hoping to find the right one; search first, then read the hit.
---

# Vera

Ranked code search over an indexed repository. Always request JSON; do not rely on Vera's Markdown rendering. Pipe through `jq` only when selecting, limiting, or formatting fields.

Use this location-preserving filter unless the task needs different fields:

```sh
vera search "config object construction" --json | jq -r '.[] | "\(.file_path):\(.line_start)-\(.line_end) \(.symbol_type // ""):\(.symbol_name // "")\n\(.content)"'
```

## Pick the tool

| You are about to... | Do this instead |
|---------------------|-----------------|
| Read files to find where something lives | `vera search "config object construction" --json \| jq ...` |
| Read files to understand how something works | `vera search "env file loading decision" --json \| jq ...` |
| Find documentation on a topic | `vera search "deploying behind proxy" --scope docs --json \| jq ...` |
| Find every occurrence of a pattern | `vera grep "TODO\|FIXME" --json \| jq ...` |
| Find callers or callees of a symbol | `vera references make_config --json \| jq ...` |
| Find definitions, routes, env reads | `vera structural env --json \| jq ...` / `vera structural routes --json \| jq ...` |
| Edit the same pattern in many files | `rg` |
| Read a file you already know | Read it directly |

## Do not use Vera when

- You already know the exact path and line: open the file.
- You are editing across many files mechanically: use `rg`.
- The answer is a literal string you can match: `vera grep` beats `vera search`.
- You have already run two searches that returned the same region: stop searching and read the code.

## Search well

- Search behavior, not nouns: `"JWT expiry handling"`, not `"auth"` or `"utils"`.
- Pass several angles in one call: `vera search "OAuth token refresh" "JWT expiry" "auth middleware" --json | jq ...`.
- Start broad with `--compact` (signatures only, fewer tokens), then narrow with `--lang`, `--path`, `--type`, `--limit`.
- Add `--intent "<goal>"` when the query is vague but the goal is clear.
- Scope to a change with `--changed`, `--since <rev>`, or `--base <rev>` when reviewing a diff.
- `--deep` rewrites the query through an LLM; use it only after normal search misses.

## Treat hits as leads

- A search hit is a lead, not evidence. Before stating how something behaves, open the cited lines.
- Follow the call graph rather than re-searching: `vera references <symbol> --json | jq ...` on a promising hit answers "who drives this" in one step.
- Cite `path:line` from code you actually read.
- After editing files, run `vera update .` before searching again.

## Recovery

| Symptom | Fix |
|---------|-----|
| `no index found` | `vera index .` |
| Stale results after edits | `vera update .` (or `vera watch .`) |
| A file is missing from results | `vera explain-path path/to/file` |
| Local model or ONNX error | `vera doctor --probe`, then `references/troubleshooting.md` |
| Missing local assets | `vera repair` |
| Install, API keys, backends | `references/install.md` |
| MCP server | `references/mcp.md` |

## References

- `references/install.md`: install, setup, API and local config, `.veraignore` rules
- `references/query-patterns.md`: more query examples and rg guidance
- `references/troubleshooting.md`: common errors and fixes
- `references/mcp.md`: optional MCP server usage
