# A model for expressing invariants of internal methods in ECMAScript and its application to Proxies

Very soon in this repo:

* [invariants.md] (/claudepache/es-invariants/blob/master/invariants.md) — a model for expressing the [Invariants of the Essential Internal Methods] (https://tc39.github.io/ecma262/#sec-invariants-of-the-essential-internal-methods) of ECMAScript (“the Invariants”);

* [proxies.md] (/claudepache/es-invariants/blob/master/invariants.md) — its application to [Proxies] (https://tc39.github.io/ecma262/#sec-proxy-object-internal-methods-and-internal-slots); more specifically, how to identify the necessary and sufficient integry checks (the “Integry Checks”) needed to the trapped internal methods of Proxies in order to maintain the Invariants;

* _(TBW)_ — discussion on issues around some edge cases such Proxy over a Proxy (when the Integry Checks invokes untrusted user code), and how to resolve them.
