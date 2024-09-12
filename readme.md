# Solana Pay: Revolutionizing Blockchain Transactions

## Executive Summary

Solana Pay represents a significant advancement in blockchain technology, offering a standardized method for encoding Solana transaction requests within URLs. This innovation facilitates seamless transaction requests across diverse Solana applications and wallets, enhancing user experience and interoperability.

Key features include:

- **Standardized Transaction Requests**: Enables uniform transaction processing across various Solana platforms.
- **Partial Transaction Signing**: Allows for multi-signature transactions, enhancing security and flexibility.
- **Transaction Gating**: Implements conditional processing of transactions based on predefined criteria.

## Introduction to Solana Pay

The Solana ecosystem continues to evolve, not just through the development of new technologies, but also by ingeniously leveraging existing features. Solana Pay exemplifies this approach, utilizing the network's robust signing capabilities in novel ways. This innovation empowers merchants and developers to:

- Request transactions efficiently
- Implement sophisticated gating mechanisms for specific transaction types

Throughout this comprehensive guide, we will explore the multifaceted capabilities of Solana Pay, including:

1. Creating transfer and transaction requests
2. Encoding requests into QR codes for easy mobile interaction
3. Implementing partial transaction signing for enhanced security
4. Developing transaction gating based on custom conditions

Our goal is to demonstrate how Solana Pay serves as a springboard for innovative client-side network interactions, encouraging developers to think creatively about blockchain technology applications.

## Technical Overview of Solana Pay

### The Solana Pay Specification

Solana Pay introduces a set of standards that revolutionize how users request payments and initiate transactions across the Solana network. This specification utilizes URL-based requests, ensuring uniformity and ease of use across various Solana-compatible applications and wallets.

### URL Structure and Functionality

Solana Pay request URLs are prefixed with `solana:`, serving as a protocol identifier. This prefix is crucial for directing the link to appropriate applications, particularly on mobile platforms. When a user interacts with a `solana:` URL, it triggers compatible wallet applications, which then process the remainder of the URL to handle the request appropriately.

### Types of Solana Pay Requests

The specification defines two primary types of requests:

1. **Transfer Requests**: Designed for straightforward SOL or SPL Token transfers.
2. **Transaction Requests**: Capable of initiating any type of Solana transaction, offering greater flexibility and complexity.

## Detailed Analysis of Transfer Requests

Transfer requests are non-interactive and follow a specific URL format:

```
solana:<recipient>?<optional-query-params>
```

Key components of a transfer request include:

- **Recipient**: A mandatory field containing the base58-encoded public key of the receiving account.
- **Optional Query Parameters**:
  - `amount`: Specifies the transfer amount (non-negative integer or decimal).
  - `spl-token`: Identifies the SPL Token mint account for token transfers.
  - `reference`: Provides transaction identification on-chain (base58-encoded 32-byte arrays).
  - `label`: Describes the transfer request source (URL-encoded UTF-8 string).
  - `message`: Explains the nature of the request (URL-encoded UTF-8 string).
  - `memo`: Includes additional information in the transaction (URL-encoded UTF-8 string).

### Example Transfer Request URLs:

For 1 SOL transfer:
```
solana:mvines9iiHiQTysrwkJjGf2gb9Ex9jXJX8ns3qwf2kN?amount=1&label=Michael&message=Thanks%20for%20all%20the%20fish&memo=OrderId12345
```

For 0.1 USDC transfer:
```
solana:mvines9iiHiQTysrwkJjGf2gb9Ex9jXJX8ns3qwf2kN?amount=0.01&spl-token=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
```

## In-Depth Look at Transaction Requests

Transaction requests offer a more dynamic and interactive approach compared to transfer requests. While still URL-based, they provide greater flexibility:

```
solana:<link>
```

In this format, `link` represents a URL to which the wallet can make HTTP requests. This design allows for more complex transactions by fetching transaction details from a specified endpoint.

### Transaction Request Workflow

When a wallet processes a transaction request URL, it follows a four-step procedure:

1. **Initial GET Request**: The wallet sends a GET request to the provided link URL, retrieving a label and icon for user display.
2. **User Information POST Request**: Following the GET request, the wallet sends a POST request containing the end user's public key.
3. **Transaction Building**: The application uses the provided public key and any additional link information to construct the transaction. It then responds with a base64-encoded serialized transaction.
4. **User Interaction**: The wallet decodes and deserializes the transaction, presenting it to the user for signing and submission.

This multi-step process allows for dynamic transaction creation based on real-time data and user-specific information, offering a powerful tool for developers to create sophisticated blockchain interactions.

**How to setup a Transaction**

# Solana Pay Transaction Request 

This guide demonstrates how to create a transaction request endpoint for Solana Pay using modern JavaScript and the latest Solana web3.js library.

## Table of Contents

