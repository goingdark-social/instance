# instance

Kubernetes manifests for running goingdark.social.

## Mastodon

Posts can be up to 1000 characters. Reply fetching waits five minutes after a post is created and then checks every fifteen minutes. It stops after 1000 replies total, 500 per post, or 500 pages so the job doesn't run forever.

## Hypebot

Hypebot boosts popular posts automatically. It runs `ghcr.io/goingdark-social/hypebot:v0.1.0`.
Pinning the version keeps the bot predictable; update the tag when you deploy a new release.

