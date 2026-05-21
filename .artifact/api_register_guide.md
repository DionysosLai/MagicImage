# API 注册与申请指南

> 本文档为 MagicImage Agent 阶段 0 的 API key 准备清单。
> 周末按本文档逐一申请，把得到的 key 填到 `magicimage-agent/.env`，跑通各家 hello-world 即视为 t0-3 完成。
>
> **价格信息时效**：截至 2026-05，搜索引擎结果可能滞后，**正式申请前请到对应官方页面再确认一次**（每家最新报价链接已在表格里给出）。

---

## 0. 总览（先看哪几家、各家定位）

| 家 | 角色 | 国内/海外 | 是否需要 VPN | 信用卡 | 免费额度 | 阶段 0 是否必申 |
|----|------|----------|------------|-------|---------|----------------|
| **腾讯混元 3D** | 主力 3D 生成 | 国内 | ❌ 不需要 | ❌ 不需要（手机号即可） | 有免费资源包（首次开通） | ✅ 必申 |
| **Tripo AI** | 备选 3D 生成 | 海外 | ⚠️ 推荐 | ✅ 充值需要 | 注册送 600 credits（一次性） | ⭕ 强烈推荐 |
| **Meshy** | 兜底 3D 生成 | 海外 | ⚠️ 推荐 | ✅ 充值需要 | 200 credits/月（永久免费） | ⭕ 推荐 |
| **DeepSeek** | LLM 编排（必需） | 国内 | ❌ 不需要 | ❌ 不需要（手机号充值即可） | 注册有少量赠送，按量计费 | ✅ 必申 |

**最少必申**：混元 3D + DeepSeek（这两家国内直连、免实名/免信用卡，一晚上能搞定 demo）。
**完整方案**：四家全申（建议有海外信用卡的同学顺手把 Tripo / Meshy 也申了，做降级链路演示用）。

---

## 1. 腾讯混元 3D（主力）

### 1.1 概览

- **官方文档入口**：https://cloud.tencent.com/document/product/1804/120838
- **服务名称**：腾讯云 → AI 与机器学习 → 腾讯混元 3D
- **核心 API**：
  - `SubmitHunyuanTo3DJob`（提交任务，返回 JobId）
  - `QueryHunyuanTo3DJob`（轮询任务，得到 GLB 下载 URL）
- **接入域名**：`hunyuan.tencentcloudapi.com`
- **鉴权**：腾讯云通用 v3 签名（TC3-HMAC-SHA256），用 SecretId + SecretKey 计算

### 1.2 申请步骤

