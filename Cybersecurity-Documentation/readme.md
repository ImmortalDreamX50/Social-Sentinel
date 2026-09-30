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

*Central repository with forks* [In place] - Contributors work in personal forks; only the team repository is authoritative.
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
*Sync before new work* [In place] - Contributors fetch and compare with `upstream/main` before starting, reducing conflicts and accidental overwrites.
*Pull request target check*  [ Recommended] Confirm the pull request's base repository is `Wecncode/Social-Sentinel` before submitting, since GitHub may default to the contributor's own fork.
*Security review checklist* [ Recommended] Reviewers check every pull request against the checklist in Section 9.

---

## 3. Secrets and Credentials

API keys, access tokens, and passwords must never be stored in the repository. Anything committed to a public repository should be treated as public, even if deleted later, because it remains in Git history and may already have been copied.

*No secrets in code*  [TEAM TO CONFIRM]  Confirm that no keys, tokens, or passwords exist in the code, notebooks, or history. 
*`.gitignore` for sensitive files* [Partially in place] The Python `.gitignore` is present and correctly excludes `.env`, `venv/`, `__pycache__/`, and `.ipynb_checkpoints/`. It does **not** yet exclude trained model files; `models/artifacts/` is not ignored. Adding `models/artifacts/` and `*.joblib` is listed in Section 12, because a committed artifact is a deserialisation risk (see Section 6) and a large binary that does not belong in Git history.
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

*Pinned package versions* [Partially in place] `requirements.txt` specifies **minimum** versions using `>=` ranges (for example `pandas>=2.0.0`), not exact pins. Two contributors installing on different days therefore run different code, and a new upstream release is pulled in without review. Committing a lockfile is listed in Section 12.
*Minimal dependency surface* [In place ] The project relies on a small set of widely used libraries.
*Valid `requirements.txt`* [In place] The file is well-formed and installs cleanly. It declares only Python packages, so the Python version itself is not pinned by it. Record the supported version in the README or a `.python-version` file instead, and keep it consistent with the `Dockerfile` base image.
*Isolated environments*  [Recommended ] Each contributor installs dependencies in a virtual environment (`venv`), never system-wide.
*Vulnerability scanning* [Recommended ] Run `pip-audit` before releases and enable GitHub Dependabot alerts to flag packages with known vulnerabilities.
*Vetting new packages* [Recommended ] Before adding a dependency, confirm it is actively maintained and check the exact package name to avoid look-alike (typosquatted) packages.

---

## 5. Dataset Security

The model learns only from its data. Whoever can change the dataset can change what the model considers "safe."

*Dataset versioning* [In place ] The training set is versioned (`social-sentinel-training-dataset-v1.csv`), so changes can be traced and reviewed like code.
*Separate training and test sets* [Partially in place] The directories are kept apart, but the separation is **not enforced by any code**. `dataset/test-dataset/samples.txt` is unlabelled and referenced by no script; the only evaluation in the project is the random split inside `train.py`, drawn from the training CSV. That CSV also contains **73 duplicated messages (9.4%)**, so a message can appear in both the training and the "unseen" test partition and be scored on text the model has already memorised. Until duplicates are removed and the split is made group-aware, any reported score is an upper bound rather than a measure of performance on unseen messages. See Section 11.
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
*Network binding* [Not in place] The dashboard launches with `server_name="0.0.0.0"`, which listens on every network interface rather than localhost only. On a shared or cloud host this exposes the service beyond the machine it runs on. Bind to `127.0.0.1` by default and widen only when a deliberate deployment requires it.
*Public share links* [Recommended ] The application launches Gradio with `share=True`, which creates a public URL that anyone with the link can use for up to one week, and routes traffic through a third-party service. Because users paste real message content into it, this is a data-egress decision as well as an exposure one (see Section 10). Use `share=False` during development and create share links only for scheduled demos.
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
*Leaked secret* Revoke or rotate it first, then remove it from the repository (see Section 3).
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
- [ ] Application code does not bind to `0.0.0.0` by default.

---

## 10. Scope and Limitations

Everything above describes how the project protects **itself**. This section describes what the tool Social Sentinel provides, and — more importantly — what it does not.

### 10.1 What This Tool Is

Social Sentinel is an **advisory triage aid**. It reads one message and returns a verdict, a probability, and the vocabulary that influenced the decision. Its purpose is to help a human decide where to look first.

### 10.2 What It Is Not

*Not an enforcement control.* Nothing in the tool blocks, quarantines, or removes a message. A verdict is information for a person, not a gate on a mail flow.
*Not a verdict of safety.* A `SAFE` result means the message resembles the benign training data. It does not mean the message is safe, and it is not a clean bill of health. See Section 11.
*Not a replacement for verification.* No model output should be the sole basis for action against a person, an account, or a sender.
*The incident response controls are placeholders.* The Quarantine, Block Sender, and Purge buttons in the dashboard return fixed status text and perform no action on any mail store. A user may see a success message where nothing happened.

### 10.3 Out of Scope

The following are not addressed by this project, and their absence should not be read as an assurance that they are handled elsewhere:

- Malware, attachment, and link-sandwich analysis. The tool classifies text only.
- Business email compromise as a distinct category. It is currently folded into `phishing`.
- Sender authentication results. SPF, DKIM, and DMARC are not evaluated.
- Non-English messages. There is no language detection or normalisation.
- Any enforcement at the mail gateway, transport, or endpoint layer.

