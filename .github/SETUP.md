# thrift-php mirror — setup

This repo mirrors full releases of the `apache/thrift` monorepo as detached
tags, but marks everything except the PHP client library `export-ignore` via
a generated `.gitattributes`, so `composer require` still only downloads the
PHP-relevant subset (compiler, every other language binding, tests, and docs
are excluded from the install artifact, not from the mirror itself). It is an
unofficial, personally-maintained mirror — not affiliated with or modifying
the ASF `apache/thrift` repository in any way.

`master` intentionally holds nothing but this README's sibling `../README.md`,
`.github/`, and a minimal `composer.json` (metadata only — no `lib/php/lib`,
so `dev-master` isn't installable; it exists solely so Packagist can register
the package). The sync job never touches `master` at all — every synced
release lives purely as a git tag (and a GitHub Release) pointing at a
detached, parentless commit that is not reachable from any branch. Don't add
project files to `master` outside `.github/` expecting the sync job to leave
them alone forever — nothing in `master` is managed by the sync job, so this
is just about keeping `master` minimal by convention, not something enforced
by the workflow.

## One-time setup

1. Create this repo on GitHub (public, e.g. `<you>/thrift-php`) and push this
   scaffold as the initial commit.
2. In the repo's Settings → Actions → General → Workflow permissions, set
   "Read and write permissions" so the default `GITHUB_TOKEN` can push
   commits/tags (the workflow needs `contents: write`, already declared in
   the workflow file, but the repo-level default must allow it too).
3. That's it — no other secrets or variables are needed. The workflow
   authenticates to the *public* `apache/thrift` repo anonymously (read-only
   clone) and pushes to *this* repo using its own automatically-provided
   `GITHUB_TOKEN`.

## How it works

- Runs daily (`cron: "17 3 * * *"`) and on manual `workflow_dispatch`.
- Checks `apache/thrift`'s latest GitHub Release via the public API, then
  checks *this* repo's API for a matching tag and a matching release — no
  state file is kept; sync state is always derived live from GitHub. "Matching"
  means the upstream tag itself or any `vX.Y.Z.N` variant of it, so a manually
  published `.N` patch label (see below) counts as already synced and the
  workflow won't try to recreate the base tag.
- If the tag doesn't exist yet: shallow-clones `apache/thrift` at that release
  tag in full (the whole monorepo, not just the PHP library), drops upstream's
  own `.git`, then drops upstream's own `.github/` entirely — GitHub refuses
  `GITHUB_TOKEN` pushes that touch `.github/workflows/**` (that requires a PAT
  with `workflow` scope), and upstream's CI workflows have no business running
  against this repo's Actions runners anyway. Rewrites the snapshot's
  `composer.json` `name` field to this repo (lowercased `owner/repo`) —
  Packagist requires every version of a package to share one name, and
  upstream's tags all say `apache/thrift`, which would make every tagged
  version get rejected as a name mismatch against whatever name got
  registered from `master`, and adds a `conflict` entry against
  `apache/thrift` (both provide the same `Thrift\` namespace, so Composer
  must refuse to install both at once). Then appends `.gitattributes`
  `export-ignore` rules generated from upstream's `composer.json`
  `archive.exclude` array (blanket excludes first, then `!`-prefixed entries
  become `-export-ignore` un-excludes — same ordering as upstream's array,
  since `.gitattributes` resolves by last-match-wins). Each un-excluded entry
  also unignores every ancestor directory on its path plus a trailing `/**`,
  because unlike Composer's own archiver, `git archive` prunes a whole
  directory the moment the directory entry itself is export-ignore'd — an
  unignore on a leaf path alone is not enough. Commits that tree as a single
  parentless commit, and pushes it to this repo as a tag only — the commit is
  never attached to any branch here.
- If the tag exists but no release does yet: publishes a GitHub Release for
  it, with upstream's own release notes plus a short metadata header (link to
  the upstream release, upstream commit SHA).
- These two steps are gated independently so a partial failure (tag pushed,
  release not yet created) resumes correctly next run without re-doing the
  snapshot.
- Paths inside each snapshot keep upstream's own layout (including the
  `lib/php/lib/` nesting), so that snapshot's own `composer.json`
  `autoload.psr-4` (`Thrift\\: lib/php/lib/`) needs no rewriting, and
  Composer/GitHub's zip-download `export-ignore` handling reduces it to just
  the PHP library on install.

## If Packagist blocks a re-synced tag

Packagist treats a published version's git ref as immutable and blocks re-syncing a tag whose
commit changed after publication (a supply-chain safeguard). If a tag here ever needs to be
recreated after being published (e.g. while debugging this workflow itself), don't fight it:
publish the same content under a `vX.Y.Z.N` label instead (create a new tag + release pointing at
the same commit, with `N` being the next unused integer) and let the base tag stay as-is. The sync
workflow already treats any such variant as covering that release, so it won't try to recreate the
base tag on its own.

## To register on Packagist

Submit **this** repo's URL (`https://github.com/<you>/thrift-php`) to
https://packagist.org/packages/submit. Packagist reads the package name from
`master`'s `composer.json`, but resolves installable versions from tags — so
the name doesn't need to be `apache/thrift` (that name is already taken by the
official package and should stay pointed at the ASF repo; publishing a fork
under that vendor name is not appropriate) and `dev-master` being uninstallable
doesn't block registration.
