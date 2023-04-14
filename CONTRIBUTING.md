<!--
SPDX-FileCopyrightText: 2023 David Runge <dvzrv@archlinux.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Contributing

These are the contribution guidelines for archlinux.install_archlinux.

Development takes place on Arch Linux's GitLab at
https://gitlab.archlinux.org/dvzrv/ansible-install_archlinux.

Please read our distribution-wide
[Code of Conduct](https://terms.archlinux.org/docs/code-of-conduct/) before
contributing, to understand what actions will and will not be tolerated.

## Writing code

This project is an
[Ansible role](https://galaxy.ansible.com/docs/finding/content_types.html#ansible-roles).

All contributions are linted using 
[ansible-lint](https://ansible-lint.readthedocs.io/) and spell checked using
[codespell](https://github.com/codespell-project/codespell).

To aide in development, it is encouraged to use the local `.ci/check.sh` script
as [git pre-commit hook](https://man.archlinux.org/man/githooks.5#pre-commit):

```shell
$ ln -f -s ../../.ci/check.sh .git/hooks/pre-commit
```

## Writing commit messages

The commit message style follows
[conventional commits](https://www.conventionalcommits.org/en/v1.0.0/).

To aide in development, install
[cocogitto](https://github.com/cocogitto/cocogitto) and add its
[git-hooks](https://man.archlinux.org/man/githooks.5):

```shell
$ cog install-hook all
```

## Creating releases

Releases are created by the developers of this project using
[cocogitto](https://github.com/cocogitto/cocogitto) by running:

```shell
cog bump --auto
```

The above will automatically bump the version based on the commits since the
last release, add an entry to `CHANGELOG.md`, create a tag and push the changes
to the default branch of the project.

## License

All code contributions fall under the terms of the
[GPL-3.0-or-later](https://www.gnu.org/licenses/gpl-3.0.en.html).

All documentation contributions fall under the terms of the
[CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/).

All configuration file contributions fall under the terms of the
[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).

License identifiers and copyright statements are checked using
[reuse](https://git.fsfe.org/reuse/tool). By using `.ci/check.sh` as git
pre-commit hook, it is run automatically.
