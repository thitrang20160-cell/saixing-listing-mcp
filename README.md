# 塞星铺货执行 MCP · Saixing Listing MCP

**List products to Walmart US from your own AI agent — end to end.**
在你自己的 AI 助手里一句话把商品铺到沃尔玛美国站：采集 → 侵权排查 → AI 重构 → 修图 → 核验 → 上架 → 查 Feed 回执，全链路一次跑通。

Saixing Listing MCP 把塞星（SellerStar）的铺货流水线开放为 [Model Context Protocol](https://modelcontextprotocol.io) 工具，任何支持 MCP 的 Agent——Claude Code、Cursor、Codex、WorkBuddy 等——都能直接跑通一整条沃尔玛上架流程，不用再打开网页后台。合规排查由 SafeSellAI 引擎驱动。

> 本仓库是该 MCP 服务的**公开说明与接入仓库**：服务端为托管服务，源码不开源。

- **远程端点**：`https://bind.safesellai.com/saixing-mcp/mcp`（Streamable HTTP）
- **欢迎面板**：https://bind.safesellai.com/saixing-mcp/welcome —— 三项体验任务（选品上架 / 图片优化 / 店铺侵权全筛查），每完成一项领 1,000 Scoin 体验额度
- **获取 Key**：https://bind.safesellai.com/saixing-mcp/bind —— 用塞星账号登录后自助签发租户 Key
- **游客浏览**：无 Key 也可 `initialize` / `tools/list` 查看全部工具；实际调用需要 Key

## 工具（11 个）

| 工具 | 作用 | 权限 |
| --- | --- | --- |
| `saixing_welcome` | 入门引导：能力地图、三项体验任务与免费额度 | 只读 |
| `claim_trial_credit` | 完成体验任务后领取试用额度（每项 1,000 Scoin） | 只读 |
| `collect_products` | 采集商品链接（9 平台 + 独立站，单次 ≤500 条）入暂存 | 草稿 |
| `list_staging_items` | 查看暂存区待处理商品 | 只读 |
| `get_staging_item` | 查看单个暂存商品详情 | 只读 |
| `check_compliance` | 上架前查商标 / 专利 / 版权 / TRO 风险；可整店全筛查 | 草稿 |
| `rewrite_listing` | AI 重构标题、五点、描述（草稿） | 草稿 |
| `process_images` | 图文一致性诊断与修复 | 草稿 |
| `validate_listing` | 上架前核验：SOP 校验、UPC 池、上架时长预估 | 只读 |
| `publish_listing` | 提交上架到沃尔玛（默认 dry_run 预检；幂等） | 发布 |
| `get_feed_status` | 查沃尔玛 Feed 提交回执状态 | 只读 |

标准调用链：`collect_products → check_compliance →（rewrite_listing / process_images）→ validate_listing → publish_listing → get_feed_status`

## 快速接入

远程 HTTP（推荐）：

```json
{
  "mcpServers": {
    "saixing-listing": {
      "url": "https://bind.safesellai.com/saixing-mcp/mcp",
      "headers": { "Authorization": "Bearer <YOUR_SAIXING_MCP_KEY>" }
    }
  }
}
```

Claude Code 一行接入：

```bash
claude mcp add --transport http saixing-listing https://bind.safesellai.com/saixing-mcp/mcp \
  --header "Authorization: Bearer <YOUR_SAIXING_MCP_KEY>"
```

只支持 stdio 的客户端，可用通用远程桥（如 `mcp-remote`）把上面的 URL 与请求头转成本地 stdio。

接入后先调用 `saixing_welcome`：返回欢迎面板链接、三项体验任务进度与额度余额。

## 安全护栏

- 发布默认 `dry_run`（只预检不真发）；真发布须显式 `dry_run=false` + 幂等键，同键同参不重复执行
- 合规 fail-closed：没有 7 天内「能卖」结论的商品拒绝上架；合规引擎不可用时返回「未排查」并拒绝发布
- 自有品牌按上架闸同口径豁免；TRO 命中不豁免
- Key 分级（只读 / 草稿 / 发布）+ 店铺白名单；真上架限流每小时 10 次 / Key
- 店铺的沃尔玛凭据只存服务端，绝不下发客户端

## 费用

免费体验额度（欢迎任务每项 1,000 Scoin）+ 入门包 ¥199 / 6,000 Scoin，用完按次扣 Scoin，与网页端共用同一钱包。参考单价：合规排查 2 Scoin/件、整店筛查 100 Scoin/次、AI 重构 3 Scoin/件、图文诊断 2 Scoin/件起；采集与上架提交现价目 0。

## 相关

- SafeSellAI MCP（上架前侵权排查引擎）：https://safesellai.com/mcp-guide
- 跨境电商 MCP 导航「MCP 码头」：https://safesellai.com/mcp-nav/

## License

MIT — see [LICENSE](LICENSE). 托管服务由塞星 / SellerStar（中山心一出海科技有限公司）运营；商标与品牌归其所有。
