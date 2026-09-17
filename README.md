# Redact - Ecko Std Lib Package

Sensitive-**key** detection and masking for Ecko - dump config, env, headers,
or request maps into logs without leaking secrets.

`secret()`/`reveal()` protect values you *know* are secret at creation time.
`redact` is the complementary output-boundary heuristic: it catches secrets by
their **key name** (`DB_PASSWORD`, `accessToken`, `userPin`) on data you didn't
create.

## Install

```bash
ecko get github.com/ecko-lang/redact
```

## Usage

```ecko
import redact

redact.is_sensitive_key("DB_PASSWORD")   # true  (token match on split words)
redact.is_sensitive_key("shipping")      # false (word equality - no substring traps)

redact.mask("db_password", "hunter2")    # "***"
redact.mask("region", "eu-west-1")       # "eu-west-1"

redact.map_of({ api_key: "sk-..", db: { password: "x", host: "h" } })
# { api_key: "***", db: { host: "h", password: "***" } }
```

## Rules

- **Exact keys** (lowercased) that are always secrets: `api_key`, `apikey`,
  `client_secret`, `access_token`, `refresh_token`, `authorization`.
- **Tokens** matched against the words of a key (split from snake_case *and*
  CamelCase): `pin`, `password`, `passwd`, `secret`, `token`, `cvv`, `otp`.
- Matching is word-**equality**, never substring - `shipping` is safe,
  `spinner` is safe, `broken_tokens` is safe, but `Pin` and `userPin` are not.
- `map_of` recurses into nested maps and lists; a sensitive key masks its
  whole value, even when that value is a container.

## API

| Export | What it does |
|---|---|
| `MASK` | What a redacted value is replaced with. |
| `is_sensitive_key(key)` | Is this key name one that holds sensitive data? |
| `mask(key, value)` | The value, unless its key is sensitive - then the mask. |
| `map_of(m)` | A copy of the map with every sensitive key's value masked, recursing into nested maps and lists. Non-sensitive scalars pass through untouched. |

## Testing

```bash
ecko test tests/
```

## License

MIT
