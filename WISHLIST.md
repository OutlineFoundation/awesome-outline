# Wishlist

This is the contributor-facing backlog for `awesome-outline`.

The main README lists Outline-compatible strategies, adapters, and tools only.
This page tracks concrete work that could produce future main-list components.

This is not a grab bag of protocols. Components must add value. For example, we don't
need Trojan, because SOCKS5 + TLS already achieves the same result. Same for VMess/Vless.


Wishlist inclusion is not endorsement and does not imply current Outline compatibility or commitment.

## Resolve

- System resolver - allows querying for any record, including the HTTPS RR for ECH.

## Shape

- Advanced tlsfrag
- fake (from https://github.com/hufrea/byedpi)

## Carry

Standard protocols:
- WebTransport - provides multiplexed, full-duplex message streams that can leverage QUIC. Implementation
  can use [quic-go](https://quic-go.net/docs/webtransport/).
- Pluggable TLS - to enable using any TLS library. See [discussion](https://github.com/OutlineFoundation/outline-sdk/discussions/590)
- System TLS - expose the system TLS stack to Go, to prevent TLS fingerprinting.

Custom strategies:
- [dnstt](https://www.bamsoftware.com/software/dnstt/) - allows communication over DNS, sometimes
  the only way out. We should make it available via its individual components (KCP, smux, resolver), not
  a monolith. See [discussion](https://github.com/OutlineFoundation/outline-sdk/discussions/574).

Leveraging high-value services:
- Cloudflare Tunnels - can create an ephemeral reverse proxy for any HTTPS endpoint. See [discussion](https://github.com/OutlineFoundation/outline-sdk/discussions/587).
- Various cloud drives - leverages high collateral domains and IPs. See [discussion](https://github.com/OutlineFoundation/outline-sdk/discussions/586)
- Cloud Storage - [paper](https://www.petsymposium.org/foci/2024/foci-2024-0011.pdf)
- AMP cache - see [Champa](https://github.com/net4people/bbs/issues/485#issuecomment-3769529313)
- Google Docs Tunnel - [Rust proof-of-concept](https://github.com/0xinf0/gdocs-tunnel)

## Relay

- CONNECT-UDP. Can leverage [quic-go](https://quic-go.net/docs/connect-udp/)
- CONNECT-IP

## Deployment

- [WATER](https://github.com/refraction-networking/water) - WebAssembly-based
  transport framework for pluggable transports. Enables deployment of strategies as data.
