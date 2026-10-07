# 🔑 Automate Secret Scanning with AccuKnox GitHub Action

The **AccuKnox Secret Scan GitHub Action** detects **hardcoded secrets, credentials, API keys and other sensitive information** in your Git repositories.
It integrates with the **AccuKnox Console**, giving you centralized visibility, risk tracking and remediation across your development lifecycle.

Catch secrets before they leak, with **shift-left security**.

---

## 🎯 Key Features

- ✅ **Hardcoded Secret Detection** – Find API keys, passwords, tokens and other secrets in code and Git history.
- 🔒 **Shift-Left Security** – Run secret scanning directly in your CI/CD pipeline.
- 📥 **Console Integration** – Upload findings to the AccuKnox Console for visibility and remediation tracking.
- ⚙️ **Flexible Configuration** – Pin the scanner version, override the scan command or add custom arguments.
- 🚦 **Fail Builds on Findings** – Fail the pipeline on detected secrets, or run in soft-fail mode.
- 📦 **Downloadable Reports** – Results are stored as a workflow artifact in SARIF format.

---

## ⚠️ Prerequisites

- 🔐 **AccuKnox Console access** – Sign in to your AccuKnox tenant.
- 🗝️ **API token** – Generated in the AccuKnox Console (see Step 1).
- 🏷️ **Label** – A label in the Console to tag scan reports.
- 🔑 **GitHub secrets** – Store the token, endpoint and label securely in your repository.

---

## 📌 Installation & Usage

### Step 1: Retrieve AccuKnox credentials

1. Log in to the AccuKnox Console.
2. Go to **Settings → Tokens → Create Token** and copy the token.
3. Create a **label** under **Dashboard → Labels** for the scan results.

### Step 2: Configure GitHub secrets

In your repository go to **Settings → Secrets and variables → Actions → New repository secret**:

| Secret Name         | Description |
|---------------------|-------------|
| `ACCUKNOX_TOKEN`    | AccuKnox API token |
| `ACCUKNOX_ENDPOINT` | AccuKnox API endpoint (e.g. `cspm.demo.accuknox.com`) |
| `ACCUKNOX_LABEL`    | Label used to group results in the AccuKnox Console |

### Step 3: Add the workflow

Create `.github/workflows/secret-scan.yml`:

```yaml
name: AccuKnox Secret Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v7

      - name: Run Secret Scan
        uses: accuknox/secret-scan-action-new@latest
        with:
          accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
          accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
          accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
          soft_fail: true
```

---

## ⚙️ Configuration Options (Inputs)

| Input | Description | Required | Default |
|---|---|---|---|
| `accuknox_token` | AccuKnox API token | Yes | – |
| `accuknox_endpoint` | AccuKnox Console endpoint | Yes | – |
| `accuknox_label` | Label used to organize results in the Console | Yes | – |
| `soft_fail` | Do not fail the workflow when secrets are found | No | `false` |
| `additional_arguments` | Extra arguments appended to the scan command | No | `""` |
| `base_command` | Replaces the default scan command (keep `--report-path results.json` for artifact upload) | No | `detect --source . --report-format sarif --report-path results.json --no-banner` |
| `scanner_version` | AccuKnox ASPM scanner CLI release to use | No | `v0.15.1` |
| `upload_results` | Upload `results.json` as a GitHub artifact | No | `true` |

---

## 🎯 Scan Scope

By default the scan covers the full git history. Pass `additional_arguments` to narrow it (optional):

| Goal | `additional_arguments` | Notes |
|---|---|---|
| Current files only, no history | `--no-git` | Secrets committed and later deleted are not found |
| Only this branch's commits | `--log-opts="origin/main..HEAD"` | Needs `fetch-depth: 0` on `actions/checkout` |
| Only the latest commit | `--log-opts="-1"` | |

```yaml
- uses: accuknox/secret-scan-action-new@latest
  with:
    accuknox_token: ${{ secrets.ACCUKNOX_TOKEN }}
    accuknox_endpoint: ${{ secrets.ACCUKNOX_ENDPOINT }}
    accuknox_label: ${{ secrets.ACCUKNOX_LABEL }}
    additional_arguments: "--no-git"
```

---

## 🔍 How It Works

1. **Code is pushed** – The workflow triggers.
2. **Scanner is set up** – The pinned AccuKnox scanner release is downloaded and the secret scan engine is installed.
3. **Repository is scanned** – Working tree and Git history are checked for secrets, and a SARIF report (`results.json`) is generated.
4. **Findings are uploaded** – Results are sent to the AccuKnox Console using your token, endpoint and label.
5. **Artifact is stored** – If `upload_results: true`, the report is attached to the workflow run as `secret-scan-results`.
6. **Review findings** – In the Console go to **Issues → Findings** and filter by *Secret Findings*.
7. **Pipeline decision** – With `soft_fail: false`, the job fails when secrets are detected.

---

## 🛠️ Troubleshooting & Best Practices

| Issue | Cause | Solution |
|---|---|---|
| `token, label, and endpoint must be provided` | GitHub secret not set | Add `ACCUKNOX_TOKEN`, `ACCUKNOX_LABEL` and `ACCUKNOX_ENDPOINT` |
| `401 Token is invalid or expired` | Wrong or expired token | Generate a new token in the Console |
| No results in AccuKnox Console | Wrong label or endpoint | Verify the label and endpoint values |
| Secrets in older commits are missed | Shallow checkout | Set `fetch-depth: 0` on `actions/checkout` |
| Workflow fails even for minor findings | `soft_fail` not set | Set `soft_fail: true` to continue despite findings |
| Artifact upload finds no file | Custom `base_command` writes elsewhere | Keep `--report-path results.json` |

**Best practices**

- Pass credentials only through `secrets.*`; they reach the scanner as environment variables, never on the command line.
- Pin the action to a release tag or commit SHA for reproducible builds.
- Pin `scanner_version` and upgrade deliberately.
- Start with `soft_fail: true`, review findings, then enforce failing builds.

---

## 📖 Support & Documentation

📚 Docs: [AccuKnox Documentation](https://help.accuknox.com)

📧 Support: support@accuknox.com

---

## 📄 License

Apache License 2.0, see [LICENSE](LICENSE).

🔐 **Shift Left with AccuKnox – Catch Secrets Before They Leak!** 🚀