---

## 11. Detection Risk and Known Model Defects

A tool that tells a person "this is safe" creates a risk that a person who ignores it would not otherwise face. This section states that risk honestly.

### 11.1 The Cost of a Wrong Answer

Detection errors are not symmetric. A benign message incorrectly flagged as phishing costs an analyst a few seconds of triage. A phishing message reported as safe can result in credential compromise, financial loss, or a compromised account, and the user has been actively told there was nothing to worry about. **A false negative is therefore considerably more costly than a false positive, and the system should be tuned to favour recall.**

### 11.2 Known Defects Affecting Analyst Decisions

These are recorded publicly rather than deferred, because both affect the numbers a user decides to act on.

*Confidence distribution is currently incorrect for benign messages.* The `label` column of the training corpus contains a trailing-whitespace variant affecting 24 rows, so the trained classifier fits three classes rather than two. At inference the two "safe" classes collapse into a single dictionary key and one overwrites the other, which means the Safe probability displayed in the dashboard is the probability of only one of the two safe categories rather than their combined total. The binary verdict remains usable; the displayed confidence figure should not be quoted.
*Token attribution sign must be re-verified when the above is fixed.* The attribution code reduces the coefficient matrix to a single row using the scikit-learn convention that positive values point toward `classes_[1]`. For a two-class label set that convention points toward the *safe* class, which would invert the reported trigger signals. This is currently masked by the three-class defect above. **The two fixes are coupled and must be verified together, not landed separately.**
*Decision threshold is tuned for accuracy, not for security.* The threshold in `config/config.yaml` is `0.50`, which is the accuracy-optimal value for a near-balanced corpus. Given 11.1, this is likely too permissive. Lowering it toward `0.35` trades false positives for false negatives, and should be validated against measured recall before adoption.
*Reported scores are an upper bound.* The training corpus contains 73 duplicated messages (9.4%), and the train/test split is not group-aware, so evaluation can score the model on text it has already seen. No performance figure should be quoted externally until this is resolved.

### 11.3 Not Yet Defined

*No target false-negative rate.* The project has not set an acceptable miss rate, so there is no threshold against which a release can be judged acceptable or rejected.
*No escalation path.* Nothing states who handles the case where the verdict is `SAFE` and the message was malicious. A `SAFE` verdict currently ends the tool's involvement with no defined next step for the user.
*No monitoring after release.* There is no mechanism to detect silent degradation in production. See Section 6.

---

## 12. Remediation Roadmap

This section records what has been corrected and what is proposed, so that progress is visible rather than implied.

### 12.1 Completed — Documentation Accuracy

The controls in this document were audited against the codebase. The following claims were found to be inaccurate and have been corrected: version pinning status, a `requirements.txt` defect that did not exist, `.gitignore` coverage of model files, train/test separation and its effect on evaluation, and the network exposure created by the interface binding. The scope of the tool, its detection risk, and its known defects are now stated explicitly.

### 12.2 Proposed — Low-Risk Configuration Changes

These are small, self-contained, and independently reviewable. None requires new features.

- [ ] Add a `.dockerignore`. The `Dockerfile` uses `COPY . .` with no ignore file, so the whole working directory — including `.git` history, any local `.env`, and untracked model artifacts — is copied into the image. `.gitignore` does not protect the build context.
- [ ] Pin the container base image by digest, add a non-root `USER`, and remove `build-essential` from the final stage.
- [ ] Pin exact dependency versions and commit a lockfile, so builds are reproducible.
- [ ] Add `models/artifacts/` and `*.joblib` to `.gitignore`.
- [ ] Bind to `127.0.0.1` by default and disable `share=True` in the committed default configuration.
- [ ] Add `models/artifacts/` handling to the container so the image is not deployed without a trained model, and add a `HEALTHCHECK`.

### 12.3 Proposed — Tooling and Process

Larger items, listed as a plan with owners to be assigned rather than as immediate work.

- [ ] Add continuous integration. There is currently no `.github/workflows`, so branch protection, secret scanning, dependency alerts, and `pip-audit` are all manual and unenforced.
- [ ] Add model integrity verification: hash the artifact at build time and verify before loading it, as described in Section 6.
- [ ] Add a `CODEOWNERS` file routing security-relevant paths to a named reviewer.
- [ ] Add a `SECURITY.md` giving a named private contact for sensitive reports. Section 8 currently asks for a private channel but does not say where.
- [ ] Write a privacy policy for pasted message content. Users paste real phishing mail, which routinely contains names, addresses, and account numbers, and `share=True` routes that content through a third party.
- [ ] Document the placeholder status of the incident response controls, and complete the privacy work in 12.2 before implementing the unused `soc_actions.quarantine_dir`, which would write pasted message content to disk.
- [ ] Remove dataset duplicates and make the train/test split group-aware, so evaluation measures performance on unseen messages.
- [ ] Set a target false-negative rate and define the escalation path described in Section 11.3.
- [ ] Expand adversarial evasion coverage beyond spelling substitution to include unicode homoglyphs, zero-width characters, and whitespace injection, none of which the pipeline currently normalises. Note that the corpus embeds markdown links, so URL tokens are a learned feature and can be manipulated directly.

