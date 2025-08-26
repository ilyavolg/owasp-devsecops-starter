# OWASP DevSecOps Starter (Zero-Budget)

This repo is a **from-scratch, free** starter to run OWASP Top 10 style checks in GitHub Actions with open‑source tools.

It includes:
- **Secrets:** Gitleaks
- **SAST:** Semgrep (OWASP Top Ten rules)
- **SCA/SBOM:** Trivy (filesystem + image + CycloneDX SBOM)
- **IaC:** Checkov
- **DAST:** OWASP ZAP (baseline per PR optional, weekly active scan optional)

> Everything here is configured to **pass by default** (no failing on vulnerabilities) to make setup easy. Once you’re comfortable, flip the switches to **fail on HIGH/CRITICAL** as explained below.

---

## 1) Quick Start (No coding required)

### A) Create a GitHub repo
1. Go to GitHub → **New repository** → name it `owasp-devsecops-starter` (or anything).
2. Choose **Public** (free Actions minutes). Keep “Initialize with README” checked.
3. Click **Create repository**.

### B) Upload these starter files
1. Click **Add file → Upload files** and upload the contents of this zip (keep the folder structure).
2. Commit to the **main** branch.

### C) See GitHub Actions run
- Open the **Actions** tab. You’ll see **security-pr** and **security-main** workflows.
- The `security-main` workflow runs automatically on pushes to **main**.
- To try a Pull Request run:
  1. Click **Code** → create a new branch (e.g., `test-pr`) using the web editor.
  2. Edit any file → **Commit** to `test-pr`.
  3. Click **Compare & pull request** → **Create pull request**.
  4. The **security-pr** workflow runs on the PR.

### D) (Optional) Add DAST target URL for ZAP
If you have a staging site you **own/control**, add a secret:
1. Repo **Settings → Secrets and variables → Actions → New repository secret**.
2. Name: `DAST_TARGET_URL`; Value: `https://your-staging.example.com`.
3. Re-run a PR to see **ZAP Baseline** produce a report (artifact).

> You can leave DAST empty for now; everything else still works.

---

## 2) What’s inside

```
.github/workflows/
  pr.yml          # fast checks on pull requests (no failures on vulnerabilities by default)
  main.yml        # checks on main branch
  zap-weekly.yml  # optional weekly “mini pentest” (runs only if you set secrets)
index.html        # simple static page (so tools have something to scan)
Dockerfile        # builds a tiny nginx container to demo image scanning
zap-auth.yaml     # (optional) ZAP authenticated scan config template
.gitignore
```

### Tools mapping to OWASP Top 10
- **Access control / auth:** ZAP (with auth via `zap-auth.yaml`), Semgrep auth rules.
- **Injection / crypto misuse:** Semgrep; ZAP active checks.
- **Misconfig / outdated components:** Checkov; Trivy (deps/image).
- **Integrity / supply chain:** SBOM via Trivy CycloneDX; (later add Sigstore/Cosign).
- **Logging/monitoring:** CI artifacts; wire into SIEM later if you want.

---

## 3) Turn on strict gates later (optional)

Find these lines in workflows and **remove the “don’t fail” flags**:

### In `.github/workflows/pr.yml`
- Trivy filesystem: remove `--exit-code 0` (or add `--exit-code 1 --severity HIGH,CRITICAL`)
- Trivy image: change to `--exit-code 1 --severity HIGH,CRITICAL`
- Checkov: remove `|| true`

### In `.github/workflows/main.yml`
- Same as above (use `--exit-code 1 --severity HIGH,CRITICAL` for Trivy).

> Keep **Gitleaks** and **Semgrep** failing by default—they’re fast and reduce risk early.

---

## 4) Weekly “mini pentest” with ZAP (optional)

1. Add repo secrets (Settings → Secrets and variables → Actions → New secret):
   - `DAST_TARGET_URL` (e.g., `https://staging.example.com`)
   - Optional: `ZAP_USER`, `ZAP_PASS` if your app requires login.
2. Edit `zap-auth.yaml` to match your login form (if needed).
3. The workflow `zap-weekly.yml` is scheduled for Mondays 02:00 UTC (you can also run it manually via **Actions → zap-weekly → Run workflow**).
4. Download the `zap-report.html` artifact and review.

**Safety**: Only scan targets you own/control. Keep timeouts conservative.

---

## 5) Reading results

- Go to the **Actions** run → **Artifacts**:
  - `*.sarif` (Gitleaks, Semgrep, Trivy, Checkov) – you can view them with SARIF viewers or GitHub’s code scanning if you enable it.
  - `sbom.cdx.json` – CycloneDX SBOM (open with any JSON viewer).
  - `zap-baseline.html` / `zap-report.html` – human-friendly DAST reports.

---

## 6) Next steps (nice to have, still free)
- Enable **GitHub code scanning** to natively view SARIF results (Security → Code scanning → Set up).
- Add **pre-commit** locally (optional) so devs catch issues before pushing.
- Add IaC samples (Terraform/K8s) to see Checkov in action.
- When comfortable, turn on **failing thresholds** for Trivy/Checkov as described above.

---

## 7) Troubleshooting

- **“No artifacts found”** → That step may be skipped (e.g., DAST without target). Normal.
- **Docker build failed** → Ensure `Dockerfile` exists at repo root (included here).
- **ZAP can’t reach target** → Check your `DAST_TARGET_URL`, public reachability, and any WAF rules.
- **Semgrep too noisy** → Replace `--config p/owasp-top-ten` with a narrower ruleset later.

---

Enjoy your zero-budget DevSecOps pipeline!




