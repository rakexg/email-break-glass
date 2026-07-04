# email-break-glass — Email Emergency Recovery

Store encrypted email backup codes at a public URL disguised as a personal journal, so you can get back into your account from any browser, on any device, with nothing but what's in your head.

> **The name:** "break glass" comes from the emergency fire alarm which is smashed when there's no other way out, and from *break-glass access* in security, an account of last resort used when normal login is impossible. That's exactly this: you reach for it only when your phone is gone and every normal way into your email has failed.

> **Template repo.** Click **Use this template** to create your own private copy, then follow the setup below. The ciphertext shipped in `index.html` is a demo — it decrypts with the passphrase `correct horse battery staple` and contains no secrets. Replace it with your own before relying on this.

## Why

Worst case: your phone is lost and you have no other trusted device. Logging into Gmail on a new device needs either an OTP sent to the lost phone or a tap to approve on it, and neither works once the phone is gone. Backup codes get you in, but you can't memorize ten of them.

This project stores your encrypted backup codes at a public URL that looks like a personal journal. The ciphertext is meant to be public; the security is all in the passphrase. Open it from any browser, on any device.

You only need to memorize three things: the URL, the passphrase, and your Gmail password.

---

## How it works

Two GitHub repos are used:

| Repo | Visibility | Contains | Purpose |
|------|------------|----------|---------|
| Private repo | Private | `index.html`, `robots.txt`, `README.md`, `.github/workflows/sync.yml` | Source of truth |
| Public repo | Public | `index.html`, `robots.txt` | GitHub Pages — what the world sees |

On every push to the private repo, a sync workflow automatically pushes `index.html` and `robots.txt` to the public repo. The public repo always has a single commit (no history).

---

## What you memorize

1. URL of the page (e.g. `yourusername.github.io/yourpublicrepo`)
2. A passphrase
3. Gmail password

---

## Repo setup (do once)

### 1. Create private repo

1. Click **Use this template** → Create a new repository → any name (e.g. `gmail-recovery`) → set **Private** → Create

### 2. Personalize the disguise (important)

Edit `index.html` in your private repo:

- Change the `<title>`, heading, date, and journal entry text to something of your own
- Keep the "— draft —" section, input box, and script unchanged

### 3. Create public repo

1. github.com → New repository → any name (e.g. `journal`) → set **Public** → Create (empty, no README)

### 4. Create personal access token

Use a fine-grained token scoped to the public repo only. A classic token with `repo` scope grants write access to **every** repo on the account, including the private one, so a leaked token would let an attacker silently modify the page that receives your passphrase.

1. GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Generate new token → Repository access: **Only select repositories** → select your public repo
3. Permissions → Repository permissions → **Contents: Read and write** (nothing else)
4. Set an expiration and copy the token (set a reminder to rotate it before expiry)

### 5. Add secrets to private repo

Private repo → Settings → Secrets and variables → Actions → New repository secret. Add all four:

| Secret name | Value |
|-------------|-------|
| `PUBLIC_REPO_TOKEN` | Fine-grained token (public repo only, Contents: read/write) |
| `GIT_USERNAME` | Your GitHub username |
| `GIT_EMAIL` | Your GitHub email (the noreply address from Settings → Emails works too) |
| `PUBLIC_REPO_NAME` | Your public repo name (e.g. `journal`) created in step 3 |

### 6. Enable GitHub Pages on public repo

1. Public repo → Settings → Pages → Source: main branch → Save
2. Page will be live at `yourusername.github.io/yourpublicrepo` after first sync

### 7. Trigger first sync

GitHub → Private repo → Actions → Sync to public repo → Run workflow

### 8. Verify the demo

Open your GitHub Pages URL, enter `correct horse battery staple` in the "Enter draft" box, click Save. You should see the demo message. This proves the whole pipeline works before any real secret is involved.

---

## One-time setup

### 1. Generate a strong memorable passphrase. 
(Optional way)Diceware method for passphrase generation shown below:

- Get EFF large wordlist: https://www.eff.org/files/2016/07/18/eff_large_wordlist.txt
- Roll 5 physical dice → 5-digit number → look up word → repeat 6 times
- Memorize all 6 words in order (a vivid mental story linking the words in sequence helps)
- Test recall the next morning before proceeding

