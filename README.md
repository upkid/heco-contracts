# heco-contracts

## Prepare

Install dependency:

```bash
yarn
```

Compile files:

```bash
yarn compile
```

Release solidity files will be generated in `cache` folder.

## unit test
Test:

```bash
yarn test
```
# aitd-sys-contracts

## Prepare

Install dependency:

```bash
npm install
```

## unit test

Generate test contract files:

```bash
node generate-mock-contracts.js
```

Start ganache:

```bash
ganache-cli -e 20000000000 -a 100 -l 8000000 -g 0
```

Test:

```bash
truffle test
```

## 验证节点申请与投票

#### 申请成为出块节点

执行 `proposal` 合约的 `createProposal` 方法
参数

- dst 需要申请添加的地址
- details 附加信息（选填）

对于genesis的预设地址，默认会在系统初始化时，自动添加到审核通过

#### 审核出块申请

选择执行 `proposal` 合约的 `voteProposal`，当前设计，合约签名地址需要为已经通过申请的用户，赞同总数或者拒绝总数，哪个先超过当前正在出块人数的 1/2+1，就执行对应的决定。这个不合理，应该是需要正在出块人中的1/2+1赞同才有效，那些之前通过审核，但不是正在出块的地址，没有审核资格。
参数

- id 对应申请的id {前面申请的id通过 event获取}
- auth 是否通过

#### 取消出块资质

选择 `proposal` 合约的执行`setUnpassed`，目前合约需要当前合约权限才能禁止掉其他人，不合理，后面进行合约修改，同样需要发起申请，通过1/2+1人通过，才会取消对应地址的资格。

#### 创建validator地址

选择合约`validators`执行`createOrEditValidator`，需要先申请，通过后才允许创建
参数

- feeAddr 该矿工领取奖励时的接收地址
- moniker 昵称（选填）
- identity 身份（选填）
- website 网址（选填）
- email 邮箱（选填）
- details 附加信息（选填）

#### 抵押系统代币

选择执行 `stake`，需要创建完validator，才能进行抵押，抵押量不小于32 ether，后面挖矿奖励会根据当前用户抵押量所占总比例进行分配。抵押后，区块链每 200 个块更新一遍活动验证器列表。

#### 矿工主动提交所得，进行分配

链程序出块后，自动执行distributeBlockReward将打包所得手续费转到合约进行池分配（后期排查，节点程序是否可作恶，抽水），代币总量到合约后，合约根据currentValidatorSet中各自地址抵押总额，以及各自占比进行分配记账（validatorInfo[地址].hbIncoming），（对于合约分配逻辑，后面单独写文章讲解）

#### 领取出块奖励

选择执行 withdrawProfits，参数validator为矿工地址，执行签名地址需要与创建时设置的feeAddr一致,领取时间间隔24小时（28800块）

#### 重新申请成为出块节点

如果，有节点因为服务器问题，未及时出块，被合约拒绝后，可以通过重新申请为出块节点，成为验证器。需要执行上面 `申请成为出块节点` `审核出块申请` `抵押系统代币` 这三个步骤。

