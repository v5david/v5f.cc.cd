# v5f.cc.cd — V5 Medical 询盘表单站

> 对客询盘表单站。表单数据 → Cloudflare Worker → ERP 自动建 Lead → 邮件通知 NO 谈判。

## 架构

```
GitHub (v5david/v5f.cc.cd)
  → Cloudflare Pages（push 自动部署；构建命令留空；输出目录 /）
  → 自定义域名 v5f.cc.cd
  → Cloudflare Worker `v5f-leads`（路由 v5f.cc.cd/api/*，代码见 worker.js）
      → ERP（Frappe）：自动创建 Lead
      → 邮件：新询盘通知 → NO（Leo）
      → R2：附件存储（复用主站私有桶）
```

## 文件说明

| 文件 | 说明 |
|---|---|
| `index.html` | 表单落地页（Tailwind CDN）。字段：company*/contact*/email*/phone/category*/target_market/annual_volume/description |
| `thank-you.html` | 提交成功页（`?ref=` 显示 reference number） |
| `js/lead-forms.js` | 表单提交/校验逻辑（复用主站，零改动） |
| `js/config.js` | 站点配置（DOMAIN 已换 v5f.cc.cd） |
| `worker.js` | **后端源码**：复用主站 worker.js，差异仅 3 处（见下） |

### worker.js 相对主站的差异

1. `QUOTE_FIELDS` 白名单新增 `target_market`、`annual_volume`（CEO 拍板加的 B2B 字段）
2. `FIELD_LABELS` 新增 `target_market: "Target market"`（`annual_volume` 的 label 主站已有）
3. 部署时环境变量 `ALLOWED_ORIGINS=https://v5f.cc.cd`（否则 Origin 校验拦掉）

### Worker 部署变量（与主站对齐）

- `ALLOWED_ORIGINS` = `https://v5f.cc.cd`
- `ERP_URL` / `ERP_API_KEY` / `ERP_API_SECRET`（复用主站低权限 ERP 用户）
- `SALES_EMAIL` = **NO（Leo）的邮箱**（CEO 拍板：直接发 NO）
- `EMAIL_FROM` / `SEND_EMAIL`（Email binding）、R2 绑定（复用主站私有桶）
- ERP 侧需配 Assignment Rule：v5f 来源 Lead 自动指派 NO（ERPNext 后台配置，非代码）

## 待 CEO/运维

- [ ] push 代码到 v5david/v5f.cc.cd（等 push 权限）
- [ ] CF Pages 建项目 + 绑自定义域名 v5f.cc.cd
- [ ] 部署 Worker `v5f-leads`，配路由 + Secrets
- [ ] ERPNext 配 Lead 自动指派规则 → NO
- [ ] 端到端测试：提交 → ERP Lead → NO 收到邮件
