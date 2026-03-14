# Naza Shipping Scanner

![Naza SecureLLM TUI](https://raw.githubusercontent.com/ornab74/naza/refs/heads/main/demonaza.png)

Naza is a terminal-based, encrypted local AI tool for shipping safety scanning and secure LLM usage. It lets you:

- run a shipping-focused safety scan and get a single-word result: `Low`, `Medium`, or `High`
- manage a local GGUF model
- encrypt the model and the chat/history database with AES-GCM
- review encrypted history
- rotate the encryption key when needed

The current scanner in [`main.py`](/home/a41371777/code/ship-scan/main.py) keeps the same classification style as the original project, but the app wording and workflow are now oriented around shipping.

## What The Program Does

When you launch the app, it opens a text menu with these options:

1. `Model Manager`
2. `Chat with model`
3. `Shipping Scanner`
4. `View chat history`
5. `Rekey / Rotate key`
6. `Exit`

The scanner asks for a location, shipping type, weather/visibility, traffic density, obstacles, and sensor notes. It builds a prompt for the local model and returns a one-word safety label.

The app also:

- stores the encryption key in `.enc_key`
- stores the encrypted history database in `chat_history.db.aes`
- stores the encrypted model as `models/llama3-small-Q3_K_M.gguf.aes`
- temporarily decrypts files only when needed, then re-encrypts them

## Files You Should Know

- [`main.py`](/home/a41371777/code/ship-scan/main.py): main application
- [`requirements.in`](/home/a41371777/code/ship-scan/requirements.in): direct Python dependencies
- [`requirements.txt`](/home/a41371777/code/ship-scan/requirements.txt): pinned dependency lockfile
- [`termux-naza-autosetup`](/home/a41371777/code/ship-scan/termux-naza-autosetup): Android/Termux setup assets
- `models/`: model storage directory, created automatically
- `.enc_key`: encryption key file, created on first run
- `chat_history.db.aes`: encrypted SQLite history database

## Requirements

You need:

- Python `3.10+` recommended
- enough disk space for the GGUF model
- enough RAM/CPU to load `llama-cpp-python` and the selected model

Python dependencies used by this project:

- `llama-cpp-python`
- `httpx`
- `aiosqlite`
- `cryptography`
- `psutil`
- `pennylane`

## Installation Guides

Pick the guide that matches your environment.

### Ubuntu Installation Guide

This is the most straightforward path for most users.

#### 1. Update your system

```bash
sudo apt update
sudo apt upgrade -y
```

#### 2. Install system packages

```bash
sudo apt install -y git python3 python3-venv python3-pip build-essential cmake libssl-dev
```

If `llama-cpp-python` gives you trouble during installation, these extra packages can help on some systems:

```bash
sudo apt install -y pkg-config libffi-dev
```

#### 3. Clone the project

```bash
git clone <your-repo-url> ship-scan
cd ship-scan
```

#### 4. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

#### 5. Install Python dependencies

Recommended:

```bash
pip install --upgrade pip
pip install -r requirements.in
```

If you specifically want the pinned lockfile:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 6. Run the app

```bash
python3 main.py
```

### Termux / Android Installation Guide

The repository includes a `termux-naza-autosetup` folder, but you can also install manually.

#### 1. Install Termux

Install Termux on your Android device.

#### 2. Update packages

```bash
pkg update -y
pkg upgrade -y
```

#### 3. Install required packages

```bash
pkg install -y git python clang cmake make rust
```

Depending on your device and package state, you may also need:

```bash
pkg install -y libcrypt libffi openssl
```

#### 4. Clone the project

```bash
git clone <your-repo-url> ship-scan
cd ship-scan
```

#### 5. Create and activate a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate
```

#### 6. Install Python dependencies

```bash
pip install --upgrade pip
pip install -r requirements.in
```

#### 7. Run the app

```bash
python main.py
```

#### Termux notes

- mobile builds of `llama-cpp-python` can be slow
- some Android devices may not have enough memory for comfortable model loading
- if direct Termux installation becomes difficult, using a Linux desktop or server is usually easier

### Windows Installation Guide Using WSL

For Windows, the recommended approach is to run the project inside Windows Subsystem for Linux using Ubuntu.

#### 1. Install WSL and Ubuntu

Open PowerShell as Administrator and run:

```powershell
wsl --install -d Ubuntu
```

If WSL is already installed, you can list distributions with:

```powershell
wsl --list --online
```

Then restart Windows if prompted and open the Ubuntu terminal.

#### 2. Update Ubuntu inside WSL

```bash
sudo apt update
sudo apt upgrade -y
```

#### 3. Install required packages inside WSL

```bash
sudo apt install -y git python3 python3-venv python3-pip build-essential cmake libssl-dev
```

#### 4. Clone the project inside WSL

Recommended:

```bash
cd ~
git clone <your-repo-url> ship-scan
cd ship-scan
```

You can also work from `/mnt/c/...`, but performance is usually better when the repo lives inside the Linux filesystem under your WSL home directory.

#### 5. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

#### 6. Install Python dependencies

```bash
pip install --upgrade pip
pip install -r requirements.in
```

#### 7. Run the app

```bash
python3 main.py
```

#### WSL notes

- run the app from the Ubuntu shell, not from plain Windows `cmd`
- keep the repo inside the WSL filesystem if possible
- if model compilation is slow, be patient on the first install

### macOS Notes

macOS can often use the same basic flow as Ubuntu with `python3`, a virtual environment, and `pip install -r requirements.in`, but native build behavior can vary by Xcode toolchain state. If you are on macOS and hit dependency build issues, install the Xcode command line tools first.

```bash
xcode-select --install
```

## First Run

The first launch usually does two important things:

1. creates or loads an encryption key
2. initializes the encrypted history database

If `.enc_key` does not exist, the app asks how you want to create it:

- `1) Generate random key (saved raw)`
- `2) Derive from passphrase (salt+derived saved)`

### Key option guidance

Choose `1` if:

- you want the simplest setup
- this is a local device you control
- you do not need to remember a passphrase

Choose `2` if:

- you want the key material tied to a passphrase
- you are comfortable entering and managing that passphrase

Important:

- if you lose `.enc_key`, you may lose access to the encrypted model and chat history
- protect `.enc_key` like a secret

## How To Use The Menu

When the app starts, you can move with arrow keys or type the menu number and press Enter.

### 1. Model Manager

Use this before scanning if you do not already have an encrypted model prepared.

Options:

1. `Download model from remote repo (httpx)`
2. `Verify plaintext model hash (compute SHA256)`
3. `Encrypt plaintext model -> .aes`
4. `Decrypt .aes -> plaintext (temporary)`
5. `Delete plaintext model`
6. `Back`

#### Recommended first-time model setup

Follow this sequence:

1. Open `Model Manager`
2. Choose `1` to download the model
3. Let the app verify the SHA256 hash
4. When prompted, choose to encrypt the downloaded model
5. When prompted, choose to remove the plaintext model
6. Return to the main menu

That leaves you with an encrypted model on disk instead of a plaintext GGUF file.

#### What each Model Manager option means

- `Download model`: fetches the model from the configured remote URL
- `Verify plaintext model hash`: computes the SHA256 for the unencrypted GGUF file
- `Encrypt plaintext model`: converts the model file into the `.aes` encrypted version
- `Decrypt .aes -> plaintext`: temporarily restores the model as a normal GGUF file
- `Delete plaintext model`: removes the unencrypted GGUF file from disk

### 2. Chat With Model

This opens a direct chat session with the model.

What happens internally:

- the encrypted model is decrypted to a plaintext GGUF file
- the model is loaded
- you can chat interactively
- your chat is saved into the encrypted history database
- on exit, the plaintext model is re-encrypted and removed

Available chat commands:

- `/exit`
- `exit`
- `quit`
- `/history`

Use this mode if you want free-form prompting instead of the dedicated shipping scanner workflow.

### 3. Shipping Scanner

This is the main safety scan flow.

You will be prompted for:

- `Location`
- `Shipping type`
- `Weather/visibility`
- `Traffic density`
- `Reported obstacles`
- `Sensor notes`

Then you choose one generation mode:

1. `Chunked generation + punkd (recommended)`
2. `Chunked only`
3. `Direct single-call generation`

#### Recommended scanner flow

Use this simple workflow:

1. Open `Shipping Scanner`
2. Enter a meaningful location
3. Choose the shipping type
4. Add weather/visibility details
5. Add traffic density
6. Add obstacles and sensor notes
7. Leave generation on option `1` unless you have a reason to change it
8. Wait for the result

The output is a single label:

- `Low`
- `Medium`
- `High`

#### Example input

- `Location`: `Port of Newark Dock 4`
- `Shipping type`: `ground`
- `Weather/visibility`: `heavy rain, reduced visibility`
- `Traffic density`: `high`
- `Reported obstacles`: `loading area congestion, standing water`
- `Sensor notes`: `intermittent camera blur`

#### After the result appears

You will see these options:

1. `Re-run with edits`
2. `Export to JSON`
3. `Save & return`
4. `Cancel`

What they do:

- `Re-run with edits`: lets you tweak the input fields and scan again
- `Export to JSON`: writes the input, generated prompt, result, and timestamp to a JSON file
- `Save & return`: stores the scan in encrypted history and returns to the main menu
- `Cancel`: exits without saving a new history entry

#### JSON export

If you export, the default filename is:

```text
shipping_scan.json
```

The JSON includes:

- the input fields
- the exact generated prompt
- the result label
- a timestamp

### 4. View Chat History

This opens the encrypted history viewer.

Features:

- paging
- search
- viewing past prompts and responses

Commands inside the viewer:

- `n` for next page
- `p` for previous page
- `s` for search
- `q` to quit

History includes both chat sessions and saved shipping scanner runs.

### 5. Rekey / Rotate Key

Use this if you want to replace the encryption key.

Menu options:

1. `New random key`
2. `Passphrase-derived`
3. `Cancel`

What it does:

- decrypts the current encrypted model and database using the old key
- writes a new key file
- re-encrypts the model and database with the new key

Be careful here. If rekeying is interrupted at the wrong time, you may need to inspect the resulting files manually.

## Practical Operating Example

Here is a full first-use example:

1. Start the program with `python3 main.py`
2. Create a key when prompted
3. Open `Model Manager`
4. Download the model
5. Encrypt it
6. Remove the plaintext file
7. Return to the main menu
8. Open `Shipping Scanner`
9. Enter your shipping conditions
10. Choose generation option `1`
11. Review the `Low`, `Medium`, or `High` result
12. Export to JSON or save the result
13. Open `View chat history` later if you want to review past scans

## How The Shipping Scanner Decides

The scanner prompt asks the model to output exactly one word:

- `Low`
- `Medium`
- `High`

The code also adds:

- system metrics from `psutil`
- an entropic score
- token weighting via the PUNKD helper logic

If the model returns extra text, the program attempts to normalize the result back into one of the three valid labels.

## Security Model

This project is designed to reduce plaintext exposure on disk.

### Encrypted components

- model file
- history database

### Encryption details

- AES-GCM is used for encryption
- keys are either randomly generated or derived from a passphrase using PBKDF2-HMAC-SHA256

### Important security caveats

- a plaintext model exists briefly during active use
- the key file is critical and must be protected
- if your machine is compromised while the app is running, local encryption does not stop live memory access

## Troubleshooting

### `No encrypted model found`

Cause:

- you have not downloaded and encrypted the model yet

Fix:

1. open `Model Manager`
2. download the model
3. encrypt it

### Model download fails

Cause:

- no network access
- remote URL unavailable
- TLS or dependency issue

Fix:

- retry later
- verify your connection
- confirm the model URL configured in [`main.py`](/home/a41371777/code/ship-scan/main.py)

### Hash mismatch during model download

Cause:

- corrupted download
- remote file changed
- tampered file

Fix:

- do not keep the file unless you explicitly trust the new artifact
- investigate before proceeding

### `Failed to load` or model load errors

Cause:

- insufficient RAM
- incompatible `llama-cpp-python` build
- missing native build requirements

Fix:

- use a smaller model if you adapt the code
- rebuild dependencies in a clean virtual environment
- confirm your platform supports the installed wheel/build

### `python: command not found`

Fix:

Use:

```bash
python3 main.py
```

### History cannot be read after key changes

Cause:

- `.enc_key` changed
- rekeying was incomplete
- wrong key file is present

Fix:

- restore the correct key file if available
- verify the encrypted files and rekey flow carefully

## Developer Notes

A few implementation details in [`main.py`](/home/a41371777/code/ship-scan/main.py):

- the model is loaded through `llama_cpp.Llama`
- history is stored in a temporary decrypted SQLite DB and then re-encrypted
- the scanner uses `build_shipping_scanner_prompt(...)`
- the shipping scanner workflow is implemented in `shipping_scanner_flow(...)`

## Quick Start

If you want the shortest path:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.in
python3 main.py
```

Then:

1. create a key
2. download and encrypt the model in `Model Manager`
3. run `Shipping Scanner`

## Disclaimer

This tool is an AI-assisted local classification utility. Treat the output as a support signal, not as a sole operational authority. For real shipping decisions, combine the result with human review, facility procedures, environmental conditions, and the actual state of the shipment and equipment.
