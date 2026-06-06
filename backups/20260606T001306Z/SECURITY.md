# Snapshot security notes

This snapshot intentionally excludes raw secrets and credentials.

Excluded by policy:
- ~/.hermes/.env values
- ~/.hermes/auth.json
- session databases and transcripts
- gateway logs and request dumps
- SSH/private keys and credential stores

Included:
- sanitized Hermes config
- command/status output after sanitizer pass
- names of configured environment variables, without values
- non-sensitive file inventory for selected Hermes paths
