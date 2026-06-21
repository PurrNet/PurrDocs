---
description: Host your PurrNet dedicated servers on Gameye — no SDK, no plugin, no egress fees
---

# Gameye

This guide covers hosting your PurrNet dedicated servers on [Gameye](https://gameye.com), a game server orchestration platform. Gameye runs your server at the container level, so there is nothing PurrNet-specific to install — if your server builds to Linux and runs in Docker, it runs on Gameye.

{% hint style="success" %}
You'll need a free [Gameye Developer Trial](https://trial.gameye.com/) account to get an API token. It's self-serve — no sales call required.
{% endhint %}

{% hint style="info" %}
Unlike some hosts, Gameye needs **no PurrNet addon or plugin**. Your server runs as a plain Linux container and Gameye manages it externally through a REST API. Nothing goes inside your Unity project.
{% endhint %}

### What you'll need

* **Docker** ([Desktop or CLI](https://www.docker.com/products/docker-desktop/)) — to build and push your server image. Make sure it's running when you build.
* **Unity Linux Dedicated Server Build Support** — add it via Unity Hub (`Installs → Add Modules`).
* A **Gameye API token** — from your [Developer Trial](https://trial.gameye.com/).
* An **OCI-compatible container registry** (Docker Hub, GitLab Registry, AWS ECR, GCP Artifact Registry — any works).

### 1. Build a Linux dedicated server

Build your project for the **Dedicated Server** platform (Linux) — PurrNet has a **Build Server** button for exactly this.

Make sure your `NetworkManager` starts as a server in the build. The simplest way is to set the NetworkManager's **Start Server Flags** to start in server builds, so it begins listening as soon as the container boots. Alternatively, call `StartServer()` from your own bootstrap script guarded by `#if UNITY_SERVER`.

{% hint style="warning" %}
Your transport binds a **fixed** internal port. Keep it fixed inside the container — Gameye maps an external port to it and returns that external port to your matchmaker.
{% endhint %}

### 2. Containerize the server

Wrap your build in a Docker image. A PurrNet server needs nothing beyond a minimal Linux base:

```docker
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY Build/ /app/
RUN chmod +x /app/MyGameServer.x86_64

# Expose the port your PurrNet UDP transport is configured to listen on.
EXPOSE 5000/udp
ENTRYPOINT ["/app/MyGameServer.x86_64", "-batchmode", "-nographics"]
```

Build and push it to your registry:

```bash
docker build -t your-registry/purrnet-server:1.0.0 .
docker push   your-registry/purrnet-server:1.0.0
```

### 3. Register the image with Gameye

During onboarding, give Gameye your registry credentials and image name. Gameye pre-pulls your image onto infrastructure in every region you deploy to, so by the time your matchmaker calls the API there's no image transfer happening — just a container start.

### 4. Start sessions from your backend

When a match is ready, call the Gameye Session API with your image and a target region:

```bash
curl -X POST https://api.gameye.io/session \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "location": "eu-west",
    "image": "your-purrnet-server",
    "args": ["-batchmode", "-nographics"],
    "labels": { "matchId": "abc123" },
    "ttl": 3600
  }'
```

The response comes back in about 0.5 seconds with the connection details:

```json
{
  "id": "session-abc123",
  "host": "185.x.x.x",
  "ports": [ { "type": "udp", "container": 5000, "host": 34521 } ]
}
```

Pass `host` and the external `host` port to your players — they connect their PurrNet client directly to the server. Gameye is not in the network path during gameplay, so there's no relay hop or added latency.

{% hint style="success" %}
That's it — no PurrNet addon, no SDK in your build, no DevOps. Push a Docker image, call an API.
{% endhint %}

### Notes

* **Any PurrNet transport works.** UDP for action games, WebSockets for browser clients, Steam, or Composite. Your server binds the transport's listen port inside the container; Gameye maps an external port to it.
* **Shut down cleanly when a match ends.** When your server process exits with code `0`, Gameye reclaims the compute and stops billing for that session. A common pattern is to quit once the last player disconnects.
* **Any matchmaker.** Gameye's Session API is plain HTTP, so trigger it from any backend — Unity Matchmaker, Nakama, or a custom service.
* **Pricing.** Gameye is `$0.07/vCPU/hr` with no egress (bandwidth) fees. PurrNet is free and MIT-licensed, so the server software adds no licensing cost.

For more detail, see the [Gameye + PurrNet integration guide](https://gameye.com/purrnet-integration/) and the [Gameye documentation](https://gameye.com/docs/).
