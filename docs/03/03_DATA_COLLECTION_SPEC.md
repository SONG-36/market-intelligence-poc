# P1.2-A 美国婴幼儿看护市场数据采集任务书 V0.1

- **项目**：`market-intelligence-poc`
- **案例**：`cases/us-baby-monitor`
- **日期**：2026-10-28
- **状态**：`SCOPE_DRAFT / SEEDS_VERIFIED / ACCESS_AND_ASIN_PENDING`
- **负责人**：业务研究人员批准对象与样本；数据工程师验证获取方式与质量；平台/法务合规负责人批准具体用途与存储。
- **版本定位**：可执行的*对象发现与访问验证任务书*，**不是已获许可的大规模爬取指令**。

## 0. 目的与严格边界

我们要为美国 0–3 岁婴幼儿家庭看护市场的商业进入判断收集证据，支持“需求是否真实、交易规模与竞争、用户购买动机、价格与功能价值、利润及进入风险”六个研究问题，**不预先决定产品形态、Wi-Fi 方案、独立屏幕、AI 功能或自研路线**。萤石 C6c 仅是中国市场的功能启发性参考，不作为美国热销商品或拟研发规格。

P1.2-A 的验收不是“抓回 N 条评论”，而是：

1. 明确数据对象、平台、时间范围、选样方式、字段、任务数量与研究问题关系；
2. 给出真实存在的官方产品页 / 公开帖子 / 视频 ID / 应用页面作为**种子**；
3. 将 Amazon US 真实榜单和搜索产生的商品形成正式 ASIN 锁定清单；
4. 为每个来源核实授权、接口、商业用途、保存权限与可行性；
5. 只有通过门禁的对象才进入 P1.2-B 实际采集。

**业务上最容易犯的错误**：把预选品牌误称为热销品牌、把帖子热度当成市场需求、把估算销量当实销、把非美国讨论当成美国家庭代表性意见。以上均不得出现。

## 1. 研究问题与采集任务映射

| RQ | 商业问题 | 主要任务 | 期望证据 | 不能据此推断 |
|---|---|---|---|---|
| RQ01 | 美国是否有足够且持续的需求？ | Q01–Q12 | 美国搜索趋势、平台商品排名/成交线索、CDC 数据 | 不能把搜索指数或出生数视为购买量 |
| RQ02 | 哪些方案实际在销售？ | Q01–Q12 + product_targets.csv | 具体 SKU、ASIN、价格、产品形态、品牌、排名快照 | 预选品牌≠当前热销排名 |
| RQ03 | 谁为什么买/不买/换机？ | C01–C10 + Q13–Q18 | 帖子/回复关系、购买阶段、使用情境、正反体验 | 论坛不是美国消费者代表性抽样 |
| RQ04 | 用户为什么付费？ | 商品对照 + C01–C10 + A01–A07 | 购买选择、功能及订阅争议、价位对照、负面意见 | 评价文字≠真实支付意愿 |
| RQ05 | 新进入者是否可能盈利？ | 商品价格 + Amazon POE + 内部数据 | 价格/费用/退货、供应链、云服务、获客成本 | 零售价减出厂价≠利润 |
| RQ06 | 合规、安全、售后是否可行？ | S01 + 官方监管与内部核查 | FCC/FTC/CPSC/FDA 适用性、召回、固件/云服务责任 | 中国可售≠美国可售 |

文件索引：

- `cases/us-baby-monitor/product_targets.csv`：**候选对照型号**（不是已验证的畅销榜）；
- `cases/us-baby-monitor/content_targets.csv`：**已找到的帖子、视频、美国区 App 页面及专题证据**；
- `cases/us-baby-monitor/discovery_queries.csv`：**商店、榜单、搜索和额外内容发现任务**。

## 2. 商品与竞品抽样：先发现，后锁定

### 2.1 固定发现任务

