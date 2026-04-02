# Smart Contract Production Checklist (Solidity / EVM)

---

## 🔴 CRITICAL (deployment-blockers)

### Audit & Security Review

- [ ] Third-party security audit completed by a reputable firm (Trail of Bits, OpenZeppelin, Cyfrin, Sherlock, Spearbit, Consensys Diligence)
- [ ] All Critical and High severity audit findings resolved and re-verified before mainnet deployment
- [ ] Slither static analysis run — all findings reviewed, false positives documented, real issues fixed
- [ ] Mythril or MythX deep analysis run — no unresolved critical findings
- [ ] Reentrancy protection verified: follow Checks-Effects-Interactions pattern AND use `ReentrancyGuard` on all external call paths
- [ ] No unchecked arithmetic in critical calculations (Solidity < 0.8.x: use SafeMath everywhere; ≥ 0.8.x: verify `unchecked` blocks are safe)
- [ ] Access control verified on ALL privileged functions — no unprotected `onlyOwner`/admin functions (use OpenZeppelin `Ownable2Step` or `AccessControl`)
- [ ] No hardcoded private keys, deployer addresses, or secrets anywhere in contract code or deployment scripts
- [ ] No `selfdestruct` / `delegatecall` to untrusted addresses — both are common exploit vectors
- [ ] All external calls to untrusted contracts are treated as potentially malicious (no assumptions about return values)
- [ ] `tx.origin` never used for authorization — only `msg.sender` (tx.origin enables phishing attacks)

### Testing

- [ ] Unit tests cover ALL public and external functions — minimum 95% line coverage, 90% branch coverage
- [ ] Integration tests cover all cross-contract interactions, including interactions with external protocols
- [ ] Fuzz testing run with meaningful corpus: Foundry `forge fuzz` (≥ 10,000 runs) or Echidna with custom property tests
- [ ] Edge cases explicitly tested: zero value transfers, zero address, max uint256, empty arrays, reentrancy attempts
- [ ] Fork tests run against live mainnet state to verify integration with deployed contracts (Hardhat fork / Foundry `--fork-url`)
- [ ] Invariant tests defined and passing: total supply consistency, balance sum = total supply, no unauthorized minting
- [ ] Gas limits tested: no function exceeds block gas limit under worst-case input

### Deployment Configuration

- [ ] Contract deployed and fully tested on public testnet first (Sepolia, Base Sepolia, Arbitrum Sepolia) — identical bytecode
- [ ] Exact same bytecode deployed to mainnet as what was audited — verify with `diff` on compilation artifacts
- [ ] Constructor arguments and initialization parameters double-checked — these are permanent and immutable once deployed
- [ ] Multi-sig wallet (Safe, formerly Gnosis Safe) controls all admin/owner functions — never a single EOA in production
- [ ] Emergency pause mechanism implemented via OpenZeppelin `Pausable` on critical contracts (mint, transfer, withdraw)
- [ ] Upgrade proxy pattern (if used) reviewed for storage collision: verify storage layout compatibility with `forge inspect` or Hardhat storage layout plugin
- [ ] Deployment scripts are deterministic and reproducible — use `CREATE2` for predictable addresses if cross-chain deployment needed
- [ ] Deployer wallet has sufficient gas but is NOT the permanent admin — transfer ownership to multi-sig immediately after deploy

---

## 🟡 IMPORTANT

### Code Quality & Standards

- [ ] Compiler version pinned exactly: `pragma solidity 0.8.24;` — never `^0.8.0` or floating ranges in production
- [ ] Latest stable Solidity compiler used (check solidity releases for security patches)
- [ ] OpenZeppelin Contracts library used for standard implementations: ERC-20, ERC-721, ERC-1155, AccessControl, Pausable
- [ ] No unused variables, imports, dead code, or unreachable branches — clean compilation output
- [ ] NatSpec documentation (`@notice`, `@param`, `@return`, `@dev`) on all public and external functions
- [ ] Code compiles with zero warnings — treat warnings as errors in CI (`--warnings-as-errors` in Foundry)
- [ ] Functions use most restrictive visibility: `external` over `public` when not called internally (gas savings + clarity)
- [ ] `assembly` / inline Yul blocks minimized and each accompanied by a safety comment explaining why it's necessary
- [ ] Custom errors used instead of require strings for gas efficiency (`error InsufficientBalance()` instead of `require(balance >= amount, "insufficient")`)
- [ ] All magic numbers extracted to named constants (e.g., `uint256 constant MAX_SUPPLY = 10_000;`)

