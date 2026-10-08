# Hedgehog

## Multi sign

Multi sign in hedgehog makes it require two different board keys to change a grid spork (mint storage, mint supply, vesting
storage and storage settings). The first key proposes the change, the second key co-signs it. Only then is the change stored and
published to the network. Read more in the Hedgehog wiki under Grid sporks.

### Propose the change

The first board member proposes the change with the private key. The node signs it and answers `202 Accepted`, the change is now
pending. A proposal is held for 60 minutes.

With the CLI. The data is the amount to grow the mint with

```
./hedgehog.bin cli gridspork-grow mint-storage --address=<address> --height=<height> --data=<amount> --key=<privatekey>
```

or the same thing against the REST interface, e.g curl to mint-storage

```
curl -X PUT -k -v -w "%{http_code}\n"
--header "Authorization: Bearer <resttoken>"
--header "Content-Type: application/json"
--header "privateKey: <privatekey>"
-d '<amount>'
https://127.0.0.1:52884/gridspork/mint-storage/<address>/<height>
```

Every REST request needs the bearer token. It is the value of `--resttoken` or `HEDGEHOG_REST_TOKEN`, or if neither is set the
content of the file `rest.token` in the data directory of the node.

### Co-sign the change

List the pending proposals and note the digest of the one to co-sign

```
./hedgehog.bin cli gridspork-pending
```

The second board member co-signs it with another private key

```
./hedgehog.bin cli gridspork-cosign --key=<privatekey> <digest>
```

or with the REST interface

```
curl -X PUT -k -v -w "%{http_code}\n"
--header "Authorization: Bearer <resttoken>"
--header "privateKey: <privatekey>"
https://127.0.0.1:52884/gridspork/pending/<digest>
```

The responses are `200` when it is stored, `404` when no proposal has that digest, `401` when the key is not a network key and
`409` when the proposal is refused, e.g when both signatures come from the same key.

The same two steps work for the other sporks. Mint supply and storage settings are proposed with `gridspork-set` and
`/gridspork/mint-supply` or `/gridspork/storage`. Vesting storage is proposed with REST only, on
`/gridspork/vesting-storage/<address>`.

### Other useful commands

```
./hedgehog.bin util key-generate
./hedgehog.bin util key-sign --data=<hexdata> --key=<privatekey>
./hedgehog.bin cli gridspork-list
./hedgehog.bin cli gridspork-log
```

`key-generate` makes a new key pair, `key-sign` signs hex data with a key, `gridspork-list` shows the stored sporks and
`gridspork-log` shows who signed each version of them.

If the network keys have changed, `gridspork-renew` followed by `gridspork-cosign` re-signs the sporks the node holds with the
new keys.

Never commit or paste a private key into documentation or a shared channel.
