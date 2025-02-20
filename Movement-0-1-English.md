## Movement基础概念

Movement 网络是首个集成 Celestia 数据可用性、去中心化共享排序器、乐观 rollup，以及通过快速最终性结算实现双重最终性选项的以太坊二层网络（L2）。它采用 Move 虚拟机（Move VM）进行执行，能提供较高的交易吞吐量。

**项目优势**：

- **Move 执行器**：支持 Move VM 和 EVM 交易，允许 Web3 开发者在单一网络上以 Move 和 EVM 字节码部署智能合约，能让开发者快速适配，并可在现有 EVM 应用上扩展新功能并利用 Move 平台的先进特性。
- **快速最终性结算模块**：依赖一组质押原生代币的验证者，通过多数共识（如 2/3 质押总量）来确认新状态的正确性，可提供快速最终性，同时提升原生代币的实用性。

简单来说，Movement可以继承Move系公链如Aptos的技术优势，又能连通以太坊网络。



## Movement开发基础


### Movement环境安装

编译movement程序

https://docs.movementnetwork.xyz/devs/movementcli

```sh
git clone https://github.com/movementlabsxyz/aptos-core/ && cd aptos-core

cargo build -p movement (generates the target/debug/movement executable)

#Copy movement into your bin or add it to your PATH depending on your system config.
sudo cp target/debug/movement /usr/local/bin/
```

如果不愿意编译，实际上也可以直接使用aptos-core原工程对应的aptos（cli文件）。

### Movement cli命令使用

movement cli命令行相关操作

```sh
movement init --network custom --rest-url https://aptos.testnet.porto.movementlabs.xyz/v1 --profile default # need given Faucet endpoint

# movement init --profile testnet --skip-faucet

movement account fund-with-faucet --account default

movement account list --query balance --account default
movement account list --query resources --account default
movement account list --query modules
```

我们可以查看一下movement生成的账户文件配置。

movement相关网络配置：

https://docs.movementnetwork.xyz/devs/networkEndpoints

**Chain ID: 177**

| Service          | URL                                                          |
| ---------------- | ------------------------------------------------------------ |
| RPC              | https://aptos.testnet.porto.movementlabs.xyz/v1              |
| Faucet endpoint  | https://fund.testnet.porto.movementlabs.xyz/                 |
| Explorer         | https://explorer.movementnetwork.xyz/                        |
| Faucet UI        | https://mizu.testnet.porto.movementnetwork.xyz/              |
| Indexer Explorer | [Public Explorer](https://cloud.hasura.io/public/graphiql?endpoint=https://indexer.testnet.porto.movementnetwork.xyz/v1/graphql) |
| Indexer Endpoint | https://indexer.testnet.porto.movementnetwork.xyz/v1/graphql |



### 第一个Move智能合约

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
        assert!(exists<MessageHolder>(addr), error::not_found(ENO_MESSAGE));
        borrow_global<MessageHolder>(addr).message
    }

    // sets or updates the message for the account
    public entry fun set_message(account: signer, message: string::String)
    acquires MessageHolder {
        let account_addr = signer::address_of(&account);
        // the first check prevents overwriting or miss-managing resources
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



MessageHolder 是一个结构类型，它也是定义资源数据的类型，资源类型的关键是要声明具有key这种能力。key的能力代表可以通过键值存储数据。MessageChange是一个事件类型，存储事件相关的信息。

get_message 是一个只读函数，因此要使用view标识。它的返回值类型是string，它要读取资源中的数据，所以它引用了MessageHolder这个资源结构，其中要使用关键字acquires。

set_message是一个修改数据的函数，它定义时前面声明了entry，在修改资源时，它同样需要先借用资源，因此也涉及了资源的引用。

sender_can_set_message是一个测试用的函数。



### Movement 智能合约部署

1. 编译

初始化一个default账户
```sh
movement init --network custom --rest-url https://aptos.testnet.porto.movementlabs.xyz/v1 --profile default
```
因为aptos和movement不是同一个网络，所以这里要指定movement专有的测试网络。

编译

```sh
movement move compile --named-addresses hello_blockchain=default
```

2. 测试合约

```sh
movement move test --named-addresses hello_blockchain=default
```

3. 部署合约

```sh
movement move publish --named-addresses hello_blockchain=default
```

4. 调用函数

修改数据
```sh
movement move  run --function-id 8773e781471ba6f5f30ecafa471eb1363a9bdae33d2a9d53697f0bb6683647d2::message::set_message --args string:Hello! --profile default
```

读取数据
```sh
movement move  view --function-id 8773e781471ba6f5f30ecafa471eb1363a9bdae33d2a9d53697f0bb6683647d2::message::get_message --args address:8773e781471ba6f5f30ecafa471eb1363a9bdae33d2a9d53697f0bb6683647d2 --profile default
```



## Scaffold-Move

https://github.com/NonceGeek/scaffold-move

### **What is Scaffold-Move？**

A modern, clean version of Scaffold-Move with NextJS, Supabase & Deno, strong-AI-powered in the future.

An open-source, up-to-date toolkit for building decentralized applications (dapps) on the Move Chains.

It's designed to make it easier for developers to create and deploy smart contracts and build user interfaces that interact with those contracts. And...We are going to add AI Abilities for Move dApp Scaffold, to generate code automatically.

### Study Guide 

1. `git clone https://github.com/rootMUD/scaffold-move.git`
2. `cd scaffold-move`
3. `yarn # Install the necessary front-end packages, pay attention to your local network environment`
4. Environment configuration, some global variables are in .env.local, which will be injected into the process started by yarn by default. Attention beginners, the testnet faucet url provided by aptos official website cannot be used directly.
5. `yarn dev` 
6. `yarn build #compiled next.js application`
7. A Quick way to deploy: `yarn vercel --prod`

