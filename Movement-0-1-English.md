## Basic Concepts of Movement

The Movement network is the first Ethereum Layer 2 (L2) that integrates Celestia's data availability, a decentralized shared sequencer, optimistic rollup, and dual finality options through fast finality settlement. It utilizes the Move Virtual Machine (Move VM) for execution, enabling high transaction throughput.

**Project Advantages**:

- **Move Executor**: Supports both Move VM and EVM transactions, allowing Web3 developers to deploy smart contracts in Move and EVM bytecode on a single network. This enables rapid developer adaptation, extends new functionalities to existing EVM applications, and leverages Move's advanced features.
- **Fast Finality Settlement Module**: Relies on a set of validators staking native tokens to confirm the correctness of new states through majority consensus (e.g., 2/3 of total staked tokens), providing rapid finality while enhancing the utility of native tokens.

In summary, Movement inherits the technical strengths of Move-based chains like Aptos while bridging the Ethereum ecosystem.

---

## Movement Development Basics

### Environment Setup

**Compiling Movement CLI**  
Reference: [Movement CLI Documentation](https://docs.movementnetwork.xyz/devs/movementcli)

```sh
git clone https://github.com/movementlabsxyz/aptos-core/ && cd aptos-core

cargo build -p movement (generates the target/debug/movement executable)

# Copy movement into your bin or add it to your PATH depending on your system config.
sudo cp target/debug/movement /usr/local/bin/
```

If you prefer not to compile, you can directly use the `aptos` CLI from the original Aptos Core repository.

---

### Movement CLI Commands

**Initialize a Profile**  

```sh
movement init --network custom --rest-url https://aptos.testnet.porto.movementlabs.xyz/v1 --profile default # Requires Faucet endpoint

# movement init --profile testnet --skip-faucet

movement account fund-with-faucet --account default

movement account list --query balance --account default
movement account list --query resources --account default
movement account list --query modules
```

You can inspect the generated account configuration files.

**Network Endpoints**:  
Refer to [Network Configuration](https://docs.movementnetwork.xyz/devs/networkEndpoints).

**Chain ID: 177**

| Service          | URL                                                          |
| ---------------- | ------------------------------------------------------------ |
| RPC              | https://aptos.testnet.porto.movementlabs.xyz/v1              |
| Faucet endpoint  | https://fund.testnet.porto.movementlabs.xyz/                 |
| Explorer         | https://explorer.movementnetwork.xyz/                        |
| Faucet UI        | https://mizu.testnet.porto.movementnetwork.xyz/              |
| Indexer Explorer | [Public Explorer](https://cloud.hasura.io/public/graphiql?endpoint=https://indexer.testnet.porto.movementnetwork.xyz/v1/graphql) |
| Indexer Endpoint | https://indexer.testnet.porto.movementnetwork.xyz/v1/graphql |

---

### First Move Smart Contract

```rust
module hello_blockchain::message {
    use std::error;
    use std::signer;
    use std::string;
    use aptos_framework::event;

    struct MessageHolder has key {
        message: string::String,
    }

     #[event]
    struct MessageChange has drop, store {
        account: address,
        from_message: string::String,
        to_message: string::String,
    }

    const ENO_MESSAGE: u64 = 0;

    #[view]
    public fun get_message(addr: address): string::String acquires MessageHolder {
        assert!(exists<MessageHolder>(addr), error::not_found(ENO_MESSAGE);
        borrow_global<MessageHolder>(addr).message
    }

    // Sets or updates the message for the account
    public entry fun set_message(account: signer, message: string::String)
    acquires MessageHolder {
        let account_addr = signer::address_of(&account);
        // The first check prevents overwriting or mismanaging resources
        if (!exists<MessageHolder>(account_addr)) {
            move_to(&account, MessageHolder {
                message,
            })
        } else {
            let old_message_holder = borrow_global_mut<MessageHolder>(account_addr);
            let from_message = old_message_holder.message;
            event::emit(MessageChange {
                account: account_addr,
                from_message,
                to_message: copy message,
            });
            old_message_holder.message = message;
        }
    }

    #[test(account = @0x1)]
    public entry fun sender_can_set_message(account: signer) acquires MessageHolder {
        let addr = signer::address_of(&account);
        aptos_framework::account::create_account_for_test(addr);
        set_message(account, string::utf8(b"Hello, Blockchain"));

        assert!(
            get_message(addr) == string::utf8(b"Hello, Blockchain"),
            ENO_MESSAGE
        );
    }
}
```

**Key Notes**:  

- `MessageHolder` is a resource type requiring the `key` capability, which enables key-value storage.  
- `MessageChange` is an event type storing event-related data.  
- `get_message` is a read-only function marked with `#[view]` and uses `acquires` to reference the `MessageHolder` resource.  
- `set_message` modifies data and uses `entry` for external invocation.  

---

### Deploying a Movement Smart Contract

1. **Compilation**  
   Initialize a default account:  

```sh
movement init --network custom --rest-url https://aptos.testnet.porto.movementlabs.xyz/v1 --profile default
```

Compile the contract:  

```sh
movement move compile --named-addresses hello_blockchain=default
```

2. **Testing**  

```sh
movement move test --named-addresses hello_blockchain=default
```

3. **Deployment**  

```sh
movement move publish --named-addresses hello_blockchain=default
```

4. **Interacting with Functions**  
   **Update Data**:  

```sh
movement move run --function-id 8773e781471ba6f5f30ecafa471eb1363a9bdae33d2a9d53697f0bb6683647d2::message::set_message --args string:Hello! --profile default
```

**Read Data**:  

```sh
movement move view --function-id 8773e781471ba6f5f30ecafa471eb1363a9bdae33d2a9d53697f0bb6683647d2::message::get_message --args address:8773e781471ba6f5f30ecafa471eb1363a9bdae33d2a9d53697f0bb6683647d2 --profile default
```

---

## Scaffold-Move

**GitHub**: [Scaffold-Move](https://github.com/NonceGeek/scaffold-move)

### **What is Scaffold-Move?**  

A modern, clean toolkit for building decentralized applications (dApps) on Move-based chains, built with NextJS, Supabase, and Deno. Future plans include integrating AI capabilities for automated code generation.

### **Quick Start Guide**  

1. Clone the repository:  

```sh
git clone https://github.com/NonceGeek/scaffold-move.git
```

2. Install dependencies:  

```sh
cd scaffold-move && yarn
```

3. Configure environment variables in `.env.local`.  
4. Start the development server:  

```sh
yarn dev
```

5. Build the project:  

```sh
yarn build
```

6. Deploy via Vercel:  

```sh
yarn vercel --prod
```

