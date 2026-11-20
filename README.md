# NoLimitNodes ShredStream Proxy

The NoLimitNodes ShredStream proxy is a drop-in replacement for Jito's
ShredStream proxy, which shut down on September 5, 2026. It is a minimal fork
of the upstream `jito-labs/shredstream-proxy` client: same binary, same CLI,
same wire protocol, pointed at the NoLimitNodes Jito Shreds gateway instead of
Jito's block engine.

ShredStream gives you the lowest-latency path to shreds from Solana leaders and
forwards them to your local consumers (for example a validator or your own
service).

## Migrating from Jito

No code changes are needed. Point `--block-engine-url` (and `--auth-url` if you
use a separate one) at `https://fra.jito.nolimitnodes.com`, keep your existing
auth keypair, and register that keypair's public key in the NoLimitNodes
dashboard so it is whitelisted. Everything else (`--dest-ip-ports`,
`--src-bind-port`, and so on) works exactly as before.

The gateway serves **Frankfurt only**. `--desired-regions` is accepted for CLI
compatibility but is not used for routing.

## Setup

1. **Auth keypair**: use your existing Solana keypair, or create one with
   `solana-keygen new --outfile shredstream-keypair.json`. It is only used to
   log in to the gateway and does not need to hold funds.
2. **Whitelist the public key**: buy Jito Shreds in the NoLimitNodes
   dashboard (https://nolimitnodes.com) and add the keypair's public key there.
3. **Open your firewall**: allow incoming UDP on your `--src-bind-port` from
   the source IP shown in the dashboard.
4. **Point at the gateway**: `https://fra.jito.nolimitnodes.com` (Frankfurt).

## Access

Jito Shreds is **$100/month**, with a **2-hour trial** per account. Manage the
subscription and the whitelisted public key in the NoLimitNodes dashboard.

## Running

### Prebuilt binary

Each [release](https://github.com/no-limit-nodes/shredstream-proxy/releases)
has a Linux x86-64 binary built for generic x86-64 CPUs (no AVX2 needed) and
glibc 2.31 or newer (Ubuntu 20.04+, Debian 11+), with its SHA-256 checksum.

```bash
curl -LO https://github.com/no-limit-nodes/shredstream-proxy/releases/latest/download/jito-shredstream-proxy-x86_64-unknown-linux-gnu
curl -LO https://github.com/no-limit-nodes/shredstream-proxy/releases/latest/download/jito-shredstream-proxy-x86_64-unknown-linux-gnu.sha256
sha256sum -c jito-shredstream-proxy-x86_64-unknown-linux-gnu.sha256
chmod +x jito-shredstream-proxy-x86_64-unknown-linux-gnu
RUST_LOG=info ./jito-shredstream-proxy-x86_64-unknown-linux-gnu shredstream \
  --block-engine-url https://fra.jito.nolimitnodes.com \
  --auth-keypair shredstream-keypair.json \
  --dest-ip-ports 127.0.0.1:8001
```

### With Cargo

```bash
RUST_LOG=info cargo run --release --bin jito-shredstream-proxy -- shredstream \
  --block-engine-url https://fra.jito.nolimitnodes.com \
  --auth-keypair shredstream-keypair.json \
  --dest-ip-ports 127.0.0.1:8001,10.0.0.1:8001
```

- `--block-engine-url`: `https://fra.jito.nolimitnodes.com`.
- `--auth-keypair`: path to your whitelisted auth keypair file.
- `--dest-ip-ports`: comma-separated `IP:PORT` destinations that receive the
  shreds (for example your validator's TVU port, found with
  `scripts/get_tvu_port.sh`).

Run `cargo run --release --bin jito-shredstream-proxy -- shredstream --help`
for every flag (multicast, dynamic endpoint discovery, gRPC entry service and
more).

### With Docker

```bash
docker build -t nln-shredstream-proxy .
docker run -d --network host \
  -e RUST_LOG=info \
  -v $(pwd)/shredstream-keypair.json:/app/shredstream-keypair.json \
  nln-shredstream-proxy \
  shredstream \
  --block-engine-url https://fra.jito.nolimitnodes.com \
  --auth-keypair /app/shredstream-keypair.json \
  --dest-ip-ports 127.0.0.1:8001
```

## Maintenance and credits

Maintained by NoLimitNodes as a faithful drop-in for the upstream Jito
ShredStream proxy client. The differences from upstream are the removal of
Jito's end-of-life kill switch, vendored protos (so a plain `cargo build`
works), a larger default UDP receive buffer, Alpenglow / transaction v1
deshred support, and the NoLimitNodes documentation.

Based on [jito-labs/shredstream-proxy](https://github.com/jito-labs/shredstream-proxy)
(Jito Labs). Licensed under the Apache License 2.0; see `LICENSE` and `NOTICE`.
Background on ShredStream: https://docs.jito.wtf/lowlatencytxnfeed/
