# redact

## `MASK`

What a redacted value is replaced with.

## `is_sensitive_key(key)`

Is this key name one that holds sensitive data?

## `mask(key, value)`

The value, unless its key is sensitive - then the mask.

## `map_of(m)`

A copy of the map with every sensitive key's value masked, recursing into
nested maps and lists. Non-sensitive scalars pass through untouched.
