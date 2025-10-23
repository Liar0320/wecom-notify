# wecom-notify

GitHub Action：读取指定 Markdown 文件内容，通过企业微信机器人 webhook 发送 Markdown 消息。适用于发布流程中推送版本更新、部署结果等信息。

## 使用示例

```yaml
- name: push 推送到企业微信
  uses: Liar0320/wecom-notify@v1.0.0
  with:
    body_path: CHANGELOG.md
    robots_key: ${{ secrets.WECOM_ROBOTS_KEY }}
```

可选输入：
- `encoding`：文件编码，默认 `utf-8`
- `dry_run`：设为 `true` 时仅打印 payload，不发送请求

## 输入参数

| 名称 | 必填 | 默认值 | 说明 |
| ---- | ---- | ------ | ---- |
| `robots_key` | 是 | - | 企业微信机器人 webhook key |
| `body_path`  | 是 | - | Markdown 文件路径，将作为消息内容 |
| `encoding`   | 否 | `utf-8` | 读取文件使用的编码 |
| `dry_run`    | 否 | `false` | 仅打印 payload，跳过真实请求 |

## 工作流示例

### 代码库内调试

```yaml
name: Test

on: [push, pull_request]

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "# 更新日志\\n\\n- 测试消息" > CHANGELOG.md
      - uses: ./
        with:
          body_path: CHANGELOG.md
          robots_key: dummy
          dry_run: true
```

### Marketplace 调用

```yaml
name: Manually Triggered WeCom Notify

on:
  workflow_dispatch:
    inputs:
      dry_run:
        default: 'true'
        description: '设为 false 执行真实推送'

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "## 发布说明\\n\\n- 手动触发测试" > CHANGELOG.md
      - uses: Liar0320/wecom-notify@v1.0.0
        with:
          body_path: CHANGELOG.md
          robots_key: ${{ secrets.WECOM_ROBOTS_KEY }}
          dry_run: ${{ github.event.inputs.dry_run }}
```

## 运行时依赖

- `curl`
- `jq`
- `iconv`

GitHub 托管 runners 默认已提供以上工具。

## 许可证

本项目基于 MIT License 发布。
