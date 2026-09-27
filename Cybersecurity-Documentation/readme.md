# Cybersecurity Documentation: Securing the Social Sentinel Project

This document describes how the **Social Sentinel** team protects the project itself: its repository, code, data, model, and deployed application.

This document covers every layer of the project:

```text
[ Team Accounts & Repository Access ]
        │
        ▼
[ Contribution Workflow ] ──► [ Secrets & Credentials ]
        │
        ▼
[ Dependencies (Supply Chain) ]
        │
        ▼
[ Dataset ] ──► [ Trained Model ]
        │
        ▼
[ Deployed Application (Gradio) ] ──► [ End Users ]
```

---

## 1. Repository and Access Security

The repository (`Wecncode/Social-Sentinel`) is the single source of truth. Whoever controls it controls what users run.

**Central repository with forks* [In place] - Contributors work in personal forks; only the team repository is authoritative.
*Branch protection on `main`* [TEAM TO CONFIRM]- Blocks direct pushes and force-pushes to `main`.
*Required pull request reviews* [Recommended]- At least one approving review from another team member before merging. Contributors should not merge their own pull requests.
*Two-factor authentication (2FA)*  [Recommended]-Every member with write access enables 2FA on GitHub, preventing account takeover through stolen passwords.
*Least-privilege roles* [Recommended]- Only maintainers hold admin or write access; other contributors submit changes through pull requests.
*Periodic access review* [Recommended]- Remove access for members who leave the project.

---

## 2. Secure Contribution Workflow

Every change reaches `main` through a visible, reviewable path.

```text
[ Fork ] ──► [ Sync with upstream/main ] ──► [ Feature branch ]
                                                    │
                                                    ▼
[ Merge to main ] ◄── [ Review & approval ] ◄── [ Pull request ]
```

*Fork-and-pull-request model* [In place]- All changes are proposed through pull requests to the team repository.
*Pull request template* [In place]- `.github/PULL_REQUEST_TEMPLATE.md` standardizes what each contributor describes.
*Bug report template* [In place] `.github/Issue_Template/bug_report.yml` provides a consistent reporting channel.
**Sync before new work* [In place]- Contributors fetch and compare with `upstream/main` before starting, reducing conflicts and accidental overwrites.
*Pull request target check*  [ Recommended] Confirm the pull request's base repository is `Wecncode/Social-Sentinel` before submitting, since GitHub may default to the contributor's own fork.
*Security review checklist* [ Recommended] Reviewers check every pull request against the checklist in Section 10.

---

## 3. Secrets and Credentials

API keys, access tokens, and passwords must never be stored in the repository. Anything committed to a public repository should be treated as public, even if deleted later, because it remains in Git history and may already have been copied.

*No secrets in code*  [TEAM TO CONFIRM]  Confirm that no keys, tokens, or passwords exist in the code, notebooks, or history. 
*`.gitignore` for sensitive files* [Recommended ] Exclude `.env`, `venv/`, `__pycache__/`, `.ipynb_checkpoints/`, and local model files.
*Environment variables* [Recommended ] Load any future credentials from environment variables or the hosting platform's secret store, never from source files. 
*GitHub secret scanning and push protection*  [Recommended ] Enable in repository settings to block commits containing recognizable secrets. 
*Clean notebook outputs* [Recommended ] Clear notebook outputs before committing; outputs can contain file paths, public share URLs, or data samples. 

### If a Secret Is Accidentally Committed

1. **Revoke or rotate the secret immediately.** This is the only step that actually removes the risk.
2. Notify the maintainers.
3. Remove the secret from the code and, if required, from Git history (e.g., with `git filter-repo`), coordinating with the team because rewriting history affects every fork.

---

## 4. Dependency and Supply-Chain Security

Social Sentinel depends on third-party packages (scikit-learn, pandas, NumPy, Gradio). A compromised or vulnerable package becomes a vulnerability in the project.

*Pinned package versions*  [In place] `requirements.txt` specifies exact versions, so every environment runs the same known code and new releases are not pulled in silently.
*Minimal dependency surface* [In place ] The project relies on a small set of widely used libraries.
*Valid `requirements.txt`* [Recommended ] The file currently contains `python==3.11.8`, which pip cannot install and which stops installation. Record the Python version in the README or a `.python-version` file instead.
*Isolated environments*  [Recommended ] Each contributor installs dependencies in a virtual environment (`venv`), never system-wide.
*Vulnerability scanning* [Recommended ] Run `pip-audit` before releases and enable GitHub Dependabot alerts to flag packages with known vulnerabilities.
*Vetting new packages* [Recommended ] Before adding a dependency, confirm it is actively maintained and check the exact package name to avoid look-alike (typosquatted) packages.

