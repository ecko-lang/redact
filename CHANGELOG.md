# Changelog

## 0.38.0

**Fixed**

- Header-style keys are now caught whatever their case and hyphens:
  `X-API-Key`, `API-KEY`, `Proxy-Authorization`, `Cookie`, `Set-Cookie`,
  `X-Auth-Token`.
- More secrets are recognised: `private_key`, `signing_key`,
  `encryption_key`, `AWS_ACCESS_KEY_ID`, `passphrase`, `credentials`, `pwd`,
  `jwt`, `ssn`, `card_number`, `credit_card`, `cvc`, `dsn`, and data-store URLs
  such as `DATABASE_URL` and `REDIS_URL`, whose values carry a password.
- Plurals are recognised: `passwords`, `secrets`, `auth_tokens`, `apiKeys`.
- `map_of` now recurses into typed records, which used to pass through with
  their secrets in clear. A record comes back as a plain map of its fields.
- `is_sensitive_key` with a non-string key, and `map_of` with a list or a
  scalar, now fail with a message naming the function and the argument.

**Breaking**

- More keys are masked than before, `broken_tokens` among them.
  Migration: none needed for logging; if a test pinned a key as safe, check it
  against the README's list.
