# OpenCode Plugin Trace Compatibility Audit

Original audit: 2026-07-09 (maida-ai/opencode-plugin#5)

Last verified: 2026-10-04 against released Maida v0.6.1.

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

## Maida v0.6.1 Verification

- The existing 26 plugin tests, TypeScript lint, and build passed with the checkout's locked `@maida-ai/core` 0.6.0 dependency. No package version or dependency selection was changed.
- The published `maida-ai==0.6.1` package validated and read the normal, tool-loop, and missing-terminal-state fixtures, then projected their expected events. The unfinished fixture remained `running` without `RUN_END`; validation did not turn it into a completed execution. The malformed span-ID fixture was rejected by both the CLI validator and Python reader.
- Fresh good and regression demo traces emitted through the plugin hooks declared `spec_version: "0.2.0"` and were read and projected by the released engine: three tool calls without a loop versus five tool calls with one loop warning. The displayed tool commands and model responses were offline fixtures.
- The first-user README now installs that released engine, requires explicit OpenCode plugin configuration, and selects the plugin's explicit trace ID. Maida v0.6.1's `maida init` / `maida check` first-report path selects initialized Claude capture, not this plugin's runs.
- This verification covers local trace compatibility and offline fixtures. It does not establish live OpenCode capture, answer correctness, or protected GitHub merge enforcement.

## Cross-Repo Conformance Fixtures

The repository includes traces under `tests/fixtures/traces/` for a normal run, a repeated-tool loop, a run with no terminal state, and a malformed trace. `tests/fixtures.test.ts` checks the expected structure with the TypeScript reader, including rejection of the malformed trace.

The fixtures intentionally declare the accepted legacy `spec_version: "0.2"` spelling in `meta.json` and omit it from individual spans. This exercises reader compatibility separately from new traces, which declare `0.2.0`.

## Original Audit Followups

- https://github.com/maida-ai/opencode-plugin/issues/6 updated the writer and compatibility documentation.
- https://github.com/maida-ai/opencode-plugin/issues/7 added the conformance fixtures and their tests.
