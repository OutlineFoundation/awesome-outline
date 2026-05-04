# Awesome Outline

A curated index of **Outline-compatible censorship-circumvention strategies,
adapters, and tools** from the Outline team and the broader community.

This repository is an index. It does not host strategy implementations. Listed
components and tools should live in their authors' own repositories and
documentation.

Inclusion is not endorsement, certification, or approval by the Outline team.
The goal is to make compatible components and tools easier to find, compare,
and choose.

Looking for non-compatible projects that would be useful to adapt, audit, or
test? See [Wishlist](WISHLIST.md).

> **Work in progress.** Structure, categories, fields, and entries are still
> evolving. Nothing here is final — feedback, corrections, and proposals for a
> different shape are welcome via issues and pull requests.

## Contents

- [How To Read This List](#how-to-read-this-list)
- [Core Concepts](#core-concepts)
- [Resolve](#resolve)
- [Shape](#shape)
- [Carry](#carry)
- [Relay](#relay)
- [Choose](#choose)
- [Mobile Proxy Adapters](#mobile-proxy-adapters)
- [Tools](#tools)
- [Contributing](#contributing)

## How To Read This List

The index uses five practical capability buckets:

- **Resolve**: turn a name into a usable address or route.
- **Shape**: make a direct flow survive inspection or interference (a.k.a
  *proxyless*).
- **Carry**: move bytes or packets across the censored network to a service.
- **Relay**: ask a service to reach the final destination (like proxies).
- **Choose**: select, race, route, and fall back across options.

Each section opens with a `Prevents:` line that names the censorship threats
the bucket addresses by default. Per-entry threat extensions or caveats live in
Notes. Threat labels are shorthand for where a component may help; they are
not guarantees that a strategy works on every network or against every censor.

The five capability sections use the same fields:

| Field | Meaning |
| :---- | :------ |
| Component | The Outline strategy, compatible adapter, or compatible tool. |
| Origin | 🧩 Outline implementation or 🌐 Community adapter/tool. |
| Package / source | Package docs plus source link when available. |
| Role | The interface role exposed to applications or other components. |
| Notes | Compatibility evidence, scope, caveat, or threats addressed beyond the bucket default. |

## Core Concepts

This list uses the composable networking vocabulary from the Outline SDK's
[interoperable interfaces](https://github.com/OutlineFoundation/outline-sdk#interoperable-and-reusable).
Components are easiest to reuse when they expose one of these roles.

**Connections** move bytes or packets between two endpoints over an abstract
transport. [`transport.StreamConn`](https://pkg.go.dev/golang.getoutline.org/sdk/transport#StreamConn)
is for stream traffic, such as TCP. [`transport.PacketConn`](https://pkg.go.dev/golang.getoutline.org/sdk/transport#PacketConn)
is for packet or datagram traffic, such as UDP; "packet" follows the Go naming
convention. Connections can be wrapped, so a stream may be direct TCP, TLS over
TCP, HTTP over TLS over TCP, QUIC, or another nested stack.

**Endpoints** create connections to a fixed destination while hiding the
underlying transport details. [`transport.StreamEndpoint`](https://pkg.go.dev/golang.getoutline.org/sdk/transport#StreamEndpoint)
creates stream connections. [`transport.PacketEndpoint`](https://pkg.go.dev/golang.getoutline.org/sdk/transport#PacketEndpoint)
creates packet connections. As an analogy, an endpoint is like a reverse proxy:
the destination is selected by the endpoint, not by each caller.

**Dialers** create connections to caller-provided `host:port` addresses while
hiding the underlying transport or proxy details. [`transport.StreamDialer`](https://pkg.go.dev/golang.getoutline.org/sdk/transport#StreamDialer)
creates stream connections. [`transport.PacketDialer`](https://pkg.go.dev/golang.getoutline.org/sdk/transport#PacketDialer)
creates packet connections. As an analogy, a dialer is like a forward proxy:
each call chooses the destination. Dialers can also be nested: a
SOCKS5-over-TLS dialer can use a TLS dialer to reach the proxy, then use SOCKS5
to reach the target.

**Resolvers** answer DNS questions through a common
[`dns.Resolver`](https://pkg.go.dev/golang.getoutline.org/sdk/dns#Resolver)
interface. They are usually used to map domain names to IP addresses, but the
implementation can use system DNS, encrypted DNS, alternate resolvers, or a
custom policy.

## Resolve

Turn a name into a usable address or route. This usually involves using
the DNS system, but it doesn't have to.

Main interface: [`dns.Resolver`](https://pkg.go.dev/golang.getoutline.org/sdk/dns#Resolver).

Prevents: **DNS-based blocking**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Outline DNS resolvers | 🧩 Outline | [dns](https://pkg.go.dev/golang.getoutline.org/sdk/dns) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/dns) | Resolver | Provides `dns.Resolver` plus DNS-over-UDP, DNS-over-TCP, DNS-over-TLS, and DNS-over-HTTPS implementations. |
| Resolver-backed stream dialer | 🧩 Outline | [dns.NewStreamDialer](https://pkg.go.dev/golang.getoutline.org/sdk/dns#NewStreamDialer) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/dns/stream_dialer.go) | Resolver plus stream dialer | Resolves hostnames with an injected resolver and connects with Happy Eyeballs v2 behavior. |
| DNS config strings | 🧩 Outline | [x/configurl DNS protection](https://pkg.go.dev/golang.getoutline.org/sdk/x/configurl#hdr-DNS_Protection) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/x/configurl/dns.go) | Config adapter | Supports DNS resolution strategies such as `do53` and `doh` in composable transport strings. |
| Address override | 🧩 Outline | [x/configurl override](https://pkg.go.dev/golang.getoutline.org/sdk/x/configurl#hdr-DNS_Protection) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/x/configurl/override.go) | Dialer wrapper | Replaces the dialed host and/or port when an app already knows a usable address. Also addresses IP blocking when an alternate address is known. |

## Shape

Make a direct flow survive inspection or interference by manipulating packets
and flows on the client side, with no relay and no server changes (a.k.a.
*proxyless*).

Often implemented as connection wrappers, or convenience Endpoints and Dialers.

Prevents: **content-based blocking** and **domain-based blocking**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| TCP stream split | 🧩 Outline | [transport/split](https://pkg.go.dev/golang.getoutline.org/sdk/transport/split) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/split) | Stream dialer wrapper | Splits outgoing stream bytes at configured positions. The SDK README lists it as a TCP-layer SNI-blocking bypass strategy. |
| TLS record fragmentation | 🧩 Outline | [transport/tlsfrag](https://pkg.go.dev/golang.getoutline.org/sdk/transport/tlsfrag) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/tlsfrag) | Stream dialer wrapper | Splits the first TLS ClientHello record into multiple TLS records. |
| TLS transport options | 🧩 Outline | [transport/tls](https://pkg.go.dev/golang.getoutline.org/sdk/transport/tls) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/tls) | TLS stream wrapper | Supports SNI, ALPN, session cache, and certificate verification options for TLS-wrapped streams. |
| Packet reordering | 🧩 Outline | [x/disorder](https://pkg.go.dev/golang.getoutline.org/sdk/x/disorder) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/disorder) | Stream dialer wrapper | Sends a selected write with low TTL or Hop Limit so TCP retransmission changes packet arrival order (a DPI-desynchronization technique). |

## Carry

Move bytes or packets across the censored network to a service.

Often implemented as connection wrappers, which let you implement Endpoints and
Dialers.

Prevents: **content-based blocking** and **domain-based blocking**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Direct TCP and UDP dialers | 🧩 Outline | [transport](https://pkg.go.dev/golang.getoutline.org/sdk/transport) / [stream source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/stream.go), [packet source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/packet.go) | Direct stream and packet dialers | Baseline carry primitives that other components wrap. No circumvention by themselves. |
| TLS stream transport | 🧩 Outline | [transport/tls](https://pkg.go.dev/golang.getoutline.org/sdk/transport/tls) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/tls) | Stream wrapper | Wraps an inner `transport.StreamConn` with TLS. SNI remains visible unless paired with another strategy. |
| WebSocket transport | 🧩 Outline | [x/websocket](https://pkg.go.dev/golang.getoutline.org/sdk/x/websocket) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/websocket) | Stream and packet endpoint | Carries streams or packets over WebSocket messages. Useful where only HTTP/WebSocket paths are allowed. |
| Shadowsocks encrypted reader/writer | 🧩 Outline | [transport/shadowsocks Reader](https://pkg.go.dev/golang.getoutline.org/sdk/transport/shadowsocks#Reader), [Writer](https://pkg.go.dev/golang.getoutline.org/sdk/transport/shadowsocks#Writer) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/shadowsocks) | Encrypted stream reader/writer | `NewReader` decrypts Shadowsocks stream data and `NewWriter` encrypts stream data before it is carried over the underlying connection ([spec](https://shadowsocks.org/doc/aead.html)). |
| LwIP to transport bridge | 🧩 Outline | [network/lwip2transport](https://pkg.go.dev/golang.getoutline.org/sdk/network/lwip2transport) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/network/lwip2transport) | IP packet bridge | Translates IP packets to TCP and UDP handlers backed by Outline transport interfaces. Useful for full-tunnel or packet strategies. |
| AmneziaWG Outline adapter | 🌐 Community | [amneziawg-go/outline](https://pkg.go.dev/github.com/amnezia-vpn/amneziawg-go/outline) / [source](https://github.com/amnezia-vpn/amneziawg-go/tree/master/outline) | Packet endpoint adapter | External Outline-compatible adapter for AmneziaWG. Also addresses WireGuard fingerprinting and active probing. Also listed under Relay because VPN services can forward packet traffic beyond the endpoint. |

## Relay

Ask a service to reach the final destination (like proxies).

Usually implemented as Stream or Packet Dialers, or a Packet Listener.

Prevents: **content-based blocking**, **domain-based blocking**, **IP-based blocking**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Shadowsocks transport and proxy | 🧩 Outline | [transport/shadowsocks](https://pkg.go.dev/golang.getoutline.org/sdk/transport/shadowsocks) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/shadowsocks) | Stream and packet proxy protocol | Implements Shadowsocks secure transport and [proxy](https://shadowsocks.org/doc/what-is-shadowsocks.html) protocols. Compatible with Outline access keys through config strings. |
| SOCKS5 transport | 🧩 Outline | [transport/socks5](https://pkg.go.dev/golang.getoutline.org/sdk/transport/socks5) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/socks5) | Stream and packet proxy protocol | Provides SOCKS5 dialers for relaying traffic through a SOCKS5 proxy. SOCKS5 is not camouflage by itself. |
| HTTP CONNECT client | 🧩 Outline | [x/httpconnect](https://pkg.go.dev/golang.getoutline.org/sdk/x/httpconnect) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/httpconnect) | Stream proxy protocol | Implements HTTP CONNECT over HTTP/1.1, HTTP/2, and HTTP/3 transports. Useful where only web-style paths are allowed. |
| HTTP proxy handlers | 🧩 Outline | [x/httpproxy](https://pkg.go.dev/golang.getoutline.org/sdk/x/httpproxy) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/httpproxy) | Local forward proxy | Routes local HTTP proxy traffic through an injected `transport.StreamDialer`. |
| MobileProxy | 🧩 Outline | [x/mobileproxy](https://pkg.go.dev/golang.getoutline.org/sdk/x/mobileproxy) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/mobileproxy) | App integration proxy | Runs a local proxy so mobile apps can route selected traffic through Outline SDK dialers. |
| AmneziaWG Outline adapter | 🌐 Community | [amneziawg-go/outline](https://pkg.go.dev/github.com/amnezia-vpn/amneziawg-go/outline) / [source](https://github.com/amnezia-vpn/amneziawg-go/tree/master/outline) | Packet relay adapter | Also listed under Carry. As a VPN strategy, the service can relay packet traffic beyond the AmneziaWG endpoint. Also addresses WireGuard fingerprinting and active probing. Backward-compatible with WireGuard. |
| Psiphon StreamDialer adapter | 🌐 Community | [x/psiphon](https://pkg.go.dev/golang.getoutline.org/sdk/x/psiphon) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/psiphon) | Stream dialer adapter | Uses Psiphon as a `transport.StreamDialer`. Also addresses protocol fingerprinting and active probing. Requires a Psiphon config and the `psiphon` build tag because of licensing. |
| SOAX proxy session adapter | 🧩 Outline | [x/soax](https://pkg.go.dev/golang.getoutline.org/sdk/x/soax) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/soax) | Stream dialer adapter | Builds SOCKS5 or HTTP CONNECT proxy clients for SOAX proxy sessions. Also addresses server-reputation blocking via endpoint rotation. |

## Choose

Select, race, route, and fall back across options (a.k.a. *strategy selection*).

No single strategy works everywhere. Selection lets a client try multiple
options, fall back when one fails, and balance cost, resilience, and
performance across contexts.

Prevents: **single-strategy failure** across networks, censors, and time.

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Happy Eyeballs stream dialer | 🧩 Outline | [transport.HappyEyeballsStreamDialer](https://pkg.go.dev/golang.getoutline.org/sdk/transport#HappyEyeballsStreamDialer) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/happyeyeballs.go) | Selector and racer | Races resolved addresses with Happy Eyeballs v2 behavior. |
| Parallel Happy Eyeballs resolver | 🧩 Outline | [transport.NewParallelHappyEyeballsResolveFunc](https://pkg.go.dev/golang.getoutline.org/sdk/transport#NewParallelHappyEyeballsResolveFunc) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/happyeyeballs.go) | Resolver racer | Coordinates parallel resolver functions, usually IPv4 and IPv6. |
| Smart Dialer | 🧩 Outline | [x/smart](https://pkg.go.dev/golang.getoutline.org/sdk/x/smart) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/smart) | Strategy finder | Searches DNS and TLS strategies for test domains, then returns a working `transport.StreamDialer`. |

## Mobile Proxy Adapters

MobileProxy is the easiest Outline SDK integration path for many mobile apps:
it runs a local proxy and lets the app route selected traffic through compatible
stream dialers or strategy selectors. This section groups adapters that are
especially relevant to MobileProxy integrations. MobileProxy itself is listed
under Relay, not here.

For extending MobileProxy with additional strategies, see the
[MobileProxy package](https://pkg.go.dev/golang.getoutline.org/sdk/x/mobileproxy)
and the Outline SDK discussion on
[registering custom strategies](https://github.com/OutlineFoundation/outline-sdk/discussions/536).

| Component | Origin | Package | Notes |
| :-------- | :----- | :------ | :---- |
| Psiphon [RegisterFallbackParser](https://pkg.go.dev/golang.getoutline.org/sdk/x/mobileproxy/psiphon#RegisterFallbackParser) | 🌐 Community | [golang.getoutline.org/sdk/x/mobileproxy/psiphon](https://pkg.go.dev/golang.getoutline.org/sdk/x/mobileproxy/psiphon) | Requires Psiphon config and the `psiphon` build tag because of licensing. |
| AmneziaWG [RegisterFallbackParser](https://pkg.go.dev/github.com/amnezia-vpn/amneziawg-go/outline#RegisterFallbackParser) | 🌐 Community | [github.com/amnezia-vpn/amneziawg-go/outline](https://pkg.go.dev/github.com/amnezia-vpn/amneziawg-go/outline) | Requires AmneziaWG config. |

## Tools

Tools are not circumvention strategies by themselves. They stay separate from
the capability buckets because they help developers inspect, test, and compare
Outline-compatible configs and components.

In the examples below, `$TRANSPORT` is a config string accepted by
[`x/configurl`](https://pkg.go.dev/golang.getoutline.org/sdk/x/configurl), such
as a Shadowsocks/Outline access key (`ss://...`) or a composed pipeline like
`tls|tlsfrag:1`.

### resolve

Resolves a domain through a chosen transport and resolver. Useful for
investigating DNS blocking/manipulation, resolver blocking, and alternate
resolver behavior.

[Docs](https://pkg.go.dev/golang.getoutline.org/sdk/x/tools/resolve),
[source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/tools/resolve).

```sh
go run golang.getoutline.org/sdk/x/tools/resolve@latest \
  -type A \
  -transport "tls" \
  -resolver 8.8.8.8:853 \
  -tcp getoutline.org.
```

### fetch

Requests a URL through a configured transport string. Useful for checking
direct shaping strategies against HTTP endpoints before embedding them in an
app.

[Docs](https://pkg.go.dev/golang.getoutline.org/sdk/x/tools/fetch),
[source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/tools/fetch).

```sh
go run golang.getoutline.org/sdk/x/tools/fetch@latest \
  -transport "override:host=cloudflare.net|tlsfrag:1" \
  -method HEAD \
  -v https://meduza.io/
```

### http2transport

Runs a local forward proxy backed by a configured transport string. Useful for
trying real app or browser traffic through an Outline-compatible transport.

[Docs](https://pkg.go.dev/golang.getoutline.org/sdk/x/tools/http2transport),
[source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/tools/http2transport).

```sh
go run golang.getoutline.org/sdk/x/tools/http2transport@latest \
  -transport "override:host=cloudflare.net|tlsfrag:1" \
  -localAddr localhost:8080
```

### test-connectivity

Tests stream and datagram connectivity through a configured proxy transport.
Useful for checking proxy reachability and packet/stream behavior.

[Docs](https://pkg.go.dev/golang.getoutline.org/sdk/x/tools/test-connectivity),
[source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/tools/test-connectivity).

```sh
go run golang.getoutline.org/sdk/x/tools/test-connectivity@latest \
  -transport "$TRANSPORT"
```

### fetch-speed

Measures download speed through a configured transport. Useful for
investigating throttling or quality degradation through a candidate path.

[Docs](https://pkg.go.dev/golang.getoutline.org/sdk/x/tools/fetch-speed),
[source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/tools/fetch-speed).

```sh
go run golang.getoutline.org/sdk/x/tools/fetch-speed@latest \
  -transport "$TRANSPORT" \
  http://speedtest.ftp.otenet.gr/files/test10Mb.db
```

## Contributing

Contributions are welcome when they help people find and choose reusable
Outline-compatible strategies, adapters, and tools.

See [Contributing](CONTRIBUTING.md) for entry rules, compatibility bars, and
threat-label guidance.

## License

This list is released under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
Linked projects keep their own licenses.
