# Week 2 Ledger State and Updates

## The Unspent Transaction Output Model

### Exercise - Read your first UTXO State

In this exercise, we will read our first UTXO state via Overledger’s autoExecuteSearchUtxo API. The documentation for this endpoint can be found [here](https://docs.overledger.io/#operation/autoExecuteSearchUtxoRequest). 

Note that unlike blocks and transactions, the state data model of utxo and accounts based DLTs do have to diverge somewhat. This is because of the wide variety of parameters in the state of both models.

#### DLT Network Information

We will be interacting with the Bitcoin testnet. The relevant Overledger location object is as follows:

``Location = {“technology”: “Bitcoin”, “network”: “Testnet”}``

#### Prerequisites

It is assumed that you have already setup your environment by following [these instructions](./Exercise1.md) and that you have completed the previous exercises to search for a block using Overledger [here](./Exercise2.md) and to search for a UTXO transaction using Overledger [here](./Exercise3.md).

#### Searching for the Largest Unspent UTXO in a block

We will demostrate searching the UTXO state through a specific example. This example will search for the largest Unspent UTXO in a recent block of the Bitcoin test network. To run the example, enter:

```
node examples/state-search/autoexecute-utxo-search.js password=MY_PASSWORD
```

You will see in the example script that we are using the `/autoexecution/search/utxo/${utxoId}` Overledger URL to search for the given utxoId.

The full details of this script is as follows. Firstly it gets the latest block, then requests the block that is 20 block's back from the current latest block. We have choosen this particular block, so that hopefully it will contain a variety of UTXO statuses (see below for the list of possible statuses). If the chosen block is empty, the script will complete. Otherwise it will ask Overledger for information on each transaction in the block. The script will then get information from Overledger on each transaction destination. The script checks if each related output is currently unspent or not by utilising Overledger's standardised UTXO status codes, which are as follows:

- UNSPENT_PENDING: This transaction output has been created in a transaction that Overledger currently deems not final (hence the PENDING suffix). This transaction has not yet been spent in any other transaction (hence the UNSPENT prefix).
- UNSPENT_SUCCESSFUL: This transaction output has been created in a transaction that Overledger deems final (hence the SUCCESSFUL suffix). This transaction has not yet been spent in any other transaction (hence the UNSPENT prefix).
- SPENT: This transaction has been spent in another transaction. 
- UNSPENDABLE_PENDING: This transaction output has been created in a transaction that Overledger currently deems not final (hence the PENDING suffix). But this transaction output can never be spent, because it has no unlocking condition (hence the UNSPENDABLE prefix).
- UNSPENDABLE_SUCCESSFUL: This transaction output has been created in a transaction that Overledger deems final (hence the SUCCESSFUL suffix). But this transaction output can never be spent, because it has no unlocking condition (hence the UNSPENDABLE prefix).

All the logic in this script is based on the Overledger standardised data model. This means that the script can easily be reused for other DLTs that are UTXO based. All that is required is to change the location object to another UTXO based DLT network.

##### Overledger Auto Execute UTXO Search API Response

See that the response has two main objects due to Overledger’s preparation and execution model:

1. preparationUtxoSearchResponse: This includes the request id and any QNT fee that must be paid for use of this endpoint.
2. executionUtxoSearchResponse: This includes information on the requested transaction destination and related metadata. 

The utxo information will be returned in cross-DLT standardised form for the utxo data model and in the DLT specific form (referred to as nativeData). This allows maximum flexibility to software developers, as they can choose to use either data models.

##### Auto Execute UTXO Search API Response Main Components

The UTXO response will contain a few main components. Notice that the metadata components (location and status) are the same for a UTXO as they are for a transaction and block. The only difference is the status codes, as described in the section above.

1. Location: Each Overledger DLT data response includes a reference to the location (technology, network) of where this data was taken from. This helps with auditing.

2. Status: Overledger responses regarding utxos come with a status. Due to some DLTs having probabilistic finality of transactions/blocks and other DLTs having deterministic finality of transaction/blocks, the status object is used to indicate to the developer when the requested data is assumed to be final (therefore status.value has the suffix “SUCCESSFUL”) or not (therefore status.value has the suffix “PENDING”). As transaction outputs can have an additional three statuses ("SPENT", "UNSPENT", "UNSPENDABLE", as described above), this part of the status is included as the prefix.

3. Destination = The requested transaction output, presented as the Overledger standardised destination and it's relevant nativeData formats.

For parameter by parameter descriptions see the [documentation](https://docs.overledger.io/#operation/autoExecuteSearchUtxoRequest). 

#### Challenges

##### Searching for a Specific Transaction Output

Take a look at a third party explorer for the Bitcoin testnet we are using, e.g. [here](https://blockstream.info/testnet/). 

Choose a transaction from a block in this explorer. Then choose a specific transaction output. Can you understand how to modify the example script to search for your chosen transaction output?

#### Troubleshooting
This exercise was tested in Ubuntu 20.04.2 LTS Release: 20.04 Codename: focal, with nvm version 0.35.3, and node version 16.3.0. 

This exercise was additionally tested in MacOS Monterey Version 12.0.1, with nvm version 0.39.0, and node version 16.3.0. 

#### Error: bad decrypt 

Description:

```
Secure-env :  ERROR OCCURED Error: error:06065064:digital envelope routines:EVP_DecryptFinal_ex:bad decrypt
```

Cause: the secure env package cannot decrypt the .env.enc file because the provided password was incorrect.

Solution: provide the password with which .env.enc was encrypted when running the script.

#### Error: .env.enc does not exist 

Description:

```
Secure-env :  ERROR OCCURED .env.enc does not exist.
```

Cause: You are missing the encrypted environment file in the folder that you are running from.

Solution: Return to the top level folder and encrypt .env as described in Exercise 1.

#### Error: Missing Password

Description:

```
Error: Please insert a password to decrypt the secure env file.
```

Cause: You did not include the password as a command line option.

Solution: Include the password as a command line option as stated in your terminal print out.