# redact

Sensitive-key detection and masking: dump config, environment, headers or
request maps into logs without leaking secrets. Pure computation - no
capabilities.

`secret()`/`reveal()` protect values you *know* are secret when you create
them. `redact` is the complementary check at an output boundary: it catches
secrets by their **key name** (`DB_PASSWORD`, `accessToken`, `X-API-Key`) on
data you did not create.

## Install

```bash
ecko get github.com/ecko-lang/redact
```

```ecko
import redact
```

## Usage

```ecko
import redact

type Login { user: String, password: String }

print(redact.is_sensitive_key("X-API-Key"))  # true
print(redact.is_sensitive_key("shipping"))  # false
print(redact.mask("db_password", "hunter2"))  # ***
print(redact.mask("region", "eu-west-1"))  # eu-west-1

print(
    redact.map_of(
        {
            headers: { "Content-Type": "text/plain", Cookie: "sid=1" },
            DATABASE_URL: "postgres://u:pw@h/db",
            login: Login("ada", "hunter2"),
        },
    ),
)
# {DATABASE_URL: ***, headers: {Content-Type: text/plain, Cookie: ***}, login: {password: ***, user: ada}}
```

## API

| function | returns |
|---|---|
| `MASK` | the replacement for a sensitive value, `"***"` |
| `is_sensitive_key(key)` | `true` when the key name marks a secret |
| `mask(key, value)` | `value`, or `MASK` when `key` is sensitive |
| `map_of(m)` | a copy of a map or record with every sensitive key's value masked, recursing into maps, lists and records |

## Notes

### Errors

| kind | reason | when |
|---|---|---|
| `bug` | `bad_argument` | `is_sensitive_key`/`mask` got a key that is not a String, or `map_of` got something other than a map or record |

### What counts as sensitive

A key is split into words at `_`, `-`, digits and camelCase seams, so
`X-API-Key`, `apiKey` and `API_KEY` all read as `api key`. A word matches only
when it **equals** a sensitive word, so `shipping`, `spinner`, `author`,
`keyboard` and `tokenizer` are safe while `Pin`, `userPin` and `auth_tokens`
are not. Plurals count: `passwords`, `secrets`, `cookies`.

- **Words**: `pin`, `password`, `passwd`, `pwd`, `passphrase`, `secret`,
  `token`, `cvv`, `cvc`, `otp`, `auth`, `authorization`, `apikey`,
  `privatekey`, `cookie`, `credential`, `cred`, `jwt`, `ssn`, `dsn`.
- **Phrases** (sensitive together, not apart): `api key`, `access key`,
  `private key`, `signing key`, `encryption key`, `master key`, `session key`,
  `session id`, `card number`, `credit card`, `connection string`.
- **Data store URLs**: a key ending in `url` or `uri` after `database`, `db`,
  `redis`, `mongo`, `mongodb`, `postgres`, `postgresql`, `pg`, `mysql`, `amqp`,
  `rabbitmq`, `broker`, `cache` or `elasticsearch` (`DATABASE_URL`,
  `REDIS_URL`), since such URLs usually carry a password.

### How `map_of` walks a value

It recurses into nested maps, lists and typed records. A sensitive key masks
its **whole** value, even a container. A typed record comes back as a plain map
of its fields, since a masked field may no longer fit its declared type. Only
keys are checked: a secret stored under an innocent key, or inside a string,
passes through.

## Testing

```bash
ecko test
```

## License

[MIT](LICENSE)
