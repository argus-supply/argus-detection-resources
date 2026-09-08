> 已停用全量同步与 Release 发布工作流。ARGUS 使用独立按需检索服务（主应用仓库的 services/argus-intel-service）；本目录仅保留旧采集实现供参考，不作为数据分发源。

# ARGUS 检测资源与覆盖索引

每日北京时间 **04:03** 独立同步模板、指纹和覆盖缺口清单，也可手动触发。供应数据不会自动进入现有 `argus-rules`、Catalog 或 ARGUS 的执行环境。

| 来源 | 跟踪方式 | 数据属性 |
| --- | --- | --- |
| projectdiscovery/nuclei-templates | 官方最新 Release 对应 Git tag | Nuclei 模板 |
| 0x727/FingerprintHub | 默认分支提交 | `plugins/` 模板、聚合组件指纹 JSON |
| edoardottt/missing-cve-nuclei-templates | `data/all.json` | 缺少模板的 CVE 清单；不是模板代码 |
| topscoder/nuclei-wordfence-cve | 默认分支提交 | 社区 Nuclei 模板 |

Git blob ID 用于识别变更与删除。默认模板源每轮最多处理 20,000 文件；覆盖缺口源处理一份 JSON。首次超出预算会保存续采状态。模板按 YAML 结构检查，指纹按 JSON 语法检查；格式错误只进入覆盖元数据，不进入资源包。不运行上游脚本、模板或 PoC。

## 发布资产

- `resources.tar.gz`：按来源命名空间保存模板、指纹和发现的许可文件，避免文件名覆盖。
- `records.jsonl.gz`：完整资源元数据、CVE 关联、文件 SHA-256、验证结果与覆盖缺口。
- `indexes.json.gz`：CVE → 资源、模板 ID → 路径、组件 → 指纹文件。一个模板 ID 可以映射多个来源路径，不能把它当作全局唯一键。
- `delta.jsonl.gz`：元数据增删改；资源内容使用完整资源包更新。
- `state.sqlite.gz`：下一轮采集所需的源数据和进度。
- `manifest.json`：格式版本、来源提交、源状态、基础版本和资产校验和。

结构检查不等于 Nuclei 引擎验证，也不意味着模板已获得执行批准。`runtime_validated: false` 明确表达这一边界。CVE 缺口清单表达上游判断，消费者应结合本地实际模板索引计算覆盖率。

下载后先校验 manifest；导入放入新版本目录并校验完成后再切换。当前 ARGUS 的消费适配尚未在本仓库实现，现有 Catalog/操作员激活流程保持独立。

## 本地验证

需要 Python 3.12、Git；发布/恢复需要 GitHub CLI。

```sh
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
python scripts/sync.py --source missing-cve --limit 10 --repository OWNER/argus-detection-resources
```

数据输出位于 `.work/` 与 `dist/`，不提交 Git。仓库初始为私有，各来源许可和归属随来源 URL、提交版本和许可文件保留。详见 [architecture.md](docs/architecture.md)。
