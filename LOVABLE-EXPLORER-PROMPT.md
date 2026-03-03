# BLOCK Blockchain Explorer — Lovable Build Prompt

Copy everything below this line and paste it into Lovable:

---

Build me a modern, professional blockchain explorer web application for the **BLOCK** blockchain. This is a custom L1 EVM-compatible blockchain using Proof of Authority (Clique) consensus.

## 🔗 Data Sources (APIs)

### Primary: Direct RPC (JSON-RPC)
```
RPC URL: http://64.52.80.204:8545
```
All calls use standard Ethereum JSON-RPC over HTTP POST with `Content-Type: application/json`.

Example call:
```javascript
const response = await fetch('http://64.52.80.204:8545', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    method: 'eth_blockNumber',
    params: [],
    id: 1
  })
});
const data = await response.json();
// data.result = "0x1D4C3" (hex block number)
```

### Secondary: Blockscout REST API (for indexed data like search, tx history)
```
Blockscout API: http://64.111.92.74/api/v2
```

Useful endpoints:
- `GET /api/v2/stats` — chain stats (total blocks, txs, addresses)
- `GET /api/v2/blocks?type=block` — latest blocks list
- `GET /api/v2/transactions` — latest transactions list
- `GET /api/v2/addresses/{hash}` — address details + balance
- `GET /api/v2/addresses/{hash}/transactions` — address tx history
- `GET /api/v2/transactions/{hash}` — single transaction details
- `GET /api/v2/blocks/{number}` — single block details
- `GET /api/v2/search?q={query}` — search for address, tx hash, or block
- `GET /api/v2/addresses?sort=balance&order=desc` — top holders

## ⛓️ Chain Configuration

| Parameter | Value |
|-----------|-------|
| Chain Name | BLOCK |
| Chain ID | 9999 |
| Consensus | Proof of Authority (Clique) |
| Block Time | ~3 seconds |
| Currency Symbol | BLOCK |
| Decimals | 18 |
| Total Supply | 10,000,000,000 BLOCK |
| RPC URL | http://64.52.80.204:8545 |

## 🎨 Design & Branding

### Theme
- **Dark mode by default** with optional light mode toggle
- Primary color: `#6366f1` (indigo/purple)
- Accent color: `#22d3ee` (cyan)
- Background: `#0f0f14` (near-black)
- Card background: `#1a1a24` (dark gray)
- Text: white/gray palette
- Modern, clean, minimalist — inspired by Etherscan + Blockscout but cleaner
- Use monospace fonts for addresses and hashes (`font-mono`)
- Responsive (mobile-friendly)

### Logo & Name
- Display "BLOCK" as the logo text in bold with the primary color
- Tagline below: "Blockchain Explorer"

## 📄 Pages & Features

### 1. Homepage (`/`)
- **Hero section** with:
  - Chain name "BLOCK Explorer"
  - Search bar (prominent) — accepts address, tx hash, or block number
  - Live stats row: Total Blocks | Total Transactions | Total Addresses | Avg Block Time
- **Latest Blocks** card (last 6 blocks) showing:
  - Block number (clickable link to `/block/{number}`)
  - Time ago (e.g., "3s ago")
  - Validator address (shortened, e.g., `0xa9Cd...5BeD`)
  - Transaction count
- **Latest Transactions** card (last 6 txs) showing:
  - Tx hash (shortened, clickable link to `/tx/{hash}`)
  - From → To (shortened addresses, clickable)
  - Value in BLOCK
  - Time ago
- Auto-refresh every 5 seconds for live data

### 2. Blocks List (`/blocks`)
- Paginated table of all blocks (50 per page)
- Columns: Block # | Age | Validator | Txns | Gas Used
- Click block number → go to block detail page
- Fetch from Blockscout API: `GET /api/v2/blocks?type=block`

### 3. Block Detail (`/block/{number}`)
- Block header info: number, hash, parent hash, timestamp, validator, gas used/limit, size
- List of transactions in this block
- Fetch block via RPC: `eth_getBlockByNumber` (with `true` for full tx objects)

### 4. Transactions List (`/txs`)
- Paginated table of all transactions
- Columns: Tx Hash | Method | Block | Age | From | To | Value | Fee
- Fetch from Blockscout API: `GET /api/v2/transactions`

### 5. Transaction Detail (`/tx/{hash}`)
- Full transaction info: hash, status (success/fail), block, timestamp, from, to, value, gas price, gas used, input data
- Show a green checkmark for success, red X for failed
- Fetch from Blockscout API: `GET /api/v2/transactions/{hash}`

### 6. Address Detail (`/address/{hash}`)
- Address balance in BLOCK (formatted with commas, e.g., "2,999,970,000 BLOCK")
- Transaction history table for this address
- Show "From" and "To" with direction indicators (IN/OUT)
- Balance fetched via RPC: `eth_getBalance`
- Tx history from Blockscout: `GET /api/v2/addresses/{hash}/transactions`