1. Amazon US：从 `https://www.amazon.com/Best-Sellers/zgbs` 进入 Baby 类目，人工确认最恰当的 *video baby monitors* 子类目与真实 URL；**记录类目路径和快照时间**，截取当次可见前 30 名。网页自动访问可能遇到反机器人限制，**不能绕过登录/CAPTCHA/限流**。
2. Amazon US：分别搜索 `baby monitor`、`video baby monitor`、`wifi baby monitor`、`non wifi baby monitor`、`baby camera`、`indoor camera baby`；各记录可见前 20 个自然搜索结果，**赞助结果另列，不进入自然排名**。保存搜索语句、排序方式、变体、时间、国家和登录状态。
3. Amazon Seller Central：先核实 Professional Seller 是否具备 Product Opportunity Explorer 访问权限；若可用，记录相关 niche 的搜索/购买/价格及退货线索，并单独标注口径。**卖家后台数据不得假定可以自由对外再分发**。
4. Walmart US 与 Target：同类搜索作为跨渠道检查；同款商品按 `品牌+具体型号+包装配置` 去重，**不同摄像头数量/屏幕组合属于不同销售变体**。
5. 形成候选池后，由业务研究人员选择 **10–12 个代表性 SKU**：至少覆盖通用室内摄像头、手机端为主的专用 Baby Monitor、Hybrid、非 Wi-Fi 屏幕型；其中必须包括价格/功能/销量线索的强弱对照。若某分组没有足够在售商品，说明缺口，不凑数。

### 2.2 产品记录内容

每个正式锁定的美国零售 SKU 必须记录：

`platform, product_url, asin_or_retailer_id, parent_asin(if applicable), brand, exact_model, bundle_or_variant, product_type, list_price_usd, selling_price_usd, coupon_or_discount, availability, rating, rating_count, listing_rank, rank_category, rank_snapshot_at, seller_or_sold_by, observed_at, evidence_url, source_method, legal_use_status`。

产品功能另建手工审阅字段：`display_type, connectivity, night_vision, audio_monitoring, background_audio, cry_alert, motion_detection, local_storage, cloud_subscription, multi_camera, multi_caregiver, offline_behavior, return_or_warranty_claim`。只记录有证据支持的功能；未核实用 `UNKNOWN`，不得补全“常识”参数。

**ASIN/榜单名次不是本次网页核查得到的事实，因此 CSV 中一律留空。** 工程师须依据真实美国商品页填写，不能推断、拼装 URL 或拿旧媒体测评作为排名。

## 3. 消费者声音采集（论坛、购物评价、视频、App）

### 3.1 Reddit：只确认种子，未经授权不得商业批量采集

- 种子社区：`r/NewParents`、`r/beyondthebump`、`r/predaddit`、`r/daddit`、`r/Buyingforbaby`；帖子与具体链接见 `content_targets.csv`。
- 重点记录购买前选择、功能替代、价格、持续监听/屏幕使用、联网故障、订阅、安全顾虑、满意体验、长期故障与换机原因。
- **收集单位**：帖子 `post_id` + 顶层评论 `comment_id` + 回复 `parent_id` + 层级 + 发布时间 + 允许存储的摘录/摘要（如果被授权）；**禁止把回复拆成脱离上下文的独立消费者评论**。
- 地理：未明确位于美国的用户统一记录 `GEO_UNVERIFIED`，不得强行设成 US。
- Reddit 2026-10-05 更新的政策规定 API 访问需审批，商业使用须明确书面许可，并对商业挖掘、抓取及 AI 训练用途设限制。当前任务仅允许**记录已公开识别的帖子标题/链接与研究问题**；批量正文及回复的采集、持久存储、模型分析须等待授权/有效商业数据许可。禁止绕过 API 限制，禁止用代理/多账号规避控制。
- 负面案例（断线、黑客传闻）需保留对应的满意用户/非 Wi-Fi 缺陷案例作为反证，避免选择性采样。

### 3.2 Amazon/Walmart/Target 买家评价

- **商品先锁定 ASIN / retailer ID，再确定评价任务**。不要从全平台抓一个无法定位产品的“baby monitor 评论池”。
- 首轮探索目标：每个有效商品至多抽取约 10–20 条**可合法获取**的评论，按星级区间和新旧日期分层，优先带实际使用过程、退货/换机线索的评论；**不得只抓最低分/最有帮助**。若评论受限，可使用 Amazon POE 的 Customer Review Insights 聚合数据、许可数据供应商或受许可人工记录。
- 记录 `review_id, parent_product_id, date, verified_purchase_if_displayed, star, helpfulness_if_displayed, language, review_variant, review_context, url, permitted_summary, capture_method, access_scope`；任何整段复制须按授权处理。
- 评价数不同、日期不同、老版本/新版本硬件不同，应分别报告，不能混成“故障率”。

### 3.3 YouTube 视频与评论

