# 本仓自治规则（内容层：自停 · 自愈 · 英文泄漏自查）

TOPICS.md 是本仓选题的**唯一真相源**。以下三条每次运行都适用；trigger 的启用/停用由外部 `verify-routine-caps` 守卫按 N_pub vs N_top 统一管，**你不要去改 trigger**。

## 1. 自停（写完 TOPICS 就不再生成）
运行开头先算：
- `N_pub` = 已发布页最高编号：`ls *-day*.html`（或本仓命名 week/issue/book/topic）去 `.en.html`、去前导零取最大。
- `N_top` = TOPICS.md 最大编号：`grep '^[-*#]* *(Day|Week|Issue|Book|Topic) *[0-9]+'` 取最大。

**若 N_pub ≥ N_top（TOPICS 已全部写完）**：本次**不生成任何页面、不修改 TOPICS.md、绝不自己发明新主题**；直接结束（至多发一条『本站已写完 TOPICS 路线图，补充新主题后自动续写』的通知，且仅在你判断这是刚写完的当次时发，不要每次都发）。**这条优先级高于本仓 routine prompt 里任何"超表就自造新主题"的旧措辞。** 新主题只由 BigCat 手动或 deep-research 反哺加进 TOPICS.md；加了之后守卫会自动重开、routine 自然续写。

## 2. 自愈（保持 TOPICS ⊇ 已发布）
**只在越界时动手**：若本期 Day/编号按 TOPICS 计划写、TOPICS 里本就有这行 → 不动。只有当你发布的编号是 **TOPICS.md 里没有的**（用户手动指定主题、或 deep-research 反哺内容，当时漏记进 TOPICS）时：
1. 用本期页面标题/副标题，在 TOPICS.md **末尾 append** 一行 `- Day N: <主题> — <要点>`（前缀 Day/Week/Issue/Book/Topic 随本仓约定，先 `grep` 看已有行照抄）。
2. 与本期内容改动**一并 commit / push 到 main**。
3. **只 append，绝不修改已有行。** 否则越界页占了 `dayN`、TOPICS 缺行，日后往 TOPICS 补的 Day N 新主题会撞车永远写不出。

## 3. 英文页中文泄漏自查
生成 `*.en.html` 后、publish 前，对它 grep 这些指纹（命中=模板槽把中文漏进了英文页）：
`class="en">[一-鿿]` / `class="name-en">[一-鿿]` / `class="name-zh">[一-鿿]` / `class="cn">[一-鿿]` / `Reflections — [一-鿿]` / `class="lang-tag">ZH`
命中就修掉（删中文节点或译成英文）再 publish。**正常情况不算泄漏**：经文/古典原典+英译、孙子兵法原文+英译、term(中文) 括注、代码/分词演示、语言切换标签『中文』、主题本身是中文的页。

## 改已发布页面：只新增独立卡，别动已有卡（省 TTS 重烤）

`bake-tts.py` 按 **h2/h3/h4 与 `class` 含 `card` 的 div** 切段，对每段的
**未归一化原文**取 sha1 当 `data-tts`。所以在一张已有卡里改一个标点，
整张卡的音频就要重烤；跨卡改一句话，牵连的每一段都重烤。

要给已发布的页面补内容（导言、术语交代、背景铺垫等），**一律新开一个
`class` 含 `card` 的独立块**（如 `<div class="sutra-card primer">`），
放在已有卡之前或之后，**已有卡的正文一个字都不要动**。这样只新增一段音频，
原有段全部命中缓存。

推之前自查（不必真的调 Azure）：用 `collect_groups` + `hash_text` 跑一遍
改动前后两版，确认除新增段外每个 hash 都不变；顺手把算出的 `data-tts`
预先写进新块，CI 连属性修正的 commit 都省了。

真要改已有卡的正文（比如内容写错了），那就改——省重烤不是不许改正文的理由，
只是别为了"顺手润色"去动它。

## 新页面必带共享脚本（免触发 inject-comments 机器人提交）

生成任何 `*.html`（含 `.en.html`）时，在 `</body>` 前直接写入这 4 行，勿遗漏：

```html
<script src="https://hub.cissychen.com/comments.js" defer></script>
<script src="https://hub.cissychen.com/search.js" defer></script>
<script src="https://hub.cissychen.com/index-button.js" defer></script>
<script src="https://hub.cissychen.com/i18n-tts.js" defer></script>
```

