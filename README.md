# A model for expressing invariants of internal methods in ECMAScript and its application to Proxies

__Work in progress__

* [invariants.md](/invariants.md) — a model for expressing the [Invariants of the Essential Internal Methods](https://tc39.github.io/ecma262/#sec-invariants-of-the-essential-internal-methods) of ECMAScript (“the Invariants”);

* [proxies.md](/proxies.md) — its application to [Proxies](https://tc39.github.io/ecma262/#sec-proxy-object-internal-methods-and-internal-slots); more specifically, how to identify the necessary and sufficient integrity checks (the “Integrity Checks”) needed on the result of the trapped internal methods of Proxies in order to maintain the Invariants;

* _(TBW)_ — discussion on issues around some edge cases revolving around invocation of untrusted user code, namely Proxy over Proxy and invocation of accessors on objects; and how to resolve them.
