# LanShare — P2P LAN File Transfer

LAN Share is a fast, open-source file sharing application that lets you transfer files between devices on the same local network — without internet, cloud services, or external servers.
Built with C++ and Qt, it enables secure, peer-to-peer file transfers with minimal setup, making it ideal for quick sharing across laptops, desktops, and local environments.

**No files ever touch the signaling server** — they transfer peer-to-peer.

## Architecture

```
Device A ──WebSocket──► Signaling Server ◄──WebSocket── Device B
Device A ◄────────────── WebRTC P2P DataChannel ──────────────► Device B
                          (actual file bytes)
```

The signaling server only exchanges WebRTC handshake messages (offer/answer/ICE candidates).


## How It Works

| Step | Description |
|------|-------------|
| 1 | Both clients connect to the WebSocket signaling server |
| 2 | Server assigns each peer a UUID and broadcasts the peer list |
| 3 | Sender creates an `RTCPeerConnection` + `DataChannel` and sends an SDP offer via the signaling server |
| 4 | Receiver replies with an SDP answer |
| 5 | ICE candidates are exchanged to establish the direct path |
| 6 | DataChannel opens — file is chunked (64 KB pieces) and sent as `ArrayBuffer` |
| 7 | Receiver reassembles chunks into a `Blob` and triggers a browser download |

## Notes

- Works on LAN (same Wi-Fi or Ethernet) and Bluetooth PAN
- STUN servers (Google) are used for ICE — no TURN server configured, so devices must be on the same network
- Multiple simultaneous transfers are supported
- No size limit beyond your browser's memory for the receiver-side reassembly

## Tech Stack

- **Frontend**: React 18 + Vite + WebRTC (`RTCPeerConnection`, `RTCDataChannel`)
- **Signaling Server**: Node.js + `ws` (WebSocket library)
- **Discovery**: Signaling server broadcasts connected peer list
- **Transfer**: Direct WebRTC DataChannel, chunked `ArrayBuffer` streaming
