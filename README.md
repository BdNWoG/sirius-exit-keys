# Sirius exit-path proving keys

This repository publishes the **Groth16 proving keys for the Sirius exit-path
circuits** so that any user can generate their own exit proof and withdraw from
the Sirius rollup **without the operator's cooperation, servers, or data**.

A proving key is **not a secret**. The "toxic waste" of a Groth16 trusted setup
is the discarded setup randomness, not the proving key. Withholding a proving
key is a distribution choice, never a cryptographic necessity — and withholding
one made a self-custody claim untrue. Hence this repo.

## Which bundle do I want?

| Bundle | Circuit | PIs | Status |
|---|---|---:|---|
| repo root (`exit_claim_*.bin`) | `ExitClaim` v1, 2026-06-23 | 5 | **Live on devnet today.** The deployed verifier PDA accepts only this one. |
| [`exit-claim-v3-2026-07-28/`](exit-claim-v3-2026-07-28) | `ExitClaim` v3, 2026-07-28 | 6 | Baked and verified, **not yet rotated on-chain** (7-day timelock, applies ~2026-08-03). |
| [`force-withdrawal-inclusion-v1-2026-07-28/`](force-withdrawal-inclusion-v1-2026-07-28) | `ForceWithdrawalInclusion`, 2026-07-28 | 6 | Baked and verified, **not yet rotated on-chain**. First bundle of this circuit whose proving key has ever existed. |

**Use the root bundle to exit today.** The 2026-07-28 bundles become the live
ones when the verifier PDAs rotate; until then proofs made with them are
rejected on-chain. Always run the on-chain check below rather than trusting this
table.

### Why there is a v3 at all: the destination-commitment cutover

Before 2026-07-28 the exit destination was fed to the circuit as a single field
element — a raw Solana pubkey read big-endian as a BN254 `Fr`. That is only
canonical for about **18.9%** of Solana addresses, so roughly four users in five
could not name their own wallet as an exit destination. The cutover feeds the
destination as **two 128-bit limbs** instead, which works for every address.

That adds one public input (5 → 6). A Groth16 verifying key's IC array length is
exactly `nr_pubinputs + 1`, so there is no reinterpretation under which the old
6-IC-point key serves the new circuit: **every pre-cutover exit key is
structurally dead**, not merely stale. Hence a new bake rather than a re-pin.

### `ForceWithdrawalInclusion`: the backstop that was decorative

`ForceWithdrawalInclusion` is the censorship-resistance backstop — the circuit
that lets you force a withdrawal when the operator will not process one. Its
verifying key was baked and pinned on-chain in June 2026, but the **matching
proving key was never retained**. No user has ever been able to produce a
force-withdrawal proof, and none could until now. The bundle here is the first
one whose proving key exists and has been shown to work.

## Contents

### Root — `ExitClaim` v1 (2026-06-23), 5 public inputs — LIVE

| File | Bytes | SHA-256 |
|---|---:|---|
| `exit_claim_pk.bin` | 1,944,021 | `e7b27c08233ec46cb387c37de3b5e340d429bae8b4846776b48cffc5ff1b81f9` |
| `exit_claim_vk.bin` | 492 | `b760b38d428d0f3e2aeb46485565b680ce7ac8f4a5f7af698c245d8e59714d62` |
| `exit_claim_r1cs.bin` | 1,103,616 | `20857b456acd1234738b1f351a76ca1da5df5f7bcef3b12b5326fdf0247c3f42` |
| `vk_exit_claim.solana.bin` | 832 | `ad13f9e3a80fb6b2ea133f8dede65b86352669a3a6c024fcbd3ad8133f82fc32` |

Canonical on-chain pin: `66838a20ef9efb66522132bf25b01792d9aa241ed200f2ae84a03b224eb16c94`

### `exit-claim-v3-2026-07-28/` — 6 public inputs, 9,859 constraints

| File | Bytes | SHA-256 |
|---|---:|---|
| `exit_claim_pk.bin` | 2,025,575 | `d57ce984abce4472007140e111db9a60ced62e900483e4bb60967c3db326d207` |
| `exit_claim_vk.bin` | 524 | `e35f67e94119452406ee5a0c1421f93356e9ef2169dd0879454a60d9121bc447` |
| `exit_claim_r1cs.bin` | 1,245,753 | `1afcd48eab877b0cc486735cf44143fb820ce5a9ca6b72f9893f60f9359c3e4b` |
| `vk_exit_claim_v6pi.solana.bin` | 896 | `6aac7328223217b365a0f532760f8f0371b9f4261efdbbb7b1de92f84f14e51a` |

Canonical on-chain pin: `73e539e3357a8f57dc6987880ebd8529eafcce713d7f20d375459ec5d5db786b`

### `force-withdrawal-inclusion-v1-2026-07-28/` — 6 public inputs, 11,642 constraints

| File | Bytes | SHA-256 |
|---|---:|---|
| `force_incl_pk.bin` | 2,206,043 | `dcdf0d0516ce73bd9d0d2cc94b98b535b6eb0949f975344b740b38804859f0e2` |
| `force_incl_vk.bin` | 524 | `49a2ab9eca2dbeeec0a6dcdc790f3bb1b04893e50b07cb159089ec38b97d661b` |
| `force_incl_r1cs.bin` | 1,466,733 | `5e4902d5eba9ff5b9a3143f48ff4053dc72437cd9a4f4838e8892584103ae43d` |
| `vk_force_incl_v6pi.solana.bin` | 896 | `be7663a5000d3e4d06e3b3a29c7aff22501ebdd9606137c0cf3e7e680dd8546c` |

Canonical on-chain pin: `9e064745190603059b6a84f09a93709bfb8ad543654e1583c9fed5b019eff21a`

