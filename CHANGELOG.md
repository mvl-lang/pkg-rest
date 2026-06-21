# Changelog

All notable changes to pkg-rest will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.4.0] - 2026-06-21

### Added
- `src/json.mvl` — typed JSON helpers (moved from pkg-http):
  - Response builders: `json_ok`, `json_created`, `json_no_content`, `json_error`, `json_ok_str`, `json_created_str`
  - HttpError constructors: `http_not_found`, `http_bad_request`, `http_unauthorized`, `http_internal_error`
  - Path param extractors: `param_string`, `param_int`
  - Body decoders: `body_json`, `body_obj`
  - JSON object helpers: `json_get_object`, `json_field_string`, `json_field_int`, `json_str`
- `src/json_test.mvl` — unit tests for JSON helpers

### Removed
- `src/server.mvl` and `src/server_test.mvl` — routing belongs in pkg-http; no need for a wrapper builder.

### Changed
- Dependency: pkg-http v0.2.3 → **v0.4.0** (HttpMethod rename, raw HTTP only scope)
- Bumped `requires-mvl` to `>=0.205.0`

### Known issues
- `mvl test` for this package currently fails due to a transpiler bug — native deps from pkg-tls (rustls et al) aren't propagated to the test Cargo.toml. See mvl#1481. Source compiles correctly via `mvl check`.

## [0.3.0] - 2026-06-20 [SUPERSEDED]

Server route builder (superseded — routing stays in pkg-http).

## [0.2.0] - 2026-06-18

### Added
- `coverage`, `prove`, and `version` Makefile targets
- `.openspec/adr/0001-rest-client-design.md` — documents JSON-in/JSON-out adapter design and IFC audit tags
- `.openspec/adr/0002-totality-policy.md` — explicit `total fn` annotation policy

### Changed
- `Makefile`: `MVL :=` now auto-selects local debug build when present (`../../target/debug/mvl`)
- `rest_error_msg`, `json_headers`, `map_http_err`, `is_success` graduated from bare `fn` to `total fn`
- `merge_headers` graduated from `partial fn` to `total fn` with `decreases keys.len() - ki` termination annotation

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
