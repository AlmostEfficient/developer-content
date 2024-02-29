---
date: Feb 29, 2024
difficulty: intermediate
title: "Full-stack Solana development with React and Anchor"
description: "Learn how to build a full-stack Solana dApp with React and Anchor"
tags:
  - web3js
  - anchor
keywords:
  - anchor
  - web3js
  - solana
  - anchor tutorial
  - intro to solana development
  - blockchain developer
  - blockchain tutorial
  - web3 developer
---

This guide will walk you through building a full-stack Solana dApp using React
and Anchor.

> The final code for this guide can be found in
> [this repository](https://github.com/AlmostEfficient/full-stack-solana-dev).

### Solana developer overview

If you've never used Solana and don't know what a blockchain is, I recommend
[this crash course](https://www.youtube.com/watch?v=uH60e4gZBSY) to get you up
to speed with the basics.

If any words or terms in this guide are confusing you, check out the
[terminology](https://solana.com/docs/terminology) page on the Solana docs,
they're pretty good!

## Project overview

We'll be building a full-stack Solana app using these tools:

- Anchor - program for building Solana programs in Rust
- Solana CLI - command line interface for interacting with Solana
- React - front-end framework
- wallet-adapter - library for connecting wallets to your app

### What you'll learn

- How to build a Solana program in Rust using Anchor
- Testing Anchor programs
- Deploying to the Solana devnet
- Building a React app that interacts with Solana from scratch
- Connecting your React app to your Solana program

### Environment setup

To begin, you'll need to set up your environment. This guide assumes you're
running MacOS or Linux. If you're on Windows, you'll need to install WSL and use
that instead,
[here's how](https://learn.microsoft.com/en-us/windows/wsl/install).

Here's a list of the tools you'll need to have installed:

- [Node.js](https://nodejs.org/en/download/)
- [Yarn](https://classic.yarnpkg.com/en/docs/install)
- [Rust](https://www.rust-lang.org/tools/install)
- [Solana CLI](https://solana.com/developers/guides/getstarted/setup-local-development)
- [Anchor](https://www.anchor-lang.com/docs/installation)

You can download Node and Yarn from their setup pages which I've linked above.
For the rest, check out the
[Solana Local development guide](https://solana.com/developers/guides/getstarted/setup-local-development)
which has detailed instructions for different operating systems.

Once you have everything installed, run this command:

```
rustc --version && solana --version && node -v && yarn --version && anchor --version
```

**Windows users** -- make sure you're running a WSL terminal, not a Windows
terminal.

You should see a number of versions printed out. If you get an error, you'll
need to install the missing tool.

If you haven't set up a Solana wallet yet, you can do that with:

```
solana-keygen new
```

You'll be prompted to enter a passphrase. This is the password you'll use to
sign transactions and interact with your wallet. I only use my wallet for
development, so I left mine blank.

The last thing you'll need is a Solana browser wallet extension to interact with
the web-app you build. I recommend [Phantom](https://phantom.app/).

## Write and deploy a Solana program

We'll start with our Solana Program (sometimes called a smart contract). This is
Rust code that will live on the Solana blockchain that we'll be interacting
with. To speed things up, we'll first deploy our program to a local Solana
network running on our machine. Configure your Solana CLI to localnet like this:

```
solana config set --url localhost
```

Now we need to set up the local validator. Open up a new terminal window and run
these commands:

```
cd ~
solana-test-validator
```

(Windows users -- `solana-test-validator` only works in the `~` directory.)

You'll see a bunch of logs that look like this:

```
endgame@~>solana-test-validator
Ledger location: test-ledger
Log: test-ledger/validator.log
⠈ Initializing...
Waiting for fees to stabilize 1...
Identity: 4GrmBaUtMM4CEKBDkwwne9AAVH9gzfqcoqm1Xwj3BfgT
Genesis Hash: E2yTivG7cf2pzfkQCtc3F3QPnerzFSS5Yya5MYnUUG8n
Version: 1.18.1
Shred Version: 18282
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
WebSocket PubSub URL: ws://127.0.0.1:8900
⠉ 00:02:13 | Processed Slot: 321 | Confirmed Slot: 321 | Finalized Slot: 289 | Full Snapshot Slot: 200 | Incremental Snapshot Slot
```

Nice! You have an entire Solana network running on your machine. Keep this
terminal window open -- if you close it the network shuts down. Head back to
terminal window 1 and run these:

```
solana airdrop 100
solana balance
```

You should see how much SOL you have on the local network. Let's write some
code!

### Set up an Anchor project

Anchor is a framework for building Solana programs. It reduces the amount of
boilerplate code you need to write and adds a lot of useful features like type
safety and testing. In your workspace, run these to create a new Anchor project
called `counter` and navigate into it:

```
anchor init counter
cd counter
```

Our program code will live in `programs/counter/src`. We'll interact and test
this program using tests in `tests/counter.ts` powered by Mocha/Chai. You can
ignore everything else in here for now.

Anchor projects come with everything we need to deploy out of the box. Run this
to build and deploy the program:

```
anchor build
```

This may take a minute on older machines the first time you run it. This command
converts the Rust code in `programs/counter/src` to a program that can be
deployed to the Solana blockchain. You'll see a new `target` folder has appeared
in your project. All the artifacts generated for your program are in there. Once
it's done, you'll see this:

```
   Compiling counter v0.1.0 (/mnt/full-stack-solana-dev/counter/programs/counter)
warning: unused variable: `ctx`
 --> programs/counter/src/lib.rs:9:23
  |
9 |     pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
  |                       ^^^ help: if this is intentional, prefix it with an underscore: `_ctx`
  |
  = note: `#[warn(unused_variables)]` on by default

warning: `counter` (lib) generated 1 warning (run `cargo fix --lib -p counter` to apply 1 suggestion)
    Finished release [optimized] target(s) in 4m 43s
```

You just built a Solana program! The output here is from the Rust compiler. If
there's any errors or issues with your code, you'll see them here.

Open up `programs/counter/src/lib.rs` and take a look at the code you just
built:

```rs
use anchor_lang::prelude::*;

declare_id!("Bims5KmWhFne1m1UT4bfSknBEoECeYfztoKrsR2jTnrA");

#[program]
pub mod counter {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}
```

The first line is bringing in the `anchor_lang` Rust library, similar to a
`import` statement in Javascript. Next we have a `declare_id!` Rust macro.
Macros in Rust are code that writes other code. The ID here is the program
address generated by Anchor.

Once Solana programs have been deployed to the network, they need to be
"initialized". This setup step is important because it sets initial states that
the program logic will depend on. It also assigns ownership of accounts to the
program, ensuring that the program has control over its state. Think of it
setting up a new office before you can begin business.

This is a Solana program does only one thing: it initializes. Once that's done,
it exits. It doesn't do anything in the initialization step here.

The most common flow in developing programs on Solana is:

- write program in Rust
- build program to check for errors (or use
  [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer)
  to get real-time errors)
- write Mocha/Chai tests to make sure it does what you expect
- test program
- repeat until you're ready to deploy

You'll find the inlcuded test for this program in `tests/counter.ts`:

```ts
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Counter } from "../target/types/counter";

describe("counter", () => {
  // Configure the client to use the local cluster.
  anchor.setProvider(anchor.AnchorProvider.env());

  const program = anchor.workspace.Counter as Program<Counter>;

  it("Is initialized!", async () => {
    // Add your test here.
    const tx = await program.methods.initialize().rpc();
    console.log("Your transaction signature", tx);
  });
});
```

If you haven't used Mocha before, this might seem unfamiliar. `describe` is used
to group related tests in a test suite and `it` is used to define individual
test cases. Anchor makes interacting with programs really easy by giving us a
nice API.

Here's what the Anchor stuff is:

- `AnchorProvider` lets us connect to our configured Solana development
  environment (local-test-validator).
- `const program = anchor.workspace.Counter` creates a reference to the program
  we just built.
- `as Program<Counter>` casts the program to the type in the IDL generated by
  Anchor when we built the program.
- `program.methods` gives us access to the methods we defined in our program.
- `initialize().rpc()` sends an RPC call with the `initialize` method.

Running this test with `anchor test --skip-local-validator` should give you:

```
  counter
Your transaction signature 2pMJRoC2h3AmCpfHQaptKTWV7Hyfuc2Gujfzrv6cffRhjptcuPbpMgahgrxtw28kRQ5Gf4d5VdMcon4j9aEmPyVy
    ✔ Is initialized! (325ms)

  1 passing (330ms)

Done in 12.65s.
```

When running tests, Anchor automatically sets up a local validator. Since we
already have one running, we can tell it to skip that step and use our one with
the `--skip-local-validator` flag.

### Write a counter program in Rust

We're ready to write our own program. We'll be building a simple counter program
that increments a counter and returns the current value.

Before we do this, delete your `target` folder to start from a clean slate. This
will remove all artifacts for your previous program, including the keypair used
to deploy it. This means you will lose control of any programs previously
deployed. While this isn't the common flow, the program we're going to deploy
next is much bigger so it isn't possible to upgrade the one we currently have
deployed.

Open up `programs/counter/src/lib.rs` and replace it with this:

```rs
use anchor_lang::prelude::*;

// Specify the program address
declare_id!("C93fyDjEmyAfr9nwDeWMVCeWVVx8fjySxnshSA9VY4KG");

// Instructions defined in program module
#[program]
pub mod counter {
    use super::*;

    // Instruction to initialize a new counter account
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        // Reference to the counter account from the Initialize struct
        let counter = &ctx.accounts.counter;
        msg!("Counter account created! Current count: {}", counter.count);
        Ok(())
    }

    // Instruction to increment a counter account
    pub fn increment(ctx: Context<Increment>) -> Result<()> {
        // Mutable reference to the counter account from the Increment struct
        let counter = &mut ctx.accounts.counter;
        msg!("Previous counter: {}", counter.count);

        // Increment the count value stored on the counter account by 1
        counter.count = counter.count.checked_add(1).unwrap();
        msg!("Counter incremented! Current count: {}", counter.count);
        Ok(())
    }
}

// Accounts required by the initialize instruction
#[derive(Accounts)]
pub struct Initialize<'info> {
    // The account paying to create the counter account
    #[account(mut)]
    pub user: Signer<'info>, // specify account must be signer on the transaction

    // The counter account being created and initialized in the instruction
    #[account(
        init,         // specifies we are creating this account
        payer = user, // specifies account paying for the creationg of the account
        space = 8 + 8 // space allocated to the new account (8 byte discriminator + 8 byte for u64)
    )]
    pub counter: Account<'info, Counter>, // specify account is 'Counter' type
    pub system_program: Program<'info, System>, // specify account must be System Program
}

// Account required by the increment instruction
#[derive(Accounts)]
pub struct Increment<'info> {
    #[account(mut)] // specify account is mutable because we are updating its data
    pub counter: Account<'info, Counter>, // specify account is 'Counter' type
}

// Define structure of `Counter` account
#[account]
pub struct Counter {
    pub count: u64, // define count value type as u64
}
```

The code here is simpler than it looks. Let's go block by block through the new
stuff.

```rs
// Instructions defined in program module
#[program]
pub mod counter {
    use super::*;

    // Instruction to initialize a new counter account
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        // Reference to the counter account from the Initialize struct
        let counter = &ctx.accounts.counter;
        msg!("Counter account created! Current count: {}", counter.count);
        Ok(())
    }

    // Instruction to increment a counter account
    pub fn increment(ctx: Context<Increment>) -> Result<()> {
        // Mutable reference to the counter account from the Increment struct
        let counter = &mut ctx.accounts.counter;
        msg!("Previous counter: {}", counter.count);

        // Increment the count value stored on the counter account by 1
        counter.count = counter.count.checked_add(1).unwrap();
        msg!("Counter incremented! Current count: {}", counter.count);
        Ok(())
    }
}
```

In the program module, we've got two instructions - these are like the endpoints
of an API. Everytime we call them via transaction sent to an RPC, we pass in
some context (`ctx`) - the state of the blockchain, which accounts are
interacting with it, any data passed in, etc. Think of this like the body of a
POST request.

`let counter = &ctx.accounts.counter;` creates a new account using the value
`counter` passed in from the context. This will make more sense when we look at
the test. `msg!` is a print statement that we can see on the logs in our
blockchain explorer.

The `increment` instruction creates a mutable reference to the counter account
we created in `initialize`. It then simply increments the value by 1

The other new things added are the necessary data structures for this program.
We have to define what accounts are required by each instruction, and what type
of data they contain.

Each Anchor project has it's own program address when it's first built. Since
you've just copied my code, the program address at the top of your `lib.rs`
won't match the one in your Anchor.toml. You can sync these with
`anchor keys sync`. Now you can hit `anchor build` to make sure there's no
errors.

Next up, let's test this. Open up `tests/counter.ts` and replace it with this:

```ts
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Counter } from "../target/types/counter";
import { Keypair } from "@solana/web3.js";

describe("counter", () => {
  // Configure the client to use the local cluster.
  const provider = anchor.AnchorProvider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.Counter as Program<Counter>;

  // Generate a new keypair to use as the address the counter account
  const counterAccount = new Keypair();

  it("Is initialized!", async () => {
    // Invoke the initialize instruction
    const transactionSignature = await program.methods
      .initialize()
      .accounts({
        counter: counterAccount.publicKey,
      })
      .signers([counterAccount]) // include counter keypair as additional signer
      .rpc({ skipPreflight: true });

    // Fetch the counter account data
    const accountData = await program.account.counter.fetch(
      counterAccount.publicKey,
    );

    console.log(`Transaction Signature: ${transactionSignature}`);
    console.log(`Count: ${accountData.count}`);
  });

  it("Increment", async () => {
    // Invoke the increment instruction
    const transactionSignature = await program.methods
      .increment()
      .accounts({
        counter: counterAccount.publicKey,
      })
      .rpc();

    // Fetch the counter account data
    const accountData = await program.account.counter.fetch(
      counterAccount.publicKey,
    );

    console.log(`Transaction Signature: ${transactionSignature}`);
    console.log(`Count: ${accountData.count}`);
  });
});
```

We've updated the `initialize` test case to match the structure the program
expects.

```ts
const counterAccount = new Keypair();
```

First, we generate a new keypair to use as the address the counter account. This
is where the data of the counter will be stored.

```ts
const transactionSignature = await program.methods
  .initialize()
  .accounts({
    counter: counterAccount.publicKey,
  })
  .signers([counterAccount]) // include counter keypair as additional signer
  .rpc({ skipPreflight: true });
```

Next, we pass in this address in the context of the initialize instruction as
`counter`. Since this account will be changed, we also need to include it as a
signer. To close out the test case we fetch the account data and print out the
value of the counter.

The `increment` test case caslls the increment instruction and fetches the
account data, but since we've already initialized it, we just pass in the
address of the counter account from global state.

Now that we're printing out messages in our logs, we'll set up a log viewer.
Open another terminal and run `solana logs`. This will stream Solana transaction
logs. You can also view transactions in the
[Solana explorer](https://explorer.solana.com/?cluster=custom&customUrl=http%3A%2F%2Flocalhost%3A8899)
by configuring it to use the local cluster.

Delete your `target` folder and run `anchor build` again. Before we can test
this, you need to replace the `declare_id!` macro with the address of your
program (cause you copy/pasted my program). You can do this with
`anchor keys sync`.

Let's run this! `anchor test --skip-local-validator` should give you:

```
Count: 1
    ✔ Increment (3973ms)


  2 passing (8s)

Done in 26.34s.
```

### Upgrade your Anchor program

The program we have is pretty good and it does what we want, but it's using an
inefficient method of storing data and is not completely secure.

Right now, we're manually creating an account for storing counter data. This
requires keeping track of the account, and if the keypair is leaked, anyone can
change it's value. We're also statically allocating the account size, which is
not ideal.

Update your `lib.rs` file to this:

```rs
use anchor_lang::prelude::*;

declare_id!("C87Mkt2suddDsb6Y15hJyGQzu9itMhU7RGxTQw17mTm");

#[program]
pub mod counter {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        counter.bump = ctx.bumps.counter; // store bump seed in `Counter` account
        msg!("Counter account created! Current count: {}", counter.count);
        msg!("Counter bump: {}", counter.bump);
        Ok(())
    }

    pub fn increment(ctx: Context<Increment>) -> Result<()> {
        let counter = &mut ctx.accounts.counter;
        msg!("Previous counter: {}", counter.count);
        counter.count = counter.count.checked_add(1).unwrap();
        msg!("Counter incremented! Current count: {}", counter.count);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(mut)]
    pub user: Signer<'info>,

    // Create and initialize `Counter` account using a PDA as the address
    #[account(
        init,
        seeds = [b"counter"], // optional seeds for pda
        bump,                 // bump seed for pda
        payer = user,
        space = 8 + Counter::INIT_SPACE
    )]
    pub counter: Account<'info, Counter>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Increment<'info> {
    // The address of the `Counter` account must be a PDA derived with the specified `seeds`
    #[account(
        mut,
        seeds = [b"counter"], // optional seeds for pda
        bump = counter.bump,  // bump seed for pda stored in `Counter` account
    )]
    pub counter: Account<'info, Counter>,
}

