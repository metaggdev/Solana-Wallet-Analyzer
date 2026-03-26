# Solana Wallet Analyzer (Token Deployment & Volume)

This repository contains a **wallet analyzer** script for Solana. It processes a list of wallet addresses, fetches tokens created by each wallet, and categorizes the wallet based on whether the wallet’s **most recently created** token has the **highest all-time trading volume** (with API-based fallbacks and rate limiting).

## Mission
Help analysts quickly understand token deployment behavior by running repeatable, local “wallet analyzer” jobs:

- For each Solana wallet, identify tokens the wallet created
- Determine the most recent token by creation timestamp
- Compare its volume against the wallet’s other tokens using all-time volume data
- Output two CSVs for wallets where the most recent token is (or isn’t) the highest-volume one

## Who can use this

- Crypto researchers and traders who want a practical Solana wallet analyzer workflow
- On-chain analysts who need a batch job to classify wallets by token performance
- Anyone with API keys and a list of Solana addresses who wants automated, resumable CSV outputs

## Requirements

### Environment / API keys
This script calls external APIs (Helius, Bitquery, Birdeye, DexScreener). You must provide API keys.

1. Copy the template:
   - `\.env.sample` -> `\.env`
2. Fill these variables in `\.env`:
   - `HELIUS_API_KEY`
   - `BIRDEYE_API_KEY`
   - `BITQUERY_API_KEY`
3. (Optional) You can also set `HELIUS_RPC_URL` directly, but it’s derived from `HELIUS_API_KEY` by default.

**Do not commit `.env`** (it’s ignored by git).

### Input file
Create `wallets.txt` in the project folder with **one Solana wallet address per line**.

Example:
```
7xKXtg2CW87d97TXJSDpbD5jBkheTqA83TZRuJosgAsU
9WzDXwBbmkg8ZTbNMqUxvQRAyrZzDsGYdLVL9zYtAWWM
```

### Dependencies
The project uses Python and the `requests` library.

## How to run (detailed)

### Windows (Python script)
1. Install Python (if needed)
   - Download: https://python.org/downloads/
   - Ensure you select **“Add Python to PATH”**.
2. Open Command Prompt and go to the project folder:
   ```cmd
   cd C:\path\to\wallet-analyzer
   ```
3. Install dependencies:
   ```cmd
   pip install -r requirements.txt
   ```
4. Create/configure:
   - `wallets.txt`
   - `.env` (from `.env.sample`)
5. Run:
   ```cmd
   python main.py
   ```

### Run as an `.exe` (optional)
1. Install PyInstaller:
   ```cmd
   pip install pyinstaller
   ```
2. Build the executable:
   ```cmd
   pyinstaller --onefile --name WalletAnalyzer main.py
   ```
3. Use:
   - Put `wallets.txt` and `.env` in the same folder as the exe (or run from the project folder).
   - Double-click `WalletAnalyzer.exe`.

## How it works
For each wallet address:

1. Fetch tokens **created by** the wallet (Helius)
2. Identify the **most recent** token by creation time
3. Fetch **all-time volume** (Bitquery), with Birdeye as fallback for 24h volume, and DexScreener as an extra fallback
4. Compute whether the most recent token is the wallet’s **highest-volume** token
5. Write a row to:
   - `wallets_with_highest_recent_volume.csv`
   - `wallets_without_highest_recent_volume.csv`

## Output files

| File | Description |
|------|-------------|
| `wallets_with_highest_recent_volume.csv` | Wallets where the most recent token has the highest volume |
| `wallets_without_highest_recent_volume.csv` | Wallets where the most recent token does NOT have the highest volume |
| `processed_wallets.log` | Checkpoint of processed wallets (resume support) |
| `failed_wallets.log` | Wallets that failed with an error message |
| `summary_report.txt` | Final summary report for the run |
| `volume_debug.log` | Diagnostic logging for volume/fetch issues (best for troubleshooting) |

## CSV columns

- `wallet_address` - Wallet that created tokens
- `total_tokens_created` - Total number of tokens created by the wallet
- `most_recent_token` - Address of the most recently created token
- `most_recent_token_symbol` - Symbol of the most recent token
- `recent_token_volume_alltime` - All-time volume of the most recent token
- `highest_volume_token` - Token with the highest volume (among tokens created)
- `highest_volume_amount` - Highest volume amount
- `all_tokens_data` - Summary of the top 5 tokens by volume

## Resume feature
If you stop the script (Ctrl+C) or it crashes:

- `processed_wallets.log` is updated as wallets complete
- Re-running will automatically skip wallets already processed

## Troubleshooting

- `No tokens found`: the wallet did not create tokens (or API returned none)
- Rate limiting: the script retries with backoff; you may need to reduce batch size
- Crashes: re-run; resume should pick up from the last processed wallet

## Configuration (via `.env`)
Edit these in `.env` if needed:

- `RATE_LIMIT_DELAY`
- `MAX_RETRIES`
- `REQUEST_TIMEOUT`
- `MAX_WORKERS`
- `BITQUERY_DELAY`

For input/output file names:

- `INPUT_FILE`
- `PROCESSED_LOG`
- `FAILED_LOG`
- `HIGH_VOLUME_CSV`
- `LOW_VOLUME_CSV`
- `SUMMARY_FILE`
- `VOLUME_DEBUG_LOG`
