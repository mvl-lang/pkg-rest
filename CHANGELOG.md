# Changelog

All notable changes to pkg-rest will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.3.0] - 2026-06-20

### Added
- `src/server.mvl` — server-side route builder module
- `RestRouter` type with fluent builder API
- `router()` constructor and `.get()`, `.post()`, `.put()`, `.delete()`, `.patch()`, `.build()` methods
- Dependency on `pkg-http` v0.2.3 for HTTP types and routing

### Changed
- Bumped `requires-mvl` to `>=0.205.0`

### Notes
- Typed handler dispatch (`.get("/path", handler_fn)`) blocked by mvl #1467
- Current builder uses string names; typed dispatch coming when #1467 is fixed

## [0.2.0] - 2026-06-18

### Added
- `coverage`, `prove`, and `version` Makefile targets
- `.openspec/adr/0001-rest-client-design.md` — documents JSON-in/JSON-out adapter design and IFC audit tags
- `.openspec/adr/0002-totality-policy.md` — explicit `total fn` annotation policy

### Changed
- `Makefile`: `MVL :=` now auto-selects local debug build when present (`../../target/debug/mvl`)
- `rest_error_msg`, `json_headers`, `map_http_err`, `is_success` graduated from bare `fn` to `total fn`
- `merge_headers` graduated from `partial fn` to `total fn` with `decreases keys.len() - ki` termination annotation

## [0.1.0] - 2026-05-31

### Added
- Initial release -- typed REST client for MVL
- `rest_post_json` -- POST a JSON body, receive a JSON response, `! Net`
- `rest_get_json` -- GET a JSON response, `! Net`
- `RestError` enum: `HttpError`, `JsonParseError`, `ApiError(Int, String)`
- `rest_error_msg` -- human-readable error formatting
- IFC security: response body detainted at `REST-JSON-RESPONSE` audit tag
- Error body truncation to 512 bytes (DoS protection)
- Header merging via `merge_headers` (extra headers override defaults)
- HTTPS transport via `pkg.tls.https`
- JSON serialization/deserialization via `std.json`
