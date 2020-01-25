# Testing in the sawtooth-core repository

- Integration tests written to validate differebt functionalities.
- Folder for the integration tests is https://github.com/hyperledger/sawtooth-core/tree/master/integration/sawtooth_integration
- Sets up a sample network as container services, runs the test driver code.
- A framework is written to facilitate some of the repeated tasks, such as creating a batch, submitting the batch, waiting for the response through REST API, timeout in REST API.
- Integration is packaged as a debian as well.
