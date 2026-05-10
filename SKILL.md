---
name: a-share-short-review
description: Generate Chinese A-share short-term daily market reviews using MXSKILLS first and Eastmoney public web APIs as fallback, with a punchy retail-trader replay style. Use when the user asks for A股复盘, 短线复盘, 涨停复盘, 主线/连板/创新高分析, or asks to imitate the provided April 2026 HTML review style.
---

# A股短线复盘

## Workflow

1. Read the requested date range. Treat weekends and exchange holidays as non-trading days; write a "周末消息/下周策略" review instead of inventing trading data.
2. Use 妙想 skills first:
   - `mx-data`: query index close, pct change, amount; 全部A股上涨家数、下跌家数、涨停家数、跌停家数、成交额.
   - `mx-search`: query same-day A股复盘、创历史新高、连板股、龙虎榜、主线、资金流向.
   - `mx-xuangu` / `MX_StockPick`: for historical-new-high stocks, query `YYYY年M月D日创历史新高的A股，显示股票简称、东财行业、概念、总市值`. This is the primary source for the `创新高历史个股分析` section.
   - For every trading-day review, also query the previous trading day for the same core market breadth and main-line context unless the user explicitly says not to. Main-line judgment must compare today with the previous trading day; do not infer a main line from a single day's strength alone.
   - For non-trading days, use `mx-search` for 周末政策、产业催化、机构策略、下周主线.
3. If 妙想 returns empty, malformed, rate-limited, irrelevant, or clearly unstable data, fall back to 东方财富 public web APIs for the missing fields. Keep the fallback low-frequency and source-tagged.
4. If both 妙想 and 东方财富 public APIs fail or do not expose a historical field, use public web review sources as the third priority. Prefer established financial/news sites such as 东方财富网/证券时报/中国证券报/新浪财经/中新经纬/金融界/21财经/经济观察网. Use these sources mainly for market breadth, limit-up/down counts, market-review context, major themes, and money-flow summaries. Do not treat social posts as primary data unless no other source exists.
5. Prefer structured `mx-data` values over news snippets when they conflict. If using fallback data, prefer 东方财富 structured JSON over text snippets, then public web review data. Use `mx-search`/资讯 only for context that structured sources do not reliably return, especially 创历史新高数量、行业分布、连板梯队、龙虎榜和主力资金.
6. Keep source facts internally traceable. Mention when a metric is from `mx-search`/资讯口径、东方财富公开接口、or 公开网络复盘口径 and avoid over-precise claims if multiple口径 differ.
7. Write one Markdown file per date using `YYYY-MM-DD+A股短线复盘.md` unless the user specifies another naming convention.

## Data Fallback: Eastmoney Public APIs

Use this only when 妙想 is missing or unstable for a needed field. These are web endpoints, not a guaranteed official quota API, so request slowly and cache results.

Common endpoint families:
- Quote/detail: `https://push2.eastmoney.com/api/qt/stock/get`
- Lists/pages: `https://push2.eastmoney.com/api/qt/clist/get`
- Unified quote list: `https://push2.eastmoney.com/api/qt/ulist.np/get`
- K-line/history when needed: `https://push2his.eastmoney.com/api/qt/stock/kline/get`
- Limit-up/market pool pages may live under Eastmoney quote/list APIs or stockrank/pool endpoints; verify response fields before trusting them.

Fallback priorities:
1. Index close, pct change, and amount: use Eastmoney index quote/K-line endpoints when `mx-data` fails.
2. Breadth and market temperature: use Eastmoney list APIs to calculate or retrieve上涨家数、下跌家数、涨停、跌停、成交额 when `mx-data` fails.
3. 涨停池/连板池/炸板池: use Eastmoney pool/list endpoints when available; if fields are absent or inconsistent, write only aggregate counts and say detailed ladder was not stable.
4. 板块强度/行业涨跌幅: use Eastmoney board/industry list endpoints as a fallback to identify candidate themes, then verify with core-stock performance.
5. 主力资金流向: use Eastmoney money-flow endpoints only if the returned field names and date match the target date. Otherwise mark资金明细未稳定返回.

Rate and reliability rules:
- Treat Eastmoney public endpoints as low-frequency fallback, not high-frequency infrastructure.
- Default to 1 request per second; use 2-3 seconds between paged/list requests.
- On failure or empty response, retry with exponential backoff such as 5s, 15s, 60s, then stop.
- Cache raw JSON/CSV under the workspace (`mx_output` or another obvious data folder) before writing reviews.
- Never hide fallback usage. In the review, say “东方财富公开接口口径” when a key metric came from fallback.
- If both 妙想 and Eastmoney disagree, use structured values only when date, market, and field definitions match; otherwise write a qualitative judgment instead of forcing a precise number.

## Third Priority: Public Web Review Sources

Use public web review sources only after 妙想 and 东方财富 public APIs are insufficient for the requested historical field.

Allowed usage:
- Historical上涨家数/下跌家数、涨停/跌停、成交额、三大指数收评.
- Same-day major themes, leading sectors, obvious risk events, and headline money-flow summaries.
- Cross-checking Eastmoney public API outputs when fields are missing or suspicious.

