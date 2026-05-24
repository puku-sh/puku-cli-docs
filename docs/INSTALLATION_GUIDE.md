# Installing `@puku/puku-cli`

> A step-by-step guide to installing, updating, and troubleshooting `@puku/puku-cli` on **macOS**, **Linux**, and **Windows**.

---

## Table of Contents

- [Prerequisites](#prerequisites)
  - [macOS — Install Node.js](#macos--install-nodejs)
  - [Linux — Install Node.js](#linux--install-nodejs)
  - [Windows — Install Node.js](#windows--install-nodejs)
  - [Verify Your Node.js Installation](#verify-your-nodejs-installation)
- [Installing `@puku/puku-cli`](#installing-pukupuku-cli)
- [Verifying the Installation](#verifying-the-installation)
- [Updating the Package](#updating-the-package)
- [Uninstalling and Reinstalling](#uninstalling-and-reinstalling)
- [Troubleshooting](#troubleshooting)
  - [Permission Errors (macOS / Linux)](#permission-errors-macos--linux)
  - [Permission Errors (Windows)](#permission-errors-windows)
  - [Command Not Found / PATH Issues](#command-not-found--path-issues)
  - [npm Prefix / Global Directory Issues](#npm-prefix--global-directory-issues)
  - [ENOENT or Module Not Found](#enoent-or-module-not-found)
- [Additional Resources](#additional-resources)

---

## Prerequisites

`@puku/puku-cli` requires **Node.js v18 or later** (which bundles **npm**). Follow the instructions for your operating system below.

---

### macOS — Install Node.js

**Option A — Official installer (simplest)**

1. Go to [https://nodejs.org](https://nodejs.org)
2. Download the **LTS** version
3. Open the `.pkg` file and follow the installation wizard

**Option B — Homebrew (recommended for developers)**

If you have [Homebrew](https://brew.sh) installed:

```bash
brew install node
```

**Option C — nvm (recommended for managing multiple Node versions)**

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload your shell config
source ~/.zshrc   # or source ~/.bashrc if you use bash

# Install the latest LTS release
nvm install --lts
nvm use --lts
```

---

### Linux — Install Node.js

**Option A — NodeSource repository (recommended for Debian/Ubuntu)**

```bash
# Add the NodeSource LTS repository and install
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Option B — Package manager**

*Debian / Ubuntu:*
```bash
sudo apt update
sudo apt install -y nodejs npm
```

*Fedora / RHEL / CentOS:*
```bash
sudo dnf install -y nodejs npm
```

*Arch Linux:*
```bash
sudo pacman -S nodejs npm
```

> **Note:** Distro-packaged Node.js versions can be outdated. If the version is below v18, use nvm or the NodeSource method above.

**Option C — nvm (recommended for managing multiple Node versions)**

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload your shell config
source ~/.bashrc   # or source ~/.zshrc

# Install the latest LTS release
nvm install --lts
nvm use --lts
```

---

### Windows — Install Node.js

**Option A — Official installer (simplest)**

1. Go to [https://nodejs.org](https://nodejs.org)
2. Download the **LTS** `.msi` installer
3. Run the installer; on the *"Tools for Native Modules"* screen, check **"Automatically install the necessary tools"** if prompted
4. Complete the installation wizard

**Option B — winget**

```powershell
winget install OpenJS.NodeJS.LTS
```

**Option C — nvm-windows (for managing multiple Node versions)**

Download and run the installer from [nvm-windows releases](https://github.com/coreybutler/nvm-windows/releases), then:

```powershell
nvm install lts
nvm use lts
```

---

### Verify Your Node.js Installation

Run the following in your terminal (any OS):

```bash
node --version
npm --version
```

Expected output (versions may vary):

```
v22.15.0
10.9.2
```

> Ensure Node.js is **v18 or later**. If either command is not found, see [PATH Issues](#command-not-found--path-issues).

---

## Installing `@puku/puku-cli`

Install the package globally so the `puku` command is available system-wide. The command is the same on all platforms:

```bash
npm install -g @puku/puku-cli
```

> **macOS / Linux:** If you see a permission error (EACCES), see [Permission Errors (macOS / Linux)](#permission-errors-macos--linux) before using `sudo`.
>
> **Windows (PowerShell):** If you see a script execution policy error, run PowerShell as Administrator and execute:
> ```powershell
> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```
> Then retry the install command.

---

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

Or install the latest version explicitly (also works as an update):

```bash
npm install -g @puku/puku-cli@latest
```

Check for outdated global packages at any time:

```bash
npm outdated -g
```

> **Windows:** If the update succeeds but the old version still runs, close and reopen your terminal. Windows caches PATH lookups in the current session.
>
> **macOS / Linux:** If you installed with nvm, make sure the correct Node version is active (`nvm use --lts`) before updating.

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

### Permission Errors (macOS / Linux)

**Symptom:** You see errors like:

```
npm WARN checkPermissions Missing write access to /usr/local/lib/node_modules
Error: EACCES: permission denied
```

**Do not use `sudo npm install -g`** — it can cause ownership issues that break future npm commands.

**Option A — Use nvm (recommended)**

nvm installs Node.js and the global npm directory under your home folder, so no elevated permissions are needed. Install nvm as described in [Linux — Install Node.js](#linux--install-nodejs) or [macOS — Install Node.js](#macos--install-nodejs), then reinstall.

**Option B — Change the npm global directory**

```bash
# Create a directory for global packages in your home folder
mkdir -p ~/.npm-global

# Point npm to it
npm config set prefix '~/.npm-global'
```

Then add the new bin directory to your PATH. Add the following line to your `~/.zshrc`, `~/.bashrc`, or `~/.profile`:

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

Reload your shell:

```bash
source ~/.zshrc   # or source ~/.bashrc
```

Then retry the install:

```bash
npm install -g @puku/puku-cli
```

---

### Permission Errors (Windows)

**Symptom:** You see errors like:

```
npm WARN checkPermissions Missing write access to C:\Users\<you>\AppData\Roaming\npm
Error: EACCES: permission denied
```

**Option A — Run as Administrator (quickest fix)**

1. Press `Win + S`, search for **Command Prompt** or **PowerShell**
2. Right-click → **Run as administrator**
3. Re-run the install command

**Option B — Fix npm directory permissions (recommended for persistent issues)**

1. Open File Explorer and navigate to `C:\Users\<YourUsername>\AppData\Roaming\npm`
   *(Create the folder if it doesn't exist)*
2. Right-click the `npm` folder → **Properties** → **Security** tab
3. Click **Edit**, select your user account, and check **Full Control**
4. Click **Apply** → **OK**
5. Retry the install without admin rights

**Option C — Change the global npm directory**

```powershell
# Create a new directory for global packages
mkdir "$env:USERPROFILE\npm-global"

# Point npm to it
npm config set prefix "$env:USERPROFILE\npm-global"
```

Then add `%USERPROFILE%\npm-global` to your PATH (see the next section).

---

### Command Not Found / PATH Issues

**Symptom:** After installation, running `puku-cli` returns:

```
# macOS / Linux
puku-cli: command not found

# Windows
'puku-cli' is not recognized as an internal or external command
```

**Why this happens:** npm installs global CLI binaries into a directory that may not be in your `PATH`.

**Step 1 — Find where npm installs global binaries:**

```bash
npm config get prefix
```

Global CLI binaries live in the `bin/` subfolder of that path (e.g. `~/.npm-global/bin` on macOS/Linux, or `C:\Users\<you>\AppData\Roaming\npm` on Windows).

**Step 2 — Add that path to PATH:**

*macOS / Linux* — add to `~/.zshrc`, `~/.bashrc`, or `~/.profile`:

```bash
export PATH="$(npm config get prefix)/bin:$PATH"
```

Then reload: `source ~/.zshrc` (or `~/.bashrc`).

*Windows* —

1. Press `Win + S`, search **"Edit the system environment variables"** and open it
2. Click **Environment Variables**
3. Under **User variables**, select **Path** and click **Edit**
4. Click **New** and paste the path from Step 1 (e.g. `C:\Users\<you>\AppData\Roaming\npm`)
5. Click **OK** on all dialogs
6. Restart your terminal

**Step 3 — Verify:**

```bash
# macOS / Linux
which puku-cli

# Windows
where puku-cli
```

---

### npm Prefix / Global Directory Issues

**Symptom:** npm installs packages in an unexpected location, or you have multiple conflicting npm installations (e.g. one from the Node.js installer and one from nvm).

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

> **nvm users (all platforms):** Avoid setting a global `prefix` manually — let nvm manage the global directory per Node version.

---

### ENOENT or Module Not Found

**Symptom:** Errors referencing missing files or incorrect paths:

```
# Windows
Error: Cannot find module 'C:\Users\<you>\AppData\Roaming\npm\node_modules\@puku\puku-cli\...'

# macOS / Linux
Error: Cannot find module '/usr/local/lib/node_modules/@puku/puku-cli/...'
```

**Solutions:**

1. **Uninstall and reinstall cleanly** (see [Uninstalling and Reinstalling](#uninstalling-and-reinstalling))

2. **Check for a corrupt cache:**

   ```bash
   npm cache verify
   npm cache clean --force
   ```

3. **Windows only — Check for spaces in your username path.** Paths like `C:\Users\John Doe\...` can break some packages. Set a global prefix without spaces:

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
- [nvm (macOS / Linux)](https://github.com/nvm-sh/nvm)
- [nvm-windows](https://github.com/coreybutler/nvm-windows)
- [Homebrew (macOS)](https://brew.sh)

---

> Found an issue with this guide? Open a pull request or file an issue.
