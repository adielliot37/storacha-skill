---
name: storacha-upload
description: >-
  Upload files to IPFS, store on Storacha, upload to decentralized storage,
  check Storacha status, view storage usage, create Storacha space, switch space,
  list spaces, retrieve files by CID, open IPFS content, get gateway link,
  decentralized storage, web3 storage, pin to IPFS, content-addressed storage,
  store on chain, get my CID, backup to IPFS, share IPFS link, upload directory,
  remove upload, Storacha delegation, IPFS file sharing, permanent storage,
  Filecoin backup, manage Storacha account
version: 1.0.0
homepage: https://github.com/adielliot37/storacha-skill
metadata:
  clawdbot:
    emoji: "🔥"
    requires:
      bins:
        - storacha
        - node
      env: []
    primaryEnv: ""
    files:
      - "scripts/*"
---

# Storacha Upload

Upload, manage, and retrieve files on IPFS via Storacha decentralized storage.

> **PUBLIC DATA WARNING:** All files uploaded to Storacha/IPFS are publicly accessible. Anyone with the CID can retrieve them. Never upload unencrypted sensitive data.

> **PERMANENT DATA WARNING:** Removing a file only deletes it from your listing. Other IPFS nodes may retain copies indefinitely. Treat every upload as permanent.

---

## Prerequisites

Run this before anything else:

```bash
which storacha && storacha --version
```

If `storacha` is not found, install it:

```bash
npm install -g @storacha/cli
```