#[account]
#[derive(InitSpace)]
pub struct Counter {
    pub count: u64, // 8 bytes
    pub bump: u8,   // 1 byte
}
```

Now that we're storing the counter data in a PDA, we can derive the address of
it anywhere without needing a keypair. This also guaranttes that only the
program itself can sign transactions. This provides a built-in security layer
since no one can maliciously create a Counter account that the program would
mistakenly use.

Here's what the new test in `tests/counter.ts` will look like:

```ts
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Counter } from "../target/types/counter";
import { PublicKey } from "@solana/web3.js";

describe("counter", () => {
  // Configure the client to use the local cluster.
  const provider = anchor.AnchorProvider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.Counter as Program<Counter>;
  const wallet = provider.wallet as anchor.Wallet;
  const connection = provider.connection;

  const [counterPDA] = PublicKey.findProgramAddressSync(
    [Buffer.from("counter")],
    program.programId,
  );

  it("Is initialized!", async () => {
    try {
      const txSig = await program.methods.initialize().rpc();

      const accountData = await program.account.counter.fetch(counterPDA);
      console.log(`Transaction Signature: ${txSig}`);
      console.log(`Count: ${accountData.count}`);
    } catch (error) {
      // If PDA Account already created, then we expect an error
      console.log(error);
    }
  });

  it("Increment", async () => {
    const transactionSignature = await program.methods.increment().rpc();

    const accountData = await program.account.counter.fetch(counterPDA);

    console.log(`Transaction Signature: ${transactionSignature}`);
    console.log(`Count: ${accountData.count}`);
  });
});
```

As you can see, we're not generating a keypair here. Everything we need to
initialize this program and interact with it is easily definable and shareable.

Solana programs don't store state - everything is stored in accounts. This makes
it easy to upgrade programs without having to worry about losing data.

Once again, delete your `target` folder. Here's what you'll need to run to test
this:

```
anchor keys sync
anchor test --skip-local-validator
```

The first command will update the program address in your `lib.rs` file. The
second command will build, deploy, and test your program.

### Deploy to devnet

From here on out, we're DONE with local development. We'll be using the devnet
for our next steps.

We'll need some Solana devnet tokens to pay for transactions. You can configure
your wallet to the devnet and get some tokens with these commands:

```
solana config set --url devnet
solana airdrop 2
```

If this doesn't work, you can print out your wallet address with
`solana address` and use the [Solana faucet](https://faucet.solana.com/) to get
some tokens.

Now we're ready to deploy our program to the devnet. Here's a summary of what
we'll be doing:

- Update our `Anchor.toml` file to use the devnet
- Deploy our program to the devnet
- Update our program address to the devnet with anchor keys sync
- Rerun the test to initialize our program and increment the counter

Update your `Anchor.toml` file to use the devnet network and add the address for
the devnet program:

```
[toolchain]

