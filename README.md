# Name Service

The goal of the application you are building is to let users buy names and to set a value these names resolve to.
The owner of a given name will be the current highest bidder. In this section, you will learn how these simple
 requirements translate to application design.

Here is the tutorial for this application: [tutorial](https://docs.cosmwasm.com/tutorials/name-service/intro)

## 合约部署及交互演示
### 账户管理
```
# 创建
xiond keys add your_account

# 查询余额
xiond query bank balances your_account --node https://rpc.xion-testnet-1.burnt.com:443
```

### 合约编译
```
cargo wasm 

# 优化合约大小
RUSTFLAGS='-C link-arg=-s' cargo wasm
```

### 部署上链
```
# 设置环境变量
RPC="https://rpc.xion-testnet-1.burnt.com:443"
export NODE=(--node $RPC)
export TXFLAG=($NODE --chain-id xion-testnet-1 --gas-prices 0.25uxion --gas auto --gas-adjustment 1.4)

# 上传
xiond tx wasm store target/wasm32-unknown-unknown/release/cw_nameservice.wasm --from dev1 $TXFLAG -y --output json

# 查询 code_id
RESP=$(xiond q tx F8FCBA1E1864A5F97F10BAC9452456C0B27B7645635C87D9CFA70BA89165C955 -o json $NODE)

CODE_ID=$(echo "$RESP" | jq -r '.events[]| select(.type=="store_code").attributes[]| select(.key=="code_id").value')

```

### 实例化合约
```
# 参数
INIT='{"purchase_price":{"amount":"100","denom":"uxion"},"transfer_price":{"amount":"999","denom":"uxion"}}'

# 实例化
xiond tx wasm instantiate $CODE_ID "$INIT" --from dev1 --label "name service" $TXFLAG -y --no-admin

# 查询合约地址
xiond query wasm list-contract-by-code $CODE_ID $NODE --output json

```

### 合约读写
```
# 参数
REGISTER='{"register":{"name":"fred"}}'
CONTRACT='xion14ftvs9elg200hecs9jqgu9fyx85lkrdx7nj5mwkdefffrcjkz4tql8jxxd'

# 注册
xiond tx wasm execute $CONTRACT "$REGISTER" --amount 100uxion --from dev1 $TXFLAG -y

# 参数
NAME_QUERY='{"resolve_record": {"name": "fred"}}'

# 查询
xiond query wasm contract-state smart $CONTRACT "$NAME_QUERY" $NODE --output json
```