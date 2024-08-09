NIP-XXX
=======

Delegations and Revocations Lists for NIP-26 Delegated Event Signing
-----

`draft` `optional`

This NIP defines a way to keep a [NIP-51](https://github.com/nostr-protocol/nips/blob/master/51.md) list of both delegations and revocations, of previously signed [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegations to a publisher public key (a media manager, another device, a sub account).

The delegations list is to be regarded as a delegation acceptance list, and clients and relays can use it to provide a list of active delegations. This list shouldn't be used as a delegation validation list, but only informational.

On the other hand, the revocations list is to be regarded as a compromised delegations list, and consequently that clients and relays should consider all events signed by that public key as compromised as well, and invalidate them.

Revocation is not intended to be the normal flow of removal of a delegation. If a key is not compromised, then simply removing it from the delegations list and assuring that that delegation tag is not used anymore by that key pair is enough. But if the key pair was compromised, then we cannot be assured that that same delegation can't be used to sign future or past events, so a "forced" revocation is needed.

All the lists can be updated by the delegator (main account) directly, of course, but they also need to be updated by another delegatee under certain circumstances, as in some use cases the master key can be in cold storage and hard to use when required.

A delegatee entry in the delegations list can only be added or removed by the delegatee himself using the respective [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegation tag. This removes the possibility of a compromised key to spam or cleanup this list.

A new revocation addition should be allowed to be signed by another delegatee (a media manager, another device, a sub account), as long as all previous revocations are present as well. The revocation list is ever growing, unless the event is signed directly by the delegator (main account) key.

### References

This proposal came together with inputs from many other proposals and discussions, but these are the main ones:

[NIP-26: Delegated Event Signing](https://github.com/nostr-protocol/nips/blob/master/26.md)<br>
[NIP-51: Lists](https://github.com/nostr-protocol/nips/blob/master/51.md)

[Why I don’t like NIP-26 as a solution for key management](https://fiatjaf.com/4c79fd7b.html)<br>
[Thoughts on Nostr key management](https://fiatjaf.com/72f5d1e4.html)

[Stateless key rotation using a series of hidden commitments](https://github.com/nostr-protocol/nips/issues/103)<br>
[Key distribution, rotation, and recovery](https://github.com/nostr-protocol/nostr/issues/45)<br>
[Key rotation verified through root key attestation](https://github.com/nostr-protocol/nips/issues/116)<br>
[Trusted public-key-bundle attestations for key rotation and group definition](https://github.com/nostr-protocol/nips/issues/123)

### Delegations List

This NIP introduces a replaceable event with kind: `10026`, which is used to advertise public keys that have been delegated and accepted to use a [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) `delegation` tag on delegated events.

The `content` field is not used and can be ignored if present.

```json
{
  "id": "567b41fc9060c758c4216fe5f8d3df7c57daad7ae757fa4606f0c39d4dd220ef",
  "kind": "10026",
  "pubkey": "8e0d3d3eb2881ec137a11debe736a9086715a8c8beeeda615780064d68bc25dd",
  "tags": [
    ["p", "07caba282f76441955b695551c3c5c742e5b9202a3784780f8086fdcdc1da3a9"],
    ["p", "a55c15f5e41d5aebd236eca5e0142789c5385703f1a7485aa4b38d94fd18dcc4"],
    ...
  ],
  "content": "",
  ...
  "sig": "a9a4e2192eede77e6c9d24ddfab95ba3ff7c03fbd07ad011fff245abea431fb4d3787c2d04aad001cb039cb8de91d83ce30e9a94f82ac3c5a2372aa1294a96bd"
}
```

The public key of the delegator can never be part of the list.

#### Append a public key by a delegatee

These events are considered valid when signed by the respective [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegatee being added or removed from the list, accompanied by the corresponding `delegation` tag, and if the delegatee public key hasn't been revoked as well (see below).

A [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegatee can append his public key to the delegation list like this:

```json
{
  "id": "d78ba0d5dce22bfff9db0a9e996c9ef27e2c91051de0c4e1da340e0326b4941e",
  "kind": "10026",
  "pubkey": "477318cfb5427b9cfc66a9fa376150c1ddbc62115ae27cef72417eb959691396",
  "tags": [
    [
      "delegation",
      "8e0d3d3eb2881ec137a11debe736a9086715a8c8beeeda615780064d68bc25dd",
      "created_at>1674834236",
      "6f44d7fe4f1c09f3954640fb58bd12bae8bb8ff4120853c4693106c82e920e2b898f1f9ba9bd65449a987c39c0423426ab7b53910c0c6abfb41b30bc16e5f524"
    ],
    ["p", "07caba282f76441955b695551c3c5c742e5b9202a3784780f8086fdcdc1da3a9"],
    ["p", "a55c15f5e41d5aebd236eca5e0142789c5385703f1a7485aa4b38d94fd18dcc4"],
    ["p", "477318cfb5427b9cfc66a9fa376150c1ddbc62115ae27cef72417eb959691396"],
    ...
  ],
  "content": "",
  ...
  "sig": "a9a4e2192eede77e6c9d24ddfab95ba3ff7c03fbd07ad011fff245abea431fb4d3787c2d04aad001cb039cb8de91d83ce30e9a94f82ac3c5a2372aa1294a96bd"
}
```

The public key of the delegatee that is signing the append has to match the one, and only one being added or removed.

#### Removal of a public key by the delegatee

The process is the same as for appending, except that the respective delegatee key is not present anymore on the delegations list, but all other keys must match the ones that were already on the list. This is to prevent a compromised delegatee from deleting the delegations list.

### Revocations List

This NIP introduces the replaceable event with kind: `10126`, which is used to advertise public keys that have been compromised and consequently are no longer considered valid when a [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) `delegation` tag is present in signed events.

The `content` field is not used and can be ignored if present.

Revocation list is ever growing, so clients and relays can consider as invalid a new event that doesn't include all the previous `p` tags, unless it is signed by the delegator key directly and not by a delegatee using the `delegation` tag. This is to prevent a compromised key pair from being able to cleanup the revocation list.

```json
{
  "id": "567b41fc9060c758c4216fe5f8d3df7c57daad7ae757fa4606f0c39d4dd220ef",
  "kind": "10126",
  "pubkey": "8e0d3d3eb2881ec137a11debe736a9086715a8c8beeeda615780064d68bc25dd",
  "tags": [
    ["p", "07caba282f76441955b695551c3c5c742e5b9202a3784780f8086fdcdc1da3a9"],
    ["p", "a55c15f5e41d5aebd236eca5e0142789c5385703f1a7485aa4b38d94fd18dcc4"],
    ...
  ],
  "content": "",
  ...
  "sig": "a9a4e2192eede77e6c9d24ddfab95ba3ff7c03fbd07ad011fff245abea431fb4d3787c2d04aad001cb039cb8de91d83ce30e9a94f82ac3c5a2372aa1294a96bd"
}
```

The public key of the delegator can never be part of the list.

#### Append a public key by a delegatee

These events are considered valid when signed by a [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegatee, with the corresponding `delegation` tag, if the delegatee public key hasn't been revoked as well.

A [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegatee can append public keys to the revocation list like this:

```json
{
  "id": "d78ba0d5dce22bfff9db0a9e996c9ef27e2c91051de0c4e1da340e0326b4941e",
  "kind": "10126",
  "pubkey": "477318cfb5427b9cfc66a9fa376150c1ddbc62115ae27cef72417eb959691396",
  "tags": [
    [
      "delegation",
      "8e0d3d3eb2881ec137a11debe736a9086715a8c8beeeda615780064d68bc25dd",
      "created_at>1674834236",
      "6f44d7fe4f1c09f3954640fb58bd12bae8bb8ff4120853c4693106c82e920e2b898f1f9ba9bd65449a987c39c0423426ab7b53910c0c6abfb41b30bc16e5f524"
    ],
    ["p", "07caba282f76441955b695551c3c5c742e5b9202a3784780f8086fdcdc1da3a9"],
    ["p", "a55c15f5e41d5aebd236eca5e0142789c5385703f1a7485aa4b38d94fd18dcc4"],
    ["p", "eff37350d839ce3707332348af4549a96051bd695d3223af4aabce4993531d86"],
    ...
  ],
  "content": "",
  ...
  "sig": "a9a4e2192eede77e6c9d24ddfab95ba3ff7c03fbd07ad011fff245abea431fb4d3787c2d04aad001cb039cb8de91d83ce30e9a94f82ac3c5a2372aa1294a96bd"
}
```

The public key of the delegatee that is signing the append can never be part of the list, so that a delegatee cannot revoke himself.

### Delegations and Revocations Lists Maintenance

As a safety measure, for the eventual case of a compromised delegatee revoke all other delegatee keys in an attempt to take control of the delegator content, the delegator can always cleanup and maintain these lists freely, without any restriction of keeping prior records, etc.

Also, because in some use cases the delegator key can be in cold storage and inaccessible online, to enable any non-compromised delegatee to maintain these lists the delegator can produce a [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegation tag exclusively for kind `10026` or `10126`, with a small validity window (less than one day, ideally one hour), and clients and relays should consider valid any event of these kinds, when in presence of such a delegation tag, without any of the restrictions, not even checking the revocations list for that key, like it was signed by the delegator key.

### Deletion

[NIP-09](https://github.com/nostr-protocol/nips/blob/master/09.md) event deletion for kinds `10026` and `10126` is only valid if signed by the delegator account directly, without a `delegation` tag. This is to prevent a compromised delegatee from deleting a delegation or a revocation list.
