---
id: gas
title: Gas
sidebar_label: Gas
---

Gas is a unified unit of cost of computation on the blockchain. It is purchased at a variable price point (depending on system load) using NEAR tokens at the moment a transaction is processed.  Remaining tokens are refunded to the account that submitted the transaction.

Accounts also pay storage rent which accumulates over time and is charged once the account submits a transaction for processing.  Storage rent is charged for data and also as a tax on short account names to discourage name squatting.

This page covers these dynamics in more detail, starting with an an excerpt from the [NEAR Whitepaper](https://nearprotocol.com/papers/the-official-near-white-paper/) below:

<blockquote class="info">
<strong>NEAR Platform Economics -- What is Gas?</strong><br><br>

Fundamentally, the NEAR platform is a marketplace between willing participants.

On the supply side, operators of the validator nodes and other fundamental infrastructure need to be incentivized to provide these services which make up the community cloud.  On the demand side, the developers and end-users of the platform who are paying for its use need to be able to do so in a way which is simple, clear and consistent so it helps them.

A blockchain-based cloud provides several specific resources to the applications which run atop it:

- **Compute (CPU)**: This is the actual computer processing (and immediately available RAM) which run the code in a contract.
- **Bandwidth ("Network")**: This is the network traffic between participants and users, including messages which submit transactions and those which propagate blocks.
- **Storage**: Permanent data storage on the chain, typically expressed as a function of both storage space and time.

Existing blockchains like Ethereum account for all of these in a single up front transaction fee which represents a separate accounting for each of them but ultimately charges developers or users only once in a single fee.

**This is a high volatility fee commonly denominated in "gas".**

Developers prefer predictable pricing so they can budget and provide prices for their end users.  The pricing for the above-mentioned resources on NEAR is an amount which is slowly adjusted based on system usage (and subject to the smoothing effect of resharding for extreme usage) rather than being fully auction-based. This means that a developer can more predictably know that the cost of running transactions or maintaining their storage.

Initially, all of these resources will be priced and paid in terms of NEAR tokens. In the future, they may also be priced in terms of a stable currency denomination (for example a token pegged to the $USD).

[read more here](https://nearprotocol.com/papers/the-official-near-white-paper/#economy)

</blockquote>

## How do I buy gas?

You don't directly buy gas.  You attach tokens to a transaction or FunctionCall access key which are used to purchase gas at specific points in time.

Accounts purchase gas automatically when ...

1. **a transaction is processed** (ie. a function call is made, an account is created, tokens are transferred, etc) \
    The amount of gas used by a transaction depends on the details of the transaction itself. At genesis, the blockchain is configured with many different cost parameters (in units of gas). Through some governance process, these parameters may be changed over time. The actual cost of this gas, its price in NEAR tokens, is calculated dynamically and on an ongoing basis based on system load, inflation and other factors.

2. **an account is charged rent** to remain on the network  \
    Accounts are charged accumulated rent whenever they submit a transaction for processing.  Account names are also taxed if they are 10 characters or fewer to discourage name squatting.

    From the NEAR whitepaper,

    *"Storage is a long-term scarce asset which is priced on a rental basis.  This means that each account or contract will be charged per byte per block for its storage use deducted automatically from the balance on that account. The storage price is fixed and is only subject to change as a major governance decision."*

    *"If an account has data stored but does not have the resources to pay its storage bill for a given block, that account is deallocated, meaning that its storage will be released back to the chain.  Should the account desire to recover their storage, it will need to reallocate the storage by paying its rent and presenting back the data from the account. The previous state is recoverable by examining the prior history of the chain or third-party backups."*

    Note that accounts with short names pay a tax to discourage name squatting.  At time of writing, all account names must be 2 characters or more and are charged the short name tax up to 10 characters.  Account names which are 11 characters or longer are not taxed.  The tax increases exponentially starting from 10 characters with tax set at some amount `R` (measured in units of gas).  9 characters is `3 x R`, 8 characters is `3 x 3 x R` and so on down to 2 characters which is taxed most heavily at `3^8 x R` or about `6,500 x R`.  Refer to the [Key Concept: Account](/docs/concepts/account) page for more detail about accounts.

Thus, accounts have their balance in NEAR tokens and gas is bought on the fly when a transaction gets processed.

## An Example Scenario

Account has its balance in NEAR tokens

A user decides to send a transaction to the network.  She decides how much gas at most she is willing to spend for the transaction and sets a gas limit for processing the transaction.  Some transactions require an almost constant amount of gas but function calls are hard to predict so a little extra never hurts since whatever remains unspent will be returned.

The transaction reaches the network and, while being processed, it automatically "buys" enough gas to finish processing at the current gas prices.

Once the transaction is processed, the related account is only "charged" for the gas that was used.  Any leftover gas is converted back to NEAR tokens on the account.


## What about Prepaid Gas?

The Near Team understands that developers want to provide their users with the best possible onboarding experience.  To realize this vision, developers can design their applications in a way that first-time users can draw funds for purchasing gas directly from an account maintained by the developer.  Once onboarded, users can then transition to paying for their own platform use.

In this sense, prepaid gas can be realized using a funded account and related contract(s) for onboarding new users.

*So how can a developer pay the gas fee for his dApp users on NEAR?*

A user can use the funds directly from the developers account suitable only for the gas fees on this dApp. Then the developer has to distinguish users based on the signers' keys instead of the account names.

Check out [Key Concept: Account](/docs/concepts/account) "Did you know?" section, item `#2`.

NEAR protocol does not provide any limiting feature on the usage of developer funds. Developers can set allowances on access keys that correspond to specific users -- one `FunctionCall` access key per new user with a specific allowance.

## Experimental Observation

You can check out the price of gas yourself right now by issuing various transactions and even directly querying the price of gas on the network.

### What's the price of gas right now?

You can directly query the NEAR platform for the price of gas on a specific block using the RPC method `gas_price`.  This price may change depending on network load.  The price is denominated in yoctoNEAR (10^-24 NEAR)

1. Take any recent block hash from the blockchain using [NEAR Explorer](https://explorer.testnet.nearprotocol.com/blocks)

   *At time of writing, `SqNPYxdgspCT3dXK93uVvYZh18yPmekirUaXpoXshHv` was the latest block hash*

2. Issue an RPC request for the price of gas on this block using the method `gas_price` [documented here](/docs/interaction/rpc)

   ```bash
   http post https://rpc.testnet.nearprotocol.com jsonrpc=2.0 method=gas_price params:='["SqNPYxdgspCT3dXK93uVvYZh18yPmekirUaXpoXshHv"]' id=dontcare
   ```

3. Observe the results

   ```json
   {
     "id": "dontcare",
     "jsonrpc": "2.0",
     "result": {
       "gas_price": "5000"
     }
   }
   ```

The price of 1 unit of gas at this block was 5000 yoctoNEAR (10^-24 NEAR).

### How much does it cost to create an account?

1. use NEAR Wallet to create a new account

   > open https://wallet.testnet.nearprotocol.com and create a new account

2. open NEAR Explorer to the account page to find the transaction representing the account creation

   You can do this by clicking `View All` under "Activity" once your account is created or just by appending your account name to this URL:  \
   **explorer.testnet.nearprotocol.com/accounts/`your_account_name`**

   > try using account named `ebbs`:  [accounts / ebbs](https://explorer.testnet.nearprotocol.com/accounts/ebbs)

   You will see a "Batch Transaction" by `@test` (the NEAR TestNet faucet account). This transaction represents the initial account creation, funding (10 NEAR) and new key addition (`FullAccess`) since these 3 steps represent the minimum actions needed to create a new account (account creation, funding via faucet and full access to an owner).

3. use the RPC interface to query the full status of the transaction

   Clicking the link to the right of this Batch Transaction will open a page specific to the transaction itself at a URL matching the following pattern:  \
   **explorer.testnet.nearprotocol.com/transactions/`transaction_hash`**

   > view a sample [Transaction 27r7Xy...](https://explorer.testnet.nearprotocol.com/transactions/27r7XycLpnmmHsB79zTRn2kLC5Lyx1kQYrq9sBJmtXtq)

   Use this `transaction_hash` to execute the query in your terminal as per the line below.  Note that you will need some way to send the request over HTTP and we recommend [HTTPie](https://httpie.org/).

   ```bash
   http post https://rpc.testnet.nearprotocol.com jsonrpc=2.0 method=tx params:='["transaction_hash", ""]' id=dontcare
   ```

   > try using sample Tx `27r7Xy...` below

   ```bash
   http post https://rpc.testnet.nearprotocol.com jsonrpc=2.0 method=tx params:='["27r7XycLpnmmHsB79zTRn2kLC5Lyx1kQYrq9sBJmtXtq", ""]' id=dontcare
   ```

4. observe the amount of gas burnt by this transaction

    ```json
    {
      "id": "dontcare",
      "jsonrpc": "2.0",
      "result": {
        "receipts_outcome": [
          {
            "block_hash": "5TmnMGuqxiwK8rhn4fW8LPRRTstys4MXKZa5NEBesAhZ",
            "id": "HY2rWvEqW7E3shXxmndSTATrmCyQX3cpE3JKvrJYg8PS",
            "outcome": {
              "gas_burnt": 937144500000,
              "logs": [],
              "receipt_ids": [],
              "status": {
                "SuccessValue": ""
              }
            }
            // ... snippet removed for clarity ...
          }
        ],
        "transaction": {
          "actions": [
            "CreateAccount",
            {
              "Transfer": {
                "deposit": "10000001000000000000000000"
              }
            },
            {
              "AddKey": {
                "access_key": {
                  "nonce": 0,
                  "permission": "FullAccess"
                },
                "public_key": "ed25519:A64JPAjVHLFWCkXPWQY2bzpSXBWmTVeKpqki63BuBage"
              }
            }
          ]
        // ... snippet removed for clarity ...
        }
      }
    }
    ```

    The transaction used to create an account includes 2 actions:
    - `CreateAccount`  \
      *(which includes a transfer of 10 NEAR although `deposit` here is measured in yoctoNEAR and 10^24 yoctoNEAR === 1 NEAR)*
    - `AddKey` \
      *(which in this case was a `FullAccess` key pair whose public key starts with `A64JPA...`)*

    To create this account someone had to buy over 900 billion units of gas (`gas_burnt` = 937,144,500,000).

    At the current gas price of 5000 yoctoNEAR per unit of gas, we can multiply `gas_burnt` by `gas_price` to arrive at the total cost in yoctoNEAR tokens for this transaction.  The answer is 4.7 x 10^15 yoctoNEAR.

    This is easier to reason about if we change the scale to picoNEAR since this comes to about 4,700 picoNEAR (10^12 picoNEAR === 1 NEAR) for the entire transaction.

    This is a vanishingly small number but who spent those tokens?  The NEAR `@test` faucet account did.  The same faucet account also deposited 10 NEAR into this new account.



