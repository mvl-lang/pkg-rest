# ADR-0001: REST Client Design — JSON-in/JSON-out over pkg.tls.https

**Status:** Accepted
**Date:** 2026-06-18
**Context:** pkg-rest needs a typed REST client that handles JSON serialization, IFC boundary crossing for tainted HTTP response bodies, and HTTP error propagation — while staying composable with the existing `pkg.tls.https` transport layer.

## Decision

**The REST client is a pure adapter layer: it accepts `Value` (from `std.json`), serializes to `String`, delegates to `pkg.tls.https`, and deserializes the response `String` back to `Value`.** No HTTP framing, TLS, or connection management lives in this package.

### Key design choices

1. **Single IFC detaint point per response path.** Response bodies are `Tainted[String]` at the HTTPS boundary. `parse_json_body` is the only call site of `trust(...)`, audited as `"REST-JSON-RESPONSE"`. Error bodies (non-2xx) are detainted separately as `"REST-ERROR-DISPLAY"` and truncated to 512 bytes before being surfaced in `RestError::ApiError`.

2. **Header merging is caller-controlled.** Default JSON headers (`Content-Type: application/json`, `Accept: application/json`) are produced by `json_headers()` and then merged with caller-supplied `extra_headers` via `merge_headers`. Extra headers win on conflict — this is the expected override semantics for auth tokens, tracing headers, etc.

3. **`RestError` wraps rather than re-exports `HttpsError`.** Callers that need to inspect transport failures use `RestError::HttpError(he)` and then match on `he`. This keeps the error type self-contained and avoids leaking `pkg.tls.https` types into call sites that only want to pattern-match on REST-level failures.

4. **No retry, no timeout, no connection pooling.** These are cross-cutting concerns that belong at a higher layer (e.g., a service-level client built on top of pkg-rest). Keeping pkg-rest thin makes it easier to reason about and test.

### Public surface

```mvl
pub type RestError = enum {
    HttpError(HttpsError),
    JsonParseError(String),
    ApiError(Int, String),
}

pub total fn rest_error_msg(e: val RestError) -> String
pub partial fn rest_post_json(url, extra_headers, body) -> Result[Value, RestError] ! Net
pub partial fn rest_get_json(url, extra_headers)      -> Result[Value, RestError] ! Net
```

### IFC audit tags

| Tag | Where | Meaning |
|---|---|---|
| `REST-JSON-RESPONSE` | `parse_json_body` | Parsed JSON from a 2xx response |
| `REST-ERROR-DISPLAY` | `rest_post_json`, `rest_get_json` | Error body surfaced in `ApiError` |

## Consequences

- Callers receive bare `Value` — they must re-taint any fields that will flow into privileged sinks.
- The 512-byte truncation in `ApiError` is a DoS guard; callers that need the full body must use `pkg.tls.https` directly.
- Adding new HTTP methods (PUT, DELETE, PATCH) follows the same pattern as `rest_get_json`: call `https_request` with the appropriate method string.

## Connected to

- Issue #1000: Initial REST client design
- Issue #1016: IFC boundary for HTTP response bodies
- Issue #1020: Error body truncation
- ADR-0002: Explicit `total fn` annotation policy
