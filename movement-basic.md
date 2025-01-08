# movement开发基础


## Movement环境安装

编译movement程序

https://docs.movementnetwork.xyz/devs/movementcli

```sh
git clone https://github.com/movementlabsxyz/aptos-core/ && cd aptos-core

cargo build -p movement (generates the target/debug/movement executable)

#Copy movement into your bin or add it to your PATH depending on your system config.
sudo cp target/debug/movement /usr/local/bin/
```



## Movement cli命令使用

movement cli命令行相关操作

```sh
movement init --network custom --rest-url https://aptos.testnet.porto.movementlabs.xyz/v1 --profile default

movement init --profile testnet --skip-faucet

movement account fund-with-faucet --account default

movement account list --query balance --account default
movement account list --query resources --account default
movement account list --query modules
```

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



## Movement 智能合约部署

1. 编译

初始化一个default账户
```sh
movement init --network custom --rest-url https://aptos.testnet.porto.movementlabs.xyz/v1 --profile default
```
编译
```sh
movement move compile --named-addresses hello_blockchain=default
```

2. 测试

```sh
movement move test --named-addresses hello_blockchain=default
```

3. 部署

```sh
movement move publish --named-addresses hello_blockchain=default
```

4. 调用

修改数据
```sh
movement move  run --function-id aa1ce323e0c3ecdd3a56b635004a137b3ebfaa7cc13def58f6a39d49ee4765be::message::set_message --args string:Hello! --named-addresses hello_blockchain=default
```

读取数据
```sh
movement move  view --function-id aa1ce323e0c3ecdd3a56b635004a137b3ebfaa7cc13def58f6a39d49ee4765be::message::get_message --args address:0xaa1ce323e0c3ecdd3a56b635004a137b3ebfaa7cc13def58f6a39d49ee4765be --profile default
```