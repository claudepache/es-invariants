# Integrity checks on Proxies

This page aims to identify minimal invariant checks needed on trapped internal methods of [Proxies] (https://tc39.github.io/ecma262/#sec-proxy-object-internal-methods-and-internal-slots)
in order to maintain the [invariants of internal methods] (https://tc39.github.io/ecma262/#sec-invariants-of-the-essential-internal-methods)
as modelled in _[invariants.md] (/invariants.md)_.

## Fundemental observation methods

The following internal methods are intended to be purely observational. We called them the **<dfn>fundamental observation methods</dfn>**
* \[\[IsExtensible]] ()
* \[\[GetPrototypeOf]] ()
* \[\[OwnPropertyKeys]] ()
* \[\[GetOwnProperty]] (_P_)

## Nonquantum objects

We first concentrate on [Proxies] (https://tc39.github.io/ecma262/#sec-proxy-object-internal-methods-and-internal-slots) whose target object can be observed without disturbing the state of the program. Specifically we define a **<dfn>nonquantum object</dfn>** as an [Object] (https://tc39.github.io/ecma262/#sec-object-type) whose fundemental observation methods satisfy the two following conditions:

1. <i>Running a fundemental observation method of the object will not observably modify the state of the program.</i>

We exclude from this criterion states depending on the time passing, like the returned value from `Date.now()`. The returned completion of the method is not part of that criterion.

<ol start=2><li><i>The results returned by calling successively (i.e., without other operation done inbetween) fundemental observation methods of the object are consistent.</i></ol>

Here, consistency means:
* the value of a _nonabrupt_ completion returned by the same fundemental observation method will always be the same;
* if the value of a nonabrupt completion of \[\[GetOwnProperty]] (_P_) is not **undefined**, respectively is **undefined**, then the value of a nonabrupt completion of \[\[OwnPropertyKey]] () shall be a List containing _P_, respectively not containing _P_, and conversly.

Note that abrupt completions are irrelevant for our purpose.

Every [standard built-in object] (https://tc39.github.io/ecma262/#sec-ecmascript-standard-built-in-objects) is nonquantum. This is probably true for most, if not all, [built-in objects] (https://tc39.github.io/ecma262/#sec-built-in-object) provided by a given ECMAScript implementation such as a web browser.

On the other hand, Proxy objects may be quantum. Although it is probably not a good idea for a non-malicious user to create quantum Proxies, we cannot rely on that.
 
## Integrity checks of trapped Proxy internal methods

When an internal method \[\[_Method_]] for which a trap is defined is called on a Proxy, the following steps are taken:

1. The trap is invoked, and the result is transformed in order to comply with the form of the value that should be returned from the internal method (e.g., for \[\[OwnPropertyKeys]](), the result is coerced to a List of property keys).
2. An integrity check is made by comparing that result with results returned by different invocations of the fundemental observation methods on the target object.

If the integrity check is not passed, a TypeError exception is thrown. Also, any abrupt completion terminates the algorithm immediately and is forwarded.

Let **<dfn>IntegrityCheck\_<i>Method</i>(_arguments_, _result_, _target_)</dfn>** denotes the abstract operation defining the integrity check associated to \[\[_Method_]], where _arguments_ is the List of the arguments with which the \[\[_Method_]] was invoked, _result_ is the value that the Proxy wants to return, and _target_ is the target object of the Proxy. When terminating nonabruptly, that operation shall return either **true** or **false**, depending on whether the integrity check is passed.

We want to define the various IntegrityCheck\_<i>Method</i> algorithms so that the two following conditions are satisfied.

1. IntegrityCheck\_<i>Method</i>_(_arguments_, _result_, _target_)</i> shall return **true** if all [observations] (/invariants.md#observations) that would be made on the Proxy by returning _result_, may be made on _target_ by invoking fundamental observation methods on it.

For example, let _P_ be a Proxy with target object _T_. Suppose that the trap triggered by calling _P_.\[\[PreventExtensions]] () wants to return **true**. That would trigger the observation “extensible: false” on _P_.
Then, if the same observation can be made on _T_, e.g. by calling T.\[\[IsExtensible]]() and getting the value **false**, the integrity check is passed.

<ol start=2><li>If the target of a Proxy is a nonquantum object that observes the <a href="https://github.com/claudepache/es-invariants/blob/master/invariants.md#invariants-of-internal-methods">Invariants of internal methods</a>, then the Proxy shall observe those invariants.</ol>

We restrict ourself to _nonquantum_ target objects, because quantum objects would lead to inextricable complications in edge cases; that issue will be discussed somewhere else (TBW).

## Necessary and sufficient conditions for the integrity checks

We want to prove the following:

> Given Condition 1 above, in order to satisfy Condition 2, it is necessary and sufficient that the IntegrityCheck\_<i>Method</i>(_arguments_, _result_, _target_) abstract operations return **false** (or an abrupt completion) unless the two following conditions hold:
>
> A. For any observation “character: _value_” that would be made on the Proxy by returning _result_, if a lock “@character: _value2_” may be put on _target_ (by invoking fundemental observation methods on it), then _value2_ must be the same as _value_.
>
> B. Assuming that all the locks that may be put on _target_ (by invoking fundemental observation methods on it) are also put on the Proxy, for any lock “@character: value” that would be put on the Proxy by returning _result_, if the observation “character: _value2_” may be performed on _target_, then _value2_ must be the same as _value_.

Proof: TBW

## Application: The integrity check for the \[\[Delete]] method

The following algorithm is a valid implementation for IntegrityCheck\_Delete(_arguments_, _result_, _target_).

Steps 4-7 of the algorithm mirror steps 9-12 of the currently specced [\[\[Delete\]\] internal method of Proxies] (https://tc39.github.io/ecma262/#sec-proxy-object-internal-methods-and-internal-slots-delete-p) (where “throw a TypeError exception” is replaced by “return false”, meaning that the integrity check is not passed, and, similarly, “return result” is replaced by “return **true**”.)

Steps 8-9 are a currently missing check that is proposed in [PR 666] (https://github.com/tc39/ecma262/pull/666/files).

1. Assert: _result_ is a Boolean.
2. Assert: _arguments_ is a List containing a unique element, which is a property key.
3. Let _P_ the unique item of _arguments_.
4. If _result_ is **false**, return **true**.
5. Let _targetDesc_ be ? _target_.\[\[GetOwnProperty]](_P_).
6. If _targetDesc_ is **undefined**, return *true*.
7. If _targetDesc_.\[\[Configurable]] is **false**, return **false**.
8. Let _extensibleTarget_ be ? _target_.\[\[IsExtensible]]().
9. If _extensibleTarget_ is *false*, return **false**.
10. Return **true**.

Let us analyse that algorithm. Steps 1-3 extract the relevant values, namely _result_ and _P_.

Step 4: if _result_ is **false**, no observation would be made on the Proxy, so that the integrity check is passed.

Otherwise, _result_ is **true**, and the following observation would be made on the Proxy:

> exists(_P_): false

Steps 5 and 6 checks whether the same observation “exists(_P_): false” may be made on the target. If so, the integrity check is passed.

Otherwise, the observation “exists(_P_): true” is performed on _target_. At this point, we should check whether this is not contradicted by a potential lock “@exists(_P_): true” on _target_ (condition A) or “@exists(_P_): false”  on the Proxy (condition B).

In step 7, if _targetDesc_.\[\[Configurable]] is **false**, then the lock “@configurable(_P_): false” is put on _target_, from which it follows that the lock “@exists(_P_): true” is also put on _target_. By Condition A, the integrity check is not passed.

Steps 8-9: If the observation “extensible: false” is made on _target_, then we have the lock “@extensible: false” on _target_. Now, assuming that such a lock is put on the Proxy, observing “exists(_P_): false” on the Proxy would put the lock “@exists(_P_): false”. By Condition B, the integrity check is not passed.

Step 10. Otherwise, no potential lock of the character “exists” may be put, either on the Proxy or on _target_. Therefore the integrity check is passed.
