# srig-action

GitHub Action for [SiliconRig](https://siliconrig.dev) hardware-in-the-loop CI/CD.

Downloads `srig-cli`, creates a session, flashes firmware, captures serial output, and cleans up.

## Usage

```yaml
- uses: raws-labs/srig-action@v1
  with:
    api-key: ${{ secrets.SRIG_API_KEY }}
    board: esp32-s3
    firmware: build/firmware.bin
```

### With custom serial timeout and log

```yaml
- uses: raws-labs/srig-action@v1
  id: hil
  with:
    api-key: ${{ secrets.SRIG_API_KEY }}
    board: esp32-s3
    firmware: build/firmware.bin
    serial-timeout: 60s
    serial-log: test-output.txt

- name: Check output
  run: grep "System ready" ${{ steps.hil.outputs.serial-log }}
```

### Publish the run as a record

With `record: true` the run is recorded and published as a run record: a page
anyone can open to play back the flash, the boot and the serial output, with
the firmware hash and the board it ran on. The link and a "Run on SiliconRig"
badge go into the job summary, and the serial output still lands in
`serial-log`. `serial-timeout` is the record length, 10 to 120 seconds.

```yaml
- uses: raws-labs/srig-action@v1
  id: hil
  with:
    api-key: ${{ secrets.SRIG_API_KEY }}
    board: esp32-s3
    firmware: build/firmware.bin
    serial-timeout: 30s
    record: ${{ github.ref == 'refs/heads/main' }}

- run: echo "Recorded at ${{ steps.hil.outputs.record-url }}"
  if: steps.hil.outputs.record-url != ''
```

An account keeps up to 20 records, so record on `main` or on tags rather than
on every push. A record is billed like any session. Needs srig-cli 0.5.0 or
newer, which the default `cli-version: latest` installs.

### Session only (no firmware)

```yaml
- uses: raws-labs/srig-action@v1
  with:
    api-key: ${{ secrets.SRIG_API_KEY }}
    board: esp32-s3
    firmware: ""
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | Yes | | SiliconRig API key |
| `board` | Yes | | Board type: `esp32-s3`, `stm32-h753`, `stm32-f446`, or `rp2350` |
| `firmware` | No | | Path to firmware binary |
| `serial-timeout` | No | `30s` | Serial capture duration |
| `serial-log` | No | `serial-output.txt` | File to save serial output |
| `cli-version` | No | `latest` | srig-cli version to install |
| `record` | No | `false` | Publish the run as a run record (needs `firmware`) |
| `record-name` | No | repository@short-sha | Name shown with the record |

## Outputs

| Output | Description |
|---|---|
| `session-id` | The session ID that was created |
| `serial-log` | Path to the serial output log file |
| `record-id` | ID of the run record, when `record` is true |
| `record-url` | Page that plays the run record back, when `record` is true |
