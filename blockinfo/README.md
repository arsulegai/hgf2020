# BlockInfo Injector

Hyperledger Sawtooth allows batch injection be done from the validator
directly into the generated block. The block info transaction family
is packaged with the Hyperledger Sawtooth validator, which can be enabled
through a settings transaction.

BlockInfo transaction family allows to add a block number and timestamp
to the block. Since these information are written to the global state,
they are available for consumption by the user smart contract.

## Hands-On

- Bring up a single node validator network with the `BlockInfo`
transaction processor.

```shell_script
$ docker-compose up
```

- Send a sample transaction through the `pc-cli` container

```shell_script
$ ./target/debug/pc-cli -C PRODUCE -I Bread -Q 10 -K /keys/validator.priv --url http://rest-api:8008
```

Expect a mew block generated upon submitting the transaction.
Run the following command to show the latest block information

Run this command either from the `shell` or the `validator` container
(where you find the `sawtooth` command installed)

```shell_script
$ sawtooth block list --url http://rest-api:8008
$ sawtooth block show <BLOCK-ID> --url http://rest-api:8008
```

- Decode the payload to see what's inside it. You may make use of the
repository [BlockInfo Decoder](https://github.com/arsulegai/blockinfo-decoder).

**Where:**
- Remember that `http://rest-api:8008` should be replaced with the REST API's
address in your deployment.
- <BLOCK-ID> gets replaced with the block header signature.
