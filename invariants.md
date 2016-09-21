# A model for invariants of internal methods
## Characters

A **<dfn>character</dfn>** is a predefined key to which is associated a value taken from a predefined set.
The association between a character and its value is denoted as:

>  character: _value_

Characters are used to describe observed characteristics of an [Object] (https://tc39.github.io/ecma262/#sec-object-type).
The table below gives the list of characters;
_P_ represents an arbitrary property key (i.e., a String or a Symbol).

 Character          | Possible values    | Meaning
--------------------|--------------------|-------
 prototype          | _Object_, **null** | the prototype of an object
 extensible         | _Boolean_          | whether the object is extensible
 exists(_P_)        | _Boolean_          | whether property _P_ exists
 configurable(_P_)  | _Boolean_          | whether a property is configurable
 enumerable(_P_)    | _Boolean_          | whether a property is enumerable
 type(_P_)          | data, accessor            | whether _P_ is a data property or an accessor property
 writable(_P_)      | _Boolean_          | whether a data property is writable
 value(_P_)         | _ECMAScript value_        | the value of a data property
 getter(_P_)        | _function_, **undefined** | the getter of an accessor property
 getter-undefined(_P_) | _Boolean_ | whether _P_ is an accessor property with an undefined getter
 setter(_P_)        | _function_, **undefined** | the setter of an accessor property
 setter-undefined(_P_) | _Boolean_ | whether _P_ is an accessor property with an undefined setter
 
## Observations

When an [internal method] (https://tc39.github.io/ecma262/#sec-object-internal-methods-and-internal-slots) of an Object returns nonabruptly,
an **<dfn>observation</dfn>**, that consists of a set of association of character and value, may be performed on that Object.

Below is the list of performed observations.
When an internal method returns a value not in the table (e.g., \[\[SetPrototypeOf]] (_V_) returns **false**),
no observation is done.

Internal method    |   Returned value  | Observation performed
-------------------|-------------------|-------------
\[\[GetPrototypeOf]] () | _V_               | prototype: _V_
\[\[SetPrototypeOf]] (_V_) | **true** | prototype: _V_
\[\[IsExtensible]] () | _V_  | extensible: _V_
\[\[PreventExtensions]] () | **true** | extensible: **false**
\[\[GetOwnProperty]] (_P_) | **undefined** | exists(_P_): **false**
\[\[GetOwnProperty]] (_P_) | { \[\[Value]]: _V_,<br> \[\[Writable]]: _W_,<br> \[\[Configurable]]: _C_,<br> \[\[Enumerable]]: _E_ } | configurable(_P_): _C_<br>exists(_P_): **true**<br>enumerable(_P_): _E_<br>type(_P_): data<br>writable(_P_): _W_<br>value(_P_): _V_
\[\[GetOwnProperty]] (_P_) | { \[\[Get]]: _G_,<br> \[\[Set]]: _S_,<br> \[\[Configurable]]: _C_,<br> \[\[Enumerable]]: _E_ } | configurable(_P_): _C_<br>exists(_P_): **true**<br>enumerable(_P_): _E_<br>type(_P_): accessor<br>getter(_P_): _G_<br>getter-undefined(_P_): (_G_ is **undefined**)<br>setter(_P_): _S_<br>setter-undefined(_P_): (_S_ is **undefined**)
\[\[DefineProperty]] (_P_, {  }) | **true** | exists(_P_): **true**
\[\[DefineProperty]]<br> (_P_, { \[\[Configurable]]: _C_ }) | **true** | configurable(_P_): _C_<br>exists(_P_): **true**
\[\[DefineProperty]]<br> (_P_, { \[\[Enumerable]]: _E_ }) | **true** | exists(_P_): **true**<br>enumerable(_P_): _E_
\[\[DefineProperty]]<br> (_P_, { \[\[Value]]: _V_ }) | **true** | exists(_P_): **true**<br>type(_P_): data<br>value(_P_): _V_
\[\[DefineProperty]]<br> (_P_, { \[\[Writable]]: _W_ }) | **true** | exists(_P_): **true**<br>type(_P_): data<br>writable(_P_): _W_
\[\[DefineProperty]]<br> (_P_, { \[\[Getter]]: _G_ }) | **true** | exists(_P_): **true**<br>type(_P_): accessor<br>getter(_P_): _G_<br>getter-undefined(_P_): (_G_ is **undefined**)
\[\[DefineProperty]]<br> (_P_, { \[\[Setter]]: _S_ }) | **true** | exists(_P_): **true**<br>type(_P_): accessor<br>setter(_P_): _S_<br>setter-undefined(_P_): (_S_ is **undefined**)
\[\[HasProperty]] (_P_) | **false** | exists(_P_): **false**
\[\[Get]] (_P_, _Receiver_) | _any_ | _see below_
\[\[Set]] (_P_, _V_, _Receiver_) | **true** | _see below_
\[\[Delete]] (_P_) | **true** | exists(_P_): **false**
\[\[OwnPropertyKeys]] ( ) | _List_ | exists(_P_): **true** for every _P_ in _List_<br>exists(_P_): **false** for every property key _P_ not in _List_

The observations triggerd by the \[\[Get]] and \[\[Set]] internal methods depends on the locks (as defined later) currently put on the Object.
(Note that a lock involving character “type(_P_)” implies the locks “@configurable(_P_): **false**” and “@exists(_P_): **true**”.)

Internal method    |   returned value  |  previously put lock | observation
-------------------|-------------------|----------------------------|----------
\[\[Get]] (_P_, _Receiver_) | _V_ |  @type(_P_): data | value(_P_): _V_
\[\[Get]] (_P_, _Receiver_) | anything but **undefined** |  @type(_P_): accessor | get-undefined(_P_): **false**
\[\[Set]] (_P_, _V_, _Receiver_) | **true** |  @type(_P_): data | value(_P_): _V_
\[\[Set]] (_P_, _V_, _Receiver_) | **true**  | @type(_P_): accessor | set-undefined(_P_): **false**

# Locks

A **<dfn>lock</dfn>** is an association of a character with some value.
Its purpose is to signal that a given character can only be observed with some fixed value. 

A lock is denoted as follows:

> @character: value

Here is the list of possible locks. There is a relation of dependance between locks, which is mirrored by the nesting in the list;
for example, the lock “@prototype: _V_” depends on “@extensible: **false**.

* @extensible: **false**
  * @prototype: _any_
  * @exists(_P_): **false**
* @configurable(_P_): **false**
  * @exists(_P_): **true**
  * @enumerable(_P_): _any_
  * @type(_P_): data
    * @writable(_P_): **false**
      * @value(_P_): _any_
  * @type(_P_): accessor
    * @getter(_P_): _any_
    * @getter-undefined(_P_): _any_
    * @setter(_P_): _any_
    * @setter-undefined(_P_): _any_

## Putting a lock

A lock “@character: value” is put on an Object when:
* all the locks on which “@character: value” depends are previously put on that Object;
* an observation “character: value” is done on that Object.

## Invariants of internal methods

The [Invariants of Essential Internal Methods] (https://tc39.github.io/ecma262/#sec-invariants-of-the-essential-internal-methods)
are of two kinds

* fixing the form of the value of a nonabrupt completion (e.g., \[\[IsExtensible]]() returns a Boolean);

* enforcing the following statement:
  > If a lock ”@character: _value_” is put on an Object, there shall be no subsequent observation ”character: _value2_” on that Object for _value2_ different from _value_.

## Example 

Let _O_ be an Object.

1. _O_.\[\[SetPrototypeOf]] (_V_) returns **true** — an observation ”prototype: _V_” is done. No lock is put.
1. _O_.\[\[IsExtensible]] () returns **true** — an observation ”extensible: **true**” is done. No lock is put.
1. _O_.\[\[PreventExtensins]] () returns **true** — an observation ”extensible: **false**” is done. The lock “@extensible: **false**” is put.
1. _O_.\[\[GetPrototypeOf]] () returns _W_ — an observation ”prototype: _W_” is done. The lock “@prototype: _W_” is put.
1. _O_.\[\[SetPrototypeOf]] (_W_) returns **true** — an observation ”prototype: _W_” is done. No new lock is put.
1. _O_.\[\[SetPrototypeOf]] (_X_), where _X_ is not _W_, returns **true** — an observation ”prototype: _X_” is done, breaking the lock “@prototype: _W_”: that shall not have happened.





