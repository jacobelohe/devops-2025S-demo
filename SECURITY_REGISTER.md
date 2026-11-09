# Security register

| # | Finding | Source | Severity | Decision | Justification / owner / revisit |
|---|---------|--------|----------|----------|---------------------------------|
| 1 | Hardcoded `SECRET_KEY` in `campus_notifier/settings.py` (B105) | Bandit (Drill 2) | Low | Remediate | Now read from `DJANGO_SECRET_KEY` in `.env` (gitignored; `.env.example` provided). The old `django-insecure-...` key is still in git history, so "removed from the file" is not "rotated". It was a dev placeholder and must never be used in production. |
| 2 | Image HIGH/CRITICAL vulnerabilities | Trivy (Drill 3) | TBD | Pending CI run | Record HIGH/CRITICAL counts from the PR's Image scan table. Remedy: bump base image / pinned packages in `Dockerfile` and `requirements.txt`, rebuild. |
| 3 | ECR repo not encrypted with a KMS CMK (CKV_AWS_136) | Checkov (Drill 4) | Medium | Accept / suppress | Suppressed inline in `terraform/main.tf`. AES256 default encryption is enough for a class demo; a CMK adds cost. Owner: course team. Revisit: 2027-02-01. Other Checkov findings: pending CI run. |

## Gate policy (Drill 5)
- **Secrets (gitleaks):** blocks on any real secret.
- **Image (Trivy):** blocks on CRITICAL (`exit-code: 1`). HIGH is reported only.
- **IaC (Checkov):** blocks (`soft_fail: false`). Accepted risks are suppressed with justifications.
- **Code (Bandit):** blocks on MEDIUM and above (`-ll`). LOW findings are reported but do not block. The B105 finding was Low and is fixed, so the gate costs nothing today. It still stops real issues such as `shell=True` or `yaml.load`, without failing the build on noise like the Low-severity assert.
