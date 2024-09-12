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



## Solana Pay QR codes


One of the standout features of Solana Pay is its easy integration with QR codes. Since transfer and transaction requests are simply URLs, you can embed them into QR codes that you make available in your application or elsewhere.

The `@solana/pay` library simplifies this with the provided `createQR` helper function. This function needs you to provide the following:

- `url` - the URL of the transaction request.
- `size` (optional) - the width and height of the QR code in pixels. Defaults to 512.
- `background` (optional) - the background color. Defaults to white.
- `color` (optional) - the foreground color. Defaults to black.

Here's an example of how to use the `createQR` function to generate a QR code for a check-in location:

```typescript
import { createQR } from '@solana/pay';

export function createCheckInQR(id: number) {
  const url = new URL(`${window.location.origin}/api/checkIn`);
  url.searchParams.append('id', id.toString());
  
  // Generate a random reference for this transaction
  const reference = new Uint8Array(32);
  window.crypto.getRandomValues(reference);
  url.searchParams.append('reference', encodeURIComponent(buffer.Buffer.from(reference).toString('base64')));

  // Create the QR code
  const qr = createQR(url, 400, 'transparent', 'black');
  return qr;
}
```

In this example:
- We create a URL for the check-in endpoint, including the location ID as a query parameter.
- We generate a random reference for the transaction and add it to the URL.
- We use `createQR` to generate the QR code, specifying a size of 400 pixels, a transparent background, and black foreground color.

You can then use this function in your React component to display the QR code:

```tsx
import { useState, useEffect } from 'react';
import { createCheckInQR } from '../utils/createQrCode/checkIn';

export default function LocationQRCode({ id }: { id: number }) {
  const [qr, setQr] = useState<HTMLElement | null>(null);

  useEffect(() => {
    const qrCode = createCheckInQR(id);
    setQr(qrCode);
  }, [id]);

  return (
    <div>
      <h2>Check-in for Location {id}</h2>
      {qr && <div ref={(ref) => ref && ref.appendChild(qr)} />}
    </div>
  );
}
```


## Setup

1. Clone the repository:
   ```
   git clone https://github.com/your-repo/solana-pay-scavenger-hunt.git
   cd solana-pay-scavenger-hunt
   ```

2. Install dependencies:
   ```
   yarn install
   ```

3. Set up environment variables:
   - Rename `.env.example` to `.env`
   - Fill in the required values in the `.env` file, for example:
     ```
     NEXT_PUBLIC_RPC_ENDPOINT=https://api.devnet.solana.com
     EVENT_ORGANIZER_SECRET_KEY=your_secret_key_here
     ```

4. Run the development server:
   ```
   yarn dev
   ```

5. Set up ngrok for HTTPS:
   ```
   ngrok http 3000
   ```
   Use the HTTPS URL provided by ngrok to access your app from mobile devices.

## Project Structure

- `pages/`: Next.js pages and API routes
- `components/`: React components
- `utils/`: Utility functions and helpers
- `public/`: Static assets

## Key Components and Code Examples

1. **Transaction Request Endpoint** (`pages/api/checkIn.ts`):

This file handles GET and POST requests for Solana Pay. Here's a simplified version of the main handler:

```typescript
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === "GET") {
    return get(res);
  } else if (req.method === "POST") {
    return await post(req, res);
  } else {
    return res.status(405).json({ error: "Method not allowed" });
  }
}

function get(res: NextApiResponse) {
  res.status(200).json({
    label: "Scavenger Hunt!",
    icon: "https://solana.com/src/img/branding/solanaLogoMark.svg",
  });
}

async function post(req: NextApiRequest, res: NextApiResponse) {
  const { account } = req.body;
  const { reference, id } = req.query;

  // ... (validation and transaction building logic)

  res.status(200).json({
    transaction: transaction,
    message: `You've found location ${id}!`,
  });
}
```

2. **QR Code Generator** (`utils/createQrCode/checkIn.ts`):

This utility creates QR codes for Solana Pay requests:

```typescript
import { createQR } from '@solana/pay';

export function createCheckInQR(id: number) {
  const url = new URL(`${window.location.origin}/api/checkIn`);
  url.searchParams.append('id', id.toString());
  const qr = createQR(url, 400, 'transparent');
  return qr;
}
```

3. **Scavenger Hunt Program** (Anchor program on Devnet):

Here's a simplified version of the Anchor program:

```rust
use anchor_lang::prelude::*;

declare_id!("your_program_id_here");

#[program]
pub mod scavenger_hunt {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>, game_id: Pubkey) -> Result<()> {
        let user_state = &mut ctx.accounts.user_state;
        user_state.user = ctx.accounts.user.key();
        user_state.game_id = game_id;
        user_state.last_location = Pubkey::default();
        Ok(())
    }

    pub fn check_in(ctx: Context<CheckIn>, game_id: Pubkey, location: Pubkey) -> Result<()> {
        let user_state = &mut ctx.accounts.user_state;
        require!(user_state.game_id == game_id, ErrorCode::InvalidGame);
        // Add logic to verify correct location order
        user_state.last_location = location;
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = user, space = 8 + 32 + 32 + 32)]
    pub user_state: Account<'info, UserState>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct CheckIn<'info> {
    #[account(mut)]
    pub user_state: Account<'info, UserState>,
    pub user: Signer<'info>,
    /// CHECK: This is not dangerous as we don't read or write from this account
    pub event_organizer: UncheckedAccount<'info>,
}

#[account]
pub struct UserState {
    pub user: Pubkey,
    pub game_id: Pubkey,
    pub last_location: Pubkey,
}
```

4. **Frontend** (`pages/index.tsx`):

Here's a simplified version of the main page component:

```tsx
import { useState } from 'react';
import { createCheckInQR } from '../utils/createQrCode/checkIn';

export default function Home() {
  const [locationId, setLocationId] = useState(1);

  return (
    <div>
      <h1>Solana Pay Scavenger Hunt</h1>
      <div>
        {createCheckInQR(locationId)}
      </div>
      <button onClick={() => setLocationId(locationId + 1)}>
        Next Location
      </button>
    </div>
  );
}
```

## Using the Latest Solana Standards

This project uses the latest Solana standards as of September 2024:

- Solana Web3.js v1.89.0 or later
- @solana/pay v0.2.0 or later
- @project-serum/anchor v0.26.0 or later

Make sure to check for updates and adjust the code if newer versions are available.

## Running the Scavenger Hunt

1. Start the development server and set up ngrok as described in the Setup section.
2. Open the ngrok HTTPS URL on your mobile device.
3. Ensure your Solana wallet is set to Devnet.
4. Scan the QR code for Location 1 to start the scavenger hunt.
5. Follow the prompts to visit each location in order.

## Important Concepts

1. **Partial Signing**: The server partially signs the transaction before sending it to the client. This allows for additional security measures and control over the transaction.

2. **PDA (Program Derived Address)**: Used to deterministically generate addresses for user state accounts.

3. **Transaction Building**: Transactions are built server-side, including both the `initialize` and `check_in` instructions when necessary.

4. **Error Handling**: Proper error handling is implemented both on the client and server side to provide a smooth user experience.

## Challenges and Extensions

After completing the basic scavenger hunt, try these challenges:

1. Implement a reward system using SPL tokens or NFTs.
2. Add time limits or other constraints to the scavenger hunt.
3. Create a leaderboard for multiple participants.
4. Integrate with real-world locations using geolocation.

## Resources

- [Solana Pay Documentation](https://docs.solanapay.com/)
- [Solana Cookbook](https://solanacookbook.com/)
- [Anchor Documentation](https://www.anchor-lang.com/)

