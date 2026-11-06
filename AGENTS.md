# Repository guidance

This is a Hugo blog. Run `hugo --minify` and `git diff --check` after changes.
GitHub-hosted runners are allowed for CI and deployment.

Keep this repository's published `main` history at exactly one root commit.
Fold every change, including CI/CD, into that commit with `git commit --amend`;
never add follow-up commits. This overrides the shared release commit split.
Before rewriting, preserve local recovery refs for the current local and remote
tips and a verified bundle. Verify the final tree and single-commit history,
fetch the current remote tip, and push with an explicit
`--force-with-lease=refs/heads/main:<sha>`.
Keep unrelated in-progress work out of the commit.
