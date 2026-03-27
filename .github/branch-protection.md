# Branch Protection Rules

Intended rules for the `master` branch (configured in GitHub Settings → Branches → Add rule).

| Rule | Setting |
|---|---|
| Require a pull request before merging | Enabled |
| Required approvals | 0 (solo project — PR still required for a clean history) |
| Dismiss stale reviews on new commits | Enabled |
| Allow force pushes | Disabled |
| Allow branch deletions | Disabled |

These rules ensure `master` is never pushed to directly and every change has a PR record.
