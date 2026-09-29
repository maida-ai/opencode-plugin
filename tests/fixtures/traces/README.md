# OpenCode Plugin Trace Fixtures

These fixtures are plugin-owned conformance assets for cross-repo Maida trace
tests. Each fixture directory contains the files that should be copied under
`<data_dir>/runs/<trace_id>/` before a reader calls `loadValidatedRun`.

The valid fixtures exercise the accepted legacy `0.2` spelling of the current `0.2.0` public trace contract. New traces declare `0.2.0`:

- `meta.json` declares the accepted `spec_version: "0.2"` spelling.
- `spans.jsonl` contains one JSON span object per line.
- Span records intentionally do not repeat `spec_version`; the version belongs
  to `meta.json`.

The malformed fixture is intentionally invalid and should fail validation.