1. 打开 [cloud.tencent.com](https://cloud.tencent.com)，注册并登录（手机号即可，不需要实名 / 信用卡）
2. 进入 **控制台 → 访问管理 → API 密钥管理**：https://console.cloud.tencent.com/cam/capi
3. 新建一个子用户密钥（不要用主账号 root key），记下 `SecretId` 和 `SecretKey`
4. 进入 **腾讯混元 3D 控制台**：https://console.cloud.tencent.com/hunyuan/start
5. 第一次访问时点击「立即开通」，会自动发放免费资源包（一次性、有效期 1 年内）
6. （可选）实名认证后，免费额度更高、调用稳定性更好

### 1.3 免费额度（截至 2026-05）

- 首次开通会一次性赠送免费调用包，具体数额以控制台显示为准（按文档：v3 有"首次开通发放"机制）
- 用完后按量计费（见 https://cloud.tencent.com/document/product/1804/billing 的最新报价）

### 1.4 .env 字段

```ini
HUNYUAN_SECRET_ID=AKIDxxxxxxxxxxxxxxxxxxxx
HUNYUAN_SECRET_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
HUNYUAN_REGION=ap-guangzhou
```

### 1.5 hello-world 验证（占位伪代码，t0-3 阶段补真代码）

```python
# 调 SubmitHunyuanTo3DJob 提交一个 "a red wooden box" 任务，
# 30~120s 后用 QueryHunyuanTo3DJob 轮询，能拿到 .glb 下载 URL 即视为通过。
```

### 1.6 常见坑

- **签名算法**：v3 签名实现有点麻烦，建议直接用腾讯云官方 Python SDK：`pip install tencentcloud-sdk-python-hunyuan`
- **地域**：建议 `ap-guangzhou` 或 `ap-shanghai`，不要选海外节点
- **配额限制**：免费阶段并发受限，单账号同时跑多任务会 429，加重试就行

---

## 2. Tripo AI（备选）

### 2.1 概览

- **官网**：https://www.tripo3d.ai
- **API 文档**：https://platform.tripo3d.ai/docs
- **定价页**：https://www.tripo3d.ai/pricing
- **核心 API**：`POST https://api.tripo3d.ai/v2/openapi/task` 提交，`GET .../task/{task_id}` 查状态
- **鉴权**：Bearer Token（一个 string，不需要签名计算）

### 2.2 申请步骤

1. 打开 https://platform.tripo3d.ai 注册（Google / 邮箱登录）
2. 控制台 → API Keys → Create New Key
3. 复制 key（只显示一次，丢了重建）
4. 走预置充值才能调 API（有些地区可能要绑信用卡）

### 2.3 免费额度（截至 2026-05）

- **注册赠送**：约 600 credits（一次性）
- **每次图生 3D**：约消耗 20~40 credits（看分辨率、是否带骨骼）
- **够跑 15~30 次测试**，足够 MVP 联调
- 充值起步：$10 起，约 1000 credits

### 2.4 .env 字段

```ini
TRIPO_API_KEY=tsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 2.5 hello-world 验证

```python
# POST /v2/openapi/task with {"type":"text_to_model","prompt":"a red wooden box"}
# 拿 task_id 后 GET /v2/openapi/task/{task_id}，
# 状态变 success 后从 result.pbr_model.url 下载 .glb 即视为通过。
```

### 2.6 常见坑

- **网络**：国内直连可能慢，建议走代理或 cloudflare workers 中转
- **绑骨**：阶段 2 的角色管线里 Tripo 自带 auto-rigging，省去 AccuRig 这一步可作为备选
- **付费起步价**：API 模式部分功能必须付费账户才能用（具体看 pricing 页 API 那一行）

---

## 3. Meshy（兜底）

### 3.1 概览

- **官网**：https://www.meshy.ai
- **API 文档**：https://docs.meshy.ai/api/pricing
- **核心 API**：
  - 文生 3D：`POST https://api.meshy.ai/openapi/v2/text-to-3d`
  - 图生 3D：`POST https://api.meshy.ai/openapi/v1/image-to-3d`
- **鉴权**：Bearer Token

### 3.2 申请步骤

1. 打开 https://www.meshy.ai 注册（邮箱 / Google）
2. 进入 https://www.meshy.ai/api-keys 创建 API key
3. 注意：免费层（200 credits/月）默认就能调 API，**不强制付费**

### 3.3 免费额度（截至 2026-05）

- **永久免费层**：200 credits/月（每月初刷新）
- **图生 3D**：每次约 5 credits → 每月约 40 次
- **文生 3D**：每次约 10 credits → 每月约 20 次
- 付费 Pro：$20/月，1000 credits

### 3.4 .env 字段

```ini
MESHY_API_KEY=msy_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 3.5 hello-world 验证

```python
# POST /openapi/v2/text-to-3d with {"mode":"preview","prompt":"a red wooden box","art_style":"realistic"}
# 拿 result id → GET /openapi/v2/text-to-3d/{id}
# 状态 SUCCEEDED 后从 model_urls.glb 下载即视为通过。
```

### 3.6 常见坑

- **两阶段生成**：Meshy 有 preview / refine 两步，preview 便宜（5 credits），refine 贵（20 credits）。MVP 阶段用 preview 就够了
- **网络**：海外服务，国内直连不稳定，建议代理
- **绑骨能力**：Meshy 不支持自动绑骨，所以只用作静态道具兜底

---

## 4. DeepSeek（LLM 编排，必需）

### 4.1 概览

- **官网**：https://platform.deepseek.com
- **定价页**：https://api-docs.deepseek.com/quick_start/pricing/
- **核心 API**：OpenAI 兼容（直接复用 openai SDK，改 base_url 即可）
  - `https://api.deepseek.com/v1/chat/completions`
- **鉴权**：Bearer Token

### 4.2 申请步骤

1. 打开 https://platform.deepseek.com，手机号注册
2. 控制台 → API Keys → 创建 API key
3. 充值（最低 ¥10 起，支持微信/支付宝），不要信用卡

### 4.3 价格（截至 2026-05）

- **deepseek-chat（V3.2 / V4 系列）**：
  - 缓存命中：约 ¥0.5/百万 tokens
  - 缓存未命中：约 ¥2/百万 tokens
  - 输出：约 ¥3/百万 tokens
- **MagicImage 用量估算**：每次生成 ≈ 2k tokens 输入 + 200 tokens 输出 ≈ ¥0.005，**充 ¥10 够跑 2000 次 demo**
- 注：不同时期可能有打折活动（比如 V4 Pro 限时 2.5 折），看官方页

### 4.4 .env 字段

```ini
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
DEEPSEEK_MODEL=deepseek-chat
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1
```

### 4.5 hello-world 验证

```python
from openai import OpenAI
client = OpenAI(api_key=os.environ["DEEPSEEK_API_KEY"], base_url="https://api.deepseek.com/v1")
resp = client.chat.completions.create(
    model="deepseek-chat",
    messages=[{"role":"user","content":"返回JSON: {\"ok\": true}"}],
    response_format={"type":"json_object"},
)
print(resp.choices[0].message.content)
# 能输出合法 JSON 即视为通过
```

### 4.6 常见坑

- **OpenAI 兼容性**：`response_format={"type":"json_object"}` 大多时候靠谱，但偶尔会有非法 JSON，所以 1.1 章节里规定了"非法 JSON 回退默认参数"
- **超时**：高峰期可能慢到 30s+，记得设 `timeout=60`
- **不要混用 OpenAI / Anthropic 的库直接调**：用 OpenAI SDK 改 base_url 是最简单的接入方式

---

## 5. 申请优先级建议（按时间）

如果你周末时间紧，按这个顺序：

| 优先级 | 家 | 预计耗时 | 完成后能做 |
|-------|---|---------|----------|
| 🟢 P0 | 腾讯混元 3D | 15 分钟 | 跑通主力 3D 生成 |
| 🟢 P0 | DeepSeek | 10 分钟 | 跑通 LLM 意图解析 |
| 🟡 P1 | Meshy | 15 分钟（含科学上网） | 完成兜底链路（免费每月 200） |
| 🟡 P1 | Tripo | 20 分钟（如要充值更慢） | 完成备选链路（用赠送的 600 credits） |

**只完成 P0 即可开搭 MVP**（pipeline_3d 里只注册 hunyuan 一家，其他家先 mock）。P1 后面补，方便做降级演示。

---

## 6. 完成检查清单（t0-3 验收）

- [ ] `magicimage-agent/.env` 文件已创建，4 个 key 至少填入混元 + DeepSeek 两个
- [ ] `magicimage-agent/.env` 已加入 `.gitignore`，未提交到 git
- [ ] `magicimage-agent/.env.example` 文件已创建（占位 key），已提交到 git
- [ ] 跑通 `python -m magicimage-agent.scripts.hello_hunyuan` → 返回有效的 GLB 下载 URL
- [ ] 跑通 `python -m magicimage-agent.scripts.hello_deepseek` → 返回合法 JSON
- [ ] （可选）Meshy / Tripo 也能跑通 hello-world

完成以上 5 项，标记 todolist 第 3 条 `t0-3-api-keys` 为 completed。

---

## 7. 申请遇到问题？

| 问题 | 处理 |
|------|------|
| 腾讯混元 3D 控制台找不到入口 | 顶部搜索框搜"混元 3D"或"hunyuan-to-3d" |
| Tripo / Meshy 注册要求 GitHub 登录 | 用 Google 邮箱注册更快 |
| 国内访问 Tripo / Meshy 超时 | 用代理（Clash / V2Ray），或者只用混元跑 demo |
| DeepSeek 充值后查不到余额 | 控制台首页就有余额显示，几分钟到账 |
| 不想用真实手机号注册海外服务 | 阶段 0 跳过 Tripo / Meshy，只用混元，照样能跑通 |

---

> 申请完别忘了把 key 填到 `.env`，**不要 push 到 git**！
> `.gitignore` 已在 stage0-bootstrap 里规划排除 `.env`，但还是手动 `git status` 二次确认一次稳。
