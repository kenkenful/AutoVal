# sanity_ssd_performance_on_diffs JSON test controls

`ocp-diag-autoval-ssd/src/autoval_ssd/cfg/test_suites/`にある
`JQ_P0_test_suite.yaml`以外のYAMLから、各テストの`name`をファイル名にした
単体実行用JSONを作成しています。

YAML間で同じテスト名かつ異なる定義があるため、suite別に
`scripts/<suite名>/<test name>.json`へ保存しています。

## 実行方法

基本形:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner <cmd> \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/<suite-name>/<test-name>.json
```

例:

```bash
sudo SITE_SETTINGS=/home/ttt/AutoVal/site_settings.json env/bin/python -m autoval.autoval_test_runner autoval_ssd.tests.storage_hw_eng.tests.ssd_tests.ssd_synthflash_workload.ssd_synthflash_workload \
  --config /home/ttt/AutoVal/hosts.json \
  --test_control /home/ttt/AutoVal/scripts/sanity_ssd_performance_on_diffs/SSD_Synthflash_Search_Sweep_1H22.json
```

各JSONに対応する`cmd`は、各suiteディレクトリ内の`<suite名>_commands.json`、
または全体一覧の`scripts/all_non_jq_p0_test_suite_commands.json`を参照してください。

## BMCを使わない設定

生成したJSONではBMC/OOBに依存しやすい収集処理を避けるため、次の値を設定しています。

- `collect_drive_data: false`
- `flash_config_logging: false`
- `skip_health_check: true`
- `disable_config_collection: true`

BMC接続自体を完全に避けたい場合は、`--config`で指定するhosts設定から`oob_addr`、`oob_username`、`oob_password`を外したファイルを指定してください。
