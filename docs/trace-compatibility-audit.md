# OpenCode Plugin Trace Compatibility Audit

Original audit: 2026-07-09 (maida-ai/opencode-plugin#5)

Last verified: 2026-09-29 against the Maida 0.6.0 line.

## Current Output

The plugin writes local Maida trace data under `<data_dir>/runs/<trace_id>/` as `meta.json` and `spans.jsonl`. `MAIDA_DATA_DIR` selects `<data_dir>` when set; otherwise the Maida default local storage directory is used. The plugin does not upload traces.

- `src/session.ts` delegates storage to `@maida-ai/core` through `createRun`, `appendSpan`, `appendEvent`, and `finalizeRun`.
- The package and lockfile resolve `@maida-ai/core` 0.6.0 and require Node.js 24 or newer.
- New `meta.json` files declare `spec_version: "0.2.0"`, a 32-hex `trace_id`, status, timing, and counts.
- `spans.jsonl` contains one JSON span per line for LLM calls, tool calls, errors, loop warnings, and the run-root summary span. Span records do not need their own `spec_version`.
- New traces do not use the legacy `run.json` and `events.jsonl` layout; tests assert those files are absent.

## Maida 0.6.0 Verification

- The plugin's 26 tests, TypeScript lint, and build pass after a clean install from the lockfile with the published `@maida-ai/core` 0.6.0 package.
- A regression demo trace produced through the plugin hooks was loaded by Python source at the Maida `v0.6.0` tag using `load_validated_run()` and projected by `load_run_for_analysis()`. It declared `spec_version: "0.2.0"`, status `ok`, eight spans, and `RUN_START`, `TOOL_CALL`, `LOOP_WARNING`, `LLM_CALL`, and `RUN_END` events.

## Cross-Repo Conformance Fixtures

The repository includes traces under `tests/fixtures/traces/` for a normal run, a repeated-tool loop, a run with no terminal state, and a malformed trace. `tests/fixtures.test.ts` checks the expected structure with the TypeScript reader, including rejection of the malformed trace.

The fixtures intentionally declare the accepted legacy `spec_version: "0.2"` spelling in `meta.json` and omit it from individual spans. This exercises reader compatibility separately from new traces, which declare `0.2.0`.

## Original Audit Followups

- https://github.com/maida-ai/opencode-plugin/issues/6 updated the writer and compatibility documentation.
- https://github.com/maida-ai/opencode-plugin/issues/7 added the conformance fixtures and their tests.