Do not use fewer than 6 dice-rolled words, and do not substitute words you picked yourself — the security comes from the dice. The ciphertext is public forever, so the passphrase must survive offline attack indefinitely.

### 2. Get Gmail backup codes

- Gmail → Security → 2-Step Verification → Backup codes → Get codes
- 10 codes will be shown on screen

### 3. Encrypt backup codes

Copy the 10 backup codes to clipboard, then run:

**Mac:**
```bash
pbpaste | openssl enc -aes-256-cbc -pbkdf2 -iter 600000 -a
```

**Linux:**
```bash
xclip -o | openssl enc -aes-256-cbc -pbkdf2 -iter 600000 -a
```

**Windows (PowerShell):**
```powershell
Get-Clipboard | openssl enc -aes-256-cbc -pbkdf2 -iter 600000 -a
```

> OpenSSL isn't bundled with Windows. It ships with **Git for Windows** (run this from Git Bash), or install it with `winget install ShiningLight.OpenSSL.Light` and reopen the terminal. For output byte-identical to Mac/Linux, encrypt in Git Bash — PowerShell uses CRLF line endings, which is harmless but shows slightly different spacing when decoded.

- Enter your passphrase when prompted
- Output is your ciphertext (starts with `U2FsdGVkX1`)
- Copy the entire ciphertext output

### 4. Paste ciphertext into index.html

Open `index.html` in the private repo, replace the demo `const DRAFT` value with your ciphertext.

### 5. Push to private repo

```bash
git add index.html
git commit -m "update ciphertext"
git push
```

The sync workflow triggers automatically and pushes `index.html` and `robots.txt` to the public repo. 
The workflow can also be triggered manually from GitHub → Actions → Sync to public repo → Run workflow.

### 6. Test recovery flow (mandatory)

1. Open incognito browser
2. Go to your GitHub Pages URL
3. Enter passphrase in the "Enter draft" input box → click Save
4. Confirm backup codes appear

Do not skip this step.

### 7. Memorize

- Primary URL: `yourusername.github.io/yourpublicrepo`
- Mirror 1 (optional): `yourproject.pages.dev` (if Cloudflare set up)
- Mirror 2 (optional): `yourproject.netlify.app` (if Netlify set up)
- Your strong passphrase

---

## Set up mirrors (optional)

Mirrors protect against GitHub Pages being unavailable. Only add mirrors on accounts you protect as strongly as your GitHub account.

**Cloudflare Pages:**
1. dash.cloudflare.com → Workers & Pages → Create → Pages → Connect to Git
2. Select your public repo → Deploy
3. URL: `yourproject.pages.dev`

**Netlify:**
1. app.netlify.com → Add new site → Import from Git
2. Select your public repo → Deploy
3. URL: `yourproject.netlify.app`

---

## Recovery - When you're locked out (from any device)

1. Open any browser → type your URL from memory (double-check the spelling, since a typo could land on a lookalike page)
2. When the page loads, enter your passphrase in the "Enter draft" input box → click **Save** (the label is deliberately misleading, part of the journal disguise)
3. Backup codes appear
4. Open Gmail → login → when asked for 2FA → choose **Use backup code**
5. Enter one code → you're in

---

## Immediately after recovery (recommended)

When back on a trusted device:

1. Gmail → Security → 2-Step Verification → Backup codes → Get new codes
2. Generate and memorize a **new passphrase**. The old one was typed on an untrusted device, so treat it as compromised (a keylogger that captured it can decrypt anything encrypted with it, since the ciphertext URL is public)
3. Encrypt the new codes with the new passphrase
4. Update `const DRAFT` in `index.html` → push to private repo → sync triggers automatically
5. Test the recovery flow again in incognito with the new passphrase

---

## Security notes

- Ciphertext is public by design — security is entirely in the passphrase
- Uses AES-256-CBC + PBKDF2-SHA256 (600,000 iterations) via browser Web Crypto API
- No external libraries loaded, support in all major browsers
- **The passphrase is the security; so create the ciphertext using a strong memorable password.**
- The private repo must stay private: its history contains your old ciphertexts and links your identity to your disguised public URL.


## License

MIT