Rules:
- Prefer reputable financial/news sources over forums. Use forum/social content only as a last resort and label it clearly.
- Cross-check at least two sources for important numbers when possible. If values differ, write “约/超/近” and cite the source口径 in prose.
- Do not copy long passages. Summarize the facts and keep source URLs in notes or final response.
- In the review, label key fallback data as “公开网络复盘口径” when it did not come from 妙想 or 东方财富 structured JSON.
- If public sources only provide qualitative descriptions, write qualitative market judgment instead of inventing missing fields.

## Style

Use the style guide in `references/style-guide.md` when imitating the sample HTML reviews.

Core voice:
- Direct, trading-log tone; not a broker research report.
- Use short judgments: "怎么说呢", "说白了", "这不是没行情，是结构性行情", "别瞎追高".
- Emphasize contradiction: index vs individual stocks, volume vs price, old main line vs new branches.
- Preserve the repeated structure: 核心结论 -> 市场全景数据 -> 创新高历史个股分析 -> 主线判断 -> 连板梯队 -> 资金 -> 风险 -> 操作建议.
- Do not write investment promises. End with risk disclaimer if the output is user-facing.

## Required Sections

For a trading day:
1. `◆ 一、今日核心结论`
2. `◆ 二、市场全景数据`
3. `◆ 三、⭐ 创新高历史个股分析`
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
- New highs: total count, industry concentration, core table with 股票/细分方向/市值/驱动因素, and the main-line judgment derived from the new-high pool.
- Limit-up chain: highest board, 3板以上, 2板 groups,晋级率 if available.
- Money flow: full-market main flow, top industry inflows/outflows, top stock inflows/outflows, institution/沪深股通龙虎榜.
- Main line judgment: strongest theme, whether it is continuation, acceleration, divergence, or high-low switching.

## Main Line Judgment

Judge short-term A-share main lines by market proof, not story logic. The core question is: after divergence, does money come back, and when it comes back does it spread through the whole sector? A one-day rise is only a rebound. Repeated inflow, core-stock new highs, and sector-wide diffusion are required before calling a direction a main line.

Use this sequence before writing `◆ 三、⭐ 创新高历史个股分析`:

1. **Market money-making effect first.** Check limit-up count, highest board, failed-board rate if available, limit-down count, total turnover, whether indices rise with volume, and whether strong stocks have premium. If money-making effect is poor, even a strong sector is only a light-position trial. If money-making effect is good, then identify the strongest direction.
2. **Always compare with the previous trading day.** A single-day surge is insufficient. Check whether yesterday's strong direction continues, repairs after divergence, or is replaced. Compare: limit-up count, limit-down count, highest board, total turnover, main inflow/outflow, new-high concentration, core-stock performance, and whether yesterday's leaders have premium or negative feedback today.
3. **Sector strength third.** A real main line usually has several traits: sector index expands volume or trends along moving averages; core stocks keep making new highs or hold high levels without collapsing; the sector resists decline on divergence days and repairs proactively the next day; the sector has a ladder of leader, large-cap center, trend stocks, and supplement stocks; after news stimulation it does not become a one-day theme but repeatedly sees money return; high-turnover and high-market-cap stocks can rise, proving institutional and large-capital participation.
4. **Core stocks matter more than杂毛.** Do not judge the main line by random small stocks. Judge whether core names lead and whether the chain diffuses. Example logic: if 中际旭创、东山精密 pull up after weak opens and CPO/PCB/光模块/算力硬件 keep receiving capital while 天华新能、湖南裕能、德方纳米 spike and fade, AI hardware is the stronger main line and新能源 is only rotational repair.
5. **Divergence-day return is the key test.** On climax days everything can rise. The real test is whether core stocks are bought after selling pressure, recover after intraday drops, keep trend levels, and see late-session buying. Weak sectors usually spike with sentiment, then lose VWAP/intraday average, lose the open, fall below zero, and fail to see afternoon return.
6. **Watch sector seesaws.** Short-term capital is limited. When one direction rises and another jumps then fades, money is choosing. Ask: which direction strengthens after a weak open? Which direction is sold after a spike? Which direction has core stocks making new highs? Which direction survives only on low-level supplement stocks? The answer usually identifies the main line.
7. **Translate judgment into action.** Main-line core-stock divergence can be bought on dips. Weak rotational selloffs are not default buys. Do not chase杂毛 on main-line climax days. Sell rotational names that spike and fade. For secondary-line positions, reduce when they lose intraday average, reduce again when they lose the open, and exit the rest when they lose zero or key daily levels.

One-sentence rule: logic is only an entry reason; 盘面强度 is the evidence. Main-line judgment must move from "I think it has logic" to "capital repeatedly verifies it is the strongest direction."

## New High Section Rule

Every trading-day review must include `◆ 三、⭐ 创新高历史个股分析`. This section follows the sample HTML review structure: first quantify today's new-high pool, then list core new-high stocks, then make the main-line judgment.

