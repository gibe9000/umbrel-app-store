# Chlud's Umbrel Community App Store

A self-hosted [Umbrel Community App Store](https://github.com/getumbrel/umbrel-community-app-store)
containing:

- **Longevity** (`chlud-longevity`) — self-hosted health and longevity
  dashboard. FastAPI + SQLite, connects Zepp/Amazfit, Android Health Connect
  and smart-scale data, computes a composite Longevity Score, and exposes
  AI-agent-friendly API endpoints.

## Install on Umbrel

1. Push this folder to a public git repository (e.g.
   `https://github.com/gibe9000/umbrel-app-store`).
2. In umbrelOS, open the **App Store**, click the **⋯** menu →
   **Community App Stores**.
3. Paste the repository URL and click **Add**.
4. Open the store and install **Longevity**.

## Prerequisite: published Docker image

Umbrel pulls app images from a registry — it does not build from source.
Before the app can install, build and push the image referenced in
[`chlud-longevity/docker-compose.yml`](chlud-longevity/docker-compose.yml):

```bash
# from the Longevity source repo
docker build -t ghcr.io/gibe9000/longevity-platform:1.0.0 .
docker push ghcr.io/gibe9000/longevity-platform:1.0.0
```

For reproducible installs, pin the image by digest after pushing:

```bash
docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/gibe9000/longevity-platform:1.0.0
```

and set `image: ghcr.io/gibe9000/longevity-platform:1.0.0@sha256:<digest>`.

## First run

The app prints a primary API key **once** in its logs. In Umbrel, open the
app's settings → logs, copy the `lgv_...` key, then paste it into the
Longevity dashboard to sign in.

## Structure

```
umbrel-app-store.yml        # store manifest (store id: chlud)
chlud-longevity/
├── umbrel-app.yml          # app manifest
└── docker-compose.yml      # Umbrel app compose (app_proxy + server)
```

App ids in a community store must be prefixed with the store id — hence
`chlud-longevity`, and the proxy target hostname `chlud-longevity_server_1`.