Requires **Node.js v18+**. Verify with `node -v`. If missing or outdated, direct the user to [nodejs.org](https://nodejs.org).

---

## First-Time Setup & Authentication

Complete these steps in order before any upload operation.

### Step 1 — Check CLI Installation

```bash
which storacha && storacha --version
```

**Expected output:**
```
/usr/local/bin/storacha
x.y.z
```

If missing, install:
```bash
npm install -g @storacha/cli
```

Then re-run the check. If install fails, verify Node.js v18+ is available.

### Step 2 — Check Authentication

```bash
storacha whoami
```

**If output contains `did:key:`** → authenticated. Proceed to Step 3.

**If error or no DID** → not logged in. Go to Step 2a.

### Step 2a — Login Flow

Ask the user for their Storacha email address. If they don't have an account, direct them to sign up (free) at https://console.storacha.network.

```bash
storacha login USER_EMAIL
```

Replace `USER_EMAIL` with the actual email address provided by the user.

Tell the user: *"Check your email for a verification link from Storacha and click it. Let me know once you've done that."*

**WAIT for user confirmation before proceeding.** The CLI blocks until the email link is clicked.

If this is a new account, inform the user they may need to select a plan after clicking the link:

| Plan | Price | Storage | Egress | Overage |
|---|---|---|---|---|
| Mild (Free) | $0/month | 5 GB | 5 GB | $0.15/GB |
| Medium | $10/month | 100 GB | 100 GB | $0.05/GB |
| Extra Spicy | $100/month | 2 TB | 2 TB | $0.03/GB |

### Step 3 — Check Spaces

```bash
storacha space ls
```

**Expected output:**
```
* did:key:z6Mk... SpaceName
  did:key:z6Mk... AnotherSpace
```

The `*` marks the active space.

- **If spaces exist with `*` marker** → active space is set. Proceed to Step 4.
- **If no spaces exist** → create one:
  ```bash
  storacha space create "MyFiles"
  ```
  Space names are permanent and cannot be changed.
- **If spaces exist but none is active** → activate one:
  ```bash
  storacha space use "SpaceName"
  ```

### Step 4 — Verify Provider Registration

```bash
storacha space info
```

**Expected output includes:**
```
Providers: did:web:web3.storage
```

If no provider is listed, the space is not registered. Direct the user to https://console.storacha.network to register the space, or create a new space.

### Step 5 — Check Storage Usage

```bash
storacha usage report
```

**Expected output format:**
```
Account: did:mailto:...
Provider: did:web:web3.storage
Space: did:key:z6Mk...
Size: 123456789
```

Parse the `Size` value and convert to human-readable format:
- < 1024 → bytes
- < 1,048,576 → KB
- < 1,073,741,824 → MB
- >= 1,073,741,824 → GB

Present a status dashboard to the user:

```
╔══════════════════════════════════════╗
║       Storacha Status Dashboard      ║
╠══════════════════════════════════════╣
║ Account:  did:mailto:user@email.com  ║
║ Space:    MyFiles (did:key:z6Mk...)  ║
║ Storage:  117.7 MB used              ║
║ Plan:     Mild (Free) — 5 GB limit   ║
╚══════════════════════════════════════╝
```

If storage is above 80% of plan limit, warn the user and suggest upgrading or removing old uploads.

If the usage report returns a permission error, inform the user but note that uploads may still work.

---

## Core Operations

### Upload a File

```bash
storacha up /path/to/file
```

**Expected output:**
```
⁂ Stored 1 file
⁂ https://storacha.link/ipfs/bafy...
```

After a successful upload, present:
- **Filename:** the uploaded file name
- **CID:** the content identifier hash
- **Gateway URLs:**
  - Path style: `https://storacha.link/ipfs/CID`
  - Subdomain style: `https://CID.ipfs.storacha.link`

Remind the user: anyone with the CID or link can access this file.

### Upload a Directory

```bash
storacha up /path/to/directory/
```

- Dotfiles (hidden files) are excluded by default. Use `--hidden` to include them.
- Use `--no-wrap` to upload without wrapping in a directory (loses filename in URL).

For directory uploads, files are accessible at:
```
https://storacha.link/ipfs/CID/filename.txt
```

### List Uploads

```bash
storacha ls
```

Displays all uploads in the current space with their CIDs.

### Remove an Upload

```bash
storacha rm CID
```

To also remove underlying data shards:
```bash
storacha rm CID --shards
```

**Warn the user:** removal only deletes from your listing. The data may persist on other IPFS nodes indefinitely.

### Retrieve / Open a File

Open in browser:
```bash
storacha open CID
```

Download programmatically:
```bash
curl -o output.txt "https://storacha.link/ipfs/CID"
```

Subdomain style:
```bash
curl -o output.txt "https://CID.ipfs.storacha.link"
```

---

## Space Management

**Create a space:**
```bash
storacha space create "ProjectName"
```
Space names are permanent and cannot be changed after creation.

**List all spaces:**
```bash
storacha space ls
```
The active space is marked with `*`.

**Switch active space:**
```bash
storacha space use "SpaceName"
```
Or by DID:
```bash
storacha space use did:key:z6Mk...
```

**View space details:**
```bash
storacha space info
```
Shows the space DID and registered providers.

---

## Sharing & Delegation

Create a UCAN delegation for another agent:
```bash
storacha delegation create AUDIENCE_DID --can store/add --can upload/add --output ./delegation.ucan
```

Full admin delegation:
```bash
storacha delegation create AUDIENCE_DID --can '*' --output ./admin.ucan --base64
```

List active delegations:
```bash
storacha delegation ls
```

---

## Error Handling

1. **"command not found: storacha"** → Install CLI: `npm install -g @storacha/cli`
2. **"no proofs available for resource"** → Re-login with `storacha login EMAIL` or switch spaces with `storacha space use "Name"`
3. **"Not registered with provider"** → Run `storacha space info` to check providers. Re-register at https://console.storacha.network or create a new space.
4. **Upload hangs or times out** → Check internet connection. Retry the upload. For large files, ensure stable connectivity.
5. **"usage/report" permission error** → This is informational only. Uploads should still work. Proceed with the operation.
6. **"no spaces" or empty space list** → Create a space: `storacha space create "MyFiles"`
7. **Storage limit errors** → Upgrade plan at https://console.storacha.network or remove old uploads: `storacha rm CID --shards`

---

## Quick Reference

| Action | Command |
|---|---|
| Install CLI | `npm install -g @storacha/cli` |
| Login | `storacha login user@email.com` |
| Check identity | `storacha whoami` |
| Create space | `storacha space create "Name"` |
| List spaces | `storacha space ls` |
| Switch space | `storacha space use "Name"` |
| Space details | `storacha space info` |
| Upload file | `storacha up /path/to/file` |
| Upload directory | `storacha up /path/to/dir/` |
| Upload without wrap | `storacha up /path --no-wrap` |
| Upload with dotfiles | `storacha up /path --hidden` |
| List uploads | `storacha ls` |
| Remove upload | `storacha rm CID` |
| Remove with shards | `storacha rm CID --shards` |
| Open in browser | `storacha open CID` |
| Check usage | `storacha usage report` |
| Create delegation | `storacha delegation create DID --can store/add --output file.ucan` |
| List delegations | `storacha delegation ls` |

---

## Important Notes

- Authentication is email-based using DIDs and UCAN. There are no API keys or tokens.
- Spaces are storage namespaces identified by `did:key`. Each space tracks its own uploads independently.
- Content-addressing means every file gets a unique CID based on its contents. Identical files produce identical CIDs.
- Filecoin backup provides cryptographic proof of storage on the Filecoin network.
- Two gateway URL styles are available:
  - Path: `https://storacha.link/ipfs/CID`
  - Subdomain: `https://CID.ipfs.storacha.link`
- The current CLI binary is `storacha`. It was previously called `w3` during the web3.storage era.
