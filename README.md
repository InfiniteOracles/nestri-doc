# Nestri Setup / Install

Assumes you already have podman and bun set up.

---

## Nestri has 3 parts:
- Runner  
- Relay  
- Website

---

## Step 1: Set up the Relay (middleman between runner and website)

Run this:
```bash
podman run --replace --name=relay \
  -e PERSIST_DIR=/persist \
  -v ~/relay-persist:/persist \
  -e AUTO_ADD_LOCAL_IP=true \
  --network host \
  ghcr.io/datcaptainhorse/nestri-relay:latest
```

It will output something like this:

```
2025/07/01 16:48:50 INFO Mesh connection addresses:
> /ip4/10.4.9.119/tcp/8088/p2p/12D3KooW...
> /ip4/10.4.9.119/tcp/8088/ws/p2p/...
> /ip4/127.0.0.1/tcp/8088/p2p/...
> /ip6/::1/tcp/8088/ws/p2p/...
```

🟡 **Important:**  
Copy the **exact** line that:
- starts with `/ip4/`
- **is not** `127.0.0.1`
- uses `/tcp/`
- does **not** contain `/ws/`
- **does** contain `/p2p/`

✅ Correct example:
```
/ip4/10.4.9.119/tcp/8088/p2p/12D3KooWGKW62k9cPYvHiHBeMyUTPLTZaaasEqUimtskfqXhNfsK
```

This is your `RELAY_URL`. Save it.

---

## Step 2: Set up the Runner

### For NVIDIA GPU:
```bash
podman run --replace -d \
  --name=nestri \
  --network host \
  --shm-size=6g \
  --cap-add=SYS_NICE \
  --device /dev/dri/ \
  --device /dev/nvidia-uvm \
  --device /dev/nvidia-uvm-tools \
  --device /dev/nvidiactl \
  --device /dev/nvidia0 \
  --device /dev/nvidia-modeset \
  -e RELAY_URL='PASTE_RELAY_URL_HERE' \
  -e NESTRI_ROOM='your-room-name' \
  -e RESOLUTION=1920x1080 \
  -e FRAMERATE=60 \
  -e NESTRI_PARAMS='--verbose=true --dma-buf=true --audio-rate-control=cbr --video-codec=h264 --video-rate-control=cbr --video-bitrate=8000' \
  -v ~/nestri:/home/nestri \
  ghcr.io/datcaptainhorse/nestri-cachyos:latest
```

> If it fails to run, try changing `--dma-buf=true` to `--dma-buf=false`.

### For Intel/AMD:
Same command, just skip the NVIDIA `--device` lines.

---

## Step 3: Set up the Website

Clone the repo:
```bash
git clone https://github.com/nestrilabs/nestri.git
cd nestri/apps/www
bun install
bun dev
```

After it starts, you’ll get a local dev URL.

Go to:
```
http://localhost:5173/play/{roomname}?peerURL={RELAY_URL}
```

Replace:
- `{roomname}` with your room name  
- `{RELAY_URL}` with the relay address from earlier

---

### Troubleshooting

If the page says **offline**, the runner isn’t connecting to the relay.

- Make sure the `RELAY_URL` is correct (only `/ip4/`, not `127.0.0.1`, no `/ws/`, must have `/p2p/`)
- Make sure the relay container is running
- Restart the runner if needed

Done.
