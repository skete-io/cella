# cella

```
________________________________________________________________________________
|                                                                              |
|     ____ _____ _     _        _                                              |
|    / ___| ____| |   | |      / \                                             |
|   | |   |  _| | |   | |     / _ \      [ Your Personal Cell ]                |
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

A self-hosted sync server for the Supernote Nomad. Replaces Ratta's cloud with your own hardware — notes, annotations, and distilled documents stay on your infrastructure, air-gapped from third-party servers.

Named after the individual cells (*cellae*) of the Egyptian desert monks at Scetis — a private, minimal space, yours alone.

Formerly `supernote-private-cloud`.

---

## architecture

```
Supernote Nomad
      |
      | HTTPS
      v
   ngrok (static tunnel)
      |
      v
   Nginx :8080  ← sub_filter strips hardcoded ports from responses
      |
      v
   Docker (Notelib + MariaDB + Redis)  :19072
```

---

## setup

### 1. install

Run the official Ratta deployment script from your target directory:

```sh
mkdir -p /mnt/data/supernote && cd /mnt/data/supernote
bash install.sh
```

`install.sh` is provided by Ratta as part of their private cloud release. It pulls the Docker images and starts the stack.

### 2. permissions

Allow the containers to write to the host filesystem:

```sh
sudo chmod -R 777 /mnt/data/supernote/supernote_data
sudo chmod -R 777 /mnt/data/supernote/sndata
```

### 3. nginx

The Ratta service hardcodes `:8080` in upload URLs. Nginx strips this so the Nomad can communicate over standard HTTPS on port 443. Gzip must be disabled so `sub_filter` can read the response body.

```nginx
server {
    listen 8080;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:19072;
        proxy_set_header Host your-domain.com;

        gzip off;
        proxy_set_header Accept-Encoding "";

        sub_filter ':8080' '';
        sub_filter 'http://your-domain' 'https://your-domain';
        sub_filter_once off;
        sub_filter_types application/json;
    }
}
```

### 4. ngrok tunnel

Point a static ngrok domain at local Nginx on port 8080:

```sh
ngrok http --url=your-static-domain.ngrok-free.app 8080
```

### 5. connect the nomad

```
Settings → Account → Private Cloud
  Host: https://your-static-domain.ngrok-free.app
```

Register an account through the Nomad UI — this creates a local user in MariaDB.

---

## cli

Shell-based until the Go rewrite lands.

**server side**

```sh
sn-register <file> <type>      # index a file moved via SCP into the database
sn-reset-password <email> <pass>  # reset credentials via Redis injection
```

**client side**

```sh
sn-push <file>   # upload a document to the server
sn-pull          # sync all server files to local machine
```

---

## roadmap

| version | description |
|---|---|
| v1 | stable Docker stack with shell-based sync (current) ✅ |
| v2 | unified Go binary replacing the web UI and shell scripts |
| v3 | Bubble Tea TUI for library management and sync status |
| v4 | automatic extraction of handwritten highlights back into Markdown |

---

## contributing

PRs improving Docker orchestration, Nginx efficiency, or the Go implementation are welcome.

---

## related

- [skete](https://github.com/yourname/skete) — distillation pipeline that produces EPUBs for pushing to cella
