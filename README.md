# renovate-pep440

Reproduction for odd PEP440 versions not being properly detected by Renovate,
in support of [discussion
30566](https://github.com/renovatebot/renovate/discussions/30566).

## PEP440 Background

[PEP440](https://peps.python.org/pep-0440/) allows for a wide range of package
versions. For the most part, this follows semver, and Renovate properly
identifies those versions. However, it also supports a format that is
`major.minor` with alpha/beta/candidate pre-release formats:

>  \<major\>.\<minor\>(a|b|dev|post|rc)\<revision\>

More information can be found in the [summary of permitted
suffixes](https://peps.python.org/pep-0440/#summary-of-permitted-suffixes-and-relative-ordering)

## Current behavior

Renovate is unable to parse versions that use these special suffixes, and marks
them as an invalid value.

Given the following entries in `pyproject.toml`:
```toml
[project]
dependencies = [
  "opentelemetry-api >=1.22.0, <2.0",
  "opentelemetry-instrumentation >=0.43b0, <1.0",
]
```

Renovate will provide the following log entries:

`opentelemetry-api` is parsed as expected, and versions are identified:
```
{
  "packageName": "opentelemetry-api",
  "depName": "opentelemetry-api",
  "datasource": "pypi",
  "depType": "project.dependencies",
  "currentValue": ">=1.22.0, <2.0",
  "updates": [],
  "versioning": "pep440",
  "warnings": [],
  "registryUrl": "https://pypi.org/pypi",
  "currentVersion": "1.26.0",
  "currentVersionTimestamp": "2024-07-25T04:01:38.000Z"
},
```

`opentelemetry-instrumentation` is not parsed as expected, and Renovate fails
to find its version:
```
{
  "packageName": "opentelemetry-instrumentation",
  "depName": "opentelemetry-instrumentation",
  "datasource": "pypi",
  "depType": "project.dependencies",
  "currentValue": ">=0.43b0, <1.0",
  "updates": [],
  "versioning": "pep440",
  "warnings": [],
  "registryUrl": "https://pypi.org/pypi",
  "skipReason": "invalid-value"
},
```

## Expected behavior

Renovate is able to parse `0.43b0`, and identify that there is a newer release
at [0.47b0](https://pypi.org/project/opentelemetry-instrumentation/0.47b0/).
