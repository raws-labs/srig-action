# srig-action

GitHub Action "SiliconRig HIL": installs the `srig` CLI, opens a session on a real board, flashes firmware, captures serial output, and always ends the session. Composite action, bash steps only, no build. Public, github.com/raws-labs/srig-action; consumed as `raws-labs/srig-action@v1`.

## Build, test, run
- No build, tests, or CI here; `action.yml` is the whole product. The only way to exercise a change is a workflow in another repo that references the branch or SHA with a real `SRIG_API_KEY` (a run needs a live board).
- Release: tag `v1.x.y`, then move the floating `v1` tag to the same commit; consumers pin `@v1`.

## Layout
- `action.yml`: inputs `api-key`, `board`, `firmware` (empty means session only), `serial-timeout` (default `30s`), `serial-log` (default `serial-output.txt`), `cli-version` (default `latest`); outputs `session-id`, `serial-log`. Steps: install CLI, `srig session create --board ... --json`, `srig flash --session`, `srig serial --timeout --log` (both only when `firmware` is set), `srig session end` under `if: always()`.

## Conventions
- Never change `name:` in `action.yml`. The Marketplace listing is keyed by that unique name and survived the org transfer and repo rename only because the name stayed. `author:` and `description:` are free to edit; keep `branding` (icon `cpu`, color `green`).
- The serial and session-end steps end in `|| true` on purpose: hitting the capture timeout is the normal path, and cleanup must never fail the job.

## Gotchas
- CLI install: `cli-version: latest` resolves the tag from `api.github.com/repos/raws-labs/srig-cli/releases/latest` and downloads the goreleaser tarball `srig_<version-without-v>_<os>_<arch>.tar.gz`; there is no bare-binary release asset. Both curls must stay `curl -fsSL`: `-L` because a moved repo answers 301 with a JSON body that makes `jq -r .tag_name` print the literal string `null`, and `-f` so a bad URL fails at the download instead of writing GitHub's 404 page to `/usr/local/bin/srig` and failing steps later with a confusing tar or exec error. Both happened after the org move; the `::error::` exit on an empty or `null` tag is the guard.
- `uses: siliconrig/action@v1` (the pre-move path) still resolves through GitHub's redirect; do not document it, but do not break it either.
