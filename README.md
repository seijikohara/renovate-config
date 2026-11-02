# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset for seijikohara repositories.

## Usage

Extend the preset from the repository's `renovate.json5`, then add only the rules that the repository needs:

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: ["github>seijikohara/renovate-config:default.json5"],
  packageRules: [
    // Repository-specific groups and version holds.
  ],
}
```

The reference must include the `.json5` file name. A bare `github>seijikohara/renovate-config` reference looks only for `default.json` ([Preset file naming](https://docs.renovatebot.com/config-presets/#preset-file-naming)).

Settings in the repository override the preset. Repository `packageRules` run after the preset rules, so a repository rule wins when both match a dependency.

## Policy

[`default.json5`](default.json5) explains each setting inline. In summary:

| Area | Setting |
|---|---|
| Base preset | `config:best-practices` |
| Commits | Conventional Commits, `chore(deps): ...` for every update |
| Schedule | Monday 00:00–08:59 Asia/Tokyo, for regular updates and lock file maintenance |
| Release age | Every release waits 3 days. Update types and sources without release timestamps are exempt, so they do not stay pending forever. Lock files rely on the package manager's cooldown ([Package manager release age](#package-manager-release-age)). |
| Ranges | `rangeStrategy: auto`; development dependencies keep their ranges instead of being pinned |
| Auto-merge | Minor, patch, pin, and digest updates, plus lock file maintenance, through GitHub native auto-merge with the rebase strategy |
| Majors | Wait for approval on the Dependency Dashboard |
| Groups | GitHub Actions updates, and every Node.js runtime declaration |
| Security | Fixes from Dependabot alerts get the `security` label and require manual review |

## Package manager release age

The 3-day hold covers only the dependencies that Renovate updates. Renovate does not hold lock file maintenance, and it does not see the transitive dependencies that the package manager resolves into the lock file ([Minimum Release Age](https://docs.renovatebot.com/key-concepts/minimum-release-age/)). Each repository therefore relies on its package manager's cooldown:

| Package manager | Repository setting |
|---|---|
| pnpm 11+ | None. pnpm holds every release for 1 day by default ([`minimumReleaseAge`](https://pnpm.io/settings/dependency-resolution#minimumreleaseage)). An explicit value also turns on [`minimumReleaseAgeStrict`](https://pnpm.io/settings/dependency-resolution#minimumreleaseagestrict), which makes a security fix newer than the cooldown fail to resolve, so repositories keep the default. |
| npm 11.10+ | `min-release-age=3` in `.npmrc` ([`min-release-age`](https://docs.npmjs.com/cli/v11/using-npm/config#min-release-age)). Renovate then skips its own `--before` flag and leaves the hold to npm. |
| Bun | `install.minimumReleaseAge = 259200` (3 days in seconds) in `bunfig.toml` ([`install.minimumReleaseAge`](https://bun.com/docs/runtime/bunfig#install-minimumreleaseage)). Bun has no hold by default. |

With npm and Bun, a security fix newer than 3 days waits until it ages.

## Requirements for a repository

- The Renovate GitHub App has access to the repository.
- "Allow auto-merge" and "Allow rebase merging" are enabled.
- A ruleset or branch protection requires at least one CI status check, so auto-merge waits for CI ([platformAutomerge](https://docs.renovatebot.com/configuration-options/#platformautomerge)).
- Dependabot alerts are enabled, so Renovate can raise security fixes ([vulnerabilityAlerts](https://docs.renovatebot.com/configuration-options/#vulnerabilityalerts)).
- Dependabot security updates stay enabled. Renovate does not fix a vulnerability that exists only in the lock file, such as one in a transitive dependency, but Dependabot can ([Dependabot security updates](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-security-updates)).

## Validation

The `Validate` workflow checks the preset and this repository's own config with the pinned `renovate-config-validator`. To check locally:

```sh
npx --yes --package renovate renovate-config-validator --strict --no-global default.json5
```

## License

[MIT](LICENSE)