这样 CI 的 inject-comments 不会再对新页面追加自动提交。

## 配色：深色由 CI 兜底，不用你操心

本仓是睡前读物，全站深色。**生成页面时照常写你觉得合适的浅色配色**——
push 之后 `darken.py` 会把内联 `<style>`、`style=""` 和内联 SVG 的 `fill=`/`stroke=`
一起按亮度推到深色，幂等，已经转过的页面会跳过。

不要为了"配合深色"自己手写死一套黑：那会让每篇文章长得一模一样。
darken.py 保留色相和彩度，原设计的性格还在，只换明暗。

**别在 `<style>` 顶部手写 `/* bigcat-dark v1 */`。** 那是"已转换"的标记，
写上去这一页就永远跳过转换、永远停在浅色。

`darken.py` 顶部的 `TINT_HUE` 是**本站的身份色相**（本仓 32°，暖褐）。
四个睡前读物仓的原始底色都是近白的米色，几乎不带彩度，照原色相推暗会全部
收敛成同一种暖黑、四个站长得一模一样；所以「接近中性」的面改用这个色相
重新上色，真正有颜色的元素保持自己的颜色。改它等于改本站的整体调性，
不要顺手动。

## 新页面必带「十条要点」（2026-10-17 起）

本站页面结构：读前导言（可无）→ 四张 `sutra-card`（每张含【经文选段】【中文详解】【跨学科联想】【生活实践】【English Summary】【每日一修】）→ 【深入思考】卡 → 返回首页。
**2026-10-17 起**，在第四张经文卡之后、【深入思考】卡之前，新增一张独立的十条要点卡（前后照样用 `<div class="divider">☸ ☸ ☸</div>` 隔开）。

- **不加文末「越界」框**：本站每张经文卡已有【跨学科联想】，再加一个越界框就是重复。
- **十条要点 `<ol class="takeaways">`**（2026-10-17 起，**必写，正好 10 条**）：读者拿它直接记笔记。每条一句完整的话，约 20–45 字，**单独抄出去也能看懂**：写出经名、宗派、人物和名相（如「如来藏」「阿赖耶识」），不用「此经」「这一派」「它」这类要靠上下文的代词。覆盖四部经最值得记的义理、修法和判别，可含【跨学科联想】里的一两条；顺序跟正文走（大致每部经两三条）。经名、祖师名不必每条都当主语开头：全组里同一个名字出现两三次就够，其余各条直接写义理、机制或事实（写「如来藏是接引畏惧无我者的方便，实义仍是空」，不写「《楞伽经》认为如来藏……」）。**只总结正文已经写过的东西**，不加新信息，不放经文引语。英文页标题 `Ten takeaways`，条目为对应英译。
  ```html
  <div class="sutra-card tk">
  <h3>【十条要点】</h3>
  <ol class="takeaways">
  <li>…</li>
  </ol>
  </div>
  ```
  CSS（各页 `<style>` 是从上一篇照抄的，旧页里没有这段；**第一次写时把它追加到 `<style>` 末尾**，之后照抄即可）。照常写浅色，CI 的 `darken.py` 会统一推深：
  ```css
  .tk h3{font-size:1.1rem;color:#5c4a2a;letter-spacing:2px;margin-bottom:14px}
  .takeaways{counter-reset:tk;list-style:none;margin:0;padding:0;border:1px solid #e3d5b8;border-radius:10px;background:#fbf6ea}
  .takeaways li{counter-increment:tk;position:relative;margin:0;padding:11px 18px 11px 52px;border-top:1px dashed #e3d5b8;font-size:0.95rem;line-height:1.8;color:#3d3429}
  .takeaways li:first-child{border-top:none}
  .takeaways li::before{content:counter(tk,decimal-leading-zero);position:absolute;left:18px;top:12px;font-family:Georgia,serif;font-size:0.82rem;color:#b8956a;font-weight:700}
  ```
  篇幅：十条要点约 350 字，`.maxchars` 已为它从 3750 上调到 4100（2026-10-17）；其余各节的篇幅不变。
  `publish.sh` 只对**从未提交过的新页面**（含 `.en.html`）检查正好 10 个 `<li>`；已发布页面不补、不改。
