# Batch injection

Sawtooth Validator supports the feature of injecting transactions into batch or blocks.

**Supported Usecases**
- Setting the information in global state that can potentially used by TPs. this information cannot be provided in every single transaction. 
For eg, Setting block number or timestamp for every blocks generated.
- Automatic submission of transactions in response to particular state. 
For eg, bond-quote matching transactions.
- Testing automation. For eg, Computation time of eecution of test transactions can be increased after certain number of batches processed.

## BatchInjector interface
BatchInjector interface is defined as follows.

```
interface BatchInjector:
  // Called when a new block is created and before any batches are added. A list of
  // batches to insert at the beginning of the block must be returned. A StateView
  // is provided for inspecting state as of the previous block.
  block_start(string previous_block_id) -> list<Batch>

  // Called before inserting the incoming batch into the pending queue for the
  // given block. A list of batches to insert before this batch must be returned.
  before_batch(string previous_block_id, Batch batch) -> list<Batch>

  // Called after inserting the incoming batch into the pending queue for the
  // given block. A list of batches to insert after this batch must be returned.
  after_batch(string previous_block_id, Batch batch) -> list<Batch>

  // Called just before finalizing and completing a Block. An ordered list of batches
  // that will be committed in the block is passed in. A list of batches to insert at
  // the end of the block must be returned.
  block_end(string previous_block_id, list<Batch> batches) -> list<Batch>
```

On-chain configuration:

- Set of BatchInjectors to be loaded by validator can be provided as on-chain setting.  
sawtooth.validator.batch_injectors config loads set of batch_injectors to load.  
- Validator parses this list prior to publish every block and appropriate injectors are loaded and stored in global state.

- This setting is controlled by settings-tp.

## On-Chain Validation rules

- Validation rules enforced for each block are stored in string fromat in the setting key ```sawtooth.validator.block_validation_rules```  

**Syntax for validation rules**

- consists of name followed by a colon and comma-separated list of arguments.  
```rulename: arg,arg,....arg```
- Separate multiple rules with semicolons.  
```rulename1: arg,arg,....arg;rulename2: arg,arg,....arg```

**Validation rules**

- ```NofX```  
Only N transactions of type X must be included in a block. first argument is integer and second is name of transaction family. 
For example, ```NofX:2,intkey``` means allow only two intekey transactions per block.

- ```XatY```  
Transaction of type X must be positioned at Y in the block. first argument is name of transaction family and second is an integer indicating index of the transaction in the block that must be checked. 
For example, ```XatY:intkey, 0``` means first transaction in the block must be of type intkey.

- ```local```  
A transaction must be signed by the same key as the block. this rule takes lis t of transaction indices in the block and enforces rule on each. This rule is useful to make sure client is not submitting transactions that should only be injected by winning validator.

