# Sample Configuration File

This is an example file with fake secrets for testing vault-sanitize.

## API Configuration

```yaml
openai:
  api_key: sk-fake1234567890abcdefghijklmnop

anthropic:
  api_key: sk-ant-fake123456789012345678901234

github:
  token: ghp_faketoken12345678901234567890123456
```

## Database

```
DATABASE_URL=postgresql://admin:supersecretpassword@10.0.1.50:5432/myapp
REDIS_URL=redis://:myredispassword@192.168.1.100:6379
```

## SSH Keys

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA0fake0key0here0for0testing0only
...more fake key data...
-----END RSA PRIVATE KEY-----
```

## Network Info

- Server IP: 192.168.1.100
- Gateway: 10.0.0.1
- MAC: aa:bb:cc:dd:ee:ff

## Test vault-sanitize

Run: `vault-sanitize examples/sample-config.md examples/sample-config-clean.md`

Expected: All secrets above should be redacted.
