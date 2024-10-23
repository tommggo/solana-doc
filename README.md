# solana-doc
## SPL（Solana Program Library）
- 低级程序库: SPL 是一组低级的Solana程序，用于处理重要功能，例如代币管理、治理、质押等。这些程序已经部署在Solana上，供各种应用使用，帮助管理代币（如spl-token）、关联账户（如spl-associated-token-account）以及其他操作。
- 无抽象层: 使用SPL直接操作时，开发者需要手动处理账户验证和内存管理等内容，相较于Anchor的高级抽象，SPL 更接近底层操作
- You can find the full list of Token Program instructions *[here](https://github.com/solana-labs/solana-program-library/blob/b1c44c171bc95e6ee74af12365cb9cbab68be76c/token/program/src/instruction.rs)*.

### Solana Account Model
On Solana, all data is stored in what are referred to as "accounts”. The way data is organized on Solana resembles a key-value store, where each entry in the database is called an "account".

<img width="554" alt="image" src="https://github.com/user-attachments/assets/1c557e9d-2d8d-4fe3-8b30-da9fc4e1230b">

Key points：
- Accounts can store up to 10MB of data, which can consist of either executable program code or program state.
- Accounts require a rent deposit in SOL, proportional to the amount of data stored, which is fully refundable when the account is closed.
- Every account has a program "owner". Only the program that owns an account can modify its data or deduct its lamport balance. However, anyone can increase the balance.
- Programs (smart contracts) are stateless accounts that store executable code.
- Data accounts are created by programs to store and manage program state.
- Native programs are built-in programs included with the Solana runtime.
- Sysvar accounts are special accounts that store network cluster state.

### Account
Each account is identifiable by its unique address, represented as 32 bytes in the format of an Ed25519 PublicKey. You can think of the address as the unique identifier for the account.

<img width="539" alt="image" src="https://github.com/user-attachments/assets/3a266c09-574b-4d5c-b0b4-2df059577523">

This relationship between the account and its address can be thought of as a key-value pair, where the address serves as the key to locate the corresponding on-chain data of the account.

### AccountInfo
Accounts have a max size of 10MB (10 Mega Bytes) and the data stored on every account on Solana has the following structure known as the AccountInfo.
The AccountInfo for each account includes the following fields:
- data: A byte array that stores the state of an account. If the account is a program (smart contract), this stores executable program code. This field is often referred to as the "account data".
- executable: A boolean flag that indicates if the account is a program.
- lamports: A numeric representation of the account's balance in lamports, the smallest unit of SOL (1 SOL = 1 billion lamports).
- owner: Specifies the public key (program ID) of the program that owns the account.
As a key part of the Solana Account Model, every account on Solana has a designated "owner", specifically a program. Only the program designated as the owner of an account can modify the data stored on the account or deduct the lamport balance. It's important to note that while only the owner may deduct the balance, anyone can increase the balance.

<img width="552" alt="image" src="https://github.com/user-attachments/assets/82a58b26-3af7-4a56-b753-380db2cab368">

To store data on-chain, a certain amount of SOL must be transferred to an account. The amount transferred is proportional to the size of the data stored on the account. This concept is commonly referred to as “rent”. However, you can think of "rent" more like a "deposit" because the SOL allocated to an account can be fully recovered when the account is closed.
### Token Program
The Token Program contains all the instruction logic for interacting with tokens on the network (both fungible and non-fungible). All tokens on Solana are effectively data accounts owned by the Token Program.

You can find the full list of Token Program instructions *[here](https://github.com/solana-labs/solana-program-library/blob/b1c44c171bc95e6ee74af12365cb9cbab68be76c/token/program/src/instruction.rs)*.

<img width="570" alt="image" src="https://github.com/user-attachments/assets/1879e56c-044e-4593-b579-f698cd03b2b6">

A few commonly used instructions include:

- InitializeMint: Create a new mint account to represent a new type of token.
- InitializeAccount: Create a new token account to hold units of a specific type of token (mint).
- MintTo: Create new units of a specific type of token and add them to a token account. This increases the supply of the token and can only be done by the mint authority of the mint account.
- Transfer: Transfer units of a specific type of token from one token account to another.
### Mint Account
Tokens on Solana are uniquely identified by the address of a Mint Account owned by the Token Program. This account is effectively a global counter for a specific token, and stores data such as:

- Supply: Total supply of the token
- Decimals: Decimal precision of the token
- Mint authority: The account authorized to create new units of the token, thus increasing the supply
- Freeze authority: The account authorized to freeze tokens from being transferred from "token accounts"

<img width="543" alt="image" src="https://github.com/user-attachments/assets/ff5e49cd-e038-49a2-8ed5-bd245f9b381b">

The full details stored on each Mint Account include the following:
```
pub struct Mint {
    /// Optional authority used to mint new tokens. The mint authority may only
    /// be provided during mint creation. If no mint authority is present
    /// then the mint has a fixed supply and no further tokens may be
    /// minted.
    pub mint_authority: COption<Pubkey>,
    /// Total supply of tokens.
    pub supply: u64,
    /// Number of base 10 digits to the right of the decimal place.
    pub decimals: u8,
    /// Is `true` if this structure has been initialized
    pub is_initialized: bool,
    /// Optional authority to freeze token accounts.
    pub freeze_authority: COption<Pubkey>,
}
```

### Token Account
To track the individual ownership of each unit of a specific token, another type of data account owned by the Token Program must be created. This account is referred to as a Token Account.

The most commonly referenced data stored on the Token Account include the following:

- Mint: The type of token the Token Account holds units of
- Owner: The account authorized to transfer tokens out of the Token Account
- Amount: Units of the token the Token Account currently holds

<img width="549" alt="image" src="https://github.com/user-attachments/assets/168ce82a-dfc4-4825-9b5d-10da163290ad">

The full details stored on each Token Account includes the following:
```
pub struct Account {
    /// The mint associated with this account
    pub mint: Pubkey,
    /// The owner of this account.
    pub owner: Pubkey,
    /// The amount of tokens this account holds.
    pub amount: u64,
    /// If `delegate` is `Some` then `delegated_amount` represents
    /// the amount authorized by the delegate
    pub delegate: COption<Pubkey>,
    /// The account's state
    pub state: AccountState,
    /// If is_native.is_some, this is a native token, and the value logs the
    /// rent-exempt reserve. An Account is required to be rent-exempt, so
    /// the value is used by the Processor to ensure that wrapped SOL
    /// accounts do not drop below this threshold.
    pub is_native: COption<u64>,
    /// The amount delegated
    pub delegated_amount: u64,
    /// Optional authority to close the account.
    pub close_authority: COption<Pubkey>,
}
```

### Associated Token Account
To simplify the process of locating a token account's address for a specific mint and owner, we often use Associated Token Accounts.

An Associated Token Account is a token account whose address is deterministically derived using the owner's address and the mint account's address. You can think of the Associated Token Account as the "default" token account for a specific mint and owner.

It's important to understand that an Associated Token Account isn't a different type of token account. It's just a token account with a specific address.

<img width="547" alt="image" src="https://github.com/user-attachments/assets/2db06a5f-4f61-41f6-80e8-660cbd0a3c8a">

For two wallets to hold units of the same type of token, each wallet needs its own token account for the specific mint account. The image below demonstrates what this account relationship looks like.

<img width="543" alt="image" src="https://github.com/user-attachments/assets/04807efc-e50e-4a35-ad1b-b6f51bc685c4">

### System Account
这是由 Solana 的系统程序创建的基本账户类型。系统账户用于存储 SOL 和管理其他账户的基本操作。
### Program Account
When new programs are deployed on Solana, technically three separate accounts are created:

- Program Account: The main account representing an on-chain program. This account stores the address of an executable data account (which stores the compiled program code) and the update authority for the program (address authorized to make changes to the program).
- Program Executable Data Account: An account that contains the executable byte code of the program.
- Buffer Account: A temporary account that stores byte code while a program is being actively deployed or upgraded. Once the process is complete, the data is transferred to the Program Executable Data Account and the buffer account is closed.

<img width="547" alt="image" src="https://github.com/user-attachments/assets/ec3d8f86-193f-48ff-a31e-fbdd6d808701">

For simplicity, you can think of the "Program Account" as the program itself.

<img width="553" alt="image" src="https://github.com/user-attachments/assets/1bf8b335-e5af-4044-adff-1b19027a2c3b">

The address of the "Program Account" is commonly referred to as the “Program ID”, which is used to invoke the program.


### Data Account
Solana programs are "stateless", meaning that program accounts only contain the program's executable byte code. To store and modify additional data, new accounts must be created. These accounts are commonly referred to as “data accounts”.

Data accounts can store any arbitrary data as defined in the owner program's code.

<img width="537" alt="image" src="https://github.com/user-attachments/assets/ecff907d-4c43-4d8c-8f39-171d428a6449">


## Anchor
- 高级框架: Anchor 是一个用于简化Solana程序（智能合约）开发的高级框架。它通过提供声明式的宏、标准化模式和工具，抽象掉了许多与编写Solana程序相关的底层复杂性。
- Rust 宏和语法: Anchor 使用Rust宏（如#[program]、#[derive(Accounts)] 和 #[account]）来管理程序账户的结构和验证，这使得Solana开发更加简洁和安全，降低了出错的几率。
- CPI（跨程序调用）: Anchor 可以通过CPI轻松调用SPL程序。例如，使用Anchor时，你可以在不直接处理低级Solana API的情况下调用SPL Token或Token Metadata程序中的功能。
### 宏介绍
`declare_id!`
- 作用: 声明程序的唯一标识符（Program ID）。
- 用法: 通常在程序的顶部定义。
```
declare_id!("YourProgramIDHere");
```
`#[program]`
- 作用: 指定一个模块为 Anchor 程序模块，其中包含程序的所有入口函数。
- 用法: 在模块上方标记，并定义函数以处理来自客户端的请求。
```
#[program]
mod my_program {
    pub fn my_function(ctx: Context<MyContext>, arg: u64) -> Result<()> {
        // 处理逻辑
    }
}

```
`#[derive(Accounts)]`
- 作用: 定义一个结构体，指定程序所需的账户，帮助进行账户验证和管理。
- 用法: 创建一个结构体来描述参与某个指令的账户。
```
#[derive(Accounts)]
pub struct MyContext<'info> {
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

```
当一个指令需要访问或操作账户中反序列化后的数据时，使用 `Account` 类型。在 Anchor 框架中，`Account` 类型不仅代表了一个账户，还提供了与该账户相关的数据结构。这使得开发者可以方便地读取和修改账户的数据，而不需要手动进行数据解析或转换。简单来说，`Account` 类型让账户的使用变得更直观和高效。

#### The Accounts Struct
The Accounts struct is where you define which accounts your instruction expects and which constraints these accounts should adhere to. You do this via two constructs: Types and constraints.
- Types: Each type has a specific use case in mind. Detailed explanations for the types can be found in the *[reference](https://docs.rs/anchor-lang/latest/anchor_lang/accounts/index.html)*. 
- Constraints: Account types can do a lot of work for you but they're not dynamic enough to handle all the security checks a secure program requires.*[Constraints reference](https://docs.rs/anchor-lang/latest/anchor_lang/derive.Accounts.html)*.

Add constraints to an account with the following format:
```
#[account(<constraints>)]
pub account: AccountType
```
Some constraints support custom Errors:
```
#[account(...,<constraint> @ MyError::MyErrorVariant, ...)]
pub account: AccountType
```

`#[account]`
- 作用: 定义一个账户的数据结构，指定其存储的数据格式。
- 用法: 在结构体上方使用，以便 Anchor 可以自动处理该账户的初始化和存储。
```
#[account]
pub struct MyAccount {
    pub data: u64,
}
```
`#[init]`
- 作用: 用于在账户初始化时指定特定的参数和设置。
- 用法: 通常与 #[account] 一起使用。
```
#[account(init, payer = user, space = 8 + 8)]
pub new_account: Account<'info, MyAccount>,
```
`#[mut]`
- 作用: 指定一个账户是可变的，可以被修改。
- 用法: 在账户声明中使用。
```
#[account(mut)]
pub user: Signer<'info>,
```
`#[close]`
- 作用: 指定一个账户可以被关闭，并将其余额转移到指定的账户。
- 用法: 在账户声明中使用。
```
#[account(close = user)]
pub my_account: Account<'info, MyAccount>,
```
`#[msg]`
- 作用: 记录信息日志，可以在链上查询。
- 用法: 在函数内部调用以记录调试信息。
```
msg!("This is a log message.");
```
`#[error]`
- 作用: 定义一个结构体来描述错误类型，帮助提供更详细的错误信息。
- 用法: 定义错误类型并使用 Result 来返回。
```
#[error]
pub enum MyError {
    #[msg("Custom error message")]
    CustomError,
}
```
`#[instruction]`
- 作用: 用于对指令进行描述，帮助理解和生成客户端代码。
- 用法: 在指令函数中使用。
```
#[instruction]
pub fn my_instruction(ctx: Context<MyContext>, amount: u64) -> Result<()> {
    // 处理逻辑
}
```

## Solana CLI
与 Solana 区块链进行交互的命令行界面工具。主要功能：
1. 账户管理: 创建、查看和管理 Solana 上的账户，包括生成新密钥对、查询账户余额等。
示例命令：solana-keygen new 用于生成新的密钥对。
2. 部署和管理智能合约: 使用 Solana CLI，开发者可以将编译好的智能合约（通常是 BPF 格式）部署到 Solana 区块链上，并进行测试和调用。
示例命令：solana program deploy <file_path> 用于部署智能合约。
3. 代币操作: 结合 SPL Token 程序，CLI 允许用户创建、管理代币，进行代币转账，查询代币信息等。
示例命令：spl-token create-token 创建一个新代币。
4. 网络状态查询: CLI 还可以用来查看当前 Solana 网络的状态，包括集群信息、块生产者状态等。
示例命令：solana cluster-version 查询当前集群的版本信息。
5. 交易管理: 用户可以通过 CLI 发起、签名并发送交易到区块链上，并查看交易历史和详情。
示例命令：solana transfer <recipient_address> <amount> 进行 SOL 代币的转账。
通过这些命令，开发者可以方便地在命令行中进行链上操作，适用于在 Solana 网络中快速开发和测试的场景。
## create token
1. 使用 spl-token 命令行工具：
这是最简单、最流行的方式之一，特别适合不需要定制逻辑的代币项目，如创建标准的 SPL 代币。spl-token 是 Solana 官方提供的工具，能够快速创建代币并管理它们。
   1. 步骤：
      - 使用 spl-token create-token 创建一个新的 Mint Account。
      - 使用 spl-token create-account 或 spl-token create-associated-token-account 创建用户的 Token Account 来存储代币。
      - 使用 spl-token mint 铸造代币到指定的账户。
      - 使用 spl-token transfer 来转移代币。
   2. 适用场景：创建标准的同质化代币（如社区代币、奖励代币等）。
   3. 优点：简单、快速、适合标准代币。
      
2. 编写定制的 Solana Program（智能合约）
如果你需要更复杂的代币逻辑（例如动态铸造、基于某种条件的转移规则、访问控制等），可以编写一个自定义的 Solana Program 来实现代币创建和管理。
   1. 步骤：
      - 编写一个 Rust 程序，定义代币的行为和规则。
      - 部署你的 Program 到 Solana 网络。
      - 使用你的自定义 Program 来创建和管理代币。
   2. 优点：灵活、可定制化。
   3. 适用场景：需要定制功能的代币，比如特殊的 DeFi 协议代币、NFT 项目或 GameFi 代币。
      
3. 使用 Solana 框架或 SDK
开发者可以使用一些第三方框架或 SDK 来帮助简化代币创建和管理的流程。这些工具通常封装了底层的 Solana 调用，提供更加简便的 API 和功能。
例子：使用 Anchor 框架，它是一个为 Solana 上开发智能合约而构建的 Rust 框架，提供了更简洁的开发流程和常见功能的支持。
    1. 步骤：
       - 使用 Anchor 编写程序，包含代币创建和管理逻辑。
       - 部署并与之交互。
    2. 优点：简化开发过程，特别是如果你要构建更复杂的应用。
    3. 适用场景：复杂的 DeFi、DAO、NFT 项目开发。
