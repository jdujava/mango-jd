# Repository workflow

- Keep implementation changes and commits in this source repository. Do not
  export patches, source pins or file inventories into the dotfiles repo.
- Dotfiles contain desktop configuration and build helpers; source fixes belong
  on this repository's `desktop` branch.
- Keep one coherent commit per custom feature and one final maintenance commit.
  Fold follow-up fixes into their feature instead of accumulating repair commits;
  remove upstreamed patches only after verifying that the behavior is preserved.
- Before implementation changes, fetch the official `upstream` remote and rebase
  the custom series onto its latest `main` by default. Do not merge upstream into
  the patch series. Use an isolated worktree when the current checkout has edits.
- Notify the user of upstream changes that affect desktop behavior,
  configuration, protocol support or build requirements. Report rebase conflicts
  and preserve the intended custom behavior when resolving them.
- Before rewriting history, preserve the previous tip in a private recovery ref
  and verified bundle. For pure regrouping, verify complete tree equality before
  rebasing; build and run relevant checks after the upstream integration.
- Preserve existing user edits. Do not push rewritten history unless the user
  explicitly requests it, and never publish private recovery refs or bundles.
