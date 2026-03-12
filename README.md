# PoC — DBXen burnBatch() msg.sender vs _msgSender() Mismatch (Mar 2026)

> **Educational / research use only.** This repository reproduces the
> `burnBatch()` sender mismatch exploit that drained ~$133K (~65.36 ETH)
> from the DBXen protocol fee pool.

## Quick Start

```bash
git clone https://github.com/NomosLabs-Security/poc-dbxen-2026
cd poc-dbxen-2026
forge install foundry-rs/forge-std --no-git
forge test -vvvv
```

## Details

- **Date:** 2026-03-11
- **Chain:** Ethereum
- **Attack TX:** [0x914a5af7...08bc37](https://etherscan.io/tx/0x914a5af790e55b8ea140a79da931fc037cb4c4457704d184ad21f54fb808bc37)
- **Loss:** ~65.36 ETH (~$133K)
- **Expected Output:** Attacker earns burn credits without burning own XEN, drains fee pool

## Vulnerability

DBXen's `burnBatch()` function has an inconsistency between `msg.sender` and
`_msgSender()` (ERC2771 meta-transaction pattern):

1. **XEN token burn:** Uses `msg.sender` — when called via the trusted Forwarder,
   this is the Forwarder's address, so the Forwarder's XEN is burned.
2. **Burn credit assignment:** Uses `_msgSender()` via the `gasWrapper` modifier —
   this resolves to the meta-transaction signer (the attacker), who receives the
   `accCycleBatchesBurned` credits.

The attacker exploited this by:

1. Calling `burnBatch()` via the trusted Forwarder across multiple cycles
2. The Forwarder's XEN tokens were burned (not the attacker's)
3. The attacker (meta-tx signer) accumulated burn credits in `accCycleBatchesBurned`
4. After accumulating ~65.36 ETH worth of credits, the attacker claimed all protocol
   fees in a single transaction and transferred the ETH to their EOA (0x425d)

- **Full Analysis:** [NomosLabs Security Intelligence Archive](https://nomoslabs.io/archive/dbxen-2026)

MIT — NomosLabs Security Research
