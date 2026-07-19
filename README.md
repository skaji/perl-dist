# perl-dist

A CPAN authoring tool for transparent, provenance-attested releases.

`perl-dist` demonstrates this release model using facilities available today:
Git, GitHub Actions, GitHub Artifact Attestations, and PAUSE bearer tokens.

## Release model

The central rule is simple:

> Every file shipped in a release must already be present in the tagged Git
> tree.

Authoring and build steps may be configurable, as long as every file intended
for release is committed before the release tag is created.

`perl-dist release` updates the version and generated metadata, commits those
changes, and creates a tag. The public GitHub Actions workflow then creates the
release artifact directly from that tag:

```text
tagged Git tree
       |
       | git archive
       v
one release tarball
       |
       +-- GitHub Artifact Attestation
       +-- GitHub Release
       `-- PAUSE
```

There is no build step between the tag and the tarball that adds or rewrites
files. The exact same tarball is attested, attached to the GitHub Release, and
uploaded to PAUSE.

This gives a release several useful properties:

- **Reviewable:** every shipped file can be inspected in the release tag.
- **Traceable:** the attestation identifies the repository, commit, triggering
  event, and workflow that produced the tarball.
- **Consistent:** GitHub and PAUSE receive the same artifact, not independently
  generated copies.
- **Auditable:** the workflow that creates and publishes the artifact is public.

GitHub Artifact Attestations use Sigstore to establish build provenance. They
prove where and how an artifact was produced; they do not claim that its
contents are safe or trustworthy. See
[Artifact attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations)
for details.

## Install

`perl-dist` itself requires Perl 5.44 or later.

```console
❯ cpm install -g https://github.com/skaji/perl-dist.git
```

See [USAGE.md](USAGE.md) for setup, commands, releasing, and verification.

## License

This software is free software; you can redistribute it and/or modify it under
the same terms as Perl itself.