- `content_targets.csv` 已指定多条真实 `watch?v=` 视频链接；优先测试、演示、长期体验，品牌演示与带货视频做 `PROMOTIONAL` 标记。
- 有授权/API 项目时：`commentThreads.list(videoId, part=snippet,replies, maxResults<=100)` 取得顶层线程；**它不保证包含全部回复**，如业务确实需要则用 `comments.list(parentId)` 补齐；保存 `video_id, thread_id, comment_id, parent_id, posted_at, likes, text_if_licensed, totalReplyCount, extracted_at`。检查 API 服务政策、配额、数据保留和删除同步要求。
- 视频来源国家与视频讨论对象为美国市场是不同属性；加拿大来源应做 `NON_US_CREATOR`，不能作为美国本土消费者证据。

### 3.4 美国区 App 评价

- 以 `content_targets.csv` 中可核实的美国区 Apple App Store/Google Play 页面为种子，特别关注 `Nanit`、`eufy Baby`、`Tapo`。
- 收集软件版本、评论日期、星级、问题类型（连线、登录、推送、后台音频、订阅、共享、隐私、固件等）以及**评论是否确实针对婴儿监控功能**。多品类 App 如 Tapo/eufy Baby 需要剔除无关品类。
- Apple App Store Connect Customer Reviews API 与 Google Play Developer API 评论管理接口主要服务于**本开发者拥有的 App**，**不得误当成竞品全量评论采集权限**。未经另行授权，只做人工公开页面调研或使用合规商业数据供应商。

### 3.5 安全与监管

- 必须包含美国 CPSC **Babysense Max View VBM55 / 显示器型号 VBM55RX** 2026-02-26 召回作为 `SAFETY` 案例；这是召回事实，不可推广为所有 Babysense 型号不安全。
- FCC 设备许可、FTC 联网隐私与产品安全、CPSC 线缆和电气风险、FDA 生命体征医疗宣称只在对应产品方案确定后做适用性分析；现阶段先建证据清单与风险项。

## 4. Discovery 查询与采样约束

见 `discovery_queries.csv`，包括：

- Amazon 类目/多关键词搜索/卖家工具；Walmart、Target 跨渠道；
- Google Trends：美国、同样的时间窗，分别观察过去 24 个月和 5 年；**网站指数 0–100 并非绝对搜索量**；搜索词与 Topic 不混用。
- Reddit 社区公开网页**人工发现**关键词；未拿到商业授权前不实施自动抓取；
- YouTube 英文测评、价格、Wi-Fi vs 非 Wi-Fi、长期体验的发现规则；
- CPSC 相关召回检索。

所有“前 N”仅为**要做的采样任务规模**，不是已经收集到的数量，也不是市场排名。

## 5. 建议首轮样本与配比（探索性，非统计推断）

| 来源 | 建议初次范围 | 约束 |
|---|---|---|
| 零售发现 | Amazon 类目最多前 30 + 每查询最多 20 自然结果 + Walmart/Target 对照 | 先形成候选池，保留采样快照 |
| 锁定产品 | 10–12 个真实 SKU | 四组替代方案均须覆盖，不将 C6c 参考样品算入 |
| 商品评价 | 获准后每 SKU 10–20 条 | 星级和时间分层、变体正确，受权限影响可减少 |
| Reddit | 10 个已核实主题帖作为阅读种子 | 批量文本/回复须获商业许可；正反证据兼顾 |
| YouTube | 5 个已核实视频起步，后补至 6–10 | 视频演示与独立测评区分、保持回复关系 |
| 应用评论 | 3 个品牌 x 2 系统优先，最多每 App 15–20 条 | 版权/平台授权先行、筛除无关品类 |
| 安全事件 | 至少 1 个官方召回案例 | 只归因于具体召回型号 |

样本少但具有来源质量和反例，优先于大量无授权、无时效、无对应产品的评论。

## 6. 工程数据契约与验收标准

### 6.1 通用事件/证据字段

```text
record_id, task_id, research_question_ids, source_platform,
object_type, source_id, source_url, parent_source_id,
product_key, exact_model_or_variant, published_at, collected_at,
region_scope, us_relevance_status, source_type,
original_claim_or_permitted_excerpt, analyst_summary,
metric_name, metric_value, metric_unit, metric_definition,
collection_method, permission_status, retention_rule,
quality_flags, extraction_status, error_reason
```

不适用项可以为空（但必须有原因）；采集失败不得返回看似正常的空记录。每条记录必须可回到明确的公开页面或授权数据源。所有日期保留原时区，标准化时间另外存 UTC，不覆盖原日期。

