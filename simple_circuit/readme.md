# simple_circuit — README

A compact, step-by-step guide to build, run, prove, and verify the `simple_circuit` Noir example using **nargo** (Noir CLI) and **Barretenberg (`bb`)** as the proving backend.

> This README is self-contained: copy it into `README.md` in the project root or keep it somewhere you prefer.


## Prerequisites

* macOS / Linux (commands shown assume macOS shell / zsh)
* Git
* A terminal (iTerm / Terminal)
* `curl` available
* Basic familiarity with command line

---

## 1) Install required tools

### Install Noir (nargo) via `noirup`

```bash
# install noirup (installer)
curl -L https://raw.githubusercontent.com/noir-lang/noirup/refs/heads/main/install | bash

# add noirup/nargo to PATH (if your shell is zsh)
echo 'export PATH="$HOME/.nargo/bin:$HOME/.noirup/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# run noirup to install nargo (if not already installed automatically)
noirup
```

Verify:

```bash
nargo --version
# expected output: nargo version = 1.0.0-beta.xx
```

---

### Install Barretenberg (bb) via `bbup`

```bash
# install bbup (installer)
curl -L https://raw.githubusercontent.com/AztecProtocol/aztec-packages/refs/heads/next/barretenberg/bbup/install | bash

# ensure ~/.bb is in your PATH
echo 'export PATH="$HOME/.bb:$PATH"' >> ~/.zshrc
source ~/.zshrc

# use bbup to download/install the actual `bb` binary
bbup
```

Verify:

```bash
bb --version
# and
bbup --version
```

---

## 2) Project structure (what to expect)

After creating the project or if you cloned this repo, you should see:

```
.
├─ Nargo.toml
├─ Prover.toml        # inputs for witness generation
├─ src/
│  └─ main.nr
└─ target/
   ├─ simple_circuit.json   # compiled circuit (after nargo execute / nargo check)
   └─ simple_circuit.gz     # witness (after nargo execute)
```

---

## 3) Prepare `Prover.toml` (public/private inputs)

Create or edit `Prover.toml` in the project root and provide inputs required by your circuit.

Example:

```toml
x = "1"
y = "2"
```

> Values are strings in TOML. `x` can be private by default (unless declared `pub` in your `.nr`), `y` may be public depending on your circuit.

---

## 4) Compile & generate witness (nargo)

From the project root:

```bash
# sanity check / compile
nargo check

# execute to generate witness (and compile if necessary)
nargo execute
```

After `nargo execute` you should see:

```
[simple_circuit] Circuit witness successfully solved
[simple_circuit] Witness saved to target/simple_circuit.gz
```

Check the `target/` directory:

```bash
ls -l ./target
# expecting: simple_circuit.json   simple_circuit.gz
```

* `simple_circuit.json` — the compiled ACIR (circuit)
* `simple_circuit.gz` — the witness produced by execution

---

## 5) Produce a proof + verification key (Barretenberg `bb`)

Use the compiled circuit and witness to produce a proof and write the verification key:

```bash
bb prove -b ./target/simple_circuit.json -w ./target/simple_circuit.gz --write_vk -o ./target
```

Explanation of flags:

* `-b ./target/simple_circuit.json` : compiled circuit
* `-w ./target/simple_circuit.gz` : witness file
* `--write_vk` : instructs `bb` to produce a verification key file `vk`
* `-o ./target` : output directory where `proof` and `vk` will be written

After success, `./target` should contain:

```
simple_circuit.json
simple_circuit.gz
proof
vk
```

---

## 6) Verify the proof locally

Verify the generated proof using the verification key:

```bash
bb verify -p ./target/proof -k ./target/vk
```

If successful, you’ll see a confirmation that the proof was verified.

---

## 7) (Optional) Generate Solidity verifier contract

Noir provides a codegen command to export a Solidity verifier (if supported in your toolchain):

```bash
nargo codegen-verifier
# or, if the command differs by version:
nargo generate-verifier
```

This writes a `target/contract.sol` (or similar) that you can deploy to an EVM-compatible chain and use to verify proofs on-chain.

Deploy contract:

* Use Hardhat / Truffle / Foundry to compile and deploy `target/contract.sol`
* Then call the verifier contract with the proof bytes and public inputs (follow the contract ABI)

---

## Troubleshooting — common issues and fixes

### `gunzip: can't stat: .trget/simple_circuit.son: No such file or directory`

Cause: typo in path or filename.
Fix: use correct paths and filenames:

```bash
bb prove -b ./target/simple_circuit.json -w ./target/simple_circuit.gz --write_vk -o ./target
```

### `jq: error: Could not open file ./target/program.json: No such file or directory`

Cause: `bb` expects a compiled program file present. Ensure `nargo execute` produced `./target/*.json`. Confirm correct file name and path.

### `Unable to open file: ./target/vk`

Cause: using `bb prove` without `--write_vk`.
Fix: include `--write_vk` when proving.

### `command not found: bb` or `bbup`

Cause: `~/.bb` not in PATH.
Fix:

```bash
echo 'export PATH="$HOME/.bb:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Then run `bbup` to install `bb`.

### Nothing happens or files not created

* Re-run `nargo check` then `nargo execute`.
* Confirm `Prover.toml` inputs are valid for your circuit.
* Confirm you are in the correct project directory (run `pwd`).

---

## Useful commands — quick reference

Install tools

```bash
curl -L https://.../noirup/install | bash
noirup
curl -L https://.../bbup/install | bash
bbup
```

Create / run project

```bash
nargo new simple_circuit   # if starting fresh
cd simple_circuit
nargo check
echo 'x = "1"
y = "2"' > Prover.toml
nargo execute
```

Prove & verify with bb

```bash
bb prove -b ./target/simple_circuit.json -w ./target/simple_circuit.gz --write_vk -o ./target
bb verify -p ./target/proof -k ./target/vk
```

Generate solidity verifier (optional)

```bash
nargo codegen-verifier
# or nargo generate-verifier (depending on version)
```


