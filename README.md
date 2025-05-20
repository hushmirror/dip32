# DIP32 Implementation Plan

This document serves as a plan to implement DIP32 "Confidential Transactions" (CTs)
as described in https://github.com/dashpay/dips/pull/161

If you have feedback on something below or see something missing or incorrect, please open an Issue .

# Deliverables

The main goal is the create a PR in the https://github.com/dashpay/dash repo
which consists of a working implementation of CTs. This code should also be
audited by third parties but the audit process is considered out of scope for
this document.

A more detailed list of deliverables is:

  * Update libsecp256k1 to a version which supports CT operations
  * Implement all new RPCs relating to CTs and modify all existing RPCs which need changes due to CTs
  * Implement consensus changes to validate CTs
  * Implement code which activates CT on Dash mainnet
  * Add C++ unit tests relating to CT internals
  * Add python tests which create valid CTs to make sure they are seen as valid
  * Add python tests which create invalid CTs to make sure they are seen as invalid
  * There should be at least one test for each permutation of arguments for each RPC which test successful behavior
  * There should be at least one test for each error condition for each RPC which tests failure behavior
  * Updated RPC docs for new and changed methods

These deliverables will be described in more detail below. Some of these
deliverables depend on others while some can be worked on independently. They
are listed in roughly the order that things should be worked on. For instance,
updating libsecp256k1 should be the first or one of the first things
done, because it has the most "unknown unknowns". It is currently unclear
exactly which fork of libsecp256k1 will be used and how the version currently
in Dash core has diverged from it. 

## Updating libsecp256k1

  * What are the exact options of forks of libsecp256k1 and their URLs?
    * Which branch/commit/release of these potential forks is best for Dash to use?
    * What is the divergence between those forks and the version of libsecp256k1 that exists currently in Dash core?
  * Once the above questions are answered and a specific commit of a specific libsecp256k1 fork is identified, that code needs to be integrated into the Dash Core codebase
    * Specifically that means updating the code in the `src/secp256k1` directory https://github.com/dashpay/dash/tree/develop/src/secp256k1
## New RPCs

The following is a list of new RPCs which will be needed to support CTs:

  * `getnewctaddress`
    * Args: none
    * Generates a random pubkey as well as a random salt or blinding factor and stores this data in wallet.dat and returns the base58 encoded representation
    * This address will have a different base58 prefix from normal Dash addresses
  * `listctaddresses`
    * Args: none
    * Returns a list of all CT addresses
  * `validateaddress`
    * Args: string (required)
    * Recognize CT addresses as valid/invalid and return metadata about them
  * `createrawcttransaction`
    * Args: 
    * Add ability to create raw CT transactions via specifying the amount of an input UTXO

Additional new RPCs may be required.

## Modified RPCs

At a minimum, the following RPCs will be modified to support CTs:

  * `importprivkey`
    * Args: string (required)
    * Add ability to import a CT address private key
  * `getrawtransaction`
    * Args: string (required)
    * Add ability to return data about CT transactions
  * `gettransaction`
    * Args: string (required)
    * Add ability to return data about CT transactions
  * `getreceivedbyaddress`
    * Args: string (required)
    * Add ability to get data about a CT address
  * `dumpprivkey`
    * Args: string (required)
    * Add ability to dump a CT address private key
  * `dumpwallet`
    * Args: none
    * Add ability to dump CT address data
  * `rescanblockchain`
    * Args: none or integer (optional) depending on if rescanning from a custom height is supported
    * Add ability to recognize CT transactions owned by current wallet during a rescan
  * `sendmany`
    * Args:
    * Add ability to send to one or more CT addresses
  * `sendtoaddress`
    * Args:
    * Add ability to send to a CT address

Other RPCs may need to be modified to support CTs.
