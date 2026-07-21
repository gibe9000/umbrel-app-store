# Chlud's Umbrel Community App Store

A self-hosted [Umbrel Community App Store](https://github.com/getumbrel/umbrel-community-app-store)
containing:

- **Longevity** (`chlud-longevity`) — self-hosted health and longevity
  dashboard. FastAPI + SQLite, connects Zepp/Amazfit, Android Health Connect
  and smart-scale data, computes a composite Longevity Score, and exposes
  AI-agent-friendly API endpoints. Self-built image.
- **Second Brain** (`chlud-second-brain`) — self-hosted LLM wiki. Plain
  markdown vault shared with the official Umbrel Obsidian app, plus an agent
  API so AI agents (Hermes) can add notes (`POST /api/v1/notes`) and recall
  knowledge (`GET /api/v1/recall`) with full-text search. **Install the
  official Obsidian app first** — the vault lives inside its data directory
  and appears in Obsidian at `/config/vaults/SecondBrain`. Self-built image.
- **SillyTavern** (`chlud-sillytavern`) — power-user frontend for LLMs and AI
  roleplay. Repackages the official upstream image; runs behind Umbrel's login.

## Install on Umbrel

1. Push this folder to a public git repository (e.g.
   `https://github.com/gibe9000/umbrel-app-store`).
2. In umbrelOS, open the **App Store**, click the **⋯** menu →
   **Community App Stores**.
3. Paste the repository URL and click **Add**.
4. Open the store and install an app.

## Prerequisite: published Docker images (self-built apps only)

Umbrel pulls app images from a registry — it does not build from source. The
**Longevity** and **Second Brain** apps use images you build and push yourself;
**SillyTavern** uses the official upstream image and needs no build.

```bash
# from the Longevity source repo
docker build -t ghcr.io/gibe9000/longevity-platform:1.0.0 .
docker push ghcr.io/gibe9000/longevity-platform:1.0.0

# from the Second Brain source repo
docker build -t ghcr.io/gibe9000/second-brain:1.0.0 .
docker push ghcr.io/gibe9000/second-brain:1.0.0
```

For reproducible installs, pin the image by digest after pushing:

```bash
docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/gibe9000/longevity-platform:1.0.0
```

and set `image: ghcr.io/gibe9000/longevity-platform:1.0.0@sha256:<digest>`.

## First run

**Longevity** prints a primary API key **once** in its logs. In Umbrel, open
the app's settings → logs, copy the `lgv_...` key, then paste it into the
Longevity dashboard to sign in.

## Updating apps

Umbrel shows an app update **only when the `version:` in that app's
`umbrel-app.yml` changes** in this repo — it does not track image tags or
`:latest` on its own. So every update is: bump the pinned image and the
manifest version together, commit, push. Umbrel then offers the update after it
re-syncs the store (nudge it with **⋯ → Community App Stores → remove + re-add
the URL**, or an umbrelOS restart).

### SillyTavern (upstream image — no build)

New releases appear at `ghcr.io/sillytavern/sillytavern`. To move to version
`X.Y.Z`, pin it by digest and match the manifest version:

```bash
TOKEN=$(curl -s "https://ghcr.io/token?scope=repository:sillytavern/sillytavern:pull" \
  | python -c "import json,sys;print(json.load(sys.stdin)['token'])")
curl -sI -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.oci.image.index.v1+json" \
  "https://ghcr.io/v2/sillytavern/sillytavern/manifests/X.Y.Z" \
  | grep -i docker-content-digest
```

Set `image: ghcr.io/sillytavern/sillytavern:X.Y.Z@sha256:<digest>` in
`chlud-sillytavern/docker-compose.yml` and `version: "X.Y.Z"` in its
`umbrel-app.yml`, then commit and push. (Pinning by digest is optional but
means an install always gets the exact bytes you tested; a plain `:X.Y.Z` tag
also works.)

### Longevity / Second Brain (self-built images)

Build and push your new image, then bump the pinned image + manifest version
the same way — see the build steps above.

## Structure

```
umbrel-app-store.yml        # store manifest (store id: chlud)
chlud-longevity/            # self-built FastAPI app
├── umbrel-app.yml
└── docker-compose.yml
chlud-second-brain/         # self-built LLM wiki (shares Obsidian's vault)
├── umbrel-app.yml
└── docker-compose.yml
chlud-sillytavern/          # repackaged upstream image
├── umbrel-app.yml
└── docker-compose.yml
```

App ids in a community store must be prefixed with the store id — hence
`chlud-longevity` / `chlud-second-brain` / `chlud-sillytavern`, and proxy
target hostnames like `chlud-sillytavern_sillytavern_1`
(`<app-id>_<compose-service>_1`).
