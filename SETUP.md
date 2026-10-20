# Setup guide: rnaseq-first

How to reproduce this project's environment and Git/GitHub setup from scratch.

## Mental model

```
working folder --git add--> staging --git commit--> local repo --git push--> GitHub
                                                                  (remote "origin")
```

- `origin` is a nickname for the GitHub repo URL.
- `main` is the branch name; it must match locally and on GitHub.
- `.gitignore` lists what Git skips (data, caches, notebook checkpoints).
- A conda environment is an isolated Python plus its packages, one per project.

## 1. Conda channels (once per machine)

```bash
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict
```

## 2. Git identity (once per machine)

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

## 3. GitHub authentication (pick one)

**SSH**

```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub     # paste into GitHub > Settings > SSH and GPG keys
ssh -T git@github.com         # should greet you by username
```

**HTTPS**: no key needed; a browser login appears on the first push.

## 4. Project folder

```bash
mkdir -p ~/projects/rnaseq-first && cd ~/projects/rnaseq-first
git init
mkdir data results figures notebooks
printf "data/\n.ipynb_checkpoints/\n__pycache__/\n" > .gitignore   # Git Bash only
```

## 5. Environment

Check that the prompt says `(rnaseq)` before installing anything.

```bash
conda create -n rnaseq python=3.11 -y
conda activate rnaseq
conda install -y pandas numpy scipy matplotlib seaborn scikit-learn jupyterlab biopython
pip install pydeseq2 gseapy GEOparse
conda env export --from-history > environment.yml
```

Order matters: conda first, pip last.

## 6. VS Code

1. Install the **Python** and **Jupyter** extensions.
2. Open the project folder: `code .`
3. Open or create a `.ipynb` file in `notebooks/`.
4. Select Kernel > Python Environments > `rnaseq`.
5. Test: `import pandas, pydeseq2`

## 7. First push

1. Create an **empty** repo on GitHub (no README, no .gitignore).
2. Then:

```bash
git add .
git commit -m "Initial project setup"
git branch -M main
git remote add origin git@github.com:USERNAME/rnaseq-first.git   # or the https:// URL
git push -u origin main
```

If `origin` already exists, use `git remote set-url origin <URL>` instead of `add`.
The URL must have no spaces and no `<angle brackets>`.

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Permission denied (publickey)` | No SSH key on GitHub | Add the key, or use HTTPS |
| `HTTP 000 CONNECTION FAILED` | Network issue | Retry, check VPN/firewall, install in smaller batches |
| `src refspec main does not match any` | Local branch is `master` | `git branch -M main` |
| `remote add` prints usage text | Space or placeholder in the URL | `git remote set-url origin <exact URL>` |
| `Did not find path entry ...` on activate | Stale PATH entry | Harmless warning |
| `grep` / `printf` not found | Using PowerShell | Use Git Bash, or `conda list pandas` |

## Daily routine

```bash
cd ~/projects/rnaseq-first
conda activate rnaseq
code .
# ...work...
git add . && git commit -m "what I did" && git push
```

## Rollback notes

- Installed into `base` by mistake: `conda list --revisions -n base`, then `conda install -n base --revision <N>` (N = the revision before the mistake). Remove pip installs separately with `pip uninstall`.
- Data is not in the repo (`data/` is ignored). Record the GEO accession and download steps in the README.