---

## 5. Dataset Security

The model learns only from its data. Whoever can change the dataset can change what the model considers "safe."

*Dataset versioning* [In place ] The training set is versioned (`social-sentinel-training-dataset-v1.csv`), so changes can be traced and reviewed like code.
*Separate training and test sets* [In place ] `dataset/training-dataset/` and `dataset/test-dataset/` are kept apart, so evaluation reflects unseen data.
*Review of dataset changes* [Recommended ] Every pull request that modifies a dataset receives label spot-checks, guarding against **data poisoning** (e.g., phishing samples deliberately labeled as safe).
*Removal of personal data* [TEAM TO CONFIRM] Real names, phone numbers, account numbers, and email addresses are replaced with placeholders.
*Defanged links* [TEAM TO CONFIRM]  Malicious URLs in samples are neutralized (e.g., `hxxp://example[.]com`) so no one can click them by accident.
*Documented data sources* [TEAM TO CONFIRM] Record where samples came from (team-written, collected, or public datasets) and confirm the team may use them.

---

## 6. Model Security

**Adversarial evasion:** attackers alter spelling (`v3rify`, `acc0unt`) or insert characters to slip past word-based TF-IDF features [Recommended ]. Add obfuscated phishing samples to the training data; consider character-level n-grams, which are more robust to misspellings.
**Malicious model files:** saved scikit-learn models use pickle/joblib, which can execute arbitrary code when loaded [Recommended ] Only load model files the team produced itself; never load models from untrusted sources; record a checksum (hash) of each released model file.
**Silent performance regression:** a change degrades detection without anyone noticing  [Recommended ]Evaluate every retrained model on the held-out test set and record accuracy and false-negative rates before release.
**Overconfidence:** a "safe" verdict gives users false assurance [Recommended ] Display a notice that the tool supports, and does not replace, independent verification.

---

## 7. Application and Deployment Security

*Empty input rejection* [In place ] The analysis function rejects empty messages before inference.
*Public share links* [Recommended ] The notebook launches Gradio with `share=True`, which creates a public URL that anyone with the link can use for up to one week. Use `share=False` during development and create share links only for scheduled demos.
*Debug mode off in deployment* [Recommended ] Run with `debug=False` outside development so error details are not exposed.
*Input length limit* [Recommended ] Cap message length to prevent excessive resource use.
*No storage of user messages*  [TEAM TO CONFIRM] Messages are analyzed in memory and are not logged, stored, or sent to third parties. Users may paste sensitive content, so this should remain true.
*Hosting configuration* [Recommended ] When deployed (e.g., Hugging Face Spaces), store any credentials in the platform's secret settings and review who can modify the deployment.

---

## 8. Incident Response

### Reporting a Vulnerability

* Report non-sensitive bugs through the repository's bug report template.
* [TEAM TO CONFIRM] Designate a maintainer to receive sensitive vulnerability reports privately, rather than in public issues.

### Responding to an Incident

*Compromised GitHub account* Change the password, revoke active sessions and tokens, confirm 2FA, and review recent commits by that account.
*Leaked secret* Revoke or rotate it first, then remove it from the repository (see Section 4).
*Suspicious commit or pull request*  Do not merge; revert the change if already merged; notify maintainers.
*Poisoned or corrupted dataset*  Restore the last reviewed dataset version and retrain from it.
*Exposed share link* Stop the running application to invalidate the link.

After any incident, the team records what happened, how it was resolved, and what will prevent a repeat.

---

## 9. Contributor Security Checklist

Before opening a pull request, confirm that:

- [ ] My fork is synced with `upstream/main`, and I worked on a feature branch.
- [ ] My pull request targets `Wecncode/Social-Sentinel`, not my own fork.
- [ ] No API keys, tokens, passwords, or `.env` files are included.
- [ ] Notebook outputs are cleared.
- [ ] Any new dependency is necessary, correctly named, and pinned in `requirements.txt`.
- [ ] Dataset changes contain no real personal data and have correct labels.
- [ ] Malicious links in samples are defanged.
- [ ] Application code does not use `share=True` or `debug=True` for deployment.