### Gas Optimization

- [ ] Gas profiling done: `forge test --gas-report` or Hardhat gas reporter — no unexpected high-cost functions
- [ ] Storage variables packed into 32-byte slots where possible (e.g., group `uint128 + uint128` together, `address + uint96`)
- [ ] `uint256` used by default — smaller uints (`uint8`, `uint128`) only when deliberately packing storage slots
- [ ] Loops have bounded iteration with explicit max — no unbounded loops over dynamic storage arrays (DoS vector)
- [ ] Events emitted for all significant state changes to enable off-chain indexing without polling
- [ ] `calldata` used instead of `memory` for external function parameters that aren't modified
- [ ] Mappings preferred over arrays for lookups — O(1) vs O(n) gas cost
- [ ] Batch operations provided where users may need to call the same function multiple times (batch mint, batch transfer)

### Operational Security

- [ ] Admin key management plan documented and implemented: hardware wallet (Ledger/Trezor), MPC wallet, or multi-sig — never a hot wallet
- [ ] Timelock contract deployed for privileged operations — minimum 24-48h delay so community can react to malicious governance (OpenZeppelin TimelockController)
- [ ] Oracle price feeds use TWAP or Chainlink price feeds with staleness checks — never spot price from a single DEX (manipulation risk)
- [ ] Front-running attack vectors assessed: use commit-reveal schemes, private mempools (Flashbots Protect), or batch auctions where needed
- [ ] Flash loan attack vectors assessed: no single-transaction price manipulation possible on key operations
- [ ] MEV protection considered for user-facing transactions: Flashbots Protect RPC or MEV-resistant design patterns
- [ ] Protocol TVL limit / deposit cap set for initial launch — start small and increase as confidence builds
- [ ] Emergency contacts documented: who can trigger pause, how to reach multi-sig signers, escalation for exploits

### Documentation & Communication

- [ ] Technical documentation published: architecture diagram, state machine, contract interaction flow, trust assumptions
- [ ] All external dependencies documented with addresses and versions: oracles, other protocols, bridges, token contracts
- [ ] Known limitations, risks, and trust assumptions disclosed publicly (user-facing risk documentation)
- [ ] Bug bounty program live BEFORE mainnet launch — rewards commensurate with TVL (Immunefi, HackerOne, Code4rena)
- [ ] Deployment addresses published in a verified registry (GitHub README, docs site, Etherscan verification)
- [ ] Upgrade/governance process documented publicly so users understand who can change what

---

## 🟢 NICE-TO-HAVE

- [ ] Formal verification run for invariant-critical logic (Certora Prover, Halmos, KEVM) — especially for DeFi core accounting
- [ ] Source code verified on Etherscan/Basescan AND Sourcify immediately after deployment
- [ ] Subgraph deployed for efficient off-chain indexing and querying (The Graph — hosted or decentralized)
- [ ] Real-time monitoring configured for anomalous on-chain activity: large transfers, admin function calls, unusual patterns (OpenZeppelin Defender, Forta, Tenderly)
- [ ] Token distribution / vesting contracts audited separately from core protocol (different risk profile)
- [ ] DAO governance contracts reviewed for vote manipulation: flash loan governance attacks, vote buying, low-quorum exploits
- [ ] Cross-chain bridge contracts receive independent bridge-specific security review (bridges are highest-risk category)
- [ ] Disaster recovery plan documented: what happens if admin key is compromised, if oracle fails, if critical bug is found post-launch
- [ ] War room / incident response drill conducted: simulate a security incident and practice the response process
- [ ] Gas sponsorship / meta-transactions implemented for better UX (ERC-2771, Account Abstraction ERC-4337)
- [ ] Contract supports ERC-165 `supportsInterface` for composability with other protocols
- [ ] Multi-chain deployment plan documented with chain-specific considerations (L1 vs L2 gas, sequencer downtime, bridge finality)
