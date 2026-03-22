# AGENTS.md

When working with GitHub Actions in this repository, please follow these best practices:

## Pinning Actions to Hashes
- Always pin GitHub Actions to a full 40-character commit hash instead of a tag or branch name. This protects against supply chain attacks where a tag could be moved to point to malicious code.
- Example: `uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd`

## Version Comments
- On the same line as the `uses:` statement, append a comment showing the version or branch that corresponds to the hash.
- Use the most specific version number available (e.g., `# v6.0.2` instead of `# v6`).
- If no tag corresponds to the hash, use the branch name (e.g., `# main`).
- Ensure there is exactly one space between `#` and the version string.

## Verification
- Before updating a hash or version comment, verify that they correctly correspond to each other.
- You can use the GitHub API or `git ls-remote` to find the tags and branches associated with a commit hash in the action's repository.
- **Always trust the hash** as the source of truth; if the hash and the comment disagree, update the comment to match the hash.

## Commented-out Actions
- Maintain accurate version comments even for actions that are currently commented out in the workflow files.
