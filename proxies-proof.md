# Proof of the main statement about Integrity checks on Proxies

Recall that **<dfn>IntegrityCheck\_<i>Method</i>(_arguments_, _result_, _target_)</dfn>** denotes the abstract operation defining the integrity check associated to \[\[_Method_]].

Given the following condition:

> I. IntegrityCheck\_<i>Method</i>_(_arguments_, _result_, _target_)</i> shall return **true** if all [observations] (/invariants.md#observations) that would be made on the Proxy by returning _result_, may be made on _target_ by invoking fundamental observation methods on it.

we want that to prove that the following condition is satisfied:

> II. If the target of a Proxy is a nonquantum object that observes the [Invariants of internal methods] (/invariants.md#invariants-of-internal-methods), then the Proxy shall observe those invariants.</ol>


when the IntegrityCheck\_<i>Method</i>(_arguments_, _result_, _target_) abstract operations returns **false** unless the two following conditions are enforced:

> A. For any observation “character: _value_” that would be made on the Proxy by returning _result_, if a lock “@character: _valueT_” may be put on _target_ (by invoking fundemental observation methods on it), then _valueT_ must be the same as _value_.
>
> B. Assuming that all the locks that may be put on _target_ (by invoking fundemental observation methods on it) are also put on the Proxy, for any lock “@character: value” that would be put on the Proxy by returning _result_, if the observation “character: _valueT_” may be performed on _target_, then _valueT_ must be the same as _value_.



## Necessity

Let us show that, if the integrity check is passed, Condition A and B must hold.

Consider the following programm:

1\. Construct an ordinary object _target_  for which the observation of characters using its [fundemental observation methods] (/proxies.md#fundemental-observation-methods) would produce some predefined result.

2\. Construct a Proxy _P_ with _target_ as its target object, and whose trap will return some predefined arbitrary result without modifying _target_, as follows:

```js
let result
let P = Proxy(target, {
    getPrototypeOf(t) { return result }
  , setPrototypeOf(t, v) { return result }
  /* etc. */
})
/* 
result = <something>
Reflect.getPrototypeOf(result); // will return <something>
*/
```

3\. For all locks that could be observed on _target_ by calling its fundemental observation methods, observe the same locks on _P_:

```js
result = Reflect.isExtensible(target)
Reflect.isExtensible(P)

result = Reflect.getPrototypeOf(target)
Reflect.getPrototypeOf(P)

result = Reflect.ownKeys(target)
Reflect.ownkeys(P)

foreach (let key of result) {
    result = Reflect.getOwnPropertyDescriptor(target, key)
    Reflect.getOwnPropertyDescriptor(P, key)
}
```

Because of Condition I, integrity checks are past and all these observations are successful.

4\. Call the corresponding internal method of that would trigger the “character: _value_” observation if successful.

```js
result = <desired returned value>
Reflect[Method](P, ...args)
```

As part of the algorithm, IntegrityCheck\_<i>Method</i> is run, that would trigger “character: _valueT_” observation on _target_. By assumption it is passed.

5\. Repeat step 3.

Now we can show:

Condition A. Let us suppose that “@character: _valueT_”  is put on _target_ during the integrity check of step 4. Then, the same lock is put on _P_ during step 3. Because “character: value” is observed on _P_ during step 4, that imposes that _value_ shall be equal _valueT_.

Condition B. Let us suppose that all locks on which “@character: value” depends are put on _target_ during the integrity check of step 4. Then the same locks are put on _P_ during step 3, and the completion of step 4 will put the “@character: _value_” lock on _P_. Finally, step 5 will trigger the observation “character: _valueT_” on _P_, imposing that _valueT_ shall be the same as _value_.


## Sufficiency

Let us assume that some “@character: _value_” lock has been put on a proxy _P_, and let us show that we cannot observe “character: _value2_” on _P_ unless _value2_ is the same as _value_. First, we show that

> The same “@character: _value_” lock is put on _target_ at the latest during the call of the internal method that put “@character: _value_” lock on _P_.

By using recursion, we can assume that this is true for all locks on which “@character: _value_” depends; so it suffices to show that “character: _value_” is observed on the _target_.
If the internal method is not trapped, this is evident. If it is trapped, the enforcement of  Condition B during the integrity check will trigger such an observation.

Now, let us attempt observe “character: _value2_” on _P_ by calling some internal method on _P_. If it is not trapped, because of the existing “@character: _value_” lock on _target_, we observe “character: _value_” again. If it is trapped, the enforcement of  Condition A will guarantee that _value2_ shall be the same as _value_.

