# VS Code User Configuration in Git

This repository stores **only the human‑readable parts** of your Visual Studio Code setup (settings, keybindings, snippets) plus a **generated list of extensions**. That makes it easy to:

* Keep VS Code config under version control
* Recreate your setup on a new machine (e.g. your laptop) in minutes
* Track changes over time (and roll back if needed)

> Repo URL: `https://github.com/onno-schuit/vscode-config` (replace with your own if you fork).

---

## Repo Location

Clone (or initialize) this Git repo **directly inside** `~/.config/Code/User`.

That directory is where VS Code stores your user-level settings on Linux, so having the repo live there means you commit the files *in place*—no copying back and forth. If you want to keep the `.git` metadata elsewhere, see the *Separate Git dir* section below.

---

## TL;DR

* **Track:** `settings.json`, `keybindings.json`, `snippets/`, `extensions.txt`
* **Ignore:** `globalStorage/`, `workspaceStorage/`, `History/`, `logs/`, etc.
* **Update after changes:**

  ```bash
  code --list-extensions --show-versions | sort > extensions.txt
  git add settings.json keybindings.json snippets extensions.txt
  git commit -m "Update VS Code config"
  git push
  ```
* **Deploy on a new machine (e.g. your laptop):**

  ```bash
  rm -rf ~/.config/Code/User    # careful!
  git clone git@github.com:onno-schuit/vscode-config.git ~/.config/Code/User
  xargs -a ~/.config/Code/User/extensions.txt -I{} code --install-extension {}
  ```

---

## What’s in this repo?

```text
~/.config/Code/User
├── settings.json        # Your editor & UI settings
├── keybindings.json     # Custom keybindings
├── snippets/            # Per-language or global snippets folder
├── extensions.txt       # Auto-generated list of extensions (with versions)
└── .gitignore           # Ignore noisy, machine-specific folders
```

### .gitignore template

```gitignore
# VS Code user data that shouldn't be versioned
globalStorage/
workspaceStorage/
History/
logs/
sync/
**/token*
```

Adjust this list as needed if `git status` shows more noisy files.

---

## Initial Setup (on an existing machine)

If you’re reading this, you’ve likely already done these steps. For completeness:

```bash
cd ~/.config/Code/User

# Initialize repo
git init

# (Optional) set your remote
git remote add origin git@github.com:onno-schuit/vscode-config.git

# Generate extension list
code --list-extensions --show-versions | sort > extensions.txt

# Commit tracked files
git add settings.json keybindings.json snippets extensions.txt .gitignore
git commit -m "Initial VS Code configuration"

git push -u origin master   # or main, if preferred
```

> If you prefer to keep `.git` elsewhere (to keep this folder clean), see the **Separate Git dir** section below.

---

## Updating the Repo After You Change Something

Whenever you tweak settings, keybindings, or snippets, or add/remove extensions:

```bash
# 1. Refresh the extension list (optional but recommended)
code --list-extensions --show-versions | sort > extensions.txt

# 2. Add and commit relevant files
git add settings.json keybindings.json snippets extensions.txt
git commit -m "Update VS Code config"

git push
```

### Automate extension list updates (pre-commit hook)

Create `.git/hooks/pre-commit` in this repo:

```bash
#!/usr/bin/env bash
set -e
code --list-extensions --show-versions | sort > extensions.txt
git add extensions.txt
```

Make it executable:

```bash
chmod +x .git/hooks/pre-commit
```

Now, every commit automatically refreshes `extensions.txt`.

---

## Re-Deploy on a New Ubuntu Installation (e.g. your laptop)

1. **Install VS Code** and ensure the `code` CLI is available.
2. **Clone the repo into place:**

   ```bash
   rm -rf ~/.config/Code/User    # careful! only if you’re sure
   git clone git@github.com:onno-schuit/vscode-config.git ~/.config/Code/User
   ```
3. **Install extensions:**

   ```bash
   xargs -a ~/.config/Code/User/extensions.txt -I{} code --install-extension {}
   ```

Done! VS Code should now reflect your exact settings and extensions.

---

## Optional: Separate Git Dir (keep `.git` outside the User folder)

If you don’t like having a `.git` folder inside `~/.config/Code/User`:

```bash
mkdir -p ~/.dotrepos
git init --separate-git-dir ~/.dotrepos/vscode.git ~/.config/Code/User

# Create a handy alias
alias vgit='git --git-dir=$HOME/.dotrepos/vscode.git --work-tree=$HOME/.config/Code/User'

# Use vgit instead of git
vgit status
vgit add settings.json keybindings.json snippets extensions.txt
vgit commit -m "Update VS Code config"
```

You can put that alias into your shell config (`~/.bashrc`, `~/.zshrc`, etc.).

---

## Troubleshooting

* **Ignored folders still show up in ********************`git status`********************:**
  They might be *already tracked*. Remove them from the index and commit:

  ```bash
  git rm -r --cached globalStorage workspaceStorage History logs sync
  git commit -m "Stop tracking noisy VS Code folders"
  ```

