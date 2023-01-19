# (PaperReading) Fraud and Data Availability Proofs: Detecting Invalid Blocks in Light Clients
[toc]
## Definitions
- Shares/DataRoot: 
It is a fixed size of transaction data chunk in bytes known as “shares”. The share will not contain all the transactions rather a fixed part of transactions. We reserve the first byte in each share to be the starting position of the first transaction. This allows the protocol message parser to establish the message boundaries without requiring every transaction in the block.

- Messages: 
They are either transactions or intermediate state roots output by function parseShares.
    - parseShares
        - Input: 
        Given a list of shares (sh0, sh1, . . . shn) we define a function parseShares which parses these shares
        - Output:
        A list of messages (m0, m1, . . .mt).

    e.g. parseShares on some shares in the middle of some block i may return (trace1i, t4i , t5i , t6i ,trace2i).

- Period:
We can not include state roots after every transaction so we can define a period, for example after p transactions of g gas we can include an intermediate state root within a block.

    - parsePeriod
    The function parses a list of messages and returns a pre-state intermediate roots tracexi and post-state intermediate root tracex+1i and a list of transaction (tig, tig+1, . . .tig+h) such that when we apply these transactions on tracexi , it must give us tracex+1i.

- Merkle Proof:
Path from leaf node to root of merkle tree is for verifying data stored in leaf node is included in the tree.

- State Witness:
Merkle Proofs with same state root for states which is used in transactions.

## How to verify the invalid state transition?
- If a sequecer provided us with an incorrect stateRoot.
- VerifyTransitionFraudProof
This function is for checking the invalidity of the stateRoot. It takes the fraud proof submitted by full nodes and verifies it.
- Fraud Proof
    - Relevant shares in the block that contain a bad state transition.
    - Merkle Proofs for those shares.
    - State witness for the transactions in the shares.

## Why we need to use erasure code? 
### 问题的引出：轻节点如何检查区块数据可用性？
- 在区块交易数据可获取的情况下：通过欺诈证明，诚实的full node可以向light node提供状态转移不合法的证明。
- 在区块交易数据不可获取的情况下：full node由于对数据可用性存疑，会拒绝信任该新出的区块；
    - 但full node无法提供数据不可用证明，因为这是一个不可归责的问题。
        - 情况一：数据有可能是因为网络条件不好而丢失。
        - 情况二：如果一个节点发布了数据不可用的警报，但是检查的时候发现数据齐全，那么这即可能是出块者故意不上传数据，然后发现警报后又补全了数据，但也可能是警报者发布错误警报。
- 因为轻节点同步区块需要两方面的安全保障
    - 1. 欺诈证明有效性的必要前提：确定区块的数据可用性
    - 2. 欺诈证明：确定数据可用后，通过欺诈证明机制保证诚实的full node能向轻节点提供同级别的安全保障
### 问题的解决：erasure code
- 需要保证轻节点无需下载所有区块数据进行验证，否则就和full node没有区别；
- 随机下载数据，检查数据可用性：
    - 如果小于50%数据可用，则随机下载100个数据块，发现数据不可用的概率为接近99.9%(1-1/2^100)。
    - 实际情况与上述的区别在于，10Mb数据中只要有100bytes缺失就足以掩护作恶交易，此时数据可用比例在99.999%(1-1/10^5)。此时，如何保证在检查数据量小的同时验证数据可用性？
- 实现方式
    - 数据扩展：将数据以点集的形式构造多项式，并对多项式取额外的点进行扩展。
    - 数据复原：对于n阶多项式，只需请求任意n个不同点即可算出原多项式，并求出原数据对应的点集。
    - 数据分片：由于不需要请求所有数据，只需要从不同分片请求足够复原原数据的点集即可。同时，数据一致性通过merkle tree proof保证。

## Problems
- 网络最大延迟时间？

## Reference
- https://www.ethereum.cn/Eth2/data-availability-checks
