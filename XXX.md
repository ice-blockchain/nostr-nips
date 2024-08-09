NIP-XXX
=======

Revocation List for NIP-26 Delegated Event Signing
-----

`draft` `optional`

This NIP defines a way to revoke a previously signed delegation to a publisher public key (a media manager, another device, a sub account), and consequently require relays to consider all events signed by that public key as compromised, and invalidate them.

This is not intended for the normal flow of removal of delegation. If a key is not compromised, them simply assuring that that delegation tag is not used anymore by that key pair is enough. But if the key pair was compromised, then we cannot be assured that that same delegation can't be used to sign past events, so this "forced" revocation is needed.

Also this revocation should be allowed to be signed by another delegatee (a media manager, another device, a sub account), as in some use cases the master key can be in cold storage and hard to use when required.

Since this revocation requires clients and relays to keep track of revoked public keys, that are no longer allowed to use the delegation tag to sign by the delegator, it introduces a [NIP-51](https://github.com/nostr-protocol/nips/blob/master/51.md) list for that purpose.

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

### Revocation List

This NIP introduces a replaceable event with kind: `10026`, which is used to advertise public keys that have been compromised, and consequently are no longer considered valid when a [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) `delegation` tag is present in signed events.

This event is considered valid when signed by a [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegatee, with the corresponding `delegation` tag, if the delegatee public key hasn't been revoked as well.

The `content` field is not used and can be ignored if present.

Revocation list is ever growing, so clients and relays can consider as invalid a new event that doesn't include all the previous `p` tags, unless it is signed by the delegator key directly and not by a delegatee using the `delegation` tag. This is to prevent a compromised key pair to be able to cleanup the revocation list.

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

The public key of the delegator cannot ever be part of the list.

#### Append a public key by a delegatee

A [NIP-26](https://github.com/nostr-protocol/nips/blob/master/26.md) delegatee can append public keys to the revocation list like this:

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
    ["p", "eff37350d839ce3707332348af4549a96051bd695d3223af4aabce4993531d86"],
    ...
  ],
  "content": "",
  ...
  "sig": "a9a4e2192eede77e6c9d24ddfab95ba3ff7c03fbd07ad011fff245abea431fb4d3787c2d04aad001cb039cb8de91d83ce30e9a94f82ac3c5a2372aa1294a96bd"
}
```

The public key of the delegatee that is signing the append cannot ever be part of the list, so that delegatees cannot revoke themselves.

### Deletion

[NIP-09](https://github.com/nostr-protocol/nips/blob/master/09.md) event deletion for kind `10026` is only valid if signed by the delegator account directly, without a `delegation` tag. This is to prevent a compromised delegatee from deleting a revocation list.
