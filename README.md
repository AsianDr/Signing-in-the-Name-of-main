# Signing in the Name of  
*An educational zero-knowledge group signature prototype built with Circom, snarkjs, and Node.js.*

## 📖 Overview
This project demonstrates a simple **anonymous group signing system** using zero-knowledge proofs.

A user can sign a document as a member of a group without immediately revealing which group member created the signature. The project uses a **Merkle tree** to prove group membership, **Poseidon hashes** for identity commitments, and **Groth16 zk-SNARK proofs** through `snarkjs`.

The system also includes a reveal flow where the original signer can later prove that they were the person who signed the document.

This project is useful for learning how anonymous signatures, Merkle proofs, trusted setup, Circom circuits, and proof verification work together in a small prototype.

---

## 🚀 Features
- Anonymous document signing using zero-knowledge proofs
- User signup with password-based identity secrets
- Poseidon hashing for identity commitments and signatures
- Merkle tree generation for group membership
- Merkle proof verification inside a Circom circuit
- Groth16 trusted setup, proof generation, and proof verification
- Document hashing into a Circom-compatible field element
- Optional signer revelation flow
- Simple terminal-based workflow using Node.js scripts
- Helper scripts and `justfile` commands for building circuits and proofs

---

## 📁 Project Structure

```
Signing-in-the-Name-of-main/
├── README.md
├── .gitignore
├── justfile
│
├── ptau/
│   └── .gitkeep
│
├── scripts/
│   ├── prepare.sh                  # Installs required tools
│   ├── build.sh                    # Builds Circom circuits
│   ├── build_depth.sh              # Builds circuits with Merkle tree depth
│   ├── set_depth.sh                # Inserts Merkle tree depth into templates
│   ├── generate_proof.sh           # Generates zk-SNARK proofs
│   ├── verify_proof.sh             # Verifies zk-SNARK proofs
│   ├── trusted_setup.sh            # Runs full trusted setup
│   ├── trusted_setup_phase1.sh     # Runs Powers of Tau phase 1
│   ├── trusted_setup_phase2.sh     # Runs circuit-specific phase 2
│   └── install_just.sh             # Installs the just command runner
│
├── solution/
│   ├── package.json
│   ├── signup.js                   # Creates users, Merkle tree, and db.json
│   ├── trustedSetup.js             # Builds circuits and runs trusted setup
│   ├── sign.js                     # Generates an anonymous document signature
│   ├── verifySignature.js          # Verifies an anonymous signature proof
│   ├── reveal.js                   # Generates a proof revealing the signer
│   ├── verifyRevelation.js         # Verifies signer revelation
│   ├── server.js                   # Core server-side helper logic
│   ├── utils.js                    # Poseidon hash and field helpers
│   ├── solution-template.circom    # General signing circuit template
│   ├── vottord.pdf                 # Example document/file
│   │
│   ├── sign/
│   │   ├── package.json
│   │   ├── merkle.circom           # Merkle proof circuit helpers
│   │   └── sign-template.circom    # Anonymous signing circuit template
│   │
│   └── reveal/
│       ├── package.json
│       └── reveal.circom           # Signer revelation circuit
│
└── utils/
    ├── README.md
    ├── generate_witness.js         # Generates witness files from WASM
    └── witness_calculator.js       # Witness calculator helper
```

---

## 📌 Key Components

### **`solution/signup.js`**
Creates the simulated user group. It asks for a username, password, number of simulated users, and a file path. It then:
- creates identity commitments
- builds a Merkle tree
- generates Merkle proofs for each user
- converts the selected document into a field element
- writes the pseudo-database to `db.json`

### **`solution/sign.js`**
Generates an anonymous signature proof for the first document in the database. The signer proves that:
- they know a valid password/identity secret
- their identity commitment is inside the group Merkle tree
- they signed the chosen document

The proof and public outputs are stored in `db.json`.

### **`solution/verifySignature.js`**
Loads the saved proof and public output from `db.json`, then verifies the signature proof using `snarkjs`.

### **`solution/reveal.js`**
Allows the signer to reveal themselves later. The script creates a second zero-knowledge proof showing that the user who reveals themselves produced the original anonymous signature.

### **`solution/verifyRevelation.js`**
Verifies the signer revelation proof and prints the username of the revealed signer.

### **`solution/sign/sign-template.circom`**
The main anonymous signing circuit. It checks:
- identity commitment using Poseidon
- Merkle tree membership
- signature generation from identity secret and document

### **`solution/reveal/reveal.circom`**
The reveal circuit. It proves that the revealing user’s identity secret produces the same signature attestation as the original anonymous signature.

### **`scripts/`**
Contains helper scripts for preparing dependencies, compiling circuits, running trusted setup, generating proofs, and verifying proofs.

---

