# Hedgehog

## Multi sign

Multi sign in hedgehog makes it requier mutliple signatuers to affect state changes in the sporks.

You take some data and let two people sign it with 
```
./hedgehog.bin util key-sign --data=<data> --key=<privatekey>
```

Do this with two keys and save the response.

Do the request to the REST. e.g curl to mintstorage

```
curl -X PUT --header "Content-Type: application/json" 
--header "privateKey: dd58f07a4896f15ed1f3a5dab030068fe353e0a16c141bdbce084ecd90fa608c9de4c0b1a1397b0f7110cd75f79745e30cd679be2994fc25d6ca09cd80b29fed2b" 
-d '{
    "amount" : "amount",
    "signatures" : [
        "signature1",
        "signature2"
    ],
    "data"; "data"
}' 
-w "%{http_code}\n" -v -k 
https://127.0.0.1:52884/gridspork/mint-storage/y3x9dL1LEwLDVhMFLTmtEeaYMfgbYoeoCn/425
```

the signature and data part is the same for all calls that uses multisign the other data in the json looks diffrent based on what call it is.