1. [Introduction](#introduction)
2. [Setting Up the API Endpoint](#setting-up-the-api-endpoint)
3. [Handling GET Requests](#handling-get-requests)
4. [Handling POST Requests](#handling-post-requests)
5. [Building and Returning Transactions](#building-and-returning-transactions)
6. [Confirming Transactions](#confirming-transactions)
7. [Implementing Gated Transactions](#implementing-gated-transactions)

## Introduction

Solana Pay transaction requests allow for dynamic creation of transactions based on user data. This guide will walk you through creating an API endpoint that handles these requests using Next.js API Routes.

## Setting Up the API Endpoint

First, create a new file in your `pages/api` directory. We'll call it `transaction-request.js`:

```javascript
import { Connection, PublicKey, SystemProgram, Transaction, LAMPORTS_PER_SOL } from '@solana/web3.js';
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'GET') {
    return handleGet(res);
  } else if (req.method === 'POST') {
    return handlePost(req, res);
  } else {
    return res.status(405).json({ error: 'Method not allowed' });
  }
}
```

## Handling GET Requests

The GET request should return basic information about the transaction request:

```javascript
function handleGet(res: NextApiResponse) {
  res.status(200).json({
    label: "My Solana Pay Store",
    icon: "https://solana.com/src/img/branding/solanaLogoMark.svg",
  });
}
```

## Handling POST Requests

The POST request is where we'll build the actual transaction:

```javascript
async function handlePost(req: NextApiRequest, res: NextApiResponse) {
  const { account } = req.body;
  if (!account) {
    return res.status(400).json({ error: 'Missing account' });
  }

  try {
    const transaction = await buildTransaction(new PublicKey(account));
    const serializedTransaction = transaction.serialize({ requireAllSignatures: false });
    const base64Transaction = serializedTransaction.toString('base64');

    res.status(200).json({
      transaction: base64Transaction,
      message: 'Transfer of 0.001 SOL',
    });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Error creating transaction' });
  }
}
```

## Building and Returning Transactions

Here's how to build a simple transfer transaction:

```javascript
async function buildTransaction(account: PublicKey): Promise<Transaction> {
  const connection = new Connection('https://api.devnet.solana.com');
  const { blockhash } = await connection.getLatestBlockhash();

  const transaction = new Transaction({
    recentBlockhash: blockhash,
    feePayer: account,
  });

  const recipientAccount = new PublicKey('INSERT_RECIPIENT_PUBLIC_KEY_HERE');
  
  const transferInstruction = SystemProgram.transfer({
    fromPubkey: account,
    toPubkey: recipientAccount,
    lamports: 0.001 * LAMPORTS_PER_SOL,
  });

  transaction.add(transferInstruction);

  return transaction;
}
```

## Confirming Transactions

To confirm transactions, you can use a reference key:

```javascript
import { findReference, FindReferenceError } from '@solana/pay';

async function confirmTransaction(reference: PublicKey, connection: Connection) {
  try {
    const signatureInfo = await findReference(connection, reference, { finality: 'confirmed' });
    return signatureInfo.signature;
  } catch (error) {
    if (error instanceof FindReferenceError) {
      return null;
    }
    throw error;
  }
}
```

## Implementing Gated Transactions

You can implement gated transactions by checking conditions before building the transaction. Here's an example that checks for NFT ownership:

```javascript
import { Metaplex } from '@metaplex-foundation/js';

async function buildGatedTransaction(account: PublicKey, collectionAddress: PublicKey): Promise<Transaction | null> {
  const connection = new Connection('https://api.devnet.solana.com');
  const metaplex = new Metaplex(connection);

  const nfts = await metaplex.nfts().findAllByOwner({ owner: account }).run();

  const hasRequiredNFT = nfts.some(nft => 
    nft.collection?.address.equals(collectionAddress)
  );

  if (!hasRequiredNFT) {
    return null; // Or throw an error
  }

  // If the user has the required NFT, build and return the transaction
  return buildTransaction(account);
}
```

To use this in your POST handler:

```javascript
async function handlePost(req: NextApiRequest, res: NextApiResponse) {
  const { account } = req.body;
  if (!account) {
    return res.status(400).json({ error: 'Missing account' });
  }

  try {
    const collectionAddress = new PublicKey('INSERT_COLLECTION_ADDRESS_HERE');
    const transaction = await buildGatedTransaction(new PublicKey(account), collectionAddress);
    
    if (!transaction) {
      return res.status(403).json({ error: 'User does not have the required NFT' });
    }

    const serializedTransaction = transaction.serialize({ requireAllSignatures: false });
    const base64Transaction = serializedTransaction.toString('base64');

    res.status(200).json({
      transaction: base64Transaction,
      message: 'Gated transfer of 0.001 SOL',
    });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Error creating transaction' });
  }
}
```

