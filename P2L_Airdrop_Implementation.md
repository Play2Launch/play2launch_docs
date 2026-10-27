# Play to Launch airdrop implementation

Each Play to Launch campaign has one immutable distribution and its own `P2LAirdrop` contract. Existing IDO contracts and flows are separate. All four projects were edited on main.

## Deployment and setup

1. Record a dedicated authorization signing key in the existing encrypted signer registry (admin campaign airdrop tab). The server needs the existing `ido.signer_encryption_key` configured. This key signs claim authorizations only, not owner transactions.
2. Deploy `src/P2LAirdrop.sol` with `(owner, campaignHash, token, signer)`. The campaign's exact hash is shown in the admin airdrop tab. Owner is the administrator's transaction wallet; signer is the address of the encrypted signing record. They should be separate. Tokens with transfer fees or rebasing are not supported.
3. Save the network, deployed contract address, deployment block, token address, signer record, total token amount and optional deadline. The backend reads and verifies the contract's version, campaign, signer, token, symbol and decimals. Configuration does not yet lock scores or open claims.
4. Preview the distribution, review client-reported scores when applicable, then choose **确认结算并锁分**. The transaction freezes activity scoring, captures per-user best valid scores, and stores the immutable root, integer allocations, proofs and audit record. Bots, excluded scores, suspended/restricted accounts and zero scores do not enter the denominator. All users are deduplicated across teams. No wallet is required at settlement.
5. Use the connected owner wallet to **同步分配到合约**, **补足合约 Token**, and **开放领取**, in that order. Each action waits for the configured chain confirmation count. Retrying cannot replace the root or create a second distribution. These are explicit wallet transactions; no extra application confirmation modal is added.
6. Refresh chain status and **同步领取与回收记录** to read confirmed logs starting at the configured deployment block. Synchronization uses resumable 2000-block windows, and the UI processes up to 20 windows per click. Events are deduplicated and include claim IDs/wallets, recovery recipients, amounts, transaction hashes and block numbers. The recent 200 events are displayed; full allocations can be exported as JSON.

The repository changes do not deploy contracts, fund them or modify live activity records automatically.

## Settlement and play after locking

Rule `score_share_v1`: `floor(totalBaseUnits * personalBest / sumOfEligiblePersonalBests)`. All amounts use integer base units. The remainder is displayed and is not allocated. Zero total score and a distribution where all allocations round to zero are rejected.

Campaign writes serialize score acceptance and snapshot creation through the existing participation transaction fence. Late results cannot increase activity scores. Schedule, scoring rules and game binding cannot be changed after locking. Arcade games continue with an ordinary non-campaign session; existing external trials and built-in practice remain available. The snapshot never recalculates after moderation or wallet changes. Newly suspended users cannot obtain claim authorizations; already issued short-lived authorizations remain valid until expiry unless the owner pauses or closes the contract.

The first implementation has an explicit 10,000 eligible-account limit for a single atomic settlement; exceeding it fails without locking or partially issuing allocations. A staged batch-settlement implementation is required before using this version for larger campaigns.

## Binding and claiming

The public page shows pool, final score, final allocation, claim state and deadline/recovery policy. Wallet ownership uses a server nonce, campaign/contract/chain-specific message and five-minute challenge. The server consumes the challenge in a transaction and binds the wallet permanently for that distribution. The current implementation verifies EOA personal signatures; smart-contract account signature binding is not supported.

Claims use a double-hashed leaf `(claimId, amount)` and sorted-pair Merkle proofs. The EIP-712 authorization also includes the root, receiver and ten-minute expiry; the domain binds chain and contract. The digest is computed locally by the server, never accepted from an external contract call. The contract requires `msg.sender` to match the signed wallet and records `claimed[claimId]` before transferring. Each account receives its entire allocation once, regardless of subsequent wallet connections. The UI waits for the configured confirmations and also reloads live claim state.

## Deadline and recovery

Deadline is optional. Zero means no automatic expiry, but the owner can recover unclaimed tokens at any time. With a deadline, recovery is allowed only once that deadline is reached. An owner may extend the deadline or remove it; it cannot be shortened or added to a previously unlimited distribution. Recovery transfers the balance to the selected nonzero external recipient, permanently closes the distribution and emits `Recovered`. A closed distribution cannot reopen, replace its root or resume claims. Repeated recovery may collect tokens accidentally transferred after closure.

The public page explicitly states the no-deadline administrator recovery policy. All receipt/claim/recovery authority comes from chain state, not browser flags.

## Verification

- Solidity tests cover one-time claims, wrong wallets/amounts, replay across contracts, insufficient funding, pause/ownership, deadline extension, recovery with/without deadline and fuzzed two-user claims.
- Go tests verify deterministic Merkle allocation/proofs/rounding and an ethers-generated EIP-712 vector.
- `P2L_AIRDROP_INTEGRATION=1 go test ./airdropcenter` uses a dedicated temporary local Mongo replica-set database and verifies the score fence, eligible totals, immutable settlement, wallet binding and nonce replay rejection.
- UI tests cover final allocation display, administrator recovery disclosure, and removal of claim actions after recovery or a prior claim.
