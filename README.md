# thrift-php

An automated, unofficial mirror of the PHP-relevant parts of
[apache/thrift](https://github.com/apache/thrift) — not affiliated with or modifying the ASF
`apache/thrift` repository in any way.

This `master` branch intentionally holds nothing but this README, `.github/` (the sync workflow),
and a minimal `composer.json` (metadata only, so Packagist can register the package — no
`lib/php/lib` or autoload here, so `dev-master` is not installable). There is no library source to
browse on `master` — the actual content lives in this repo's [tags and releases](../../releases),
each of which is a **full, detached snapshot** of `apache/thrift` at one upstream release tag (not
just `master`'s history — these commits have no parent and aren't reachable from any branch).

Each snapshot carries a generated `.gitattributes` with `export-ignore` rules derived from
upstream's own `composer.json` `archive.exclude` list, so installing a tag via Composer or
downloading its GitHub source zip yields just the PHP library, the same subset upstream's own
release archives would. A few things are deliberately changed rather than mirrored verbatim:
upstream's own `.github/` is dropped (its CI workflows have no business running against this
repo's Actions runners, and GitHub refuses token pushes that touch `.github/workflows/**` in the
first place), each snapshot's `composer.json` has its `name` field rewritten to this repo —
Packagist requires every version of a package to share the same name, and upstream's tags all say
`apache/thrift` — and a `conflict` entry against `apache/thrift` is added, since both packages
provide the same `Thrift\` namespace and Composer must refuse to install both at once rather than
let one silently clobber the other's classes.

To use this as a Composer dependency, require a specific released version (`dev-master` has no
autoload and is not installable):

```
composer require rtm-ctrlz/thrift-php:^0.24
```

See each [release](../../releases) for upstream's original release notes plus the upstream commit
this snapshot was built from. License and copyright notices for the mirrored code are unchanged
from upstream and are included in every snapshot.

`v0.24.0.1` exists alongside `v0.24.0` for one reason: Packagist treats a published version's git
ref as immutable and blocked re-syncing `v0.24.0` after it got re-tagged a few times while this
mirroring setup itself was still being debugged. `v0.24.0.1`'s content is identical to the intended
`v0.24.0` snapshot — it's a version-label workaround, per Packagist's own guidance to bump the
version rather than retag. Tags produced by the sync workflow going forward won't hit this, since
each upstream release is only ever tagged here once.
