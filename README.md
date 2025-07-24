# VS Code User Configuration in Git

This repository stores **only the human‑readable parts** of your Visual Studio Code setup (settings, keybindings, snippets) plus a **generated list of extensions**. That makes it easy to:

* Keep VS Code config under version control
* Recreate your setup on a new machine in minutes
* Track changes over time (and roll back if needed)

> Repo URL: `https://github.com/onno-schuit/vscode-config` (replace with your own if you fork).

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
* **Deploy on a new machine:**

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

## Re-Deploy on a New Ubuntu Installation

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

* **Ignored folders still show up in `git status`:**
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

* **`code` command not found:**
  VS Code may not have added itself to PATH. Use "Shell Command: Install 'code' command in PATH" from the Command Palette (macOS) or reinstall VS Code on Linux.

---

## License / Notes

* This repo mostly contains your personal configuration JSON; no special license is required unless you’d like one.
* Don’t store secrets in `settings.json`. Use environment variables or a separate ignored file.
* If you also use **VS Code Insiders**, repeat this setup for `~/.config/Code - Insiders/User/` or script the detection.

---

Happy coding! If you need this merged into a larger dotfiles bootstrap, or want Windows/macOS equivalents, add an issue or ping me.
