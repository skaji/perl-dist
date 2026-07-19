# Usage

## Configure the author

`perl-dist new` reads the author name, email address, and GitHub user from Git:

```console
❯ git config --global user.name "Your Name"
❯ git config --global user.email you@example.com
❯ git config --global github.user your-github-user
```

## Commands

| Command | Action |
| --- | --- |
| `perl-dist new Foo::Bar` | Creates a new `perl-Foo-Bar` repository with a module, test, metadata, license, and GitHub Actions workflow. |
| `perl-dist build` | Updates `META.json` and, when managed by `perl-dist`, `README.md`. |
| `perl-dist tarball` | Creates a local tarball directly from the current Git `HEAD`. |
| `perl-dist release` | Updates release files, commits and tags them, and pushes `main` and the tag to GitHub. |

The generated workflow tests the distribution on Linux, macOS, and Windows.
For a tag, it then:

1. creates the tarball with `git archive`;
2. generates a GitHub Artifact Attestation for that tarball;
3. attaches the tarball to a GitHub Release; and
4. uploads the same tarball to PAUSE.

## Create a distribution

```console
❯ perl-dist new Foo::Bar
❯ cd perl-Foo-Bar
```

This creates and stages:

```text
.gitattributes
.github/workflows/ci.yaml
.gitignore
Build.PL
Changes
LICENSE
META.json
README.md
lib/Foo/Bar.pm
t/00_use.t
```

It also initializes the Git repository and configures `origin` as:

```text
git@github.com:your-github-user/perl-Foo-Bar.git
```

Create that GitHub repository, review the generated files, and make the initial
commit.

## Build a distribution

`perl-dist` does not infer dependencies. To add or update prerequisites, edit
the `prereqs` section of `META.json` directly. `perl-dist build` preserves those
entries while updating generated metadata such as `provides`.

Run `build` when the module documentation or package list changes:

```console
❯ perl-dist build
❯ git add README.md META.json
❯ git commit -m "Update generated files"
```

## Prepare a release

Keep the source, tests, metadata, and `Changes` entries committed as normal.
Before releasing, add at least one entry under `NEXT` in `Changes` and make
sure the working tree is ready to release.

```console
❯ perl-dist release
---> version v0.0.1 -> v0.0.1, trial 0
---> do you want to proceed to release? press CTRL+C to stop.
```

Press Enter to continue or Ctrl+C to stop. The command creates the release
commit and annotated tag, then pushes both to `origin`.

After the first release, the patch component is incremented automatically. Set
an explicit version with `VERSION`:

```console
❯ VERSION=v1.0.0 perl-dist release
```

Create a trial release with `TRIAL`:

```console
❯ TRIAL=1 perl-dist release
```

## Configure PAUSE upload

Generate a bearer token from
[Generate a New Token](https://pause.perl.org/pause/authenquery?ACTION=new_token)
on PAUSE, then store it as the GitHub Actions repository secret `PAUSE_TOKEN`.
The release workflow passes this token only to the PAUSE upload step.

PAUSE tokens represent the upload authority of the PAUSE account that created
them; they are not scoped to one distribution or one set of packages. A
repository containing `PAUSE_TOKEN` must therefore be inside the same trust
boundary as all CPAN packages that the PAUSE account is allowed to upload.
This is especially important for repositories maintained by multiple people.

`perl-dist` does not change PAUSE's package indexing permissions. PAUSE still
decides which packages from an uploaded distribution are eligible for the CPAN
index according to the uploader's existing permissions.

## Verify a release

Download a release tarball from GitHub or CPAN and verify its provenance with
the GitHub CLI:

```console
❯ gh attestation verify Foo-Bar-v1.0.0.tar.gz \
    --repo your-github-user/perl-Foo-Bar
```

A successful verification establishes that the tarball was attested by the
specified GitHub repository and reports the workflow and commit associated
with it. Verification does not audit the source code or the workflow; those
remain available for the consumer to inspect.

## Provenance notes

The release model does not require authoring or build steps to be
configuration-free. A distribution may use additional steps as long as every
file intended for release is committed before the release tag is created.

`perl-dist release` may update versions and generated metadata, but it commits
those results before creating the tag. After that point, the release pipeline
must not generate or modify anything that will be included in the tarball.

For another approach to CPAN release provenance, see
[Signing CPAN Releases with SigStore](https://blogs.perl.org/users/timothy_legge/2026/05/signing-cpan-releases-with-sigstore.html),
which describes signing a locally generated tarball with an individual OIDC
identity. `perl-dist` instead attaches provenance to the public workflow that
produces the release artifact.
