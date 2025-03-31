# 1. Implementation of Eliza – Practical Exercise 
Original Repo from: https://github.com/elizaOS/eliza?tab=readme-ov-file#
## Prerequisites

- [Python 2.7+](https://www.python.org/downloads/)
- [Node.js 23+](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- [pnpm 23.3.0](https://pnpm.io/installation) 

## Step by Step Guide
- After cloning everything needs to be run in the /eliza folder
```bash

git clone repo
cp .env.example .env
```
- change .env:
  - OPENROUTER_API_KEY= # OpenRouter API Key
  - OPENROUTER_MODEL=  # Default: uses hermes 70b/405b
  - EVM_PRIVATE_KEY=          # Add the "0x" prefix infront of your private key string                 
  - EVM_PROVIDER_URL=

```bash
pnpm clean
pnpm i (e.g. with --no-frozen-lockfiles)
pnpm build
pnpm start --characters="Path/to/eliza/characters/dobby.character.json"
```

On another terminal:
```bash
pnpm start:client
```

You can now chat on the provided address.

Limitation: The Chatbot is not able to send tokens on chain... After reinstalling running and debugging everything for a couple of days. I am pretty sure that I will only be able to make this work by writing my own plugin/ not using the .alpha version of the tool but for that the challenge seems to be out of scope... 
```
Failed to import plugin: @elizaos-plugins/plugin-evm Error: Dynamic require of "net" is not supported
    at path/project_dw/eliza/packages/plugin-evm/dist/chunk-Q5Y5NR5U.js:12:11
    at ../../node_modules/.pnpm/@phala+dstack-sdk@0.1.7_bufferutil@4.0.9_typescript@5.6.3_utf-8-validate@5.0.10_zod@3.24.1/node_modules/@phala/dstack-sdk/dist/index.js (path/project_dw/eliza/packages/plugin-evm/dist/index.js:13:37)
    at __require2 (path/project_dw/eliza/packages/plugin-evm/dist/chunk-Q5Y5NR5U.js:18:52)
    at path/project_dw/eliza/packages/plugin-evm/dist/index.js:35927:33
    at ModuleJob.run (node:internal/modules/esm/module_job:274:25)
    at async onImport.tracePromise.__proto__ (node:internal/modules/esm/loader:644:26)
    at async path/project_dw/eliza/agent/src/index.ts:299:40
    at async Promise.all (index 0)
    at async handlePluginImporting (path/project_dw/eliza/agent/src/index.ts:297:33)
    at async jsonToCharacter (path/project_dw/eliza/agent/src/index.ts:179:25)
```

# 2. Knowledge – Theoretical Exercise
## 1. Can you identify potential security issues related to the Agent’s trading functionality?
- The agent has a big overhead in my opinion which might make it to slow for active trading -> applicable to sandwich attacks
- It is very complex so there is a limited amount of people who are able to work with it -> therefore it is only security wise checked by a few experts (also might be a positive thing)
- As far as I can see transaction amount is not limited also the number of transfers is not capped 
- private key written in plain text is a security risk

## 2. How can these security risks be mitigated in the code (e.g. maximal transaction amount)?
- Add restrictions on amount in the transferAction.handler, before calling action.transfer (somehthing in the way of this): 
```python
// New types to add to types/index.ts
interface TransactionLimits {
    maxAmountPerTransfer: bigint;
    dailyLimit: bigint;
    weeklyLimit: bigint;
    monthlyLimit: bigint;
    maxTransactionsPerHour: number;
    cooldownPeriodSeconds: number;
}

// New class to handle transaction tracking
class TransactionTracker {
    private transactions: Map<string, Transaction[]>;
    
    constructor() {
        this.transactions = new Map();
    }
    
    addTransaction(address: string, amount: bigint) {
        // Track transaction with timestamp
    }
    
    checkLimits(address: string, amount: bigint): boolean {
        // Check all limits
    }
}

// Modified TransferAction class
export class TransferAction {
    private transactionTracker: TransactionTracker;
    private limits: TransactionLimits;

    constructor(
        private walletProvider: WalletProvider,
        limits: TransactionLimits
    ) {
        this.transactionTracker = new TransactionTracker();
        this.limits = limits;
    }

    async transfer(params: TransferParams): Promise<Transaction> {
        // Add validation before transaction
        const address = this.walletProvider.getWalletClient(params.fromChain).account.address;
        
        if (!this.transactionTracker.checkLimits(address, parseEther(params.amount))) {
            throw new Error("Transaction exceeds limits or frequency caps");
        }

        // Rest of the existing transfer code...
    }
}
```
- use a key storage system instead of .env for example 