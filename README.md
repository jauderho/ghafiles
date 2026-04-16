# ghafiles

This contains baseline GitHub Actions that may be useful for any new project. 

- Adhere to least privilege principles for workflow permissions
- Use of commit hashes for pinning GitHub Actions dependencies
- Use of Dependabot to update commit hashes as necessary
- Use of [andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills/blob/main/CLAUDE.md) as CLAUDE.md starting point
- Use of OpenSSF's [Security Scorecard](https://github.com/ossf/scorecard) (SCORECARD_TOKEN setup required)
- Use of Step Security's [Harden Runner](https://github.com/step-security/harden-runner)
- Codespell
- Super-Linter
- Semgrep (SEMGREP_APP_TOKEN setup required)
- (optional) Sync to GitLab (GITLAB_TOKEN setup required)
