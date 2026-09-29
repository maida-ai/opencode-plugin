# Normal Plugin Session

Expected behavior:

- Accepted legacy `spec_version: "0.2"` metadata.
- Completed `ok` run.
- One LLM call and one tool call.
- A terminal run-root span with `parent_span_id: null`.
