## What was the bug?

When `request(..., { api: true })` received `oauth2Token` as a plain object (`{ accessToken, expiresAt }`) instead of an `OAuth2Token` instance, it did not refresh the token and returned without an `Authorization` header.

## Why did it happen?

The refresh condition only handled two cases: missing token and expired `OAuth2Token` instance. A truthy plain object bypassed both checks. Then header assignment was guarded by `instanceof OAuth2Token`, so no header was set.

## Why does this fix solve it?

The condition now refreshes when the token is missing, not an `OAuth2Token` instance, or expired. That converts a plain-object token into a fresh `OAuth2Token` before header assignment, so this request path sets `Authorization` from a real token instance.

## One realistic edge case not covered

A token that is valid when checked but expires between validation and the outbound network call is not covered. That race can still produce a rejected request and would need retry logic at a higher layer.