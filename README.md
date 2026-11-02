# NekoBox 远程分流规则（rule-set 版）

把原来写死在手机自定义配置里的 85 条域名分流名单，拆成 6 个按出站方向分组的
sing-box rule-set 文件，放到网上托管；手机只留一份瘦配置，每天自动拉一次最新名单。
以后改分流只改托管端的文件，手机不用再动。

## 文件清单

| 文件 | 内容 |
|---|---|
| `nekobox-remote-profile.json` | 手机端瘦配置，导入 NekoBox 用（导入前把里面 6 处 `HOST` 换成实际托管域名） |
| `rules/ruleset-us-static.json` | →「美静态」：RedPocket、IMS（3gppnetwork），共 3 个域名后缀 |
| `rules/ruleset-us.json` | →「US 22 NTT x1.0」：Meta、X、Reddit、Discord、Amazon、Spotify、Disney、Zoom、Dropbox、Notion、Slack、Google、OpenAI 及其他 AI，共 52 个 |
| `rules/ruleset-tw.json` | →「TW 21 Hinet x1.0」：Binance，共 3 个 |
| `rules/ruleset-hk.json` | →「HK 14 家宽 x1.0」：ClubSIM、YouTube、GitHub、Cloudflare、Telegram、抖音，共 17 个 |
| `rules/ruleset-jp.json` | →「JP 22 GMO x1.0」：WhatsApp，共 2 个 |
| `rules/ruleset-direct.json` | → 直连：Apple、Steam、.cn 域名，共 8 个 |

原 v2 配置里还有两条本地规则原样保留在瘦配置里，没有远程化：
`geosite cn → 直连`、`geoip cn → 直连`。它们用的是名单数据库而不是域名清单，
拆不成这种小文件；将来若想全远程化，可换成官方发布的 geosite-cn / geoip-cn
远程 rule-set（另行处理，本次未动）。

## 托管要求

- 任意能放静态文件的 HTTPS 托管即可：自己的域名、GitHub、Cloudflare 都行，
  要求只有两个：链接能公开直达（或带固定路径），且支持 HTTPS。
- 把 `rules/` 下 6 个 JSON 原样放上去，保持文件名不变，路径形如
  `https://你的域名/rules/ruleset-us.json`。
- 名单文件是 JSON 源格式（format: source），不需要编译成 .srs；改名单直接改
  JSON 文本就行，手机下次拉取自动生效。

## 导入步骤（手机，只做一次）

1. 先完成托管，确认 6 个链接在浏览器里能打开。
2. 打开 `nekobox-remote-profile.json`，把 6 处 `https://HOST/` 换成你的实际
   托管地址前缀。
3. NekoBox → 设置 → 自定义配置：删掉 v2 那份，粘贴这份瘦配置，保存。
4. 节点名（美静态、US 22 NTT x1.0 等）与 v2 完全一致，不用改；链式代理仍在
   NekoBox 界面里配，这份配置不碰它。
5. 重启一次 NekoBox 让它首次拉取；之后每 24 小时自动更新一次名单
   （update_interval 1d），改完名单想立刻生效就重启 NekoBox。

## 与现在（v2）的差异

- **规则明细在 App 里不再逐条可见**：自定义配置里只剩 6 条「组规则」
  （一个规则集 → 一个节点），85 个域名都在托管端的 JSON 里，要看明细、改名单
  都去改那 6 个文件，不在手机上逐条翻。
- 改名单不用再碰手机：以前改一条要重贴整份配置，现在改托管端文件即可。
- 多了一个依赖：托管地址必须活着。首次拉取成功后 sing-box 会用已下载的名单
  继续工作，托管短暂挂掉不影响已拉到的规则，只是拉不到更新。
- 拉取走直连（download_detour: direct），避免「规则还没拉到就决定不了路由」
  的先有鸡还是先有蛋问题；名单文件本身也不含任何敏感信息。

## 校验结论（2026-11-02，sing-box 1.12.4 官方二进制实测）

- 6 个 rule-set 全部 `sing-box rule-set compile` 成功；反编译回读与源文件
  域名集合逐一比对一致（以 ruleset-us 的 52 条为样本全量核对）。
- 瘦配置展开成完整配置后 `sing-box check` 通过（6 条 rule_set 规则及远程
  rule-set 定义部分）。geosite/geoip 两条在官方原版 1.12 里会报错是因为原版
  已移除 geosite/geoip 数据库支持——这两条与 v2 里逐字未改，NekoBox 用的是
  自带 geo 数据库的内核，行为与现在完全相同，不是本次改动引入的问题。

## 关于按应用分流（package_name）

已查实 sing-box 官方文档：`package_name` 是 rule-set 所用的 headless rule 的
合法字段（官方文档明确列出，含义为匹配安卓包名），且实测能正常 compile 进
rule-set，所以将来加「按应用分流」的规则同样可以远程化。
说明：**当前这份名单里没有 package_name 规则**——v2 原文件里只有域名规则，
按应用的分流目前是在 NekoBox 界面侧设置的，不在这份配置内，本次未动它。
