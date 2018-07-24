# 环境安装

```
➜  ~ gaiad version 
0.17.2-187be1a5
➜  ~ 

```

## 创建创世文件

```
➜  ~ gaiad init gen-tx --name=gcoin --home=/Users/wen/gopath/src/github.com/cosmos/cosmos-sdk/testnet/gcoin
➜  ~ gaiad init gen-tx --name=hcoin --home=/Users/wen/gopath/src/github.com/cosmos/cosmos-sdk/testnet/hcoin
{
  "app_message": {
    "secret": "require clean energy unlock prefer fix volume hurdle hope belt code fantasy insane lounge attitude abandon"
  },
  "gen_tx_file": {
    "node_id": "61bb02ef2ef32cc35fb22031d073357bc18e0bca",
    "ip": "192.168.3.44",
    "validator": {
      "pub_key": {
        "type": "AC26791624DE60",
        "value": "ZyfRMfpcV+MeE3RvTp0lI7a/zObL83UU18MGg9ChNcA="
      },
      "power": 100,
      "name": ""
    },
    "app_gen_tx": {
      "name": "gcoin",
      "address": "B7661BE212D6126D83A6F907CE6042E1C4883361",
      "pub_key": {
        "type": "AC26791624DE60",
        "value": "ZyfRMfpcV+MeE3RvTp0lI7a/zObL83UU18MGg9ChNcA="
      }
    }
  }
}
{
  "app_message": {
    "secret": "depth wait pond salute mule artwork point speak ride change emerge art robust erode arena abandon"
  },
  "gen_tx_file": {
    "node_id": "086c4931dbab706d1188068bbb7386ecc6c284a9",
    "ip": "192.168.3.44",
    "validator": {
      "pub_key": {
        "type": "AC26791624DE60",
        "value": "fTS2i7mV6JcmNzpUkETI0aVCCpeqrJtl7JRKy+uYo/8="
      },
      "power": 100,
      "name": ""
    },
    "app_gen_tx": {
      "name": "hcoin",
      "address": "38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA",
      "pub_key": {
        "type": "AC26791624DE60",
        "value": "fTS2i7mV6JcmNzpUkETI0aVCCpeqrJtl7JRKy+uYo/8="
      }
    }
  }
}
```


## 初始化创世交易

```
➜  ~ gaiad init --gen-txs --home=/Users/wen/gopath/src/github.com/cosmos/cosmos-sdk/testnet/gcoin --chain-id=ghcoin
➜  ~ gaiad init --gen-txs --home=/Users/wen/gopath/src/github.com/cosmos/cosmos-sdk/testnet/hcoin --chain-id=ghcoin

{
  "chain_id": "ghcoin",
  "node_id": "61bb02ef2ef32cc35fb22031d073357bc18e0bca",
  "app_message": null
}
{
  "chain_id": "ghcoin",
  "node_id": "086c4931dbab706d1188068bbb7386ecc6c284a9",
  "app_message": null
}

```

## 查询账户信息

```
➜  ~ gaiacli account B7661BE212D6126D83A6F907CE6042E1C4883361
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "B7661BE212D6126D83A6F907CE6042E1C4883361",
    "coins": [
      {
        "denom": "gcoinToken",
        "amount": 1000 //链G上的用户g拥有1000个gcoinToken
      },
      {
        "denom": "steak",
        "amount": 50
      }
    ],
    "public_key": null,
    "sequence": 0
  }
}

➜  ~ gaiacli account 38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA",
    "coins": [
      {
        "denom": "hcoinToken",
        "amount": 1000//链H上的用户h拥有1000个hcoinToken
      },
      {
        "denom": "steak",
        "amount": 50
      }
    ],
    "public_key": null,
    "sequence": 0
  }
}


```

## 账号信息

* 链H上用户h的地址:  
`38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA`
* 链G上用户g的地址：  
 `B7661BE212D6126D83A6F907CE6042E1C4883361`

## 转账

* 链G上的用户g向链H上的用户h转账666gcoinToken
```

//第一次转账，向链H的用户h转账333个gcoinToken
➜  ~ gaiacli send --amount=333gcoinToken --to=38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA  --name=gcoin --chain-id=ghcoin
Defaulting to next sequence number: 0
Password to sign with 'gcoin':
Committed at block 478. Hash: 2747A3B72DA985CC4A70D640E9557F72B805F66C

//第二次转账,向链H的用户h转账333个gcoinToken
➜  ~ gaiacli send --amount=333gcoinToken --to=38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA  --name=gcoin --chain-id=ghcoin
Defaulting to next sequence number: 1
Password to sign with 'gcoin':
Committed at block 483. Hash: BF119966B7953D1FFE7F848DA13ABBA8A663E398
➜ 

```

* 链H上的用户h向链G上的用户g转账666hcoinToken

```

//第一次转账,向链G上的用户g转账333个ghcoinToken
➜  ~ gaiacli send --amount=333hcoinToken --to=B7661BE212D6126D83A6F907CE6042E1C4883361  --name=hcoin --chain-id=ghcoin
Defaulting to next sequence number: 0
Password to sign with 'hcoin':
Committed at block 520. Hash: B1D6D2C555530C16FEDBC8A85D527FB2A9A1218E

//第二次转账,向链G上的用户g转账333个ghcoinToken
➜  ~ gaiacli send --amount=333hcoinToken --to=B7661BE212D6126D83A6F907CE6042E1C4883361  --name=hcoin --chain-id=ghcoin
Defaulting to next sequence number: 1
Password to sign with 'hcoin':
Committed at block 526. Hash: AECE23DCC13BD1F777642DB6751F8416D38AB302
➜
```
## 再次查询余额
* 链H的上用户h的余额：
```
➜  ~ gaiacli account 38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "38F5CDA4903EC8ECA6DC124C2FC1FD5A460E0EBA",
    "coins": [
      {
        "denom": "gcoinToken",
        "amount": 666     //成功收到链G的用户g转的666个gcoinToken
      },
      {
        "denom": "hcoinToken",
        "amount": 334  //成功向链H的用户h转账666个hcoinToken，剩余334个hcoinToken
      },
      {
        "denom": "steak",
        "amount": 50
      }
    ],
    "public_key": {
      "type": "AC26791624DE60",
      "value": "xXfv2YVy6Ck4RkS/41WSL9E+SsHKG0wBRXeGDTL5DyE="
    },
    "sequence": 2
  }
}
```
* 链G上用户g的地址：

```
➜  ~ gaiacli account B7661BE212D6126D83A6F907CE6042E1C4883361
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "B7661BE212D6126D83A6F907CE6042E1C4883361",
    "coins": [
      {
        "denom": "gcoinToken",
        "amount": 334 //成功向链H的用户h转出666个gcoinToken,还剩余334个gcointoken
      },
      {
        "denom": "hcoinToken",
        "amount": 666 //收到链H的用户h转的666个hcoinToken 
      },
      {
        "denom": "steak",
        "amount": 50
      }
    ],
    "public_key": {
      "type": "AC26791624DE60",
      "value": "rszYSp0rKRMJ5VFgCJ80suEgtv+XJ6S8hcMqNquZgIQ="
    },
    "sequence": 2
  }
}

```
