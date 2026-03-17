# AI 对美国就业市场的影响

分析美国经济中每个职业对 AI 和自动化的暴露程度，数据来源于美国劳工统计局 [职业展望手册](https://www.bls.gov/ooh/)（OOH）。

**在线演示：[yuevthins.github.io/jobs](https://yuevthins.github.io/jobs/)**

![AI 暴露度树形图](jobs.png)

## 项目内容

BLS 职业展望手册涵盖 **342 个职业**，覆盖美国经济的各个行业，提供工作职责、工作环境、学历要求、薪资和就业预测等详细数据。我们爬取了所有数据，使用大语言模型为每个职业的 AI 暴露度评分，并构建了交互式树形图可视化。

## 数据流水线

1. **爬取** (`scrape.py`) — 使用 Playwright（非无头模式，BLS 会屏蔽机器人）下载所有 342 个职业页面的原始 HTML 到 `html/`。
2. **解析** (`parse_detail.py`, `process.py`) — 使用 BeautifulSoup 将原始 HTML 转换为干净的 Markdown 文件，存储在 `pages/`。
3. **制表** (`make_csv.py`) — 提取结构化字段（薪资、学历、就业人数、增长前景、SOC 代码）到 `occupations.csv`。
4. **评分** (`score.py`) — 将每个职业的 Markdown 描述发送给 LLM（通过 OpenRouter 调用 Gemini Flash）和评分标准。每个职业获得 0-10 的 AI 暴露度评分及理由。结果保存到 `scores.json`。
5. **构建网站数据** (`build_site_data.py`) — 将 CSV 统计数据和 AI 暴露度评分合并为前端使用的精简 `site/data.json`。
6. **网站** (`site/index.html`) — 交互式树形图可视化，面积 = 就业人数，颜色 = AI 暴露度（绿色到红色）。

## 关键文件

| 文件 | 描述 |
|------|------|
| `occupations.json` | 342 个职业的主列表，包含标题、URL、分类、slug |
| `occupations.csv` | 汇总统计：薪资、学历、就业人数、增长预测 |
| `scores.json` | 所有 342 个职业的 AI 暴露度评分（0-10）及理由 |
| `html/` | BLS 的原始 HTML 页面（数据源，约 40MB） |
| `pages/` | 每个职业页面的干净 Markdown 版本 |
| `site/` | 静态网站（树形图可视化） |

## AI 暴露度评分

每个职业在 **AI 暴露度** 单一维度上评分 0 到 10 分，衡量 AI 将在多大程度上重塑该职业。评分同时考虑直接自动化（AI 取代工作）和间接效应（AI 使工人效率大幅提升，从而减少所需人数）。

一个关键信号是该工作的成果是否本质上是数字化的——如果工作可以完全在家通过电脑完成，AI 暴露度本身就很高。反之，需要现场操作、手工技能或实时人际互动的工作有天然屏障。

**数据集中的校准示例：**

| 分数 | 含义 | 示例 |
|------|------|------|
| 0-1 | 极低 | 屋顶工、清洁工、建筑工人 |
| 2-3 | 低 | 电工、管道工、护理助手、消防员 |
| 4-5 | 中等 | 注册护士、零售工人、医生 |
| 6-7 | 高 | 教师、管理人员、会计师、工程师 |
| 8-9 | 极高 | 软件开发人员、律师助理、数据分析师、编辑 |
| 10 | 最高 | 医学转录员 |

所有 342 个职业的平均暴露度：**5.3/10**。

## 可视化

主要可视化是一个交互式**树形图**，其中：
- 每个矩形的**面积**与就业人数成正比
- **颜色**表示 AI 暴露度，从绿色（安全）到红色（高暴露）
- **布局**按 BLS 分类分组
- **悬停**显示详细工具提示，包括薪资、就业人数、前景、学历、暴露度评分和 LLM 评分理由

## 安装

```
uv sync
uv run playwright install chromium
```

需要在 `.env` 中配置 OpenRouter API 密钥：
```
OPENROUTER_API_KEY=your_key_here
```

## 使用方法

```bash
# 爬取 BLS 页面（只需执行一次，结果缓存在 html/）
uv run python scrape.py

# 从 HTML 生成 Markdown
uv run python process.py

# 生成 CSV 汇总
uv run python make_csv.py

# AI 暴露度评分（使用 OpenRouter API）
uv run python score.py

# 构建网站数据
uv run python build_site_data.py

# 本地启动网站
cd site && python -m http.server 8000
```
