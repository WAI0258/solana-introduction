# Decode a Solana Transaction

## 目录

- [1. 使用@solana/web3.js](#1-使用solanaweb3js)
  - [基本 transaction](#基本-transaction)
    - [message](#message)
      - [accountKeys](#accountkeys)
      - [header](#header)
      - [recentBlockhash](#recentblockhash)
      - [instructions](#instructions)
      - [addressTableLookups](#addresstablelookups)
    - [Signatures](#signatures)
  - [meta](#meta)
    - [computeUnitsConsumed](#computeuintsconsumed)
    - [err](#err)
    - [fee](#fee)
    - [innerInstructions](#innerinstructions)
    - [loadedAddresses](#loadedaddresses)
    - [logMessages](#logmessages)
    - [postBalances/preBalances](#postbalancesprebalances)
    - [postTokenBalances/preTokenBalances](#posttokenbalancespretokenbalances)
    - [returnData](#returndata)
- [Reference](#reference)

---

## 1. 使用@solana/web3.js

`getTransaction()` 获取 raw 格式(也可 parse json)，如果使用 `getParsedTransaction()` 会获得更直观的 json 数据

- 单个 Tx 分析建议用 `getParsedTransaction`，其他情况入区块分析、slot 分析用 `getTransaction`（因为 `getBlock` 获取的 Tx 数据是 `getTransaction` 的 raw 格式）

### 基本 transaction

#### message

- **accountKeys** `<array[string]>`：由 `getParsedTransaction()` 获得，==`getTransaction()` 的 `staticAccountKeys` + `meta.loadedAddresses.readonly` + `meta.loadedAddresses.writable`，index 由三个数组顺序拼接

  - Tx 用的 base-58 编码公钥列表
    ```json
    // parsed过后
    {
      "pubkey": "EF3cbGuhJus5mCdGZkVz7GQce7QHbswBhZu6fmK9zkCR",
      "signer": true,
      "source": "transaction",
      "writable": true
    }
    ```
  - 包括指令使用的 accounts 和 singer account
  - 前 `numRequiredSignatures` 个公钥必须签名

- **header** `<object>`

  - 详细说明 accounts 类型和所需签名
  - `numRequiredSignatures`: 使 Tx 有效所需的总签名数
  - `numReadonlySignedAccounts`: 签名密钥中最后几个只读 accounts 的数量
  - `numReadonlyUnsignedAccounts`: 未签名密钥中最后几个只读 accounts 的数量

- **recentBlockhash** `<string>`

  - 账本中最近区块的 base-58 编码哈希
  - 用于防止 Tx 重复并给 Tx 设置生命周期

- **instructions** `<array[object]>`：`getTransaction` 为 `compiledInstructions`

  ```json
  // parsed 过后
  {
    "parsed": {
      "info": {
        "destination": "4LAyP5B5jNyNm7Ar2dG8sNipEiwTMEyCHd1iCHhhXYkY",
        "lamports": 100000000,
        "source": "EF3cbGuhJus5mCdGZkVz7GQce7QHbswBhZu6fmK9zkCR"
      },
      "type": "transfer"
    },
    "program": "system",
    "programId": "11111111111111111111111111111111",
    "stackHeight": null
  }
  ```

  - **programIdIndex** `<number>`：指向 `message.accountKeys` 数组的索引->指示执行此指令的 program account address
  - **accounts** `<array[number]>`：要传递给 program 的 account
  - **data** `<string>`：以 base-58 字符串编码的 program 输入数据

- **addressTableLookups** `<array[object]|undefined>`

  - Tx 从链上 address lookup tables 动态加载地址的 address lookup tables
  - 如果未设置 `maxSupportedTransactionVersion` 则为 undefined
  - **accountKey** `<string>`: 存 address lookup tables 的 account base-58 编码公钥
  - **writableIndexes** `<array[number]>`: 从 lookup table 获取可写 accounts 的 index
  - **readonlyIndexes** `<array[number]>`: 从 lookup table 获取可读 accounts 的 index

#### Signatures

- Tx 的 base-58 编码签名列表
- 长度始终等于 `message.header.numRequiredSignatures` 且不为空
- 索引 i 处的签名对应于 `message.accountKeys` 中索引 i 处的公钥
- 第一个签名用作交易 ID==`message.accountKeys[0]` 为 singer

### meta

#### computeUnitsConsumed

#### err

如果 != null，代表错误发生，Tx 一定失败

#### fee

#### innerInstructions

Tx 指令的内层指令，一般分析行为都由此分析

- **index**：该内层指令集的父指令在 `transaction.message.instructions` 或 `transaction.message.compiledInstructions` 的指标
- **instructions**：详细内层指令

  ```json
  {
    "accounts": [0, 19, 1, 20, 13, 14, 2, 10, 23, 11, 3],
    "data": "wZRp7wZ3czsd7UQsaAKiU9ybTWR9yCVRrrskcdrNrrxdeHZfVthXxbXu",
    "programIdIndex": 26,
    "stackHeight": 2
  }
  ```

  - **accounts**：program 需要的 accounts 的 index，具体 account 信息在 `transaction.message.accountKeys` 或 `transaction.message.staticAccountKeys` + `meta.loadedAddresses.readonly` + `meta.loadedAddresses.writable`
  - **data**：base-58 数据，可以根据对应 program 的 IDL 判断具体指令类型
  - **programIdIndex**: program 的地址 index，也是放在 accounts 的具体 account 信息里的
  - **stackHeight**：指令调用栈高度

#### loadedAddresses

从 on-chain look up table 加载下来的 accounts

- **readonly**
- **writable**

#### logMessages

program 定义的日志内容，decode 参考性较低

#### postBalances/preBalances

交易后/前 token accounts 的余额变化(lamports 单位)

- 数组大小 == 交易相关 accounts 大小，数组下标 index 对应于 accounts

#### postTokenBalances/preTokenBalances

交易后/前余额变化的 token accounts

- **token accounts**：有时候会是 ATA，如果 program 新建了 ATA，preTokenBalances 不会出现，因为

  ```json
  {
    "accountIndex": 2,
    "mint": "ts3foLrNUMvwdVeit1oNeLWjYk7e4qsn8PqSsqRpump",
    "owner": "2BDKfBgjC1sokR97YHtexEAx4SuCe1kuvNTDWceEArbb",
    "programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
    "uiTokenAmount": {
      "amount": "55621076544",
      "decimals": 6,
      "uiAmount": 55621.076544,
      "uiAmountString": "55621.076544"
    }
  }
  ```

  - **accountIndex**: accountKeys 的数组的 index(该 token account 的 address)
  - **mint**：account 拥有的 token 的 address
  - **owner**: 该 token account 的拥有者
  - **programId**：token 的版本
  - **uiTokenAmount**：人类可读(ui 显示)的 token 余额

#### returnData

Tx 某个指令调用返回的数据

一般为情况：

- 系统调用 SBF 程序的返回值，供 caller 或 client 读取
- CPI 时，callee 会自动传给 caller

内容：

- **programId**：产生该 data 的 program
- **data**：[内容字符串, 加密方式]

## Reference

https://solana.com/zh/docs/rpc/json-structures
