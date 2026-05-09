---
name: a-share-short-review
description: Generate Chinese A-share short-term daily market reviews using MXSKILLS market/search data and a punchy retail-trader replay style. Use when the user asks for A股复盘, 短线复盘, 涨停复盘, 主线/连板/创新高分析, or asks to imitate the provided April 2026 HTML review style.
---

# A股短线复盘

## Workflow

1. Read the requested date range. Treat weekends and exchange holidays as non-trading days; write a "周末消息/下周策略" review instead of inventing trading data.
2. Use 妙想 skills first:
   - `mx-data`: query index close, pct change, amount; 全部A股上涨家数、下跌家数、涨停家数、跌停家数、成交额.
   - `mx-search`: query same-day A股复盘、创历史新高、连板股、龙虎榜、主线、资金流向.
   - For non-trading days, use `mx-search` for 周末政策、产业催化、机构策略、下周主线.
3. Prefer structured `mx-data` values over news snippets when they conflict. Use `mx-search` for context that `mx-data` does not reliably return, especially 创历史新高数量、行业分布、连板梯队、龙虎榜和主力资金.
4. Keep source facts internally traceable. Mention when a metric is from `mx-search`/资讯口径 and avoid over-precise claims if multiple口径 differ.
5. Write one Markdown file per date using `YYYY-MM-DD+A股短线复盘.md` unless the user specifies another naming convention.

## Style

Use the style guide in `references/style-guide.md` when imitating the sample HTML reviews.

Core voice:
- Direct, trading-log tone; not a broker research report.
- Use short judgments: "怎么说呢", "说白了", "这不是没行情，是结构性行情", "别瞎追高".
- Emphasize contradiction: index vs individual stocks, volume vs price, old main line vs new branches.
- Preserve the repeated structure: 核心结论 -> 市场全景数据 -> 创新高/主线 -> 连板梯队 -> 资金 -> 风险 -> 操作建议.
- Do not write investment promises. End with risk disclaimer if the output is user-facing.

## Required Sections

For a trading day:
1. `◆ 一、今日核心结论`
2. `◆ 二、市场全景数据`
3. `◆ 三、⭐创历史新高与主线分析`
4. `◆ 四、连板股梯队`
5. `◆ 五、资金流向与龙虎榜`
6. `◆ 六、风险提示`
7. `◆ 七、操作建议`

For a non-trading day:
1. `◆ 一、周末核心结论`
2. `◆ 二、上个交易日盘面回放`
3. `◆ 三、周末消息面`
4. `◆ 四、下周主线推演`
5. `◆ 五、风险提示`
6. `◆ 六、操作建议`

## Data Checklist

Capture these whenever available:
- Index table: 上证指数、深证成指、创业板指、科创50、北证50.
- Breadth: 成交额、上涨家数、下跌家数、涨停、跌停、涨跌比.
- New highs: total count, industry concentration, representative stocks.
- Limit-up chain: highest board, 3板以上, 2板 groups,晋级率 if available.
- Money flow: full-market main flow, top industry inflows/outflows, top stock inflows/outflows, institution/沪深股通龙虎榜.
- Main line judgment: strongest theme, whether it is continuation, acceleration, divergence, or high-low switching.

## Writing Rules

- Start with a blunt one-paragraph conclusion before tables.
- Use tables for data, but do not let tables replace judgment.
- If data is missing, say the metric was not returned by 妙想 and use a qualitative `mx-search` summary.
- Keep each daily review around 900-1500 Chinese characters unless the user asks for a long version.
- When creating files, save under the current workspace unless the user specifies another folder.