### 6.2 记录类型的必填条件

| 类型 | 必须的额外键 | 不能缺少的结构 |
|---|---|---|
| `RETAIL_RANK_SNAPSHOT` | category_url, rank, captured_at, sort | 同一排序/时刻/地区口径 |
| `PRODUCT_LISTING` | retailer, retailer_product_id, model, bundle | SKU 与变体对应 |
| `PRODUCT_REVIEW` | review_id, retailer_product_id, rating, posted_at | 评论与具体 SKU 关联 |
| `FORUM_POST` | post_id, subreddit, created_at | 社区、帖子链接 |
| `FORUM_COMMENT` | comment_id, post_id, parent_comment_id, depth | 回复树关联 |
| `YOUTUBE_VIDEO` | video_id, channel, published_at | 视频地区与内容商业属性 |
| `YOUTUBE_COMMENT` | comment_id, video_id, parent_comment_id | 线程回复层级 |
| `APP_REVIEW` | store, app_id, review_date, rating | 软件版本/品类相关性 |
| `TRENDS_SERIES` | country, geo, term_or_topic, period, index | 相同归一化查询口径 |
| `SAFETY_NOTICE` | authority, recall_id_or_url, model, notice_date | 具体型号与风险范围 |

### 6.3 最低质量门槛（仅用于可获取的实际样本）

1. **已入库记录来源标识 + 来源 URL 完整率 100%**；不完整的单独进入异常队列。
2. **已入库评论的 parent_id / product_id 关联正确率 100%**，抽检不通过则拒收对应批次。
3. 仅在 `permission_status=APPROVED_FOR_THIS_USE` 且符合保存条款时存储正文；其余仅存批准范围内的 URL/元数据/研究摘要，或阻断任务。
4. 商业指标必须标记 `OBSERVED`, `OFFICIAL_AGGREGATE`, `THIRD_PARTY_ESTIMATE` 或 `UNKNOWN`；估算不能冒充实销。
5. 排名/价格必须保留 `snapshot_at` 和具体站点；不同日期的价格不得直接比较成“趋势”。
6. `country=US` 不等于 Reddit 发帖者就是美国消费者；用户地域无法核实时用 `UNVERIFIED`。
7. 每次采集交付 `success/partial/blocked`、成功数、失败数、页面/字段不可用原因、授权说明、覆盖率和可复核链接。
8. 去重遵循 `(platform, object_type, native_id, variant)`；同内容跨平台转发须标记可能关联，但不得冒充多个独立买家。
9. 不获取儿童影像、家庭地址、用户名映射、健康信息等无关的私人/敏感资料；不做人群敏感属性推断。

## 7. 权限矩阵与硬门禁

| 来源 | 当前状态 | P1.2-B 允许的下一步 | 必须阻断的动作 |
|---|---|---|---|
| Amazon 商品榜单/买家评价 | `ACCESS_ROUTE_TO_VERIFY` | 在合规渠道确认后台、授权 API/供应商或人工记录是否可用 | 未许可批量抓取、绕过阻拦或大量复制评论 |
| Amazon Product Opportunity Explorer | `SELLER_ACCESS_PENDING` | 业务负责人核实专业卖家账户、字段与数据使用范围 | 未授权访问、擅自对外共享后台数据 |
| Walmart/Target | `ACCESS_ROUTE_TO_VERIFY` | 核实允许的获取方式与商业数据许可 | 未经许可持续高频抓取 |
| Reddit | `COMMERCIAL_APPROVAL_REQUIRED` | 留存帖子种子 URL、咨询许可/合规提供商 | 商用批量下载帖子与回复、绕过控制 |
| YouTube | `API_PROJECT_AND_POLICY_CHECK` | 使用正式 API 做小量读取和保存政策核查 | 绕过 API/删帖保留要求/权限限制 |
| 竞争 App 评价 | `THIRD_PARTY_REVIEW_ACCESS_UNVERIFIED` | 人工公开调研或取得有许可的数据 | 用自有开发者接口试图导出别家 App 评论 |
| Google Trends | `PUBLIC_UI_RESEARCH` | 按统一查询参数记录/导出许可数据 | 当成搜索量/销量或不注明归一化口径 |
| CPSC | `PUBLIC_OFFICIAL_SOURCE` | 官方公开案例与数据，记录日期及型号 | 对不相关型号或品牌泛化风险 |

