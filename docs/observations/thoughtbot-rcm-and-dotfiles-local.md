# Observations: thoughtbot/rcm + dotfiles-local setup

Discovery date: 2025-02-15. For use when planning migration (e.g. to chezmoi) or other dotfiles work.

---

## Repos and roles

- **~/dotfiles** (this repo): Fork of thoughtbot/dotfiles. Origin: `ralphreid/dotfiles`, upstream: `thoughtbot/dotfiles`. Base set: vim, zsh, git, tmux, aliases, bin scripts.
- **~/dotfiles-local**: Override layer only. Origin: `ralphreid/dotfiles-local` (private). No upstream. Holds only local/custom files.

---

## rcm and rcrc

- Tool: rcm (`rcup` to apply); config: `~/.rcrc` (from this repo’s `rcrc`).
- **rcrc**: `DOTFILES_DIRS="$HOME/dotfiles-local $HOME/dotfiles"` (local first, overrides win). `EXCLUDES="*.md LICENSE"`. `COPY_ALWAYS="git_template/HEAD"`.
- Install: `env RCRC=$HOME/dotfiles/rcrc rcup` once; thereafter `rcup` only.

---

## What lives where

**Base (this repo):** zshenv, zprofile, zshrc, aliases; ~/.zsh/{functions,configs,completion}; gitconfig, gitignore, gitmessage, git_template/ (hooks); vimrc, vimrc.bundles, vim/; tmux.conf, psqlrc, rspec, ctags, agignore, gemrc, hushlogin, asdfrc; bin/.

**Overrides (dotfiles-local):** zshrc.local, aliases.local; zsh/completion/_docker-compose, zsh/configs/post/chpwd; gitconfig.local (+ includeIf), gitconfig-me.local, gitconfig-other.local, gitignore.local; git_template.local/hooks/; vimrc.local, vimrc.bundles.local; tool-versions, default-gems, default-npm-packages; aider.conf.yml; .psqlrc.local (blank).

**Home:** Almost all dotfile symlinks point into one repo or the other. ~/.bin is a real directory of symlinks to ~/dotfiles/bin/*. ~/.git_template and ~/.zsh are real dirs with per-file symlinks from both repos (rcm merges DOTFILES_DIRS).

---

## Load order and delegation

- **Zsh:** zshenv → .zshenv.local → zprofile → zshrc → ~/.zsh/functions → _load_settings ~/.zsh/configs (pre → main → post) → .zshrc.local → .aliases → .aliases.local → fzf, iTerm2, Docker completions.
- **Git:** Base gitconfig `[include] path = ~/.gitconfig.local`. gitconfig.local uses includeIf "gitdir:..." to choose gitconfig-me.local (e.g. ~/dev/me, ~/dotfiles, ~/dotfiles-local, specific projects) or gitconfig-other.local.
- **Vim:** vimrc sources .vimrc.bundles then .vimrc.local. Base git hooks source ~/.git_template.local/hooks/<hook> if present.

---

## Notable technical details

- PATH: path.zsh sets $HOME/.bin first and asdf; zshrc.local adds Homebrew sbin, curl, pipx, .local/bin, AWS, mcfly, pyenv, conda, Go, direnv, etc.
- ASDF: Base loads asdf; versions from dotfiles-local tool-versions. default-gems / default-npm-packages in dotfiles-local.
- post-up (hooks/post-up): After rcup, ensures .psqlrc.local exists, vim-plug upgrade, PlugUpdate/PlugClean!, and cleans old git_template/HEAD if main.
- Trunk: This repo and dotfiles-local each have .trunk/trunk.yaml; independent of thoughtbot layout.
- Identities: Git identity by directory via includeIf and two .local gitconfigs.

---

## For future migration (e.g. chezmoi)

- Two-tier layout (base + local) implemented via rcm DOTFILES_DIRS and per-dotfile symlinks; merged dirs ~/.zsh, ~/.bin.
- Sensitive/machine-specific content is in dotfiles-local (identities, paths, AWS, conda); map to private or machine-specific templates.
- Git: includeIf + multiple identity files; git template base + optional local hooks → map to templates and run_/run_once_.
- Zsh: Layered load order and shared ~/.zsh tree (pre/config/post + local chpwd, _docker-compose); preserve in new layout.
- Vim: Base bundles + local bundles and vimrc.local; vim-plug and post-up PlugUpdate; spellfile and other paths assume rcm-managed home.
- Bin: Only base provides bin/ today; ~/.bin is all symlinks to ~/dotfiles/bin.
- Dependencies: rcm (brew), asdf (brew + dotfiles-local tool-versions/default-gems/default-npm-packages), vim-plug, Homebrew; optional mcfly, direnv, conda. List what stays system vs managed.

---

*Other observation docs can be added under `docs/observations/` with distinct filenames.*