[features]
seeds = false
skip-lint = false

[programs.localnet]
counter = "Bims5KmWhFne1m1UT4bfSknBEoECeYfztoKrsR2jTnrA"

[programs.devnet]
counter = "Bims5KmWhFne1m1UT4bfSknBEoECeYfztoKrsR2jTnrA"

[registry]
url = "https://api.apr.dev"

[provider]
cluster = "devnet"
wallet = "/home/endgame/.config/solana/id.json"

[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

What's changed? I've added a new `[programs.devnet]` section and changed the
`cluster` to `devnet`.

We're ready to deploy! Run `anchor test` and you should see this:

```
Deploying cluster: https://api.devnet.solana.com
Upgrade authority: /home/endgame/.config/solana/id.json
Deploying program "counter"...
Program path: /full-stack-solana-dev/counter/target/deploy/counter.so...
Program Id: F7nbbscQpQucypsjzJccBDLBSVAVKBHumNLpxqHXym4U

Deploy success

Found a 'test' script in the Anchor.toml. Running it as a test suite!

Running test suite: "/full-stack-solana-dev/counter/Anchor.toml"
```

Copy the address and check it out on the
[Solana explorer](https://explorer.solana.com/?cluster=devnet).

You're off localhost! Anyone on the internet can now interact with your program.
Let's give them a front-end to do that.

## Build a React client for your app

Now that your Solana program is live, you have several options for interacting
with it. Any client you can think of will work - Node.js script, React app,
mobile app, even a serverless function. We'll be building a React app for this
guide.

### Set up your React project

There's a bunch of Solana templates available for front-ends that give you
everything you need. **We're not gonna use a Solana template**. They often come
with a lot of stuff added in and can be overwhelming for a beginner. Instead,
I'm gonna walk you through doing it yourself. This will help you understand what
the templates are doing and show you how simple integrating Solana wallets is.

We'll be using [Vite](https://vitejs.dev/) because it's simpler than Next.js and
keeps things light. What you'll learn here will apply to ALL React frameworks -
Next, Remix, etc. Start by opening a new terminal window and creating a new vite
project inside your workspace:

```
yarn create vite
```

Name it whatever you want (I've named mine `front-end`). Select React as your
framework and Typescript as the variant. Don't worry, we're not gonna be doing
any fancy Typescript stuff!

If you're on Windows and using WSL, you'll need to update your `vite.config.ts`
file so hot reloading works. Open it up and change it to this:

```
// WSL USERS ON WINDOWS ONLY (NOT NECESSARY FOR LINUX/MACOS)
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
    watch: {
      usePolling: true,
    },
  },
})

```

Next, navigate into the front-end folder and run `yarn` to install all the
dependencies:

```
cd front-end
yarn
```

Your front-end is ready! Hit `yarn dev` and open up `http://localhost:5173/`.

### Create a connect wallet button

We're not gonne be doing this bit from scratch, Solana has some really nice
wallet adapter libraries that make adding wallets plug-n-play. You can even
customize the styling!

Throw this command into your terminal to install all the `wallet-adapter` stuff
we need:

```
yarn add @solana/wallet-adapter-base @solana/wallet-adapter-react @solana/wallet-adapter-react-ui @solana/wallet-adapter-wallets @solana/web3.js react
```

Ready to see some magic happen? Open up your `front-end/src/App.tsx` file and
replace the code in there with this:

```tsx
import { useMemo } from "react";
import {
  ConnectionProvider,
  WalletProvider,
} from "@solana/wallet-adapter-react";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";
import { PhantomWalletAdapter } from "@solana/wallet-adapter-wallets";
import {
  WalletModalProvider,
  WalletMultiButton,
} from "@solana/wallet-adapter-react-ui";
import { clusterApiUrl } from "@solana/web3.js";
import "./App.css";

// Default styles that can be overridden by your app
import "@solana/wallet-adapter-react-ui/styles.css";

function App() {
  // The network can be set to 'devnet', 'testnet', or 'mainnet-beta'.
  const network = WalletAdapterNetwork.Devnet;
  // You can also provide a custom RPC endpoint.
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  const wallets = useMemo(
    () => [new PhantomWalletAdapter()],
    // eslint-disable-next-line react-hooks/exhaustive-deps
    [network],
  );

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          <WalletMultiButton />
          <h1>Hello Solana</h1>
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}

export default App;
```

This is setup and config code taken from the
[wallet-adapter docs](https://github.com/anza-xyz/wallet-adapter/blob/master/APP.md).
All we're doing here is bringing in the `wallet-adapter` imports, setting up a
Solana network connection, and wrapping our app with the necessary context
providers.

Head back to `http://localhost:5173/` and you should see this:

![image](https://hackmd.io/_uploads/HJWI35ana.png)

We're ready to rumble!

P.S. you can change the app title in `index.html`.

### Reading from the blockchain

Ready to read from the blockchain? This is easier than you think!

First we need to build the layer that connects our deployed program with our
front-end React app. We'll define what our program is and how to interact with
it using the IDL.

Create a new folder named `anchor` in `front-end/src` and copy over
`target/types/counter.ts` (from the Anchor program) to a new file called
`idl.ts`. We'll use the IDL to create Typescript objects that let us interact
with our program.

We'll need Anchor to create the interfaces for the program, install it with:

```
yarn add  @coral-xyz/anchor
```

Now for the "connection layer" code. Create a `setup.ts` file in the
`front-end/src/anchor` folder and add this to it:

```ts
import { IdlAccounts, Program } from "@coral-xyz/anchor";
import { IDL, Counter } from "./idl";
import { clusterApiUrl, Connection, PublicKey } from "@solana/web3.js";

const programId = new PublicKey("B2Sj5CsvGJvYEVUgF1ZBnWsBzWuHRQLrgMSJDjBU5hWA");
const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

// Initialize the program interface with the IDL, program ID, and connection.
// This setup allows us to interact with the on-chain program using the defined interface.
export const program = new Program<Counter>(IDL, programId, {
  connection,
});

// To derive a PDA, we need:
// A seed - think of this like an ID or key (in a key-value store)
// The program address of the program the PDA belongs to

// This gives us the mintPDA that we'll reference when minting stuff
export const [mintPDA] = PublicKey.findProgramAddressSync(
  [Buffer.from("mint")],
  program.programId,
);

// Similarly, derive a PDA for when we increment the counter, using "counter" as the seed
export const [counterPDA] = PublicKey.findProgramAddressSync(
  [Buffer.from("counter")],
  program.programId,
);

// This is just a TypeScript type for the Counter data structure based on the IDL
// We need this so TypeScript doesn't yell at us
export type CounterData = IdlAccounts<Counter>["counter"];
```

What we're doing here is using the interface to lay the groundwork for
interacting with our program. We're generating the PDAs for the mint and counter
accounts. We need these because anytime we send a Solana transaction, we have to
specify all the accounts that will be changed by that transaction.

If we're minting a token, or changing the value of the counter, those accounts
will change, and we need to include their addresses in the transaction. These
are in their own file in`setup.ts` so we can access them in anywhere in our app.

Make sure you remember to replace the PublicKey to the address of your program.
Now that we have everything set up, we can READ!

Create a new directory `components` in `front-end/src` and add a
`counter-state.tsx` file in it with this code:

```ts
import { useEffect, useState } from "react";
import { useConnection } from "@solana/wallet-adapter-react";
import { program, counterPDA, CounterData } from "../anchor/setup";

export default function CounterState() {
  const { connection } = useConnection();
  const [counterData, setCounterData] = useState<CounterData | null>(null);

  useEffect(() => {
    // Fetch initial account data
    program.account.counter.fetch(counterPDA).then((data) => {
      setCounterData(data);
    });

    // Subscribe to account change
    const subscriptionId = connection.onAccountChange(
        // The address of the account we want to watch
      counterPDA,
        // callback for when the account changes
      (accountInfo) => {
        setCounterData(
          program.coder.accounts.decode("counter", accountInfo.data)
        );
      }
    );

    return () => {
      // Unsubscribe from account change
      connection.removeAccountChangeListener(subscriptionId);
    };
  // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [program]);

    // Render the value of the counter
  return <p className="text-lg">Count: {counterData?.count?.toString()}</p>;
}
```

Pretty straight-forward what's happening here. We use
`program.account.counter.fetch(counterPDA)` to fetch the value of counter on
load and then subscribe to any changes with `connection.onAccountChange`, which
takes in a callback as the second value. The callback decodes the data we get
back and sets it. EZPZ!

The final step is to add the component to our `App.tsx` file. Add this to the
bottom of the file:

```ts
// ... previous imports
// Import the component we just created
import CounterState from "./components/counter-state";

function App() {
  const network = WalletAdapterNetwork.Devnet;
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  const wallets = useMemo(
    () => [new PhantomWalletAdapter()],
    [network]
  );

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          <WalletMultiButton />
          <h1>Hello Solana</h1>
          // ADD THIS LINE
          <CounterState />
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}

export default App;
```

Now if you open up `http://localhost:5173/` and you should see something like:

```
Count: 2
```

WOAH. WE JUST READ DATA FROM THE BLOCKCHAIN. Let's keep the momentum rolling by
writing!

### Writing to the blockchain

Start by adding the `spl-token` library. We'll use this to get the associated
token account address for the user's wallet.

```
yarn add @solana/spl-token
```

The associated token account is where the user's tokens for this program are
stored. We need this cause we're going to mint a token in the increment
transaction.

Now to add increment functionality. We're going to put it all in a button
component. Create a new file `components/increment-button.tsx` and add this
code:

```tsx
import { useState } from "react";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import { getAssociatedTokenAddressSync } from "@solana/spl-token";
import { program, mintPDA } from "../anchor/setup";

export default function IncrementButton() {
  const { publicKey, sendTransaction } = useWallet();
  const { connection } = useConnection();
  const [isLoading, setIsLoading] = useState(false);

  const onClick = async () => {
    if (!publicKey) return;

    setIsLoading(true);

    try {
      const associatedTokenAccount = getAssociatedTokenAddressSync(
        mintPDA,
        publicKey,
      );

      const transaction = await program.methods
        .increment()
        .accounts({
          user: publicKey,
          tokenAccount: associatedTokenAccount,
        })
        .transaction();

      const transactionSignature = await sendTransaction(
        transaction,
        connection,
      );

      console.log(
        `View on explorer: https://solana.fm/tx/${transactionSignature}?cluster=devnet-alpha`,
      );
    } catch (error) {
      console.log(error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <>
      <button className="w-24" onClick={onClick} disabled={!publicKey}>
        {isLoading ? "Loading" : "Increment"}
      </button>
    </>
  );
}
```

You know most of this code. The only new thing happening here is the transaction
we're sending. We're using the `program.methods.increment()` method to create a
transaction that increments the counter and mints our token. The accounts passed
in are the user's wallet and the associated token account for the user's wallet.

Add this to to `App.tsx` to see the button:

```tsx
// ... previous imports
// Import the component we just created
import IncrementButton from "./components/increment-button";

function App() {
  const network = WalletAdapterNetwork.Devnet;
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  const wallets = useMemo(() => [new PhantomWalletAdapter()], [network]);

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          <WalletMultiButton />
          <h1>Hello Solana</h1>
          <CounterState />
          <IncrementButton />
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}

export default App;
```

Load up `http://localhost:5173/` and you should see a button. Make sure you're
connected to the
[devnet](https://docs.phantom.app/introduction/developer-settings) on Phantom
and and click the button. You should see the counter increment and the
associated token account minted.

LET'S GOOOOOOOOOOOOOOOOOO. You did it! You are now a full-stack Solana
developer.

## What now?

The world is yours for the taking. Anything you can imagine, you can create with
Solana.

- Join a Hackathon and get your hands dirty - https://solana.com/hackathon
- Pick a bounty and get some money - https://earn.superteam.fun/
- Build more advanced Solana apps - https://www.soldev.app/course
- Check out these templates -
  [solana-dapp-scaffold](https://github.com/solana-labs/dapp-scaffold),
  [create-solana-dapp](https://github.com/solana-developers/create-solana-dapp),
  [sample mobile apps](https://docs.solanamobile.com/sample-apps/sample_app_overview)

Whatever you make, share it with the world! Tag me on Twitter @almostefficient,
it makes me happy seeing y'all build :)

Good luck!