# Contract state

State declarations are provided in a form of `global`|`owned` STATE\_NAME POSTFIX ::  STATE\_TYPE, where POSTFIX denotes how the state is accumulated with each of the operations. The following postfixes are allowed:

| Postfix  | Applicable to     | Meaning                                                                                                                                                                     |
| -------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| none     | `global`, `owned` | _Singular state_: contract has just a single value or defined single-use-seal for the state of that time                                                                    |
| `?`      | `global`, `owned` | _Optional state_: contract may have no state of this type - or can have a singular state.                                                                                   |
| `*`      | `global`, `owned` | State is a _list_, to which genesis and contract operations can add more item(s). The list may be empty, genesis may not assign an initial value for the state.             |
| `+`      | `global`, `owned` | State is a _non-empty list_, to which genesis and contract operations can add more item(s). The list can't be empty, genesis must assign an initial value for the state     |
| `{*}`    | `global` only     | State is a _set_, to which genesis and contract operations can add more item(s). The set may be empty, genesis may not assign an initial value for the state.               |
| `{+}`    | `global` only     | State is a _non-empty set_, to which genesis and contract operations can add more item(s). The set can't be empty, genesis must assign an initial value for the state       |
| `{Key*}` | `global` only     | State is a _map_, to which genesis and contract operations can add more keyed item(s). The map may be empty, genesis may not assign an initial value for the state.         |
| `{Key+}` | `global` only     | State is a _non-empty map_, to which genesis and contract operations can add more keyed item(s). The map can't be empty, genesis must assign an initial value for the state |

Lists, sets and maps are called _state collections_. Lists in case of a global state are ordered according to the [consensus ordering](https://github.com/RGB-WG/rgb-core/issues/117); for owned state the data are not ordered in any specific order.&#x20;

Only a part of the owned state can be known, i.e. postfix doesn't guarantee the availability of all owned state data as a part of the known contract state.&#x20;

Postfix puts a requirements on the state which can or must be provided by genesis and state transitions. This is the summary of these requirements:

| Schema                                                   | Genesis                                                                                                                                                                                               | Transitions                                                                                                                                                                                           | Contract state                                                    |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `State :: Type`                                          | `<- State`                                                                                                                                                                                            | <p>none, <br><code>&#x3C;- State</code><br><code>&#x3C;- State?</code></p>                                                                                                                            | Single, always present value                                      |
| <p><code>State?</code> <br><code>:: Type</code></p>      | <p>none,<br><code>&#x3C;-State?</code></p>                                                                                                                                                            | <p>none,<br><code>&#x3C;-State?</code></p>                                                                                                                                                            | Optional value of type                                            |
| <p><code>State*</code> <br><code>:: Type</code></p>      | <p>none,<br><code>&#x3C;-State?</code><br><code>&#x3C;-State</code><br><code>&#x3C;-[State]</code><br><code>&#x3C;-[State+]</code><br><code>&#x3C;-{State}</code><br><code>&#x3C;-{State+}</code></p> | <p>none,<br><code>&#x3C;-State?</code><br><code>&#x3C;-State</code><br><code>&#x3C;-[State]</code><br><code>&#x3C;-[State+]</code><br><code>&#x3C;-{State}</code><br><code>&#x3C;-{State+}</code></p> | List of values, which may be empty, ordered by consensus ordering |
| <p><code>State+</code> <br><code>:: Type</code></p>      | <p><code>&#x3C;-State</code><br><code>&#x3C;-[State+]</code><br><code>&#x3C;-{State+}</code></p>                                                                                                      | <p>none,<br><code>&#x3C;-State?</code><br><code>&#x3C;-State</code><br><code>&#x3C;-[State]</code><br><code>&#x3C;-[State+]</code><br><code>&#x3C;-{State}</code><br><code>&#x3C;-{State+}</code></p> | List of values with at least one value                            |
| <p><code>state{*}</code> <br><code>:: Type</code></p>    | <p>none,<br><code>&#x3C;-State?</code><br><code>&#x3C;-State</code><br><code>&#x3C;-{State}</code><br><code>&#x3C;-{State+}</code></p>                                                                | <p>none,<br><code>&#x3C;-State?</code><br><code>&#x3C;-State</code><br><code>&#x3C;-{State}</code><br><code>&#x3C;-{State+}</code></p>                                                                | Set of values, which may be empty, ordered by values              |
| <p><code>State{+}</code> <br><code>:: {Type+}</code></p> | <p><code>&#x3C;-State</code><br><code>&#x3C;-{State+}</code></p>                                                                                                                                      | <p>none,<br><code>&#x3C;-State?</code><br><code>&#x3C;-State</code><br><code>&#x3C;-{State}</code><br><code>&#x3C;-{State+}</code></p>                                                                | Set of values with at least one value, ordered by values          |
| <p><code>state{Key*}</code> <br><code>:: Type</code></p> | <p>none,<br><code>&#x3C;-{Key->State}</code><br><code>&#x3C;-{Key->+State}</code></p>                                                                                                                 | <p>none,<br><code>&#x3C;-{Key->State}</code><br><code>&#x3C;-{Key->+State}</code></p>                                                                                                                 | Map from keys to types, which may be empty, ordered by key value  |
| `State{Key+} :: {Type}`                                  | <p>none,<br><code>&#x3C;-{Key->+State}</code></p>                                                                                                                                                     | <p>none,<br><code>&#x3C;-{Key->State}</code><br><code>&#x3C;-{Key->+State}</code></p>                                                                                                                 | Map from keys to types, with at least one key and value           |

Other confined bounds (like `[T; N]`, `[T^A..B]` etc) than the above listed comprehensions are prohibited to appear in the schema state declaration. The maximum number of elements in a global state collection is always restricted to `0xFFFF`, the maximum number of state items in an owned state is not bounded to any upper limit (due to inability of enforcing that restriction).

If a global state needs to be a collection type by itself (for instance to be a string), it can be declared in a usual manner, with all comprehensions allowed.

It is important to remember about the difference between collections provided as a state value type, and a state itself operating as a collection. For instance,

```haskell
schema Some
    global State :: [Utf8]
```

means that the `State` consists of a single string, and it is replaced with new operations providing that state, while

```haskell
schema Some
    global State[] :: [Utf8]
```

means that the state is list of strings which accumulate over the time. If both of the contracts are created with `State := "abc"` in genesis, and than receive three consequent transitions with `State := "xyz"`, `State := "klm"` and `State := "qrs"`, the resulting contract state in the first case would be `State := ["abc", "xyz", "klm", "qrs"]` and in the second case `State := "qrs"`
