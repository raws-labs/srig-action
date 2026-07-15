# siliconrig-action

GitHub Action for [siliconrig](https://siliconrig.dev) hardware-in-the-loop CI/CD.

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
| `api-key` | Yes | | siliconrig API key |
| `board` | Yes | | Board type: `esp32-s3`, `stm32-h753`, `stm32-f446`, or `rp2350` |
| `firmware` | No | | Path to firmware binary |
| `serial-timeout` | No | `30s` | Serial capture duration |
| `serial-log` | No | `serial-output.txt` | File to save serial output |
| `cli-version` | No | `latest` | srig-cli version to install |

## Outputs

| Output | Description |
|---|---|
| `session-id` | The session ID that was created |
| `serial-log` | Path to the serial output log file |
