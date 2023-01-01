是时候回到最初开始的地方，从零开始，一路回到幼儿园。 还记得你学到的第一件事是什么吗？ 字母表。 一旦你征服了所有的 26 个字母，你就学会了如何阅读。 这里是您成为 Solana 开发人员的旅程开始的地方。 掌握英语这项独特技能使您在开发的道路上驾轻就熟。

是时候重做一次了。 我们将从您的老师停下来的地方开始——从区块链读取数据。

#### 👜 Solana 账户
从 Solana Alphabet 开始，我们用 A 代表账户。 为什么从账户开始？ 因为 Solana 上的智能合约被称为“程序”，是无状态的——这意味着它们除了代码之外不存储任何东西。 所有一切都发生在账户中，因此它们是 Solana 的核心，它们用于存储、合同和本地区块链程序。

Solana 上有三种类型的账户——
1. 数据账户——这些存储数据 哈哈
2. 程序账户——这些存储可执行程序（又名智能合约）
3. 本地账户——这些账户用于核心区块链功能，如 Stake(质押)、Vote(投票)。

本地账户是区块链运行所必要的，我们稍后会深入探讨。 现在，我们将只处理数据和程序帐户。

在数据帐户中，您还有两种类型 -
1. 系统自有账户
2. PDA（Program Derived Address）账户

我们很快就会知道这些到底是什么™️，现在让我们继续吧。

每个帐户都有一些您应该了解的字段：

| 名词        | 注释                             |
| ---------- | -------------------------------- |
| lamports   | 该账户拥有的 lamport 数量   |
| owner      | 此帐户的程序所有者            |
| executable | 该帐户是否可以处理指令(可执行) |
| data       | 该账户存储的原始数据字节数组 |
| rent_epoch | 下一个 epoch，该帐户将拖欠租金 |

我们现在只关注需要知道的事情，所以如果有什么不合理的地方，继续向前——我们将开始填补我们前进路上的空白。

Lamports 是 Solana 的最小单元。 如果您熟悉以太坊生态系统，这有点像 Gwei。 一个 lamport = 0.000000001 SOL，所以这个字段只是告诉我们帐户有多少 SOL。

每个帐户都有一个公钥 - 它就像帐户的地址一样。 你知道用你的钱包地址，来接收那些火辣的 NFT 吗？ 同样的！ Solana 地址只是一串 base58 编码的字符串。

`executable` 是一个布尔字段，告诉我们帐户是否包含可执行数据。 数据是存储在帐户中的内容，稍后我们将介绍租金！

现在让我们看一些简单的东西:)

#### 📫 读取网络

好吧，我们已经知道了什么是帐户，我们该如何读取它们？ 我们将使用一种叫做 JSON RPC 端点的东西！ 看看这张图，就这是客户端，我们试图从 Solana 网络读取内容。

![](https://hackmd.io/_uploads/HyGV685Gi.png)

构建一个 API 对 JSON RPC 进行调用以获取你需要的内容，随之与网络通信并为你提供数据。

如果我们想要获取帐户余额，API 调用如下所示：

```ts
async function getBalanceUsingJSONRPC(address: string): Promise<number> {
    const url = clusterApiUrl('devnet')
    console.log(url);
    return fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            "jsonrpc": "2.0",
            "id": 1,
            "method": "getBalance",
            "params": [
                address
            ]
        })
    }).then(response => response.json())
    .then(json => {
        if (json.error) {
            throw json.error
        }

        return json['result']['value'] as number;
    })
    .catch(error => {
        throw error
    })
}
```

这里发生了很多事情。 我们正在发起一个 post 请求，其中 body 具有特定的参数告诉 RPC 要做什么。 我们需要指定 RPC 的版本、id、方法（在本例中为 getBalance）以及该方法所需的参数（在本例中为地址）。

我们有一堆非常简单的样板文件，因此我们可以使用 Solana 的 Web3.js SDK。 示例代码如下：

```ts
async function getBalanceUsingWeb3(address: PublicKey): Promise<number> {
    const connection = new Connection(clusterApiUrl('devnet'));
    return connection.getBalance(address);
}
```

那不是很漂亮吗？ 要获得某人的 Solana 余额，我们需要的就是这三行。 想象一下，如果获得任何人的银行存款余额那么容易该多好。

现在您知道如何从 Solana 上的帐户读取数据了！ 我知道这可能*看起来*微不足道，但只需使用这个功能，您就可以获得 Solana 上**任何**账户的余额。 想象一下，能够获得地球上任何一个银行账户的余额，这就是它的强大之处。

#### 🤑 构建一个余额获取器

是时候构建一个通用的余额获取器了（假设整个宇宙都在 Solana 上）。 这将是一个简单但功能强大的应用程序，可以获取 Solana 上任何帐户的余额。

在您的工作区创建一个文件夹。 我把我的放在桌面上， 克隆这个代码库 [starter repo](https://github.com/buildspace/solana-intro-frontend/tree/starter) 并进行设置：

```bash!
git clone https://github.com/buildspace/solana-intro-frontend.git
cd solana-intro-frontend
git checkout starter
npm i
```

这是一个简单的 `Next.js` 应用程序，因此一旦安装了所有依赖项，您就可以在终端中运行`npm run dev` 启动它。 你应该在本地主机上看到这个：

![](https://hackmd.io/_uploads/HklwcED9zj.png)

我们为您提供了一个普通`Next.js` 样例程序，如果您在地址字段中输入内容并点击“Check SOL Balance”按钮，您将看到 1,000 SOL 的余额。 是时候让它发挥作用了。