## 🛠️ Technologies Used
- **Node.js** for command-line scripts and project logic
- **Circom 2** for writing zero-knowledge circuits
- **snarkjs** for trusted setup, proof generation, and verification
- **Groth16** as the zk-SNARK proving system
- **circomlib / circomlibjs** for Poseidon hashing
- **just** for command automation
- **Bash** for setup and build scripts
- **JSON** as a simple local pseudo-database

---

## ⚙️ Prerequisites
Before running the project, install or prepare:

- Node.js and npm
- Rust and Cargo
- Circom
- snarkjs
- just
- Bash-compatible terminal

You can use the included preparation script:

```bash
just prepare
```

This script attempts to install `just`, Rust, Circom, and `snarkjs`.

---

## ▶️ How to Run

### 1. Clone the repository

```bash
git clone <repository-url>
cd Signing-in-the-Name-of-main
```

### 2. Install dependencies

From the project root, run:

```bash
just prepare
```

Then install the Node.js dependencies from the `solution` folder:

```bash
cd solution
npm install
```

The circuit subfolders also contain their own `package.json` files. The build scripts install those dependencies when the circuits are compiled.

---

## 🔐 Main Workflow

All Node commands below should be run from the `solution/` folder.

### 1. Sign up and create the local database

```bash
node signup.js
```

You will be asked for:
- your username
- your password
- number of simulated users
- path to the document/file you want to register

This creates:
- `db.json`
- `debug.json`
- Merkle tree data
- user membership proofs
- document field representation

---

### 2. Run trusted setup

```bash
node trustedSetup.js
```

This builds the signing and reveal circuits, then runs the Groth16 trusted setup. You may be asked to enter entropy for the setup process.

Generated files are stored inside circuit `target/` folders.

---

### 3. Sign the document anonymously

```bash
node sign.js
```

The script asks for your username and password, generates a zero-knowledge proof, and stores the proof in `db.json`.

---

### 4. Verify the anonymous signature

```bash
node verifySignature.js
```

This checks whether the saved proof is valid for the selected document and group Merkle root.

---

### 5. Reveal the signer

```bash
node reveal.js
```

This creates a proof that links the signer’s identity to the anonymous signature without changing the original signing flow.

---

### 6. Verify the revelation

```bash
node verifyRevelation.js
```

This verifies the revelation proof and prints the username of the signer who revealed themselves.

---

## 🧪 Example Flow

```bash
cd Signing-in-the-Name-of-main
just prepare
cd solution
npm install
node signup.js
node trustedSetup.js
node sign.js
node verifySignature.js
node reveal.js
node verifyRevelation.js
```

---

## 📝 Generated Files

The project generates several files during execution:

```text
db.json                         # Local pseudo-database
debug.json                      # Merkle tree debug data
ptau/*.ptau                     # Powers of Tau artifacts
solution/sign/sign.circom       # Generated from sign-template.circom
solution/sign/target/           # Signing circuit build/proof files
solution/reveal/target/         # Reveal circuit build/proof files
```

These files are ignored by Git because they are generated during setup, proving, or verification.

---

## ⚠️ Notes
- This is an educational prototype, not a production-ready cryptographic system.
- The database is simulated using local JSON files.
- The signup flow creates random simulated users to form a group.
- The signing flow currently signs the first file in the database for simplicity.
- The trusted setup script uses generated or user-provided entropy for the ceremony.
- The scripts assume the required command-line tools are available in your terminal.

---

## 🧩 Troubleshooting

### `just: command not found`
Run:

```bash
./scripts/install_just.sh
```

or install `just` manually for your operating system.

### `circom: command not found`
Run:

```bash
just prepare
```

or install Circom manually and make sure it is available in your terminal path.

### `snarkjs: command not found`
Install it globally:

```bash
npm install -g snarkjs
```

### `Cannot find module ...`
Run npm install inside the `solution` folder:

```bash
cd solution
npm install
```

### Linux issue with `sed -i ''`
The `scripts/set_depth.sh` file uses the macOS/BSD `sed -i ''` format. On many Linux systems, this may fail. Change this line:

```bash
sed -i '' "s/<DEPTH_OF_MERKLE_TREE>/$DEPTH_OF_MERKLE_TREE/g" "$CIRCOM_FILE.circom"
```

to:

```bash
sed -i "s/<DEPTH_OF_MERKLE_TREE>/$DEPTH_OF_MERKLE_TREE/g" "$CIRCOM_FILE.circom"
```

---

## 🔮 Future Improvements
- Add a clearer multi-file signing workflow
- Add automated tests for signup, signing, verification, and reveal
- Replace the JSON pseudo-database with a real database
- Improve error handling for missing files and invalid users
- Add Docker support for easier setup
- Add Linux/macOS compatible setup scripts
- Add a frontend interface for signing and verification
- Document the exact public and private circuit inputs

---

## 📄 License
No license file was included in the uploaded project. Add a license before publishing or reusing the project publicly.
