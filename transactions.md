## 交易 \| Transactions

交易让用户可以花费聪（satoshis）。每一笔交易是由多个部分构成的，使单纯直接的支付和复杂的交易成为可能。这一章节将对其每一部分做详解并展示如果将它们拼接起来构建一个完成的交易。

为了便于理解，这一节假设不存在 coinbase 交易。只有比特币矿工可以创建 coinbase 交易，它们对于下面会提到的很多规则都将是例外。这里并不会指出 coinbase 对哪一条规则是例外，你可以通过阅读本指南中区块链章节中的 coinbase 交易小节来做了解。

![](./en-tx-overview.svg)

上图展示了一个比特币交易的主要部分。每一个交易至少有一个输入和一个输出。每一个输入会花费掉上一个输出的聪。一个输出会作为一个未花费交易输出（UTXO）等待着作为输入被花费掉。当你的比特币钱包显示你有 10,000 聪余额时，它真实的表意是你有 10,000 聪作为 UTXOs 在等待着。

每一个交易具有一个四字节的交易版本号，它告知比特币节点和矿工应使用哪一套规则来验证它。这使得开发者在为未来的交易创建新规则时可以不验证之前的交易。

![](./en-tx-overview-spending.svg)

每一个输出都有一个默认的索引数值，由其在交易中的位置决定，第一个输出就是输出零。输出包含以聪为计数单位的金额，通过满足一个特定条件的 pubkey 脚本来支付。任何可以满足 pubkey 脚本特定条件的人都可以花费对应金额的聪。

输入通过使用上一笔交易的交易 id（txid）和输出索引数值（常被称为输出向量的"vout"）来辨识哪一个输出被花费。它同时包含一个签名脚本，这个脚本提供数据参数来满足 pubkey 脚本中设置的条件。（序列数值 Sequence Number和锁定时间 Locktime是关联的，将会在后续小节中被一起涵盖。）

下图展示了使用这些特性的一个流程，Alice 给你 Bob 发送了一个交易，使得 Bob 之后可以花费这个交易。Alice 和 Bob 使用了支付给公开哈希（P2PKH）这种常用的交易类型。P2PKH 让 Alice 可以发送 satoshis 到一个比特币地址，然后让 Bob 之后可以花费这些 satoshis，其实使用了了一个简单的密码密钥对。

![](./en-creating-p2pkh-output.svg)

Bob 必须在 Alice 能够发给他交易前就生成一对私有/公开密钥对。比特币使用椭圆曲线数字签名算法（ECDSA），选用的是 secp256k1 曲线，secp256k1 的私钥是 256 位的随机数据。这份数据的副本可以被准确的转化为 secp256k1 的公钥。因为转化可以在之后被准确的进行，公钥是没有存储的必要性的。

然后公钥（pubkey）进行密码哈希。这个公钥哈希之后也可以准确的算出，所以它也是不需要被存储的。这个哈希缩短并混淆了公钥，使得手动的交易更容易并提供了安全性，用以抵抗一些未曾预料的问题，比如在未来可能出现通过公钥重新构建出私钥。

Bob 提供给 Alice 公钥哈希。公钥哈希通过编码后就得到了比特币地址，使用 base58 编码得到的字符串包含了一个地址版本数字、哈希值和一个错误校验的校验和用来捕获错别字。比特币地址可以通过任何媒介传播，包括单向媒介从而省去了消费者与接收者间的沟通，它可以之后编码为任何其他格式，比如一个包含了 `bitcoin:` URI 的二维码。

