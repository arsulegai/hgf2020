## Following defines the Validator service with BlockInfo transaction settings applied 

```
validator:
    build:
      context: ../../..
      dockerfile: ./validator/Dockerfile
      args:
        - http_proxy
        - https_proxy
        - no_proxy
    image: sawtooth-validator$INSTALL_TYPE:$ISOLATION_ID
    volumes:
      - $SAWTOOTH_CORE:/project/sawtooth-core
    expose:
      - 4004
      - 8800
    # the namespaces for the intkey family are purposefuly incorrect
    command: "bash -c \"\
        sawadm keygen && \
        sawset genesis \
          -k /etc/sawtooth/keys/validator.priv \
          -o config-genesis.batch && \
        sawset proposal create \
          -k /etc/sawtooth/keys/validator.priv \
          sawtooth.validator.transaction_families='[
            {\\\"family\\\": \\\"block_info\\\",
             \\\"version\\\": \\\"1.0\\\",
             \\\"namespaces\\\": [\\\"00b10c\\\"]},
            {\\\"family\\\": \\\"intkey\\\",
             \\\"version\\\": \\\"1.0\\\",
             \\\"namespaces\\\": [\\\"1cf125\\\",\\\"1cf127\\\"]},
            {\\\"family\\\":\\\"sawtooth_settings\\\",
             \\\"version\\\":\\\"1.0\\\",
             \\\"encoding\\\":\\\"application/protobuf\\\"},
            {\\\"family\\\":\\\"xo\\\",
             \\\"version\\\":\\\"1.0\\\",
             \\\"namespaces\\\":[\\\"5b7349\\\"]}]' \
          -o config_transaction_families.batch && \
        sawset proposal create \
          -k /etc/sawtooth/keys/validator.priv \
          sawtooth.consensus.algorithm.name=Devmode \
          sawtooth.consensus.algorithm.version=0.1 \
          sawtooth.validator.batch_injectors=block_info \
          'sawtooth.validator.block_validation_rules=NofX:1,block_info;XatY:block_info,0;local:0' \
          -o config.batch && \
        sawadm genesis \
          config-genesis.batch config_transaction_families.batch config.batch && \
        sawtooth-validator --endpoint tcp://validator:8800 -vv \
            --bind component:tcp://eth0:4004 \
            --bind network:tcp://eth0:8800 \
            --bind consensus:tcp://eth0:5005 \
    \""
```

## Below service defines BlockInfo transaction processor

```
block-info-tp:
    build:
      context: ../../..
      dockerfile: ./families/block_info/Dockerfile
      args:
        - http_proxy
        - https_proxy
        - no_proxy
    image: sawtooth-block-info-tp$INSTALL_TYPE:$ISOLATION_ID
    volumes:
      - $SAWTOOTH_CORE:/project/sawtooth-core
    expose:
      - 4004
    depends_on:
      - validator
    command: block-info-tp -vv -C tcp://validator:4004
    stop_signal: SIGKILL
```
