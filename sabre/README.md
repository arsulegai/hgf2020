# PRODUCE-CONSUME
**A sample Hyperledger Sawtooth-Sabre application**

A sample application that is written in rust and compiled into WASM.
The compiled WASM is deployed on the Sabre as a smart contract.

## What is it?

The application can PRODUCE items and CONSUME items after that.

## Prerequisites

The application can be run as docker container. It has been tested on following
version of the docker components.
* Docker version 19.03.4
* Docker compose version 1.24.1

## How to run?

Note: The following commands make it easier for the application developer.
Many of the Hyperledger Sawtooth-Sabre deployment commands are abstracted for the
developer. Please refer to the files `registry.sh` and `submit-txn.sh` files
to know more details.

1. Clone the repository, run the following command

```shell script
$ git clone https://github.com/arsulegai/produce-consume
$ cd produce-consume
$ docker-compose up
```

Wait for all the containers to be up, please ensure that processor container is
up and running before proceeding further. Note that the container may exit with
the status 0. The reason for exit is because, container is used only for
building the WASM smart-contract.

To bring down the setup, please run the following command

```shell script
$ docker-compose down
```

2. Login to the Produce Consume CLI, run the following command

```shell script
$ docker exec -it pc-cli bash
$ cd ..
$ ./cli/target/debug/pc-cli -C PRODUCE -I Bread -Q 10 -K /keys/validator.priv
```

This command produces 10 units of the item "Bread".

3. Login to the Sabre CLI, run the following command

```shell script
$ docker exec -it sabre-cli bash
$ cd /project
$ ./registry.sh
$ ./submit-txn.sh
```

This command would submit the built `processor` to the Sawtooth-Sabre, also
sends the generated `default.batch` (found in the root folder `produce-consume`)
to the `WASM` smart-contract. A sample `contract-definition.yaml` is also 
submitted to the network. Note that the directory paths are hardcoded now.

## Event handler

To add the event handler, run the command from the folder [events](./events)

```shell_script
$ docker-compose -f docker-compose-go.yaml up
```

This will bring up the handler written in Go.

## Debug

A debug docker-compose file for running the smart-contract as a Hyperledger
Sawtooth TP is also provided in the repository.

Run the following command to bring up the Hyperledger Sawtooth network
with the PRODUCE-CONSUME TP and a CLI.

```shell script
$ docker-compose -f debug-sawtooth-docker-compose.yaml up
```

To bring down the setup, please run the following command

```shell script
$ docker-compose -f debug-sawtooth-docker-compose.yaml down
```

Imn order for the client to send the transaction directly to the Sawtooth
network, the `pc-cli` is provided an `--url` or `-U` option. i.e. The
client request to the validator would look like

```shell script
$ ./cli/target/debug/pc-cli -C PRODUCE -I Bread -Q 10 -K /keys/validator.priv --url http://rest-api:8008
```

## Contributing

This software is in development phase and is Apache 2.0 licensed. We accept
contributions via [GitHub](https://github.com/arsulegai/produce-consume) pull
requests.
Each commit must include a `Signed-off-by:` in the commit message
(`git commit -s`). This sign-off means you agree the commit satisfies the
[Developer Certificate of Origin (DCO)](https://developercertificate.org/).

## License
This software is licensed under the [Apache License Version 2.0](LICENSE)
software license.

&copy; Copyright 2019, Walmart Inc.
