# movement开发基础

编译movement程序

https://docs.movementnetwork.xyz/devs/movementcli

```sh
git clone https://github.com/movementlabsxyz/aptos-core/ && cd aptos-core

cargo build -p movement (generates the target/debug/movement executable)

#Copy movement into your bin or add it to your PATH depending on your system config.
sudo cp target/debug/movement /usr/local/bin/
```



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