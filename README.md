# syl-listing-rules

独立规则中心仓库（仅管理 `syl-listing` 的规则文件与发布产物）。

## 目录结构

- `rules/`：规则源文件（`title.yaml`、`bullets.yaml`、`description.yaml`、`search_terms.yaml`）
- `scripts/build_bundle.sh`：打包脚本，生成 `dist/rules-bundle.tar.gz`
- `.github/workflows/release.yml`：发布流程（tag 触发），自动把规则包上传到 GitHub Releases

## 规则 Schema（必填）

`syl-listing` 已取消规则向后兼容，`rules/*.yaml` 必须显式包含以下字段：

- `execution.generation.protocol`：`text` 或 `json_lines`
- `execution.repair.granularity`：`whole` 或 `item`
- `execution.fallback.disable_thinking_on_length_error`：`true/false`

约束关系：

- `protocol=text` 时，`output.format` 必须是 `plain_text`
- `protocol=json_lines` 时，`output.format` 必须是 `json_object`
- `granularity=item` 时，`output.lines` 必须大于 1

## 版本号规范

发布 tag 必须匹配：`rules-v*`（否则不会触发发布 CI）。

推荐格式：

- `rules-vYYYY.MM.DD`（当天第一次发布）
- `rules-vYYYY.MM.DD-N`（当天第 N 次修订，例如 `rules-v2026.02.24-2`）

## 发布前检查清单

1. 仅修改 `rules/*.yaml`（规则源），不要改无关文件。
2. 本地确认四个规则文件都存在：
   - `rules/title.yaml`
   - `rules/bullets.yaml`
   - `rules/description.yaml`
   - `rules/search_terms.yaml`
3. 执行一次本地打包，确认产物可生成。
4. 用 `syl-listing` 做至少 1 次真实生成回归（建议用固定测试样例）。

## 本地打包（可选但推荐）

```bash
./scripts/build_bundle.sh
```

输出：

- `dist/rules-bundle.tar.gz`

## 标准发布流程

```bash
# 1) 提交规则变更
git add rules/*.yaml
git commit -m "feat(rules): 更新规则说明"
git push origin master

# 2) 打发布 tag（示例）
git tag rules-v2026.03.01

# 3) 推送 tag，触发 GitHub Actions 发布
git push origin rules-v2026.03.01
```

发布工作流会自动执行：

1. 校验 4 个规则文件是否存在
2. 打包 `rules-bundle.tar.gz`
3. 创建 GitHub Release 并上传规则包资产

## 发布后验收

### 在规则仓库验收

```bash
# 查看最近 release
gh release list --limit 5

# 查看指定 release 资产
gh release view rules-v2026.03.01
```

需要确认：

1. 新 tag 已出现在 release 列表中；
2. 资产里存在 `rules-bundle.tar.gz`。

### 在客户端验收（syl-listing）

运行一次生成命令（示例）：

```bash
syl-listing pinpai.md
```

日志中应看到规则中心更新成功或已是最新版本的信息，并可正常生成输出文件。

## 回滚 / 紧急修复

### 方案 A：快速修复（推荐）

1. 在 `master` 修复规则；
2. 发布一个新 tag（如 `rules-v2026.02.24-3`）；
3. 客户端使用 `release: latest` 会自动拉到修复版。

### 方案 B：临时固定到旧版本

在客户端 `~/.syl-listing/config.yaml` 中临时指定：

```yaml
rules_center:
  release: rules-v2026.02.24-2
```

等新规则发布稳定后，再改回 `latest`。

## 客户端配置

在 `syl-listing` 的 `~/.syl-listing/config.yaml` 中：

```yaml
rules_center:
  owner: hooziwang
  repo: syl-listing-rules
  release: latest
  asset: rules-bundle.tar.gz
  timeout_sec: 20
  strict: false
```
