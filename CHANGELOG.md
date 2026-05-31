# Changelog

All notable changes to pkg-rest will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
