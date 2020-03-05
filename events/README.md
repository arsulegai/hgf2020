# Event Handler

The demo in this section shows a sample event listener attached
to the Sawtooth network. It is for educational purpose only.
One may monitor the logs on the terminal of event-listener to
understand what is being sent back by the smart-contract.

This example demostrates events for block commit and state
delta change being published to event listener.

## Hands-On

- Bring up a single node validator network.

```shell_script
$ docker-compose up
```

- Attach the event listener to the validator instance running

```shell_script
$ docker-compose -f event-handler.yaml
```

- Send a sample transaction through the `pc-cli` container

```shell_script
$ ./target/debug/pc-cli -C PRODUCE -I Bread -Q 10 -K /keys/validator.priv --url http://rest-api:8008
```

You should be seeing a mew block information, along with the state
delta exchange information is sent to the event listener.