* **Extensions fail to install in bulk:**
  Sometimes the network or Marketplace throttles you. Just re-run:

  ```bash
  xargs -a extensions.txt -I{} code --install-extension {}
  ```

* **`code`**\*\* command not found:\*\*
  VS Code may not have added itself to PATH. Use "Shell Command: Install 'code' command in PATH" from the Command Palette (macOS) or reinstall VS Code on Linux.

---

## License / Notes

* This repo mostly contains your personal configuration JSON; no special license is required unless you’d like one.
* Don’t store secrets in `settings.json`. Use environment variables or a separate ignored file.
* If you also use **VS Code Insiders**, repeat this setup for `~/.config/Code - Insiders/User/` or script the detection.

---

Happy coding! If you need this merged into a larger dotfiles bootstrap, or want Windows/macOS equivalents, add an issue or ping me.

---

## PHP\_CodeSniffer & Moodle Coding Style

You don’t need to commit PHPCS itself—just install it quickly on your laptop (or any new machine). Here’s the exact sequence I ran on my laptop (and reuse on any other machine):

### 1. Install PHP & Composer prerequisites

```bash
sudo apt update
sudo apt install -y php-cli php-xml php-mbstring php-zip php-curl php-intl composer
```

### 2. Configure Composer globally

```bash
# Allow the PHPCS installer plugin
composer global config allow-plugins.dealerdirect/phpcodesniffer-composer-installer true

# So dev-only deps like phpcompatibility can be installed
composer global config minimum-stability dev
composer global config prefer-stable true
```

### 2b. If the distro Composer crashes (e.g. "Normalizer not found") or you prefer the official installer

```bash
mkdir -p ~/.local/bin
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=$HOME/.local/bin --filename=composer
rm composer-setup.php
# Ensure it's on PATH for this shell and future sessions
export PATH="$HOME/.local/bin:$PATH"
```

> After installing Composer this way, you can remove the distro package (`sudo apt remove composer`) to avoid conflicts.

### 3. Install PHPCS + Moodle rules (one command)

```bash
composer global require -W \
  squizlabs/php_codesniffer:^3 \
  dealerdirect/phpcodesniffer-composer-installer:^0.7.2 \
  moodlehq/moodle-cs:^3
```

> The `-W` flag lets Composer resolve/depend on whatever versions Moodle CS needs (including dev branches).

### 4. Ensure the Composer bin dir is on your PATH

```bash
# Usually this is the path used by Composer:
export PATH="$HOME/.config/composer/vendor/bin:$PATH"
# Add that line to ~/.bashrc or ~/.zshrc for permanence, then reload your shell
```

### 5. Verify PHPCS sees Moodle

```bash
~/.config/composer/vendor/bin/phpcs -i
# Expect to see: moodle (and possibly PHPCompatibility, PHPCSUtils, etc.)
```

### 6. VS Code integration (user settings)

Make sure you have the extension **wongjn.php-sniffer** installed, then add/update these keys in `settings.json`:

```jsonc
// Disable other formatters
"php.format.enable": false,
"php-cs-fixer.documentFormattingProvider": false,

// Use PHPCS for PHP
"[php]": {
  "editor.defaultFormatter": "wongjn.php-sniffer"
},

// Wongjn settings
"phpSniffer.executablesFolder": "/home/onno/.config/Code/User/phpcs-bin", // script that suppresses the output of phpcs
"phpSniffer.autoDetect": false,
"phpSniffer.standard": "Moodle" // or a path to a local phpcs.xml
```

> If you ever see missing-sniff errors (e.g., NormalizedArrays, Universal, etc.), either stick to the base `Moodle` standard or install those extra standards. For most Moodle work you only need `Moodle`.

### 7. (Optional) Project-specific ruleset

If you edit older Moodle branches, create a `phpcs.xml` in that project to pin PHPCompatibility’s version range:

```xml
<?xml version="1.0"?>
<ruleset name="Moodle 3.9 Ruleset">
  <rule ref="Moodle"/>
  <rule ref="PHPCompatibility">
    <properties>
      <property name="testVersion" value="7.0-7.4"/>
    </properties>
  </rule>
  <exclude-pattern>*/vendor/*</exclude-pattern>
  <exclude-pattern>*/node_modules/*</exclude-pattern>
</ruleset>
```

Then in that workspace’s `.vscode/settings.json`:

```jsonc
"phpSniffer.standard": "${workspaceFolder}/phpcs.xml"
```

### 8. Troubleshooting quickies

* **`spawn ENOTDIR`**: `phpSniffer.executablesFolder` must be a directory, not a file.
* **`Referenced sniff ... does not exist`**: You’re using a ruleset that references a sniff you haven’t installed. Use base `Moodle` or install the missing standards.
* **VS Code not picking changes**: `Ctrl+Shift+P → Developer: Reload Window`.
* **Confirm command**: Check Developer Tools console to see what command the extension runs and execute it manually in a terminal.

---
