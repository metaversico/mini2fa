**mini2fa**
minimal self custody 2fa keys for time based one-time passwords

---

# 🔐 Self-Hosted TOTP Authenticator Web App — Development Specifications

## 📄 Overview

This project is a **static, self-hosted web application** that reproduces the core functionality of mobile-based TOTP (Time-based One-Time Password) apps like Google Authenticator, but without relying on any external cloud provider or mobile app.

It securely stores and displays rotating 2FA codes for various services (e.g., GitHub, Kraken, etc.) entirely in the browser, with **local decryption using a master password**. No backend or external server interaction is required.

---

## 🎯 Goals

* **Security**: Secrets are AES-encrypted and decrypted only in-browser using a user-provided password.
* **Simplicity**: No backend, no dependencies beyond a CDN or optional local JavaScript libraries.
* **Portability**: Works in any browser, on desktop or mobile, from local file or static host.
* **Self-Sovereignty**: Users control their secrets entirely, with optional manual or encrypted backup workflows.

---

## 🧰 Architecture

* **Frontend-only static site** (`index.html`)
* Uses:

  * **[otplib](https://github.com/yeojz/otplib)** – for TOTP generation (RFC 6238)
  * **[crypto-js](https://github.com/brix/crypto-js)** – for AES encryption/decryption
* Optional: Self-contained, offline bundle with all JS libs included

---

## 🔑 Security Model

| Threat                    | Mitigation                                                                    |
| ------------------------- | ----------------------------------------------------------------------------- |
| Secrets exposure          | All secrets are encrypted at rest; only decrypted in memory at runtime        |
| Weak password brute force | Passwords should be strong; optional use of PBKDF2/scrypt to slow brute-force |
| Code injection            | No dynamic loading or eval; if hosted, restrict CSP                           |
| Backup leakage            | Encrypted backup format; no plaintext JSON exports                            |

---

## 🧱 Core Features

### 1. 🔐 Password-Protected Access

* App prompts for a **master password**
* AES decrypts an embedded (or externally loaded) JSON blob of TOTP secrets
* Invalid password shows error, no secrets are exposed

### 2. 🔄 TOTP Code Generation

* Once decrypted, app shows a **list of current 6-digit codes**, one per service
* Codes update every 30 seconds
* Visual countdown bar per service (optional)

### 3. 💾 Backup & Restore

* Manual export of current secrets (as encrypted JSON string)
* Ability to paste/import an encrypted string for testing or migration
* (Optional) Encrypt/decrypt helper tool or page

---

## 🧑‍💻 Developer Notes

### 1. Encryption Format

```js
const encrypted = CryptoJS.AES.encrypt(JSON.stringify(secrets), password).toString();
const decrypted = JSON.parse(CryptoJS.AES.decrypt(encrypted, password).toString(CryptoJS.enc.Utf8));
```

Use `CryptoJS.AES` with a string password. Optionally strengthen with PBKDF2 or WebCrypto-derived key.

### 2. Data Schema (Secrets)

```json
{
  "GitHub": "JBSWY3DPEHPK3PXP",
  "Kraken": "GEZDGNBVGY3TQOJQ",
  "Hetzner": "KNXW4ZDPO5XXE3DE"
}
```

Each entry maps a **label** to a **base32-encoded TOTP secret**.

### 3. Code Generation

Use `otplib.authenticator.generate(secret)`
Set options:

```js
otplib.authenticator.options = {
  step: 30, // seconds
  digits: 6,
  algorithm: 'SHA1'
};
```

---

## ✍️ UI/UX Guidelines

* Clean and minimal interface
* Password prompt screen:

  * One input + unlock button
  * Show error if password fails
* Main screen:

  * List of labels + codes
  * Auto-updating codes every second
  * (Optional) Countdown timer or progress bar
* Accessible on both mobile and desktop

---

## ⚙️ Optional Enhancements

| Feature             | Description                                   |
| ------------------- | --------------------------------------------- |
| QR Code Import      | Scan QR codes from services to import secrets |
| WebCrypto           | Use native browser crypto instead of CryptoJS |
| LocalStorage Save   | Optionally store encrypted blob in browser    |
| Hardware Key Unlock | Use WebAuthn to unlock secrets with a YubiKey |
| Multi-user          | Allow multiple vaults or identities           |
| Multi-device sync   | Manual or encrypted QR-based vault transfer   |

---

## 🔄 Update & Maintain

* Vault updates are **manual**:

  * Decrypt → edit → re-encrypt → paste updated blob into HTML
* Consider creating a companion **encrypt/decrypt editor** as a separate page or CLI tool
* Vault can be copied/backed up as a text blob

---

## 📁 Project Structure

```
selfhosted-auth/
├── index.html         # Main app
├── totp-vault.json    # (Optional) encrypted secrets vault
├── encrypt.html       # (Optional) vault generator
└── libs/              # Optional local copies of otplib, crypto-js
```

---

## 🧪 Development & Testing

* Open `index.html` in any browser
* Test decryption and TOTP generation with known secrets (e.g., from `otpauth://` URLs)
* Validate with known test vectors (Google Authenticator compatible)
* Ensure timer sync accuracy within ±1s of Google Authenticator or Authy

---

## 📦 Deployment

* **Local use**: Double-click the file or run `python -m http.server` in the folder
* **Self-hosting**: Serve via Nginx, Caddy, GitHub Pages, etc.
* **Airgapped option**: Bundle into a single `index.html` with no external dependencies

---

## ✅ Summary

| Requirement     | Approach                        |
| --------------- | ------------------------------- |
| Self-hosted     | ✅ Static files, no server logic |
| Secure          | ✅ AES-encrypted secrets         |
| Portable        | ✅ Works in any browser          |
| Dependency-free | ✅ CDN or local-only mode        |
| Reproducible    | ✅ Manual backup/restore flow    |

---