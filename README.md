# Renovate Demo

## Setup

1. Fork [pygoat](https://github.com/adeyosemanputra/pygoat)
1. Enable Issues
1. Install [Renovate Github App](https://github.com/apps/renovate)

## On-Boarding

```sh
git clone git@github.com:mhennecke/pygoat.git
cd pygoat

# modify on-boarding config
git checkout renovate/configure
```

Best Practice [presets](https://docs.renovatebot.com/presets-default) and no limits (for demo)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:best-practices",
    ":prHourlyLimitNone",
    ":prConcurrentLimitNone"
  ]
}
```

Validate renovate.json locally and commit

```sh
npx --yes --package renovate -- renovate-config-validator
git add renovate.json
git commit -m'chore(renovate): no limits, best-practices'
git push
```

Validate with pre-commit hook

```sh
tee .pre-commit-config.yaml <<EOF
repos:
  - repo: https://github.com/renovatebot/pre-commit-hooks
    rev: 37.333.1
    hooks:
      - id: renovate-config-validator
EOF
pre-commit install
```

Merge On-Boarding PR

## Docker Image Digest Pinning

Why? Enables updates for floating tags, e.g `ubuntu:22.04`. Enabled thanks to `config:best-practices` preset which includes `docker:pinDigests` preset.

## Package Grouping

Renovate will auto-close PRs if they become obsolete due to config change.

### All Non-Major

[Group All Non-Major Preset](https://docs.renovatebot.com/presets-group/#groupallnonmajor)

### All Pip Requirements

Example: Group all requirements.txt packages

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:best-practices",
    ":prHourlyLimitNone",
    ":prConcurrentLimitNone"
  ],
  "packageRules": [
    {
      "groupName": "all non-major pip requirements",
      "groupSlug": "all-minor-patch",
      "matchManagers": [
        "pip_requirements"
      ],
      "matchUpdateTypes": [
        "minor",
        "patch"
      ]
    }
  ]
}
```

## Automerge

### Docker Digests

Set `Dockerfile` python image to some "old" digest

```sh
sed -i 's/^FROM.*$/FROM python:3.11.4-buster@sha256:19b2bd12076f6910d93ef0d0a2d4bd7d46611b05db3fb7e27d20e7657274ccbc/' Dockerfile
```

REMARK: Dockerfile vanished from "pin dependencies" PR

Add automerge config

```json
{
  ...
  "packageRules": [
    ...
    {
      "groupName": "Automerge docker digests",
      "matchDatasources": ["docker"],
      "automerge": true,
      "automergeType": "branch",
      "matchUpdateTypes": [
        "digests"
      ],
      "ignoreTests": true
    }
  ]
}
```

REMARK: If branch pipeline validations exist, `ignoreTests` can be removed. First renovate run will create branch. Subsequent runs will merge branch, if validation pipeline succeeded.

## Update outside of Renovate Flow

```sh
sed -i 's/Django==[0-9.]*/Django==4.2.11/' requirements.txt
git add requirements.txt
git commit -m'chore(deps): manual Django update tp 4.2.11'
git push
```

## Custom Regex Manager

[Regex Presets](https://docs.renovatebot.com/presets-regexManagers/)

Add Preset: `regexManagers:dockerfileVersions`

Add to Dockerfile:

```Dockerfile
...
# renovate: datasource=pypi depName=pip
ENV PIP_VERSION=22.0.4
...
RUN python -m pip install --no-cache-dir pip==${PIP_VERSION}
```

Add to Dockerfile:

```Dockerfile
...
# renovate: datasource=github-releases depName=tmccombs/hcl2json
ENV HCL2JSON_VERSION=v0.5.0

RUN curl -sL https://github.com/tmccombs/hcl2json/releases/download/${HCL2JSON_VERSION}/hcl2json_linux_amd64 -o /usr/local/bin/hcl2json \
 && chmod a+x /usr/local/bin/hcl2json
...
```

## Update Restrictions

```json
{
  "packageRules": [
    {
    "matchPackageNames": ["requests"],
    "allowedVersions": "<2.30.0"
    }
  ]
}
```
