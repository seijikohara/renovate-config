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
| Release age | Every release waits 3 days. Update types and sources without release timestamps are exempt, so they do not stay pending forever. |
| Ranges | `rangeStrategy: auto`; development dependencies keep their ranges instead of being pinned |
| Auto-merge | Minor, patch, pin, and digest updates, plus lock file maintenance, through GitHub native auto-merge with the rebase strategy |
| Majors | Wait for approval on the Dependency Dashboard |
| Groups | GitHub Actions updates, and every Node.js runtime declaration |
| Security | Fixes from Dependabot alerts get the `security` label and require manual review |

## Requirements for a repository

- The Renovate GitHub App has access to the repository.
- "Allow auto-merge" and "Allow rebase merging" are enabled.
- A ruleset or branch protection requires at least one CI status check, so auto-merge waits for CI ([platformAutomerge](https://docs.renovatebot.com/configuration-options/#platformautomerge)).
- Dependabot alerts are enabled, so Renovate can raise security fixes ([vulnerabilityAlerts](https://docs.renovatebot.com/configuration-options/#vulnerabilityalerts)).

## Validation

The `Validate` workflow checks the preset and this repository's own config with the pinned `renovate-config-validator`. To check locally:

```sh
npx --yes --package renovate renovate-config-validator --strict --no-global default.json5
```

## License

[MIT](LICENSE)
