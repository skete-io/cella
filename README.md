# cella

```
________________________________________________________________________________
|                                                                              |
|     ____ _____ _     _        _                                              |
|    / ___| ____| |   | |      / \                                             |
|   | |   |  _| | |   | |     / _ \      [ Your Personal Cell ]                 |
|   | |___| |___| |___| |___ / ___ \                                           |
|    \____|_____|_____|_____/_/   \_\                                          |
|                                                                              |
|           _______               |                                            |
|          |       |              |  $ cella status                            |
|          |  [ ]  |              |  DEVICE: Nomad (Online)                    |
|          |       |              |  SYNC:   24 files pending                  |
|       ___|_______|___           |  SERVER: Local Docker (Running)            |
|      |_______________|          |  TUNNEL: Active (ngrok)                    |
|                                 |                                            |
|_________________________________|____________________________________________|
```

A self-hosted sync server for the Supernote Nomad. Replaces Ratta's cloud with your own hardware, keeping notes, annotations, and documents off third-party servers.

Named after the individual cells (*cellae*) of the Egyptian desert monks at Scetis — a private, minimal space, yours alone.

---

## architecture

- **core** — Dockerized Ratta service (MariaDB, Redis, Notelib)
- **proxy** — Nginx with `sub_filter` to patch hardcoded port issues
- **tunnel** — `ngrok` for a static HTTPS endpoint the Nomad can reach

---

## setup

```sh
git clone https://github.com/yourname/cella
bash install.sh
```

Configure Nginx to proxy `:8080` to the container and handle HTTPS. Point your Nomad to your `ngrok` domain under **Settings → Private Cloud**.

---

## cli (current)

Shell functions available until the Go CLI lands:

```sh
sn-push <file>   # upload a document to the server
sn-pull          # sync all server files to local machine
sn-register <file>  # index files moved via SCP into the server database
```

---

## roadmap

| version | description |
|---|---|
| v1 | stable Docker stack with shell-based sync (current) |
| v2 | unified Go binary replacing the web UI and scripts |
| v3 | TUI for library management and sync status |
| v4 | automatic extraction of handwritten highlights back into Markdown |

---

## related

- [skete](https://github.com/yourname/skete) — distillation pipeline that produces EPUBs for pushing to cella
