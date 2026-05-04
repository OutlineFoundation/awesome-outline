# Contributing

Contributions are welcome when they help people find and choose reusable
Outline-compatible strategies, adapters, and tools.

The main list is a menu of compatible components and tools, not a general
protocol catalog. Submit entries only when they meet one of these compatibility
bars:

- implementation shipped by Outline;
- external adapter that implements or exposes Outline-compatible interfaces;
- tool that works with Outline-compatible interfaces, transport strings,
  configs, or adapters.

Runnable integrations, example apps, measurement recipes, research references,
and general inspirations do not belong in the main list unless they are
compatible tools in their own right. If they point toward a future compatible
component, add a concrete adapter, tool, or source-audit task to
[Wishlist](WISHLIST.md).

## Entry Rules

- Every entry needs a source link.
- Every entry needs a taxonomy placement: Resolve, Shape, Carry, Relay,
  Choose, or multiple. The bucket's `Prevents:` line carries the default
  threat coverage.
- If the entry addresses threats beyond the bucket default, call them out in
  the Notes column using the labels below.
- Every entry needs an interface role, such as resolver, endpoint, wrapper,
  proxy, selector, adapter, or tool.
- Every entry needs an origin marker: 🧩 Outline or 🌐 Community.
- Every entry must be implemented by Outline or compatible through an external
  adapter or tool.
- Non-compatible projects belong in the wishlist instead of the main list.
- Avoid endorsement language. Inclusion is not certification or approval by
  the Outline team.

## Entry Fields

A useful main-list entry should include:

- component name;
- original repository or documentation URL;
- compatibility evidence;
- origin: 🧩 Outline or 🌐 Community;
- capability bucket: Resolve, Shape, Carry, Relay, Choose, or multiple;
- role: resolver, endpoint, wrapper, proxy, selector, adapter, tool, etc.;
- any threats addressed beyond the bucket's `Prevents:` default;
- short caveat about server requirements, licensing, or evidence if known.

## Threat Labels

Use concise threat labels from the talk taxonomy when possible:

- DNS blocking/manipulation.
- Domain or URL blocking above DNS.
- SNI/TLS ClientHello blocking.
- QUIC Initial / HTTP/3 blocking.
- IP blocking.
- Protocol blocking/fingerprinting.
- TLS fingerprinting.
- Active probing.
- Replay attacks.
- Traffic volume or packet-shape analysis.
- Reset or packet injection.
- Throttling / quality degradation.
- Allowlist censorship.
- Server reputation blocking.
- Distribution artifact blocking.

Keep descriptions factual. Avoid ranking, endorsement language, or claims that
a strategy works universally.
