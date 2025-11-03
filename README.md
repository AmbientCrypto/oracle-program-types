# Oracle Program Types

Rust type bindings for the Ambient Tool Oracle Solana program, generated from the program's IDL using Anchor's `declare_program!` macro.

## Overview

This crate provides type-safe Rust bindings for interacting with the Ambient Tool Oracle program deployed on Solana. The Tool Oracle is a decentralized system that enables on-chain requests for LLM-powered tool use, where off-chain workers process requests and submit results back to the blockchain.

## Program Address

```rust
// Program ID: 721QWDeUzVL77UCzCFHsVGCMBVup8GsAMPaD2YvWvw97
use oracle_program_types::ID;
```

## Example App

An example app using the oracle program can be found at the [oracle-crypto-ticker-app](https://github.com/AmbientCrypto/oracle-crypto-ticker-app) repository.

## Features

The Tool Oracle program provides the following instructions:

### Core Instructions

- **create_request**: Create a new oracle request with an initial prompt
- **claim_request**: Workers claim pending requests for processing
- **submit_intermediate_input**: Submit intermediate results and request additional LLM calls
- **submit_success**: Submit the final successful output
- **submit_failure**: Report request failure with a specific error code
- **reclaim_accounts**: Reclaim lamports from completed or failed request accounts

## Data Structures

### ToolOracleRequestAccount

The main account structure representing an oracle request:

```rust
pub struct ToolOracleRequestAccount {
    pub state: ToolOracleRequestState,
    pub initial_prompt: ToolOracleRequestInput,
    pub max_requests: u8,
    pub output_filter: Option<String>,
}
```

### Request States

```rust
pub enum ToolOracleRequestState {
    Requested,  // Initial state, awaiting worker
    Started {   // Being processed by a worker
        worker: Pubkey,
        completed_requests_arr: Vec<CompletedRequest>,
    },
    Completed { // Successfully completed
        completed_requests_arr: Vec<CompletedRequest>,
        output: AccountOrString,
    },
    Failed {    // Failed with error
        reason: FaultCode,
        completed_requests_arr: Vec<CompletedRequest>,
    },
}
```

### Input/Output Types

```rust
pub enum ToolOracleRequestInput {
    Account(Pubkey),     // Large prompts stored in separate account
    Direct(String),      // Small prompts (≤100 chars) stored inline
    Context([u8; 16]),   // Context reference for content-addressable storage
}

pub enum AccountOrString {
    Account(Pubkey),     // Large outputs in separate account
    Direct(String),      // Small outputs (≤100 chars) inline
}
```

## Usage

Add this to your `Cargo.toml`:

```toml
[dependencies]
oracle-program-types = "0.1.0"
anchor-lang = "0.31.1"
```

### Client Integration

For a complete working example of how to create and monitor oracle requests, see the [tool-oracle-example](../ambient/tool-oracle/tool-oracle-example) crate.

Key steps:
1. Create a request with an initial prompt and escrow funds
2. Monitor the request account for state changes
3. Handle completion (success or failure)
4. Reclaim lamports when done

## Account Seeds

The program uses PDA (Program Derived Addresses) with these seeds:

- **Request Account**: `["tool-oracle-request", signer_pubkey]`
- **Output Account**: `["tool-oracle-output", signer_pubkey]` (optional, for large outputs)

## Workflow

1. **Client** creates a request with `create_request`, providing:
   - Initial prompt (inline or in separate account)
   - Maximum number of LLM requests allowed
   - Optional regex filter for output
   - Escrow lamports to pay for processing

2. **Worker** (oracle-service) claims the request with `claim_request`

3. **Worker** processes the request:
   - Makes LLM inference calls
   - Can submit intermediate results with `submit_intermediate_input`
   - Tracks completed requests in the state

4. **Worker** completes with either:
   - `submit_success`: Final output returned
   - `submit_failure`: Error code reported

5. **Client** reclaims lamports with `reclaim_accounts`

## Links

- Homepage: https://ambient.xyz
- Repository: https://github.com/AmbientCrypto/oracle-program-types
