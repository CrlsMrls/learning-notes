#  Secrets

Secrets objects allow to store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. The secrets are stored in a `tmpfs` volume, which is mounted to the container. The secrets are stored in memory and never written to disk. The secrets are stored in base64-encoded format, which is not encrypted (as of today).

Aliases: `secret` or `secrets`