### 7. Search Results (`/search?q={query}`)
- If query is 42-char hex (0x + 40) → treat as address → redirect to `/address/{query}`
- If query is 66-char hex (0x + 64) → treat as tx hash → redirect to `/tx/{query}`
- If query is a number → treat as block number → redirect to `/block/{query}`
- Otherwise, call Blockscout search API: `GET /api/v2/search?q={query}`

### 8. Top Accounts (`/accounts`)
- Table of addresses sorted by balance (descending)
- Columns: Rank | Address | Balance | % of Supply | Tx Count
- Fetch from Blockscout: `GET /api/v2/addresses?sort=balance&order=desc`

## 🔧 Technical Implementation

### RPC Helper (ethers.js or raw fetch)
Use `ethers.js` v6 or raw fetch for RPC calls:

```javascript
// Configuration
const RPC_URL = 'http://64.52.80.204:8545';
const BLOCKSCOUT_API = 'http://64.111.92.74/api/v2';
const CHAIN_ID = 9999;
const CURRENCY = 'BLOCK';
const DECIMALS = 18;

// RPC call helper
async function rpcCall(method, params = []) {
  const res = await fetch(RPC_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ jsonrpc: '2.0', method, params, id: 1 })
  });
  const data = await res.json();
  return data.result;
}

// Blockscout API helper
async function blockscoutGet(path) {
  const res = await fetch(`${BLOCKSCOUT_API}${path}`);
  return res.json();
}

// Useful RPC methods:
// eth_blockNumber → latest block number (hex)
// eth_getBlockByNumber(blockNum, true) → block with txs
// eth_getTransactionByHash(txHash) → single tx
// eth_getBalance(address, "latest") → balance in wei (hex)
// eth_gasPrice → current gas price (hex)
// net_peerCount → number of peers (hex)
```

### Formatting Helpers
```javascript
// Convert wei (hex or BigInt) to BLOCK with proper decimals
function weiToBlock(weiHex) {
  const wei = BigInt(weiHex);
  const block = Number(wei) / 1e18;
  return block.toLocaleString('en-US', { maximumFractionDigits: 4 });
}

// Shorten address: 0xa9Cd0D1a193Ccb1ba01035CF32012C2f28D35BeD → 0xa9Cd...5BeD
function shortenAddress(addr) {
  return addr ? `${addr.slice(0, 6)}...${addr.slice(-4)}` : '';
}

// Shorten tx hash
function shortenHash(hash) {
  return hash ? `${hash.slice(0, 10)}...${hash.slice(-6)}` : '';
}

// Convert hex to number
function hexToNum(hex) {
  return parseInt(hex, 16);
}

// Time ago
function timeAgo(timestamp) {
  const seconds = Math.floor(Date.now() / 1000 - timestamp);
  if (seconds < 60) return `${seconds}s ago`;
  if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
  if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
  return `${Math.floor(seconds / 86400)}d ago`;
}
```

### "Add BLOCK to MetaMask" Button
Include a button that calls:
```javascript
async function addToMetaMask() {
  await window.ethereum.request({
    method: 'wallet_addEthereumChain',
    params: [{
      chainId: '0x270F', // 9999 in hex
      chainName: 'BLOCK',
      nativeCurrency: { name: 'BLOCK', symbol: 'BLOCK', decimals: 18 },
      rpcUrls: ['http://64.52.80.204:8545'],
      blockExplorerUrls: [window.location.origin]
    }]
  });
}
```

## 🔗 Footer

Include these links in the footer:

- 📱 **Telegram**: https://t.me/bl0ck_official
- 🐦 **Twitter / X**: https://x.com/bl0ck_gg
- 🎵 **TikTok**: https://www.tiktok.com/@bl0ck_official

Also include:
- "Powered by BLOCK" text
- Current year copyright
- Link to "Add BLOCK to MetaMask"

## 🛠️ Tech Stack Preferences

- React + TypeScript
- Tailwind CSS for styling
- React Router for navigation
- React Query or SWR for data fetching with auto-refresh
- ethers.js v6 for RPC interaction (optional, raw fetch works too)
- Lucide React icons

## ⚙️ Environment Variables

Make these configurable (so the user can change RPC URL later):
```
VITE_RPC_URL=http://64.52.80.204:8545
VITE_BLOCKSCOUT_API=http://64.111.92.74/api/v2
VITE_CHAIN_ID=9999
VITE_CHAIN_NAME=BLOCK
VITE_CURRENCY_SYMBOL=BLOCK
VITE_CURRENCY_DECIMALS=18
```

## Summary

Build a beautiful, modern, dark-themed blockchain explorer for the BLOCK L1 chain. It should fetch real-time data from the RPC endpoint and indexed data from the Blockscout API. Include homepage with live stats, blocks list, transactions list, address pages, search, top holders, and an "Add to MetaMask" button. Make it fast, responsive, and professionally branded.
