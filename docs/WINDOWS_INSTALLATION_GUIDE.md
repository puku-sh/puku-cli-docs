# Installing `@puku/puku-cli` on Windows

> A step-by-step guide to installing, updating, and troubleshooting `@puku/puku-cli` on Windows.

---

## Table of Contents

- [Prerequisites](#prerequisites)
  - [1. Install Node.js and npm](#1-install-nodejs-and-npm)
  - [2. Verify Your Installation](#2-verify-your-installation)
- [Installing `@puku/puku-cli`](#installing-pukupuku-cli)
- [Verifying the Installation](#verifying-the-installation)
- [Updating the Package](#updating-the-package)
- [Uninstalling and Reinstalling](#uninstalling-and-reinstalling)
- [Troubleshooting](#troubleshooting)
  - [Permission Errors](#permission-errors)
  - [Command Not Found / PATH Issues](#command-not-found--path-issues)
  - [npm Prefix / Global Directory Issues](#npm-prefix--global-directory-issues)
  - [ENOENT or Module Not Found](#enoent-or-module-not-found)

---

## Prerequisites

### 1. Install Node.js and npm

`@puku/puku-cli` requires **Node.js** (which bundles **npm**) to be installed on your system.

**Recommended: Use the official installer**

1. Go to [https://nodejs.org](https://nodejs.org)
2. Download the **LTS** (Long Term Support) version — recommended for most users
3. Run the `.msi` installer
4. On the *"Tools for Native Modules"* screen, check **"Automatically install the necessary tools"** if prompted
5. Complete the installation wizard

> **Tip:** If you manage multiple Node.js versions, consider using [nvm-windows](https://github.com/coreybutler/nvm-windows) instead of the direct installer.

---

### 2. Verify Your Installation

Open **Command Prompt** (`cmd`) or **PowerShell** and run:

```bash
node --version
npm --version
```

You should see output similar to:

```
C:\Users\poridhi>node --version
v22.15.0

C:\Users\poridhi>npm --version
10.9.2
```

> Make sure to update to the latest version. Older versions might show warnings.

If either command returns `'node' is not recognized` or `'npm' is not recognized`, see [PATH Issues](#command-not-found--path-issues) below.

---

## Installing `@puku/puku-cli`

Install the package globally so the `puku` command is available system-wide:

```bash
npm install -g @puku/puku-cli
```

> **Note for PowerShell users:** If you see a script execution policy error, run PowerShell as Administrator and execute:
> ```powershell
> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```
> Then retry the install command.

## Verifying the Installation

After installation, confirm the CLI is available:

```bash
puku-cli --version
```

Or check where it was installed:

```bash
npm list -g @puku/puku-cli
```

---

## Updating the Package

To update `@puku/puku-cli` to the latest version:

```bash
npm update -g @puku/puku-cli
```

Or install the latest version explicitly (this also works as an update):

```bash
npm install -g @puku/puku-cli@latest
```

**Check for outdated global packages at any time:**

```bash
npm outdated -g
```

> **Windows path tip:** If the update succeeds but the old version still runs, close and reopen your terminal. Windows caches PATH lookups in the current session.

---

## Uninstalling and Reinstalling

If an update fails or the CLI behaves unexpectedly, a clean uninstall and reinstall often resolves the issue.

**Step 1 — Uninstall:**

```bash
npm uninstall -g @puku/puku-cli
```

**Step 2 — Clear the npm cache:**

```bash
npm cache clean --force
```

**Step 3 — Reinstall:**

```bash
npm install -g @puku/puku-cli
```

**Step 4 — Verify:**

```bash
puku-cli --version
```

---

## Troubleshooting

### Permission Errors

**Symptom:** You see errors like:

```
npm WARN checkPermissions Missing write access to C:\Users\<you>\AppData\Roaming\npm
Error: EACCES: permission denied
```

**Solutions:**

**Option A — Run as Administrator (quickest fix)**

1. Press `Win + S`, search for **Command Prompt**
2. Right-click → **Run as administrator**
3. Re-run the install command

**Option B — Fix npm directory permissions (recommended for persistent issues)**

1. Open File Explorer and navigate to `C:\Users\<YourUsername>\AppData\Roaming\npm`  
   *(If the folder doesn't exist, create it)*
2. Right-click the `npm` folder → **Properties** → **Security** tab
3. Click **Edit**, select your user account, and check **Full Control**
4. Click **Apply** → **OK**
5. Retry the install without admin rights

**Option C — Change the global npm directory**

```bash
# Create a new directory for global packages
mkdir "%USERPROFILE%\npm-global"

# Point npm to it
npm config set prefix "%USERPROFILE%\npm-global"
```

Then add `%USERPROFILE%\npm-global` to your `PATH` (see the next section).

---

### Command Not Found / PATH Issues

**Symptom:** After installation, running `puku-cli` returns:

```
'puku-cli' is not recognized as an internal or external command
```

**Why this happens:** npm installs global CLI binaries into a directory that may not be in your Windows `PATH`.

**Step 1 — Find where npm installs global binaries:**

```bash
npm config get prefix
```

This returns a path like `C:\Users\<you>\AppData\Roaming\npm`. Global CLI binaries live here.

**Step 2 — Add that path to your system PATH:**

1. Press `Win + S`, search **"Edit the system environment variables"**, open it
2. Click **Environment Variables**
3. Under **User variables**, select **Path** and click **Edit**
4. Click **New** and paste the path from Step 1 (e.g. `C:\Users\<you>\AppData\Roaming\npm`)
5. Click **OK** on all dialogs

**Step 3 — Restart your terminal** (any open terminal will not pick up the new PATH until restarted)

**Step 4 — Verify:**

```bash
where puku-cli
```

---

### npm Prefix / Global Directory Issues

**Symptom:** npm installs packages in an unexpected location, or you have multiple conflicting npm installations (e.g. one from the Node.js installer and one from nvm-windows).

**Diagnose:**

```bash
# Check current prefix
npm config get prefix

# Check the full config chain
npm config list
```

**Reset the prefix to the default:**

```bash
npm config delete prefix
```

**If using nvm-windows**, avoid setting a global `prefix` manually — let nvm manage the global directory per Node version.

---

### ENOENT or Module Not Found

**Symptom:** Errors referencing missing files or paths with double slashes or incorrect separators, e.g.:

```
Error: Cannot find module 'C:\Users\<you>\AppData\Roaming\npm\node_modules\@puku\puku-cli\...'
```

**Solutions:**

1. **Uninstall and reinstall cleanly** (see [Uninstalling and Reinstalling](#uninstalling-and-reinstalling))

2. **Check for corrupt cache entries:**

   ```bash
   npm cache verify
   npm cache clean --force
   ```

3. **Check for spaces in your Windows username path.** Paths like `C:\Users\John Doe\...` can break some packages. Workaround — set a global prefix without spaces:

   ```bash
   npm config set prefix "C:\npm-global"
   ```

   Then add `C:\npm-global` to your PATH as described in the [PATH Issues](#command-not-found--path-issues) section.

4. **Reinstall Node.js** if the issue persists — a corrupted or partial Node.js installation can cause module resolution failures.

---

## Additional Resources

- [Node.js Official Downloads](https://nodejs.org/en/download)
- [npm Documentation](https://docs.npmjs.com)
- [`@puku/puku-cli` on npm](https://www.npmjs.com/package/@puku/puku-cli)
- [nvm-windows (optional Node version manager)](https://github.com/coreybutler/nvm-windows)

---

> Found an issue with this guide? Open a pull request or file an issue.