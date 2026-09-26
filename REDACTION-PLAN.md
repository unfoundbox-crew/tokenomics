# WorldTrainer Redaction — Reuse Plan (lane 5 of 6)

Status: plan only. No engine work done. Verify against `~/code/unfoundbox/agentworth/crates/redaction/` before building — redaction rules drift.

## What Agentworth already has (verified 2026-09-12, read-only)

Engine: `crates/redaction/` — `lib.rs`, `rules.rs`, `redactor.rs`, `report.rs`, `entropy.rs`.

- **Rule suite** (`rules.rs::default_rules`, 14 rules): PEM private keys, JWTs, Anthropic / OpenAI / Google API keys, GitHub tokens, AWS access keys, Bearer tokens, sensitive env vars (`PASSWORD|SECRET|API_KEY|…=value`, keeps the key name), credential-bearing URLs, home-dir paths (macOS/Linux/Windows → `~`, username only), emails, private IPv4, high-entropy-secret fallback (regex candidates + Shannon-entropy filter so SHAs/UUIDs don't false-positive).
- **Two profiles**: `Redactor::new()` (default, local surfaces) vs `Redactor::for_publication()` (default + `publication_rules`: `~/path-tail` blanking, forge `owner/repo` URLs). Publication exists because a measured 2026-09-10 test showed default-redacted payloads still leaked org/repo names. Plus per-trace `repository_identity_rule` masking the session's own repo/workspace string.
- **Coverage layers** (`redactor.rs`): every `EventPayload` variant — user/assistant text, tool-call args (recursive JSON, keys included), tool results, shell command + cwd + output, file paths + diffs, outcome summaries, errors, model switch, human intervention, compaction, custom — plus `provenance.source_path`, metadata, identity sightings. Numerics (tokens, costs, latencies, exit codes) pass through untouched.
- **Contract**: redacted-by-default everywhere (MCP tools, `session_show`, handoff, asks, forgotten); raw only via per-call `include_raw: true` opt-in. Never a global raw flag.
- **Receipt**: `RedactionReport` — per-category counts, `total_redactions`, `breakdown_by_category`, `merge`, `is_clean`, plus `preview_redactions` (dry-run scan without mutating).

## Thin-layer design: WorldTrainer calls Agentworth

No new engine. WorldTrainer imports `agentworth-redaction` as a library crate (same workspace dependency style as `export-atif`, which already does `Redactor::new()` in its tests) and adds only upload-side policy:

1. **Redact at the upload boundary**: run `Redactor::for_publication().for_trace(&trace)` over every trace before it leaves the machine. `for_publication` (not `new`) because WorldTrainer uploads are outbound by definition.
2. **Per-repo / per-adapter / per-rung upload controls** (new, small config — the only real code): a table mapping (repo, adapter, evidence-rung) → {upload, hold-local, drop}. Default-deny on unknown repos; rung 0 (unflown, no outcome evidence) never uploads content, only counts.
3. **Credit estimates**: tokens/cost fields already survive redaction untouched, so price the upload from `trace.stats` without seeing raw text. Estimate = f(tokens, rung) computed pre-redaction locally, attached alongside the receipt.
4. **Ship the receipt, not just the trace**: attach the `RedactionReport` (counts by category + `is_clean`) to every upload so the receiver can audit what was stripped without ever seeing it.

## Gaps to fill (no engine changes, all additive)

1. **Secret deny-lists (BMS-style)**: the engine has regex rules but no literal deny-list input. Need: caller-supplied extra `RedactionRule`s (via existing `with_rules`/`add_rule`) built from per-repo secrets (private model IDs, internal hostnames, customer names). Small constructor, same pattern as `repository_identity_rule`.
2. **RedactionBench-style fixtures**: `crates/redaction/tests/` has `redactor_test.rs` + `publication_profile_test.rs` — good seed, but no adversarial fixture set for the WorldTrainer upload path (prose-named orgs, near-miss secrets, non-secret hex). Add a fixture corpus + `preview_redactions` assertion per upload profile before enabling any new repo.
3. **Known unfixable (document, don't chase)**: org/repo named in free prose ("we pushed to acme/api") passes through — stated in `rules.rs` docs. Publication stays an operator decision; the per-repo allow-list is the backstop, not a smarter regex.
