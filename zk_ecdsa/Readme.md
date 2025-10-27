Perfect! Let’s create a **detailed, comprehensive README** that includes everything — step-by-step instructions, wallet setup, casting addresses, signing, generating inputs, references to all relevant repos, and links. This will be fully beginner-friendly.

---

````markdown
# zk ECDSA Circuit in Noir

This project demonstrates a zero-knowledge ECDSA signature verification circuit using [Noir](https://noir-lang.org/) and `ecrecover-noir`. It allows verifying Ethereum signatures in a ZK-proof.

This README includes detailed steps for setting up your wallet, generating inputs, running the circuit, and producing proofs.

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Project Setup](#project-setup)  
3. [Wallet Setup and `cast` Commands](#wallet-setup-and-cast-commands)  
4. [Generating Inputs](#generating-inputs)  
5. [Generating Prover.toml](#generating-provertoml)  
6. [Compiling and Executing the Circuit](#compiling-and-executing-the-circuit)  
7. [Generating Proofs](#generating-proofs)  
8. [Verification](#verification)  
9. [References](#references)  

---

## Prerequisites

- [Foundry](https://getfoundry.sh/) installed for `cast` commands.  
- [Noir](https://noir-lang.org/) installed (`nargo` CLI).  
- Your Ethereum keystore account ready for signing transactions and messages.  
- Basic knowledge of terminal commands.

---

## Project Setup

1. Clone the project:

```bash
git clone https://github.com/Cyfrin/zk-ecrecover-cu.git
cd zk-ecrecover-cu/circuits
````

2. Check dependencies:

```bash
nargo check
```

This will create a template `Prover.toml` in the `circuits` folder.

---

## Wallet Setup and `cast` Commands

1. **Create or list a wallet account**:

```bash
# Create a new wallet account named "defaultKey"
cast wallet new --account defaultKey

# List all keystore accounts
ls ~/.foundry/keystores/
```

2. **Get your wallet address**:

```bash
cast wallet address --account defaultKey
```

This will output your Ethereum address for signing and verification.

3. **Get your wallet public key**:

```bash
cast wallet public-key --account defaultKey
```

* The **first 32 bytes** are `pub_key_x`.
* The **second 32 bytes** are `pub_key_y`.

4. **Sign a message**:

```bash
# Generate a keccak256 hash of your message
cast keccak "hello"

# Use the resulting hash to sign
cast wallet sign --no-hash --account defaultKey <hash_bytes>
```

---

## Generating Inputs

Create a file named `inputs.txt` in the `circuits` folder. Example:

```toml
expected_address = "0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266"
hashed_message = "0x1c8aff950685c2ed4bc3174f3472287b56d9517b9c948127319a09a7a36deac8"
pub_key_x = "0x8318535b54105d4a7aae60c08fc45f9687181b4fdfc625bd1a753fa7397fed75"
pub_key_y = "0x3547f11ca8696646f2f3acb08e31016afac23e630c5d11f59f61fef57b0d2aa5"
signature = "0x73eebf81a611136662d65778960c853fdcaf6eca86793ed9cabc30f2195937af78a07e601627da5b4cc80c0ab35f6894da19b4a01759d90c101d9c9dd1c6745d1b"
```

> **Note:** All hex values should include `0x`.

---

## Generating Prover.toml

Convert `inputs.txt` to a Noir-compatible `Prover.toml`:

1. Make the script executable:

```bash
chmod +x generate_inputs.sh
```

2. Run the script:

```bash
./generate_inputs.sh
```

This will create `Prover.toml` with:

* `hashed_message`, `pub_key_x`, `pub_key_y`, `signature` as decimal arrays.
* `expected_address` as a hex string.

---

## Compiling and Executing the Circuit

1. **Compile the circuit** (optional if you only want to create a verifier later):

```bash
nargo compile
```

2. **Execute the circuit and generate a witness**:

```bash
nargo execute
```

---

## Generating Proofs

1. Generate a proof:

```bash
bb prove -b ./target/circuits.json -w ./target/circuits.gz -o ./target
```

Or with Keccak oracle:

```bash
bb prove --oracle_hash keccak -b ./target/circuits.json -w ./target/circuits.gz -o ./target
```

---

## Verification

### Off-chain verification

```bash
bb verify -k ./target/vk -p ./target/proof
```

### On-chain verifier contract

```bash
bb write_solidity_verifier -k ./target/vk -o ./target/Verifier.sol
```

---

## Notes and Troubleshooting

* Ensure `inputs.txt` is correctly populated before running `generate_inputs.sh`.
* `Prover.toml` is overwritten each time the script runs.
* If you see compilation errors (bit-width, keccak, or parameter issues), patch the libraries:

  * **Bit-width:** `u64 << u64` instead of `u64 << u8` in `noir-array-helpers/src/lib.nr`
  * **keccak256:** use `noir_std::crypto::keccak256` in `ecrecover-noir/src/secp256k1.nr`
  * **ecrecover params:** only 4 arguments

---

## References and Useful Links

* [Cyfrin zk Ecrecover Repo](https://github.com/Cyfrin/zk-ecrecover-cu/tree/main)
* [Noir Modules, Packages, and Dependencies](https://noir-lang.org/docs/noir/modules_packages_crates/dependencies)
* [ecrecover-noir GitHub](https://github.com/colinnielsen/ecrecover-noir)
* [Foundry Docs](https://getfoundry.sh/)