Required order:
1. One blunt paragraph: today's 创历史新高 count, main industry concentration, and judgment. Example: `12只个股收盘价创历史新高，电子/建筑装饰/机械设备最集中。数量不大，但在恐慌后修复日能新高，说明资金只认核心，不是全市场主升。`
2. `板块分布` table when source data supports industry distribution. Columns: `板块`, `数量`, `代表股`, `强度/判断`. If industry distribution is unavailable after all fallback sources, write one sentence explaining the missing field instead of silently skipping it.
3. Required `创新高历史核心个股` table. Columns must be exactly: `股票`, `细分方向`, `市值`, `驱动因素`.
4. `主线分析` paragraph after the table. This paragraph must connect the new-high pool to the broader main-line judgment.

Rules:
- Do not omit `创新高历史核心个股` table. If data is unavailable after 妙想、东方财富公开接口、and public web review sources, write `公开源未稳定返回完整新高池` in the table and explain that the main-line judgment falls back to breadth/limit-up/sector strength.
- `市值` should use same-day A股总市值 or流通市值 when public sources provide it. If only partial market-cap data is available, write the available value and mark others as `公开源未披露`; do not invent market cap.
- New-high data must feed the main-line judgment. A sector with many new highs but poor breadth is局部抱团; a sector with repeated new highs after divergence has main-line evidence.
- Prefer representative core/high-turnover stocks over long lists of small caps. Use 5-8 rows for the core table unless the user asks for a full list.

## New-High Primary Source: MX_StockPick

For `创新高历史个股分析`, first use 妙想选股能力 through the local `mx-xuangu` skill, which maps to `MX_StockPick`.

Recommended query:

```powershell
$env:PYTHONIOENCODING='utf-8'; $env:PYTHONUTF8='1'; python 'C:\Users\33256\.codex\skills\mx-xuangu\mx_xuangu.py' "YYYY年M月D日创历史新高的A股，显示股票简称、东财行业、概念、总市值"
```

Usage rules:

- Read the CSV/JSON path printed by `mx_xuangu.py`; do not rely on memory. On Windows, keep `PYTHONIOENCODING=utf-8` and `PYTHONUTF8=1` if the command hits GBK encoding errors.
- Treat returned row count as today's historical-new-high stock count, after checking the date fields match the target trading day.
- Use `名称`/`股票简称` as the stock name, `东财行业分类二级` plus high-relevance concepts as `细分方向`, `总市值` as market cap, and `概念`/same-day catalyst research as `驱动因素`.
- Group the new-high pool by `东财行业分类二级` and high-frequency concepts to write `板块分布`; do not just list stocks.
- The core table should prioritize large market-cap, high-turnover, sector-center, or repeated-new-high names. Typical roles include `趋势核心`, `中军`, `弹性核心`, and `新高核心`.
- Feed the new-high concentration into main-line judgment and stock selection. A direction with multiple core new highs after prior-day divergence receives stronger main-line evidence; a direction with only isolated small-cap new highs remains rotational.
- If `MX_StockPick` returns no rows, malformed rows, wrong-date rows, or unrelated fields, explicitly write that the `MX_StockPick`口径 was unstable, then use the Tonghuashun fallback below.

## Writing Rules

- Start with a blunt one-paragraph conclusion before tables.
- Use tables for data, but do not let tables replace judgment.
- If data is missing from 妙想, try 东方财富公开接口 fallback; if that is insufficient, use public web review sources. If all sources fail or disagree materially, say the metric was not stable/returned and use a qualitative summary instead of inventing a number.
- Keep each daily review around 900-1500 Chinese characters unless the user asks for a long version.
- When creating files, save under the current workspace unless the user specifies another folder.

## New-High Fallback: Tonghuashun Data Center

When writing `创新高历史个股分析`, use `MX_StockPick` first. If `MX_StockPick` returns empty, malformed, wrong-date, or only unrelated fields for historical-new-high data, query Tonghuashun Data Center before falling back to general news/review articles.

Preferred page:

- `https://data.10jqka.com.cn/rank/cxg/` - 同花顺数据中心，技术选股，创新高.

Usage rules:

- Use the `历史新高` view/category when available; do not confuse it with monthly, half-year, or yearly highs.
- Extract at minimum: stock name, stock code, close/latest price if shown,涨跌幅, 所属行业/概念 when shown, and the listed high type/date.
- Summarize the historical-new-high pool by direction: count names by industry/theme, identify 3-8 representative core/high-turnover stocks, and then write the main-line judgment from that distribution.
- If Tonghuashun blocks scripted access, requires browser interaction, or does not expose historical rows for the target date, state `同花顺数据中心未稳定返回目标日期历史新高明细`; then use public review sources only as a lower-confidence supplement.
- Label this source in the review as `同花顺数据中心创新高口径`.
- Do not invent market cap or direction. If Tonghuashun does not provide market cap, use `同花顺未披露`; if direction requires inference from industry/concept, write `按行业/概念归类`.
