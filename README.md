# Nexus Onchain Wallet Connect

**Developed by [Switch-900](https://github.com/switch-900/nexus-ocw)**

A universal Bitcoin wallet connector supporting 8 major wallets with a single `signPsbt()` interface. Deployed entirely as Bitcoin inscriptions -- immutable, decentralized, zero dependencies. Also works as a local library for bundler-based projects.

**[Live on Bitcoin](https://ordinals.com/content/7cf63b82b244d41121ef823a6532705cd25257d7420a405daa388394de5b529ei0)** | **Used in production by MyTXO**

---

## Why Nexus?

- **One interface, 8 wallets** -- `signPsbt()`, `sendBitcoin()`, `signMessage()`, and 60+ more methods work identically across UniSat, Xverse, OKX, Leather, Phantom, Wizz, Magic Eden, and Oyl
- **On-chain first** -- The entire library lives as Bitcoin inscriptions. Load it from a SAT reference -- no npm, no CDN, no servers
- **Local-ready** -- Copy the provider files into your project and import directly. No build chain dependency
- **Modular architecture** -- Each wallet provider is a separate inscription, updatable independently
- **Normalized data** -- Consistent response formats regardless of which wallet is connected

## Supported Wallets

UniSat | Xverse | OKX | Leather | Phantom | Wizz | Magic Eden | Oyl

---

## Two Ways to Use

### Option A: Load from Bitcoin (On-Chain)

Load the library directly from its Bitcoin inscription. No install, no build process.

```html
<script type="module">
  import NWC from '/r/sat/650232570297610/at/-1/content';
  window.NexusWalletConnect = NWC;
</script>
```

### Option B: Local / Bundler

Copy the provider files into your project and import them directly. This is how [MyTXO](https://mytxo.space) uses Nexus in production.

```
your-project/
  lib/wallet/nexus/
    loader.js              # Main SDK orchestrator
    providers/
      01-base-provider.js
      02-normalizers.js
      03-wallet-connector.js
      04-unisat-provider.js
      ... (04-11 wallet providers)
```

```javascript
import NWC from './lib/wallet/nexus/loader.js';
```

Both paths expose the same `NexusWalletConnect` API.

---

## Quick Start

```javascript
const NWC = window.NexusWalletConnect; // or import from local

// 1. Detect installed wallets
const wallets = NWC.detectWallets();
console.log('Available:', wallets.map(w => w.name));

// 2. Connect
await NWC.connect('UniSat');
const state = NWC.getState();
console.log(state.address, state.balance, 'BTC');

// 3. Sign a PSBT
const signed = await NWC.signPsbt(psbtHex);
const txId = await NWC.pushPsbt(signed);

// 4. Disconnect
await NWC.disconnect();
```

---

## Wallet Compatibility Matrix

### Core Capabilities

| Wallet | Balance | Inscriptions | Sign PSBT | Send BTC | Sign Message | Push PSBT | Get Public Key | Create Inscription |
|--------|---------|--------------|-----------|----------|--------------|-----------|----------------|-------------------|
| **UniSat** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | -- |
| **Xverse** | Yes | Yes | Yes | Yes | Yes | -- | Yes | Yes |
| **OKX** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| **Leather** | -- | -- | Yes | Yes | Yes | Yes | -- | -- |
| **Phantom** | Yes | Yes | Yes | Yes | Yes | -- | Yes | -- |
| **Wizz** | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| **Oyl** | Yes | Yes | Yes | Yes | Yes | -- | -- | -- |
| **Magic Eden** | -- | -- | Yes | Yes | Yes | -- | -- | -- |

### Advanced Features

| Feature | UniSat | Xverse | OKX | Leather | Phantom | Wizz | Oyl | Magic Eden |
|---------|--------|--------|-----|---------|---------|------|-----|------------|
| **Runes Support** | -- | Yes | -- | -- | -- | Yes | -- | -- |
| **BRC-20 Tokens** | Yes | -- | Yes | -- | -- | Yes | -- | -- |
| **Stacks Support** | -- | -- | -- | Yes | -- | -- | -- | -- |
| **Multi-Address** | -- | Yes | -- | -- | -- | -- | -- | -- |
| **Batch Operations** | -- | Yes | -- | -- | -- | -- | -- | Yes |
| **Hardware Wallet** | -- | -- | -- | -- | -- | -- | -- | Yes |
| **NFT Support** | Yes | Yes | Yes | -- | Yes | Yes | Yes | Yes |
| **UTXO Access** | Yes | -- | Partial | -- | -- | Yes | -- | -- |
| **Capabilities API** | -- | Yes | -- | -- | -- | -- | -- | -- |
| **BRC-20 Listing** | Yes | -- | Yes | -- | -- | -- | -- | -- |

---

## API Reference

### Core Connection Functions

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `connect(walletName)` | Connect to a specific wallet | `Promise<Provider>` | Auto-detects and connects |
| `disconnect()` | Disconnect current wallet | `Promise<void>` | Clears all state |
| `detectWallets()` | Find all installed wallets | `Array<Object>` | Returns wallet info + detection |
| `getState()` | Get current connection state | `Object` | `{isConnected, walletType, address, balance, provider}` |
| `subscribe(callback)` | Subscribe to state changes | `Function` | Returns unsubscribe function |
| `getCurrentProvider()` | Get active provider instance | `Provider\|null` | Direct access to provider |

### Wallet Information

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `getWalletInfo(name)` | Get wallet metadata | `Object` | Features, download URL, detection |
| `getAllWalletInfo()` | Get all wallet metadata | `Object` | Complete wallet registry |
| `getWalletFeatures()` | Get current wallet features | `Object` | Provider-specific capabilities |

### Basic Wallet Operations

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `getBalance()` | Get wallet balance | `Promise<number>` | Always returns BTC (not satoshis) |
| `getAddress()` | Get current address | `Promise<string>` | Primary wallet address |
| `getPublicKey()` | Get public key | `Promise<string>` | If supported by wallet |
| `getAccounts()` | Get all accounts | `Promise<Array>` | Multi-account wallets |
| `getNetwork()` | Get current network | `Promise<string>` | `'mainnet'`, `'testnet'`, etc. |
| `switchNetwork(network)` | Switch networks | `Promise<void>` | If supported |

### Transaction Operations

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `signPsbt(psbt, options)` | Sign a PSBT | `Promise<string>` | Returns signed PSBT hex |
| `signPsbts(psbts)` | Sign multiple PSBTs | `Promise<Array>` | Batch signing |
| `sendBitcoin(to, amount)` | Send Bitcoin | `Promise<string>` | Returns transaction ID |
| `sendBTC(to, amount)` | Alias for sendBitcoin | `Promise<string>` | Same as sendBitcoin |
| `pushPsbt(psbt)` | Broadcast signed PSBT | `Promise<string>` | Returns transaction ID |
| `pushTx(txHex)` | Broadcast raw transaction | `Promise<string>` | Returns transaction ID |
| `signMessage(message)` | Sign arbitrary message | `Promise<string>` | Returns signature |

### Inscription Operations

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `getInscriptions(offset, limit)` | Get inscriptions paginated | `Promise<Array>` | Normalized inscription objects |
| `getAllInscriptions()` | Get all inscriptions | `Promise<Array>` | Auto-handles pagination |
| `sendInscription(to, inscriptionId)` | Send an inscription | `Promise<string>` | Returns transaction ID |
| `createInscription(data)` | Create new inscription | `Promise<Object>` | Xverse, Wizz only |
| `inscribe(content, options)` | Inscribe content | `Promise<Object>` | Generic inscribe method |

### Xverse-Specific Functions

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `getAddresses(purposes)` | Get addresses by purpose | `Promise<Array>` | `['payment', 'ordinals', 'stacks']` |
| `createRepeatInscriptions(data)` | Batch inscriptions | `Promise<Object>` | Repeat same content multiple times |
| `sendInscriptions(params)` | Send multiple inscriptions | `Promise<string>` | Batch transfer |
| `getRunesBalance()` | Get Runes balance | `Promise<Object>` | Runes token balances |
| `transferRunes(params)` | Transfer Runes | `Promise<Object>` | `{recipient, runeName, amount}` |
| `mintRunes(params)` | Mint Runes | `Promise<Object>` | `{runeName}` |
| `etchRunes(params)` | Create new Rune | `Promise<Object>` | `{runeName, symbol, divisibility, premine}` |
| `getRunesOrder(orderId)` | Check Runes order status | `Promise<Object>` | Order tracking |
| `signMultipleTransactions(psbts)` | Sign multiple transactions | `Promise<Array>` | Batch operations |

### Leather-Specific Functions (Stacks)

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `getProductInfo()` | Get wallet product info | `Promise<Object>` | Leather version, features |
| `getURL()` | Get wallet URL | `Promise<string>` | Wallet app URL |
| `signStructuredData(data)` | Sign structured data | `Promise<Object>` | Stacks structured signing |
| `authenticate(options)` | Authenticate with wallet | `Promise<Object>` | Stacks authentication |
| `sendStacksTransaction(txOptions)` | Send Stacks transaction | `Promise<Object>` | Stacks network |
| `updateProfile(profileData)` | Update user profile | `Promise<Object>` | Profile management |

### Magic Eden-Specific Functions

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `isHardware()` | Check if hardware wallet | `Promise<boolean>` | Ledger, Trezor detection |
| `call(method, params)` | Call Magic Eden RPC | `Promise<*>` | Direct RPC access |
| `signMultipleTransactions(psbts)` | Sign multiple transactions | `Promise<Array>` | Batch operations |

### OKX-Specific Functions

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `inscribeTransfer(ticker, amount)` | Inscribe BRC-20 transfer | `Promise<Object>` | BRC-20 operations |
| `splitUtxo(options)` | Split UTXO | `Promise<string>` | UTXO management |
| `transferNft(to, inscriptionId)` | Transfer NFT | `Promise<string>` | NFT operations |
| `watchAsset(asset)` | Add asset to watch list | `Promise<boolean>` | Asset tracking |
| `mint(mintData)` | Mint tokens | `Promise<Object>` | Generic minting |
| `createInscription(data)` | Create inscription | `Promise<Object>` | OKX inscription creation |

### UniSat-Specific Functions

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `inscribeTransfer(ticker, amount)` | Inscribe BRC-20 transfer | `Promise<Object>` | BRC-20 operations |

### Utility Functions

| Function | Description | Returns | Notes |
|----------|-------------|---------|-------|
| `getCapabilities()` | Get wallet capabilities | `Promise<Object>` | Feature matrix for current wallet |
| `getUtxos()` | Get UTXOs | `Promise<Array>` | UniSat/Wizz only |
| `getBRC20List()` | Get BRC-20 tokens | `Promise<Array>` | Token holdings |
| `loadNormalizers()` | Load data normalizers | `Promise<Object>` | Format conversion utilities |
| `loadWalletConnector()` | Load wallet connector utils | `Promise<Object>` | Advanced connection utilities |
| `createProvider(walletName)` | Create provider instance | `Provider` | Direct provider creation |

---

## On-Chain Inscription Architecture

The entire Nexus SDK is deployed as modular Bitcoin inscriptions. Each wallet provider is its own inscription, allowing independent updates without re-inscribing the whole library.

### Core Library Inscriptions

| Module | SAT Number | Description |
|--------|-----------|-------------|
| **Base Provider** | `1408319431385218` | Core wallet interface |
| **Normalizers** | `1408319431385764` | Data formatting utilities |
| **Wallet Connector** | `1180016128405661` | Connection management |
| **UniSat Provider** | `1180016128407426` | UniSat wallet support |
| **Xverse Provider** | `1180016128407972` | Xverse wallet support |
| **OKX Provider** | `1180016128408518` | OKX wallet support |
| **Leather Provider** | `1180016128409064` | Leather (Stacks) wallet support |
| **Phantom Provider** | `1180016128409610` | Phantom wallet support |
| **Wizz Provider** | `1180016128410156` | Wizz wallet support |
| **Magic Eden Provider** | `1180016128410702` | Magic Eden wallet support |
| **Oyl Provider** | `1180016128411248` | Oyl wallet support |

### Application Inscriptions

| Component | SAT Number | Description |
|-----------|-----------|-------------|
| **Wallet Loader** | `650232570297610` | Main library loader (12-loader.js) |
| **App Styles** | `650232570299189` | CSS styles (22.71 KB) |
| **Main App** | `650232570299735` | React application bundle (105.97 KB) |
| **HTML Entry** | [View on Ordinals](https://ordinals.com/content/7cf63b82b244d41121ef823a6532705cd25257d7420a405daa388394de5b529ei0) | Entry point |

### How On-Chain Loading Works

```javascript
// Providers load from Bitcoin SAT references
import { BaseWalletProvider } from '/r/sat/1408319431385218/at/-1/content';
import { UniSatProvider } from '/r/sat/1180016128407426/at/-1/content';
import { XverseProvider } from '/r/sat/1180016128407972/at/-1/content';
// ... 8 wallet providers total

// The loader combines everything and exposes window.NexusWalletConnect
```

**Why inscriptions?**

- **Immutable** -- Code can never be changed or taken down
- **Decentralized** -- No servers, CDNs, or dependencies
- **Modular** -- Update individual providers without re-inscribing everything
- **Compressed** -- Brotli compression saves ~85% on-chain space
- **Permanent** -- Lives on Bitcoin forever

### Inscription Deployment Order

1. Inscribe core modules (01-03)
2. Update SAT numbers in code, re-run build, inscribe wallet providers (04-11)
3. Update SAT numbers, re-run build, inscribe loader (12)
4. Deploy frontend bundle

---

## Project Structure

```
nexus-ocw/
├── inscriptions local/           # Source provider files (11 modules)
│   ├── 01-base-provider.js
│   ├── 02-normalizers.js
│   ├── 03-wallet-connector.js
│   ├── 04-unisat-provider.js
│   ├── 05-xverse-provider.js
│   ├── 06-okx-provider.js
│   ├── 07-leather-provider.js
│   ├── 08-phantom-provider.js
│   ├── 09-wizz-provider.js
│   ├── 10-magiceden-provider.js
│   └── 11-oyl-provider.js
│
├── frontend/                     # React testing application
│   ├── App.jsx
│   ├── main.jsx
│   ├── components/
│   │   ├── dev-loader-simple.js  # Loader source (generates 12-loader.js)
│   │   ├── WalletTester.jsx
│   │   ├── XversePanel.jsx
│   │   └── walletCapabilities.js
│   └── styles/
│
├── prepare-inscriptions.js       # Generates inscription-ready files
├── fix-local-imports.js
├── vite.config.js
└── package.json
```

**Source files** (edit these): `inscriptions local/01-11` and `frontend/components/dev-loader-simple.js`

**Generated files** (don't edit): Everything in `ready-to-inscribe/` -- regular, minified, and Brotli-compressed versions.

---

## Development

### Setup

```bash
# Install dependencies
npm install

# Start dev server
npm run dev
# Open http://localhost:5173
```

### Available Scripts

```bash
npm run dev                    # Development server
npm run build                  # Build production frontend
npm run build:core             # Build core library for inscription
npm run preview                # Preview built frontend
npm run prepare-inscriptions   # Generate inscription-ready files
npm run fix-local-imports      # Fix import paths (if needed)
```

### Preparing Inscriptions

```bash
npm run prepare-inscriptions

# Generates ready-to-inscribe/ with:
#   regular/    -- readable source files
#   minified/   -- optimized (44% smaller)
#   compressed/ -- Brotli compressed (91% smaller)
```

For detailed inscription guides, check the generated files:
- `ready-to-inscribe/INSCRIPTION_GUIDE.md`
- `ready-to-inscribe/MANIFEST.json`

### Important

1. Never edit generated files in `ready-to-inscribe/`
2. Always edit source files in `inscriptions local/` and `frontend/components/`
3. SAT references use `/r/sat/XXX/at/-1/content` (not ordinals.com URLs)

---

## Credits

Built by **[Switch-900](https://github.com/switch-900/nexus-ocw)**.

For off-chain web apps that don't need inscription-level permanence, also check out [Laser Eyes](https://www.lasereyes.build/).

---

**Permanent. Decentralized. On Bitcoin.**
