NoLimitNodes ShredStream Proxy: a drop-in replacement for Jito's ShredStream proxy, which shut down on September 5, 2026. Same CLI, same wire protocol, pointed at the NoLimitNodes Jito Shreds gateway (Frankfurt).

**Download**
- `jito-shredstream-proxy-x86_64-unknown-linux-gnu`: Linux x86-64, runs on any x86-64 CPU (no AVX2 needed), glibc 2.31 or newer (Ubuntu 20.04+, Debian 11+).
- `jito-shredstream-proxy-x86_64-unknown-linux-gnu.sha256`: its SHA-256 checksum.

**Run**
```bash
sha256sum -c jito-shredstream-proxy-x86_64-unknown-linux-gnu.sha256
chmod +x jito-shredstream-proxy-x86_64-unknown-linux-gnu
RUST_LOG=info ./jito-shredstream-proxy-x86_64-unknown-linux-gnu shredstream \
  --block-engine-url https://fra.jito.nolimitnodes.com \
  --auth-keypair shredstream-keypair.json \
  --dest-ip-ports 127.0.0.1:8001
```
Register the keypair's **public** key in the NoLimitNodes dashboard first, and allow incoming UDP on `--src-bind-port` (default 20000) from the source IP shown there.

**In this release**
- Works with Alpenglow blocks; entry batches with transaction v1 are delivered to gRPC subscribers (`--grpc-service-port`). Decode them with a v1-aware Solana SDK.
- Larger default UDP receive buffer, so bursts are not dropped.
- Jito's end-of-life shutdown removed; protos vendored, so a plain `cargo build` works.

Based on jito-labs/shredstream-proxy (Apache License 2.0); see `LICENSE` and `NOTICE`.
