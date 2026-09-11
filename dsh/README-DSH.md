# DSH 侧对接说明（本 fork 的 delta）

本 fork 是 **[FuRongJun-1999/dsh-memory](https://github.com/FuRongJun-1999/dsh-memory)（灵枢 / 白箱知识库）**
的**对接分支**：上游原样保留，本侧只在其上叠**一层薄改动**，供 DSH 的 engram 记忆图谱
当"白箱智能器官"接进来。

> **定位（重要）**：灵枢是**白箱智能**——自带推演（`dex_predict` / `dex_chain` / `dex_compose` /
> `dex_analyze`）、判断（`dex_auto_verify` 的 D_norm 四维信息差）与知识层（学科卡）。
> 它**不是**一张查表用的知识库；engram 才是机器侧的记忆系统（跨会话分层记忆 / 唤醒 / 因果图）。
> 两者是**一个主体的两枚器官**：知识=可共享去情景的，经验=带情景第一人称的。改动取向
> 一律是"让灵枢的判断力进来"，不是"把卡抠出来存进表里"。

## 与上游的差异（`git diff upstream/main` 应只剩这些）

| 路径 | 性质 | 说明 |
|---|---|---|
| `md_cg/whitebox_kb/wisdom/wisdom_cloud.py` | **改** | DSH 侧扩展（见下，文件内以 `---- DSH 侧扩展 ----` 注释标记） |
| `md_cg/whitebox_kb/wisdom/semantic_algo.py` | **增** | engram SemanticScorer 三通道（词汇/共现）重排，零 LLM；`aeis_core` 优先导入 |
| `dsh/start_lingshu.py` | **增** | DSH 侧启动器：whitebox_kb 布局 sys.path、卡库空播种、崩溃 1s 重启、`busy_timeout`、黑匣子日志 |
| `dsh/README-DSH.md` | **增** | 本文 |

### `wisdom_cloud.py` 的 DSH 侧扩展（同步上游时必须回植）

1. **`add_card` op**：engram 缺口闭环的补卡端点（同名查重 + 学习统计 + `add_entry` 落库）。
2. **`learn_stats` op** + `_LEARN_STATS` / `_WEAK_HITS`：收敛观测仪表盘（强/弱/未命中/建簇/补卡计数）。
3. **`respond` 增强**：保留上游四路融合检索，另加
   - `semantic_algo.algo_rerank` 三通道重排（可用 `algo=0` 关闭对比）；
   - 弱命中自动补词网 `_auto_cluster`（同查询 ≥3 次弱命中 → 抽取俗语词建簇写 `clusters_ext.json`）；
   - **口语直答条目字段规整**：上游新翻译层会返 `id=null` / `_from_daily` / 只有 `daily` 文本的条目，
     调用方按 `L{level}/{status}：{action}` 渲染时会渲染成空 → 补 `action`/`status`；
   - **保底卡锚**：整批都无卡背书时，按「具体出招卡优先、模板学科卡兜底」追加卡库出招（直答 + 可溯源两全）；
   - **模板卡降权 0.5 + 排序**：上游种子学科卡的 `response.action` 是模板句「以X知识点回应」，
     是知识容器而非出招；触发词长，会在通用词上与具体识别卡同分甚至更高（实测把「化学_氧化还原」
     顶掉）→ 模板卡降权、直答恒排首位；
   - **尊重 `params.limit`**：上游把 `graph_retrieve` 条数硬编码为 8、无视调用方 limit → 按 limit 截断。
4. **`_seed_cards` 查重窗口 500 → 5000**：上游取 `existing` 用 `limit=500`，近千节点的库会把
   窗口外的既有卡名当成"不存在"→ 重复建卡。

## 同步上游（推荐流程）

```bash
git remote add upstream https://github.com/FuRongJun-1999/dsh-memory   # 首次
git fetch upstream main
git diff upstream/main -- md_cg/whitebox_kb        # 只看本侧 delta
git rebase upstream/main                            # 或 merge，冲突必在 wisdom_cloud.py
```

冲突几乎必落在 `wisdom_cloud.py`：按文件内 `---- DSH 侧扩展 ----` 标记把上述四处扩展回植即可。
`start_lingshu.py` 与 `semantic_algo.py` 是新增文件，上游不会有冲突。

## 不随本 fork 分发

运行期数据**不进版本库**（与上游"数据不随包分发"的约定一致）：
学科卡库 `*.db`、`audit_log/`、`db-backups/`、以及 DSH 侧自建的内部知识卡
（识别卡/补卡，敏感度：内部）。

## 来源与许可

上游工程代码 MIT（见上游 `LICENSE`）；协议理论文本（智能论 v3.x）权利归属上游协议方，
本 fork 只做工程对接，不改协议文本。
