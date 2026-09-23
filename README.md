# Outline Components Catalog

A curated index of **Outline-compatible censorship-circumvention strategies,
adapters, and tools** from the Outline team and the broader community. This is
an index — implementations live in their authors' own repositories. Looking for
projects worth adapting? See [Wishlist](WISHLIST.md).

> **Work in progress.** Structure, categories, fields, and entries are still
> evolving. Feedback, corrections, and proposals welcome via issues and pull
> requests.

## Contents

- [Core Concepts](CONCEPTS.md)
- [Resolve](#resolve)
- [Shape](#shape)
- [Carry](#carry)
- [Relay](#relay)
- [Measure](#measure)
- [Choose](#choose)
- [Integrate](#integrate)
- [Tools](#tools)
- [Contributing](#contributing)

## Resolve

Turn a name into a usable address or route. This usually involves using
the DNS system, but it doesn't have to.

Main interface: [`dns.Resolver`](https://pkg.go.dev/golang.getoutline.org/sdk/dns#Resolver).

Prevents: **DNS-based blocking**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Outline DNS resolvers | 🧩 Outline | [dns](https://pkg.go.dev/golang.getoutline.org/sdk/dns) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/dns) | Resolver | Provides `dns.Resolver` plus DNS-over-UDP, DNS-over-TCP, DNS-over-TLS, and DNS-over-HTTPS implementations. |
| Resolver-backed stream dialer | 🧩 Outline | [dns.NewStreamDialer](https://pkg.go.dev/golang.getoutline.org/sdk/dns#NewStreamDialer) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/dns/stream_dialer.go) |  Stream dialer | Resolves hostnames with an injected resolver and connects with Happy Eyeballs v2 behavior. |
| Happy Eyeballs | 🧩 Outline | [transport.HappyEyeballsStreamDialer](https://pkg.go.dev/golang.getoutline.org/sdk/transport#HappyEyeballsStreamDialer) / [source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/happyeyeballs.go) | Stream dialer | Races resolved addresses with Happy Eyeballs v2 behavior. |


## Shape

Make a direct flow survive inspection or interference by manipulating packets
and flows on the client side, with no relay and no server changes (a.k.a.
*proxyless*).

Often implemented as connection wrappers, or convenience Endpoints and Dialers.

Prevents: **content-based blocking** and **domain-based blocking**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| TCP stream split | 🧩 Outline | [transport/split](https://pkg.go.dev/golang.getoutline.org/sdk/transport/split) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/split) | Writer, stream dialer | Splits outgoing stream bytes at configured positions. The SDK README lists it as a TCP-layer SNI-blocking bypass strategy. |
| TLS record fragmentation | 🧩 Outline | [transport/tlsfrag](https://pkg.go.dev/golang.getoutline.org/sdk/transport/tlsfrag) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/tlsfrag) | Connection wrapper, stream dialer | Splits the first TLS ClientHello record into multiple TLS records. |
| Packet reordering | 🧩 Outline | [x/disorder](https://pkg.go.dev/golang.getoutline.org/sdk/x/disorder) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/disorder) | Writer, stream dialer | Sends a selected write with low TTL or Hop Limit so TCP retransmission changes packet arrival order (a DPI-desynchronization technique). |

## Carry

Move bytes or packets across the censored network to a service.

Often implemented as connection wrappers, which let you implement Endpoints and
Dialers.

Prevents: **content-based blocking** and **domain-based blocking**

### Standards-based

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Direct TCP and UDP | 🧩 Outline | [transport](https://pkg.go.dev/golang.getoutline.org/sdk/transport) / [stream source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/stream.go), [packet source](https://github.com/OutlineFoundation/outline-sdk/blob/main/transport/packet.go) | Direct stream and packet dialers | Baseline carry primitives that other components wrap. No circumvention by themselves. |
| TLS | 🧩 Outline | [transport/tls](https://pkg.go.dev/golang.getoutline.org/sdk/transport/tls) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/tls) | Connection wrapper, stream dialer | Wraps an inner `transport.StreamConn` with TLS. Supports SNI, ALPN, session cache, and certificate verification options for TLS-wrapped streams. |
| WebSocket | 🧩 Outline | [x/websocket](https://pkg.go.dev/golang.getoutline.org/sdk/x/websocket) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/websocket) | Stream and packet endpoint | Carries streams or packets over WebSocket messages. Useful where only HTTP/WebSocket paths are allowed. |

### Custom

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Shadowsocks encryption | 🧩 Outline | [transport/shadowsocks Reader](https://pkg.go.dev/golang.getoutline.org/sdk/transport/shadowsocks#Reader), [Writer](https://pkg.go.dev/golang.getoutline.org/sdk/transport/shadowsocks#Writer) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/shadowsocks) | Encrypted stream reader/writer | `NewReader` decrypts Shadowsocks stream data and `NewWriter` encrypts stream data before it is carried over the underlying connection ([spec](https://shadowsocks.org/doc/aead.html)). |
| LwIP to transport bridge | 🧩 Outline | [network/lwip2transport](https://pkg.go.dev/golang.getoutline.org/sdk/network/lwip2transport) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/network/lwip2transport) | IP packet bridge | Translates IP packets to TCP and UDP handlers backed by Outline transport interfaces. Useful for full-tunnel or packet strategies. |

## Relay

Ask a service to reach the final destination (like proxies).

Usually implemented as Stream or Packet Dialers, or a Packet Listener.

Prevents: **content-based blocking**, **domain-based blocking**, **IP-based blocking**

### Standards-based

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| SOCKS5 | 🧩 Outline | [transport/socks5](https://pkg.go.dev/golang.getoutline.org/sdk/transport/socks5) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/socks5) | Stream and packet proxy protocol | Provides SOCKS5 dialers for relaying traffic through a SOCKS5 proxy. SOCKS5 is not camouflage by itself. |
| Web proxy client (HTTP CONNECT) | 🧩 Outline | [x/httpconnect](https://pkg.go.dev/golang.getoutline.org/sdk/x/httpconnect) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/httpconnect) | Stream proxy protocol | Implements HTTP CONNECT over HTTP/1.1, HTTP/2, and HTTP/3 transports. Useful where only web-style paths are allowed. |
| Web proxy CONNECT handler (server) | 🧩 Outline | [x/httpproxy](https://pkg.go.dev/golang.getoutline.org/sdk/x/httpproxy) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/httpproxy) | Local forward proxy | Routes local HTTP proxy traffic through an injected `transport.StreamDialer`. |

### Custom

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Shadowsocks | 🧩 Outline | [transport/shadowsocks](https://pkg.go.dev/golang.getoutline.org/sdk/transport/shadowsocks) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/transport/shadowsocks) | Stream dialer, packet listener | Implements Shadowsocks secure transport and [proxy](https://shadowsocks.org/doc/what-is-shadowsocks.html) protocols. Compatible with Outline access keys through config strings. |
| AmneziaWG | 🌐 Community | [amneziawg-go/outline](https://pkg.go.dev/github.com/amnezia-vpn/amneziawg-go/outline) / [source](https://github.com/amnezia-vpn/amneziawg-go/tree/master/outline) | Stream dialer | As a VPN strategy, the service can relay packet traffic beyond the AmneziaWG endpoint. Also addresses WireGuard fingerprinting and active probing. Backward-compatible with WireGuard. |
| Psiphon  | 🌐 Community | [x/psiphon](https://pkg.go.dev/golang.getoutline.org/sdk/x/psiphon) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/psiphon) | Stream dialer | Uses Psiphon as a `transport.StreamDialer`. Also addresses protocol fingerprinting and active probing. Requires a Psiphon config and the `psiphon` build tag because of licensing. |

## Choose

Select, race, route, and fall back across options (a.k.a. *strategy selection*).

No single strategy works everywhere. Selection lets a client try multiple
options, fall back when one fails, and balance cost, resilience, and
performance across contexts.

Prevents: **single-strategy failure** across networks, censors, and time.

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Smart Dialer | 🧩 Outline | [x/smart](https://pkg.go.dev/golang.getoutline.org/sdk/x/smart) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/smart) | Strategy finder | Searches DNS and TLS strategies for test domains, then returns a working `transport.StreamDialer`. |
| Transport URL config | 🧩 Outline | [x/configurl](https://pkg.go.dev/golang.getoutline.org/sdk/x/configurl) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/configurl) | Strategy composer | Parses composable transport strings (e.g. `ss://…`, `tls\|tlsfrag:1`, `doh\|override:host=…`) into a working dialer. Covers strategies from all buckets — Resolve, Shape, Carry, Relay — in a single config language. Custom parsers can be registered to extend the config language with new strategies. Used by all SDK tools. |

## Measure

Gather consistent evidence on path availability, performance, and censor interference to allow for remote measurements across diverse networks.

Main interfaces: Telemetry reporting and endpoint verification hooks.

Prevents: **blind strategy failure** and **invisible quality degradation**

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| SOAX proxy client | 🧩 Outline | [x/soax](https://pkg.go.dev/golang.getoutline.org/sdk/x/soax) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/soax) | Proxy network client | Builds SOCKS5 or HTTP CONNECT proxy clients for SOAX sessions leveraging measurement-driven endpoint rotation to address reputation blocking, allowing for remote measurements across diverse networks. |
| Connectivity Check | 🧩 Outline | [x/connectivity](https://pkg.go.dev/golang.getoutline.org/sdk/x/connectivity) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/connectivity) | Telemetry probe | Baseline library package for checking stream and datagram network reachability and generating diagnostic logs. |
| Telemetry Reporting | 🧩 Outline | [x/report](https://pkg.go.dev/golang.getoutline.org/sdk/x/report) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/report) | Telemetry collector | Library package providing structural error, performance, and metric event logging to support data-driven selection. |

## Integrate

Easily embed network resilience and censorship circumvention into existing applications without implementing custom low-level SDK engines from scratch.

Main interface: Local proxy listeners, pre-compiled binary framework packages, and custom streaming engine handlers.

Prevents: **high implementation barriers** and **monolithic strategy forks**

### Outline MobileProxy
Provides a local lookback proxy so mobile apps can route selected traffic through compatible SDK strategy dialers.

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| MobileProxy Core | 🧩 Outline | [x/mobileproxy](https://pkg.go.dev/golang.getoutline.org/sdk/x/mobileproxy) / [source](https://github.com/OutlineFoundation/outline-sdk/tree/main/x/mobileproxy) | App loopback proxy API | Core package running a local loopback proxy for routing mobile client application traffic. |
| `mobileproxylib` | 🧩 Outline | [mobileproxylib](https://github.com/OutlineFoundation/mobileproxylib) | Pre-built static library | Repository providing pre-compiled binary static/dynamic frameworks for cross-platform inclusion without native tooling builds. |
| Psiphon Fallback Adapter | 🌐 Community | [x/mobileproxy/psiphon](https://pkg.go.dev/golang.getoutline.org/sdk/x/mobileproxy/psiphon) | MobileProxy plugin parser | RegisterFallbackParser plugin allowing MobileProxy to parse and inject custom Psiphon configuration tags. |
| AmneziaWG Fallback Adapter | 🌐 Community | [amneziawg-go/outline parser](https://pkg.go.dev/github.com/amnezia-vpn/amneziawg-go/outline#RegisterFallbackParser) | MobileProxy plugin parser | RegisterFallbackParser plugin allowing MobileProxy to inject and spin up AmneziaWG packet tunnels. |

### IIF SmartProxy
Separate Android resilience integration framework.

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| SmartProxy Library | 🌐 Community | [SmartProxy Android SDK](https://github.com/Internet-Innovations-Foundation/SmartProxy) | Integration framework | High-level `ProxyManager` wrapping Outline SDK and ByeDPI, offering pre-built media streaming extensions (ExoPlayer blocks) for fast integration. |

### App Maker
Container generators turning web properties into censorship-resistant app packages.

| Component | Origin | Package / source | Role | Notes |
| :-------- | :----- | :--------------- | :--- | :---- |
| Outline App Maker | 🧩 Outline | [outline-app-maker](https://github.com/OutlineFoundation/outline-app-maker) | Native app wrapper package | CLI container generator wrapping websites into native iOS/Android containers utilizing MobileProxy under the hood. |


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
