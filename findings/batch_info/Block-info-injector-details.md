# BlockInfoInjector

- BlockInfoInjector interface facilates insertion of BlockInfo transaction at the begining of every block. 
- BlockInfo transaction updates global state with information about last committed block as well as timestamp.

# BlockInfo Transaction Family

- BlockInfo transaction family provides way for storing information about configurable number of historic blocks.
- BlockInfo transactions should only be added to block by BlockInfo injector and validation rules should ensure only one transaction of this type is added at the begining of the block.

## How block information is stored and addressed by BlockInfo transaction family?

**Address**  
top level namespace of this transaction family is 00b10c. this namespace is further subdivided on next two characters as:  
- 00b10c01 - metadata namespace. Provides information about block info config and metadata.  
zero-address formed by concatenating metadata namespace and enough zeroes to form valid address will be used to store BlockInfo configuration.

- 00b10c00 - block info namespace. Provides historic block information.
Additional block information is stored under block info namespace address at global state is derived from block number.  

Procedure to compute address as follows:
- Convert block_num to a hex string and remove the leading “0x”
- Left pad the string with 0s until it is 62 characters long
- Concatenate the block info namespace address and the string from step 2

For example: address can be constructed in python as
```
'00b10c00' + hex(block_num)[2:].zfill(62)
```

## BlockInfo 

**BlockInfoConfig protobuf structure:**

```
message BlockInfoConfig {
  uint64 latest_block = 1;
  uint64 oldest_block = 2;
  uint64 target_count = 3;
  uint64 sync_tolerance = 4;
}
```

**Block information stored at the block info namespace address in protobuf:**

```
  // Block number in the chain
  uint64 block_num = 1;

  // The header_signature of the previous block that was added to the chain.
  string previous_block_id = 2;

  // Public key for the component internal to the validator that
  // signed the BlockHeader
  string signer_public_key = 3;

  // The signature derived from signing the header
  string header_signature = 4;

  // Approximately when this block was committed, as a Unix UTC timestamp
  uint64 timestamp = 5;
}
```

**Transaction Payload**

BlockInfo transaction Payload is defined by protobuf message as follows:
```
message BlockInfoTxn {
  // The new block to add to state
  BlockInfo new_block = 1;

  // If this is set, the new target number of blocks to store in state
  uint64 target_count = 2;

  // If set, the new network time synchronization tolerance.
  uint64 sync_tolerance = 3;
}
```

### Transcation Header

**Inputs and Outputs**
Inputs for BlockInfo transactions must include
```
Address of BlockInfoConfig
BlockInfo namespace
```

Outputs for BlockInfo transactions must include
```
Address of BlockInfoConfig
BlockInfo namespace
```

**Dependencies**
None.

**Family**
```
family_name: "block_info"
family_version: "1.0"
```

## Execution

BlockInfo TP execution follow below steps.
- Transaction validation: Validation is done for block number, previous block id, signer public key, header signature (proper hex value) and make sure time stamp is greater than 0. If any of these checks fail, transaction is invalid.

- Read latest and oldest block numbers, target number of blocks and synchronization tolerance from BlockInfoConfig address

- If the config does not exits, treat it as first transaction entry. Add sync time and target block count to the config object along with block number from transaction payload. Add this updated config to state.

- If config exists, do following checks.
  - If target_count was set in the transaction, use the new value as 
    the target number of blocks for the rest of the procedure and update the config.
  - If sync_tolerance was set in the transaction, use the new value as 
    the synchronization tolerance for the rest of the procedure and update the config. 
  - Verify block number in new BlockInfo protobuf message is one greater than 
    most recent block in state. If not, transaction is invalid.
  - Verify timestamp in the nw BlockInfo message following the rules below. 
    If not, then transaction is invalid.
     - Verify previous block id in the BlockInfo is equal to the block id of 
       the most recent block stored in state. If not equal, transaction is invalid.
- Finally calculate the address of the new block number. Write BlockInfo 
     BlockInfo message to the state at the constructed address for that block.
- If number of blocks stored in the state is greater than the target number of blocks, delete the oldest BlockInfo message from state.
- Write most recent and oldes block numbers, target number of blocks to the config zero-address.

## Timestamp concerns.

- Timestamp handling in distributed network is difficut because peers may not have synchronized clocks. 
- Clock of the network may change due to either peers or bad actors.
- If clock of the network is skewed then transactions that depend on clock may become invalid. 
- If block validation depends on timestamp validation, peers may not be able to publish blocks untill their clocks are adjusted to match network clock.

**BlockInfo transaction family uses following timestamp validation rules**

- Timestamp in the new BlockInfo message must be greater than timestamp in most recent BlockInfo message in the state.
- Timestamp in the new BlockInfo message must be less than peer's local time adjusted to UTC plus network sync tolerance. 