Once Alice has the [address](https://bitcoin.org/en/glossary/address) and decodes it back into a standard hash, she can create the first transaction. She creates a standard P2PKH transaction [output](https://bitcoin.org/en/glossary/output) containing instructions which allow anyone to spend that [output](https://bitcoin.org/en/glossary/output) if they can prove they control the [private key](https://bitcoin.org/en/glossary/private-key) corresponding to Bob’s hashed [public key](https://bitcoin.org/en/glossary/public-key). These instructions are called the [pubkey script](https://bitcoin.org/en/glossary/pubkey-script) or [scriptPubKey](https://bitcoin.org/en/glossary/pubkey-script).

Alice得到了地址并将地址反编译成标准的hash，她便可以创建第一笔交易了。她创建了一笔标准的P2PKH的交易输出，该交易输出包含一些规则要求：允许任何人花费该交易输出，只要他们能够证明他们拥有与Bob公钥相符的私钥。这些规则要求就称为pubkey script或者scriptPubkey。

Alice broadcasts the transaction and it is added to the [block chain](https://bitcoin.org/en/glossary/block-chain). The [network](https://bitcoin.org/en/developer-guide#term-network) categorizes it as an Unspent Transaction [Output](https://bitcoin.org/en/glossary/output)\([UTXO](https://bitcoin.org/en/glossary/unspent-transaction-output)\), and Bob’s [wallet](https://bitcoin.org/en/glossary/wallet) software displays it as a spendable balance.

When, some time later, Bob decides to spend the [UTXO](https://bitcoin.org/en/glossary/unspent-transaction-output), he must create an [input](https://bitcoin.org/en/glossary/input) which references the transaction Alice created by its hash, called a[Transaction Identifier](https://bitcoin.org/en/glossary/txid)\([txid](https://bitcoin.org/en/glossary/txid)\), and the specific [output](https://bitcoin.org/en/glossary/output) she used by its index number \([output index](https://bitcoin.org/en/developer-guide#term-output-index)\). He must then create a [signature script](https://bitcoin.org/en/glossary/signature-script)—a collection of data parameters which satisfy the conditions Alice placed in the previous [output’s](https://bitcoin.org/en/glossary/output)[pubkey script](https://bitcoin.org/en/glossary/pubkey-script).[Signature scripts](https://bitcoin.org/en/glossary/signature-script) are also called [scriptSigs](https://bitcoin.org/en/glossary/signature-script).

[Pubkey scripts](https://bitcoin.org/en/glossary/pubkey-script) and [signature scripts](https://bitcoin.org/en/glossary/signature-script) combine [secp256k1](http://www.secg.org/sec2-v2.pdf)[pubkeys](https://bitcoin.org/en/glossary/public-key) and [signatures](https://bitcoin.org/en/glossary/signature) with conditional logic, creating a programmable authorization mechanism.

![](https://bitcoin.org/img/dev/en-unlocking-p2pkh-output.svg)

For a P2PKH-style[output](https://bitcoin.org/en/glossary/output), Bob’s[signature script](https://bitcoin.org/en/glossary/signature-script)will contain the following two pieces of data:

1. His full \(unhashed\)[public key](https://bitcoin.org/en/glossary/public-key), so the[pubkey script](https://bitcoin.org/en/glossary/pubkey-script)can check that it hashes to the same value as the[pubkey hash](https://bitcoin.org/en/glossary/p2pkh-address)provided by Alice.

2. An[secp256k1](http://www.secg.org/sec2-v2.pdf)[signature](https://bitcoin.org/en/glossary/signature)made by using the[ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_DSA)cryptographic formula to combine certain transaction data \(described below\) with Bob’s[private key](https://bitcoin.org/en/glossary/private-key). This lets the[pubkey script](https://bitcoin.org/en/glossary/pubkey-script)verify that Bob owns the[private key](https://bitcoin.org/en/glossary/private-key)which created the[public key](https://bitcoin.org/en/glossary/public-key).

Bob’s[secp256k1 signature](https://bitcoin.org/en/glossary/signature)doesn’t just prove Bob controls his[private key](https://bitcoin.org/en/glossary/private-key); it also makes the non-[signature](https://bitcoin.org/en/glossary/signature)-script parts of his transaction tamper-proof so Bob can safely broadcast them over the[peer-to-peer network](https://bitcoin.org/en/developer-guide#term-network).

![](https://bitcoin.org/img/dev/en-signing-output-to-spend.svg)

As illustrated in the figure above, the data Bob signs includes the[txid](https://bitcoin.org/en/glossary/txid)and[output index](https://bitcoin.org/en/developer-guide#term-output-index)of the previous transaction, the previous[output’s](https://bitcoin.org/en/glossary/output)[pubkey script](https://bitcoin.org/en/glossary/pubkey-script), the[pubkey script](https://bitcoin.org/en/glossary/pubkey-script)Bob creates which will let the next recipient spend this transaction’s[output](https://bitcoin.org/en/glossary/output), and the amount of[satoshis](https://bitcoin.org/en/glossary/denominations)to spend to the next recipient. In essence, the entire transaction is signed except for any[signature scripts](https://bitcoin.org/en/glossary/signature-script), which hold the full[public keys](https://bitcoin.org/en/glossary/public-key)and[secp256k1 signatures](https://bitcoin.org/en/glossary/signature).

After putting his[signature](https://bitcoin.org/en/glossary/signature)and[public key](https://bitcoin.org/en/glossary/public-key)in the[signature script](https://bitcoin.org/en/glossary/signature-script), Bob broadcasts the transaction to Bitcoin[miners](https://bitcoin.org/en/glossary/mining)through the[peer-to-peer network](https://bitcoin.org/en/developer-guide#term-network). Each[peer](https://bitcoin.org/en/glossary/node)and[miner](https://bitcoin.org/en/glossary/mining)independently validates the transaction before broadcasting it further or attempting to include it in a new[block](https://bitcoin.org/en/glossary/block)of transactions.