`APPROVED_FOR_THIS_USE` 是**内部需要实际核实后才可填写的状态**，此任务书没有替任何平台授予许可。

## 8. 交付分工与顺序

### 8.1 业务研究人员（第一责任人）

- 检查 `discovery_queries.csv` 是否覆盖 6 个研究问题；审核产品是否具备代表性、是否遗漏低价/非 Wi-Fi/通用摄像头替代方案。
- 用美国零售快照补完候选产品的 ASIN、变体、在售状态；**签署锁定版本**。不允许让工程师随意“抓前 10 个看起来热门的”。
- 选定正反证据的抽样比例、可接受的非美国/未知地域内容、报告的风险表达方式。

### 8.2 合规/平台授权负责人（强制门禁）

- 为每个平台填写许可依据、商业用途、保存正文或元数据的范围、必要删除/过期处理方式。
- 对涉及个人内容、消费者隐私、未成年人场景的权限和最小化数据范围进行审核。

### 8.3 数据工程师（权限通过后）

- 先做 1 个榜单/1 个商品/1 个官方视频/1 个 App 页面的小样本字段可用性测试；生成可复核的字段映射和失败原因。
- 再按照已批准的任务清单采样；所有记录附任务 ID、源链接、快照时间、实际读取方法及偏差说明。
- 不自行扩大范围、抓取私有内容、突破访问限制或自行替换目标。

## 9. 状态与阶段门禁（截至 2026-10-28）

| Gate | 条件 | 当前状态 |
|---|---|---|
| G0 研究范围一致 | 明确美国 0–3 岁婴幼儿看护商业机会；产品定义后置 | `PASS` |
| G1 种子对象存在 | 预选品牌官方页面、部分帖子/视频/App 页面已查实 | `PARTIAL_PASS` |
| G2 美国零售目标锁定 | 真实 Amazon 类目快照、在售 SKU、ASIN、变体和名次 | `PENDING` |
| G3 平台获取授权 | 商业用途、接口与保存规则逐平台确认 | `PENDING`；Reddit 大规模采集 `BLOCKED_UNTIL_APPROVED` |
| G4 实际字段测试 | 真实样本和原始返回/保存能力证据 | `NOT_STARTED` |
| G5 正式批量采集 | 只有 G2–G4 完成后放行 | `NOT_AUTHORIZED` |

**因此本交付标为 P1.2-A 任务书草案已生成，而非 P1.2 数据采集完成，也非美国市场商业可行性已经得到验证。**

## 10. 核查的官方/专业原始来源

- Amazon Best Sellers 指南：https://sell.amazon.com/blog/product-ideas
- Amazon 商品商机探测器：https://sell.amazon.com/tools/product-opportunity-explorer
- Seller Central 工具访问说明：https://sellercentral.amazon.com/help/hub/reference/external/GNJ4YRTXWLMBY38U?locale=enUS
- Google Trends 解释：https://support.google.com/trends/answer/4365533?hl=en
- Reddit 政策（2026-10-05）：https://support.reddithelp.com/hc/en-us/articles/42728983564564-Responsible-Builder-Policy
- Reddit 商业接入：https://support.reddithelp.com/hc/en-us/articles/14945211791892-Developer-Platform-Accessing-Reddit-Data
- YouTube 评论接口：https://developers.google.com/youtube/data/reference/rest/v3/commentThreads/list
- YouTube 回复结构：https://developers.google.com/youtube/data/reference/rest/v3/commentThreads
- Apple 自有 App 评论 API：https://developer.apple.com/documentation/appstoreconnectapi/customer-reviews
- Google Play 开发者评论 API：https://developers.google.com/android-publisher/api-ref/rest/v3/reviews/list
- TikTok Research API 资格：https://developers.tiktok.com/docs/en/research-api-faq
- 美国 CPSC Babysense Max View 召回：https://www.cpsc.gov/Recalls/2026/Babysense-Max-View-Baby-Monitors-Recalled-Due-to-Fire-Hazard-Manufactured-by-Hisense
- Consumer Reports 2026 婴儿监控器评测：https://www.consumerreports.org/babies-kids/baby-monitors/best-baby-monitors-a8598638140
- Babylist 2026 婴儿监控器研究：https://www.babylist.com/hello-baby/best-baby-monitor

> 本清单是研究与工程交付标准，不构成法律意见，也不声称已得到任何平台的商业内容使用许可。
