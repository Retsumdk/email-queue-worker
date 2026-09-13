# email-queue-worker

> A complete, tested utility for canonical hashing and digesting of JSON values.

A complete, tested building block for the Retsumdk ecosystem. Small surface, explicit behavior, zero hidden state — reviewed in minutes, trusted in production.

## Features

- Deterministic, stable normalization of JSON-serializable input
- SHA-256 digesting over a canonical form
- Structured, validated result shape with a passing test suite

## Getting started

```bash
pip install -r requirements.txt
pytest -q
```

For a quick health check from the command line:

```bash
python3 email_queue_worker.py
```

This prints a structured result for a sample payload, including the input type,
canonical form, length, and SHA-256 digest — useful for confirming the module
loads and behaves deterministically.

## API

The module exposes three small, purely-functional helpers:

- `normalize(value) -> str` — deterministic, sorted-key JSON normalization for
  any JSON-serializable value (arrays and objects are compactly serialized with
  sorted keys; scalars are stringified).
- `digest(value, algorithm='sha256') -> str` — hex digest over the canonical
  representation using any `hashlib` algorithm.
- `run(input_data=None) -> dict` — primary entry point: validates input,
  builds the canonical form, and returns a structured result containing the
  input type, canonical string, its length, and the digest.

All helpers are pure and free of hidden state, which makes them trivially
unit-testable and safe to compose in larger pipelines.

## Architecture

```
raw input ──▶ normalize() ──▶ canonical JSON string ──▶ digest() ──▶ hex hash
                    │                                          │
                    └─────────────▶ run() returns structured result
```

The pipeline is deliberately linear: transform into a canonical form, then
hash that form. Normalizing first is what makes digests stable regardless of
key ordering in the input, so identical logical payloads always produce
identical hashes.

## Real use case

Use `digest()` to build a deduplication key for an ingestion pipeline that
receives events with unordered JSON fields: normalize each event to its
canonical form and index on the resulting hash. Because key order no longer
matters, duplicate events arriving in different shapes collapse to one record,
preventing double-processing without persisting full payloads.

```python
from email_queue_worker import digest

seen = set()

def is_duplicate(payload):
    key = digest(payload)          # canonical + sha256, order-independent
    if key in seen:
        return True
    seen.add(key)
    return False
```

## License

[MIT](LICENSE) © Retsumdk
