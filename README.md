# AutoVal NVMe CLI Test Runbook

This directory contains local run settings for running the AutoVal SSD
`nvme_cli` test against the data NVMe drive only.

## Files

- `hosts.json`: DUT connection settings.
- `control.json`: Test control for direct single-test execution.
- `test.yaml`: Suite file for suite-based execution.
- `site_settings.json`: Local writable log/result directory settings.

## Current Target

The current configuration targets only:

```text
nvme0n1
```

The SATA/system boot drive is not included.

## Direct Test Execution

Run `nvme_cli` directly with `control.json`:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.nvme_cli.nvme_cli \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/control.json
```

## Suite Execution

Run the same test through `test.yaml`:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner \
  --suite /home/ttt/AutoVal/test.yaml \
  --config /home/ttt/AutoVal/hosts.json
```

## Notes

- `control.json` and `test.yaml` currently skip `_validate_power_mode`.
- `remove_partition` is set to `false`.
- `unmount_before_test` is set to `false`.
- If the target NVMe name differs from `nvme0n1`, update both `control.json` and `test.yaml`.
- Logs are written under `/home/ttt/AutoVal/autoval_results/`.
