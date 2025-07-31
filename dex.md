### 借贷平台中渐渐

### Pump Fun

### Lifinity

- 预言机定价+集中流动性+惰性流动性+pools 的 delayed 再平衡

  - 自研预言机：只有预言机在当前 slot 成功更新才会进行 tx(自以为置信区间很短，所以防止抢先交易)
  - 集中流动性：简单 x⋅y=c⋅k=K，c 为所需集中度
    - 流动性不由外部提供(ban LP)，只有 LFNTY 支持
    - 用 POL
  - 再平衡：
    - increase liquidity forx :$K_{adjusted} = K \cdot (x \cdot p / y)^z$
    - decrease liquidity forx :$K_{adjusted} = K \cdot (y/(x \cdot p))^z$
      - p 是预言机提供的 x 价格，z 是调整幅度的遍量
      - 例子：x 占 pool 的 50%以下时，减少 x 买家的 K，增加 x 卖家的 K->回归 bonding curve

- MMaaS：自动做市商
- LaaS：POL 的流动性作为服务，为项目租赁流动性
- v2: 不按照传统 amm 的 50/50->固定基准资产（如 USDC）的数量，只有价格变动超过预设才平衡为 50/50
- veIDO：不直接卖 LFNTY，而是用 USDC 买投票托管代币 veLFNTY，按需求锁仓
  - Weight = D / (1 - (1/2) \* (L/1460))
    - D 是投入 USDC 金额
    - L 锁仓天数，最长 4 年（1460 天）
    - 1/2 是最大折扣。也就是锁了 1460 天给 5 折
