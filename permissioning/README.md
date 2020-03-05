# PERMISSIONS IN SAWTOOTH

Hyperledger Sawtooth allows us to set permissions to
who can send the transactions to the network as well
as which node can join the validator network.

The permissioning related to transaction is called and
referred to as `Transactor Permissioning`. The
permissioning related to joining a validator to the
network is called `Validator Permissioning`.

The permissioning can be both on-chain and off-chain.
This section is for hands-on with the on-chain permissioning.

## Pre-requisites

* Sawtooth Setting TP (stores the settings on-chain)
* Sawtooth Identity TP (provides the RBAC mechanism)

## Steps

1. Add an `admin` user to the list of keys allowed to
perform permissioning related changes. By default only
the genesis validator has all the permissions to do so.
2. Create a `policy`. The policy is a assertion that
either grants a permission or denies a permiossion to
the list of public keys associated with the users.
3. Create a `role`. The role is to identify set of `policy`.
A role can be associated with a set of policies.
4. Assign `policies` to the `role`.

## Hands-On

**Validator Permissioning**

Four docker-compose files are placed under `permissioning`
folder. Each file corresponds to one organization. The
validator in node 4 is not authorized in the demo scripts,
thus when it is attempted to join the network the other
validators won't allow that particular validator becoming
part of the network.

- Run the following command to add the user key to the
administration list

```shell_script
$ sawset proposal create --key /pbft-shared/validators/validator-1.priv sawtooth.identity.allowed_keys=$(cat ~/.sawtooth/keys/root.pub) --url http://rest-api-1:8008
```

- Run the following command to add a new policy to the network

```shell_script
$ sawtooth identity policy create all_are_welcome_here "PERMIT_KEY *" --url http://rest-api-1:8008
```

It can be verified using

```shell_script
$ sawtooth identity policy list --url http://rest-api-1:8008
```

- Run the following command to create a new role and assign roles

```shell_script
$ sawtooth identity role create transactor all_are_welcome_here
```