All `*_pk.bin` / `*_vk.bin` / `*_r1cs.bin` are gnark (`consensys/gnark`, BN254)
native serializations. The `*.solana.bin` files are the same verifying key in
the on-chain wire layout (`alpha_g1(64) | beta_g2(128) | gamma_g2(128) |
delta_g2(128) | IC[i](64)…`, G2 in EIP-197 order).

**Two different hashes, never conflate them.** The tables above give the
**file** SHA-256. The **canonical pin** is a different value —
`sha256([proof_type, nr_pubinputs] ‖ alpha ‖ beta ‖ gamma ‖ delta ‖ IC)` with
the IC array zero-padded to 768 bytes — and it is what the on-chain
`VerifierState` stores.

## Verify what you downloaded

```bash
sha256sum -c SHA256SUMS
```

## Verify this is the key the deployed program actually accepts

Do not trust the tables above, or filenames, or dates. Check it against the
chain. The Solana devnet program `2sz5LgzjAQC9BFz4xPPAR3WbHCsQqU2PECWjmpb18d1H`
stores its `ExitClaim` verifying key in the PDA
`FoneiBweDYm6W9zgnbCcrETemsCo8rmpcHc46atL4S51`
(seeds `["verifier", 0x02]`), and its `ForceWithdrawalInclusion` key in
`D2h6Cqrrdybp9DBhVdqWYcTK568KMAJTwxnHE16kDMBc`. Reconstruct the VK from the
account and compare:

```bash
solana account FoneiBweDYm6W9zgnbCcrETemsCo8rmpcHc46atL4S51 \
  --url https://api.devnet.solana.com --output-file onchain.bin

python3 - <<'PY'
import hashlib
d = open('onchain.bin','rb').read()
nr = d[9]
vk = d[10:458] + d[458:458 + (nr+1)*64]
print('nr_pubinputs      ', nr)
print('on-chain VK sha256', hashlib.sha256(vk).hexdigest())
print('stored vk hash    ', d[1227:1259].hex())
PY
```

`nr_pubinputs` tells you which bundle that PDA currently holds: **5** means the
root (v1) bundle, **6** means the 2026-07-28 bundle. `on-chain VK sha256` must
equal the corresponding `*.solana.bin` hash above. If it matches neither,
**every bundle here is stale for that PDA** — open an issue rather than using
one.

## Verify the proving key really pairs with that verifying key

The strongest check, and the only one that rules out a mismatched setup: make a
real proof with the proving key and verify it under the verifying key. In the
`sirius-main` repo, with the files renamed to `pk.bin` / `vk.bin` / `r1cs.bin`
(the names the tests expect):

```bash
# ExitClaim
go test ./circuit -run TestExitClaim_KeyPairBinding -v \
  -args -exit-claim-keys=/path/to/bundle \
        -exit-claim-solana-vk-sha256=<the .solana.bin hash for that bundle>

# ForceWithdrawalInclusion
go test ./circuit -run TestForceWithdrawalInclusion_KeyPairBinding -v \
  -args -force-withdraw-keys=/path/to/bundle \
        -force-withdraw-solana-vk-sha256=be7663a5000d3e4d06e3b3a29c7aff22501ebdd9606137c0cf3e7e680dd8546c
```

Both run five gates: the r1cs loads, the r1cs matches a fresh compile of the
current circuit source (catching "the circuit moved after the bake"), the keys
load, a real `groth16.Prove` succeeds, and `groth16.Verify` accepts it under the
verifying key. Only the last two prove the two files came from the *same* setup
run. Every bundle published here has passed all five.

## Why this matters / non-reproducibility

gnark's `groth16.Setup` is **unseeded**. These circuits are fully deterministic,
but running the setup yourself produces a *different* proving/verifying key
pair, and the deployed on-chain verifier only accepts proofs under the specific
verifying key it was initialised with. So these exact files cannot be
regenerated by anyone — they can only be preserved and distributed. That is why
they are here, and why they are mirrored.

A third `ExitClaim` bundle exists that is **not** deployed and never was (a
2026-07-20 re-bake; Solana VK `334323d6…`). Proofs made with it are rejected
on-chain. Always run the on-chain check above.

## Circuits

- **`ExitClaimCircuit`** — BN254. v1: 9,337 constraints, 5 public inputs
  (`exit_root`, `account_id`, `shares_withdrawable`, `exit_nonce`,
  `destination`). v3: 9,859 constraints, 6 public inputs, with `destination`
  replaced by `destination_lo` / `destination_hi`.
- **`ForceWithdrawalInclusionCircuit`** — BN254, 11,642 constraints, 6 public
  inputs (`exit_root`, `account_id`, `amount_shares`, `exit_nonce`,
  `destination_lo`, `destination_hi`).

Both take a depth-32 Merkle path as private witness (`Siblings[32]`,
`PathBits[32]`). Proving is cheap: the key loads in well under a second and a
proof takes ~260 ms on one CPU core. No GPU, no large machine.

## Trusted setup

These keys come from a **single-operator** setup, not a multi-party ceremony:
one machine, one process, one party that transiently held the setup randomness.
That party could forge proofs against these verifying keys. The security
argument is operator trust, not cryptography. See
`docs/TRUSTED_SETUP_DISCLOSURE.md` in `sirius-main` for the full statement. A
multi-party ceremony precedes public launch.

## Mirrors

- This repository (canonical)
- `gs://sirius-exit-keys-public/exit-claim/v1-2026-06-23/`
- `gs://sirius-exit-keys-public/exit-claim/v3-2026-07-28/`
- `gs://sirius-exit-keys-public/force-withdrawal-inclusion/v1-2026-07-28/`

(Google Cloud Storage, object versioning enabled.)

## License / status

Published so the exit path is executable by anyone. Provided as-is.
