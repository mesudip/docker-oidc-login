# docker-oidc-login

GitHub Action for logging in to a Docker registry with a GitHub Actions OIDC token.

This is useful for registries that trust `https://token.actions.githubusercontent.com` directly and expect the GitHub OIDC JWT as the Docker password.

## Usage

```yaml
name: Push Image

on:
  push:
    branches:
      - main

permissions:
  contents: read
  id-token: write

jobs:
  push-image:
    runs-on: ubuntu-latest
    env:
      REGISTRY: registry.example.com
      IMAGE: registry.example.com/team/app:latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: |
          docker build -t "$IMAGE" .

      - name: Log in to registry with GitHub OIDC
        uses: mesudip/docker-oidc-login@v1
        with:
          registry: ${{ env.REGISTRY }}
          audience: ${{ env.REGISTRY }}

      - name: Push image
        run: |
          docker push "$IMAGE"
```

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `registry` | Yes | | Registry host to log in to |
| `audience` | No | `registry` | OIDC audience to request from GitHub |
| `username` | No | `github-actions` | Username passed to `docker login` |
| `min-token-ttl-seconds` | No | `120` | Warn when OIDC token TTL is below this threshold |

## Failure mode

If the workflow does not have OIDC access, the action fails early with a clear message telling you to add:

```yaml
permissions:
  contents: read
  id-token: write
```

## Notes

- The token is masked before use.
- This action does not print the token or its claims.
- Your registry must validate GitHub's OIDC JWTs itself.
- GitHub's OIDC token is short-lived, typically around 5 minutes, so log in immediately before `docker push` rather than before a long `docker build`.
- The action prints token timing (`now`, `iat`, `exp`, `ttl`) and warns for low or already-expired TTL.
