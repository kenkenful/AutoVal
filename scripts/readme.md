# AutoVal SSD test JSON controls

`ocp-diag-autoval-ssd/src/autoval_ssd/cfg/test_suites/`にある
`JQ_P0_test_suite.yaml`以外のYAMLから、各テストの`name`をファイル名にした
単体実行用JSONを作成しています。

YAML間で同じテスト名かつ異なる定義があるため、suite別に
`scripts/<suite名>/<test name>.json`へ保存しています。

## 実行方法

1. 実行したいテストの`cmd`とJSONパスを確認します。

```bash
env/bin/python -m json.tool /home/ttt/AutoVal/scripts/all_non_jq_p0_test_suite_commands.json
```

suite別に確認する場合:

```bash
env/bin/python -m json.tool /home/ttt/AutoVal/scripts/sanity_ssd_performance_on_diffs/sanity_ssd_performance_on_diffs_commands.json
```

2. 確認した`cmd`とJSONパスを指定して実行します。

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner <cmd> \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/<suite-name>/<test-name>.json
```

## 実行例

SSD Synthflash:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.storage_hw_eng.tests.ssd_tests.ssd_synthflash_workload.ssd_synthflash_workload \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/sanity_ssd_performance_on_diffs/SSD_Synthflash_Search_Sweep_1H22.json
```

NVMe CLI:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.nvme_cli.nvme_cli \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/JQ_P1_test_suite/NVMECLI.json
```

NVMe format:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.nvme_format.nvme_format \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/JQ_P1_test_suite/NVME_Format_10_Cycles.json
```

SED take ownership:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.sed_check.sed_take_ownership \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/sed_take_ownership.json
```

ファイル名にスペースが含まれる場合は、JSONパスを引用符で囲んでください。

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.flash_firmware_update.flash_firmware_update \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control "/home/ttt/AutoVal/scripts/JQ_P1_test_suite/Flash Firmware Update.json"
```

## JSONの場所

生成したJSONは次のディレクトリにあります。

- `scripts/JQ_P1_test_suite/`
- `scripts/JQ_P2_DIX_FDP_test_suite/`
- `scripts/QLC_test_suite/`
- `scripts/sanity_dix_on_diffs/`
- `scripts/sanity_fdp_on_diff/`
- `scripts/sanity_ssd_performance_on_diffs/`
- `scripts/sanity_uboott_on_diffs/`
- `scripts/sanity_ussdt_on_diffs/`

各JSONに対応する`cmd`は、各suiteディレクトリ内の`<suite名>_commands.json`、
または全体一覧の`scripts/all_non_jq_p0_test_suite_commands.json`を参照してください。

## ログ出力先

ログと結果は`site_settings.json`の設定に従って、主に次の場所へ出力されます。

- `/home/ttt/AutoVal/autoval_logs/`
- `/home/ttt/AutoVal/autoval_results/`

## Flash firmware update

`flash_firmware_update`で使うFirmwareファイルは、`site_settings.json`の
`repository_dir`配下に配置します。

現在の`repository_dir`は次の場所です。

```text
/home/ttt/AutoVal/autoval_repository/
```

SSD用Firmwareは次のディレクトリ構成で配置してください。

```text
/home/ttt/AutoVal/autoval_repository/bin/flash/<vendor>/<MODEL>/
  fw_version_map.json
  fw_binary/
    <firmware file>
```

`<vendor>`はドライブの`manufacturer.lower()`、`<MODEL>`はドライブの
`model.upper()`です。モデル名にスペースがある場合は`_`に変換されます。

例:

```text
/home/ttt/AutoVal/autoval_repository/bin/flash/samsung/MZQLB960HAJR/
  fw_version_map.json
  fw_binary/
    firmware.bin
```

`fw_version_map.json`の例:

```json
{
  "latest": {
    "name": "FW_VERSION_NAME",
    "bin": "firmware.bin",
    "md5": "optional_md5sum"
  },
  "stable": {
    "name": "STABLE_FW_VERSION",
    "bin": "stable_firmware.bin"
  }
}
```

`flash_firmware_update`のJSONで指定する`versions`には、`fw_version_map.json`の
キーを指定します。たとえば`"versions": ["latest", "stable"]`の場合、
上記JSONの`latest`と`stable`が使われます。

## BMCを使わない設定

生成したJSONではBMC/OOBに依存しやすい収集処理を避けるため、次の値を設定しています。

- `collect_drive_data: false`
- `flash_config_logging: false`
- `skip_health_check: true`
- `disable_config_collection: true`

BMC接続自体を完全に避けたい場合は、`--config`で指定するhosts設定から`oob_addr`、`oob_username`、`oob_password`を外したファイルを指定してください。

## 対象ドライブ設定

生成した各テストJSONには、`nvme0n1`だけを対象にするため次の設定を入れています。

```json
{
  "drive_interface": "nvme",
  "drives": [
    "nvme0n1"
  ],
  "include_boot_drive": false,
  "only_boot_drive": false,
  "remove_partition": false,
  "unmount_before_test": false
}
```
