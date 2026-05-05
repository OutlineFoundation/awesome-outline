# Core Concepts

This list uses the composable networking vocabulary from the Outline SDK's
[interoperable interfaces](https://github.com/OutlineFoundation/outline-sdk#interoperable-and-reusable).
Components are easiest to reuse when they expose one of these roles.

**Resolvers** answer DNS questions through a common
[`dns.Resolver`](https://pkg.go.dev/golang.getoutline.org/sdk/dns#Resolver)
interface. They are usually used to map domain names to IP addresses, but the
implementation can use system DNS, encrypted DNS, alternate resolvers, or a
custom policy.

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
