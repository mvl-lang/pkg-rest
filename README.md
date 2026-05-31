# pkg-rest

Typed REST client for [MVL](https://github.com/LAB271/mvl_language) — JSON POST/GET over HTTPS with full IFC security.

Built on [`pkg-tls`](https://github.com/mvl-lang/pkg-tls) for the transport layer and `std.json` for serialization.

## Install

```bash
mvl add github.com/mvl-lang/pkg-rest v0.1.0
mvl install
```

## Usage

```mvl
use pkg.rest.client.{rest_post_json, rest_get_json, RestError, rest_error_msg}
use std.json.{Value}

partial fn fetch_data(url: String) -> Result[Value, RestError] ! Net {
    rest_get_json(url, Map::new())
}

partial fn post_data(url: String, body: Value) -> Result[Value, RestError] ! Net {
    rest_post_json(url, {"Authorization": "Bearer token"}, body)
}
```

## API

| Function | Signature | Effect |
|----------|-----------|--------|
| `rest_post_json` | `(url, extra_headers, body: Value) -> Result[Value, RestError]` | `! Net` |
| `rest_get_json` | `(url, extra_headers) -> Result[Value, RestError]` | `! Net` |
| `rest_error_msg` | `(e: val RestError) -> String` | pure |

### Error Type

```mvl
pub type RestError = enum {
    HttpError(HttpsError),
    JsonParseError(String),
    ApiError(Int, String),   // status code + truncated body (<=512 bytes)
}
```

## Security

### IFC Model

| Data | Label | Why |
|------|-------|-----|
| Response body | `Tainted[String]` (from TLS layer) | External data -- untrusted until parsed |
| Parsed JSON | bare `Value` | Detainted at `REST-JSON-RESPONSE` audit tag |
| Error body | truncated to 512 bytes | Prevent DoS via hostile response allocation |

### Effect Requirement

Both `rest_post_json` and `rest_get_json` require `! Net`. Pure functions cannot accidentally make network calls.

## Dependencies

- [`pkg-tls`](https://github.com/mvl-lang/pkg-tls) -- HTTPS transport
- `std.json` -- JSON serialization/deserialization

## License

Apache License 2.0 -- see [LICENSE](LICENSE).
