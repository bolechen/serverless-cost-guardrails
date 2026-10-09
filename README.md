# Serverless Cost Guardrails

An agent skill for auditing serverless and usage-based systems for runaway cloud bills and denial-of-wallet paths.

It turns public incident patterns from [ServerlessHorrors](https://serverlesshorrors.com/) and practical review prompts such as [Cell's Cloudflare checklist](https://x.com/cellinlab/status/2108021292071067953) into a vendor-neutral audit workflow. It focuses on loss bounds: recursion, retries, fan-out, scans, storage growth, bandwidth, paid API abuse, missing alerts, and ineffective stop controls.

[中文说明](#中文说明)

## Install

Install from [skills.sh](https://www.skills.sh/p/CeCib20mkAR8I0HC):

```sh
npx skills add https://skills.sh/p/CeCib20mkAR8I0HC
```

Or clone the repository and copy it into your agent's skills directory:

```sh
git clone https://github.com/bolechen/serverless-cost-guardrails.git
```

The skill follows the standard `SKILL.md` layout and includes Codex UI metadata in `agents/openai.yaml`. The example prompts use Codex's `$skill` syntax; in other agents, name the skill in plain text.

## Quick start

One line is enough:

```text
Use $serverless-cost-guardrails to audit this project for runaway billing risks.
```

With that prompt, the skill:

- audits read-only. It does not change infrastructure, purge queues, rotate keys, run load or fault tests, or touch the target repository's branches. It runs a change only when you authorize that specific action;
- asks once for your loss tolerance or monthly spend, then continues with a stated default instead of waiting;
- reads metered dependencies and every entry point first (routes, server actions, webhooks, schedules, queue consumers), then states what it did not read;
- gives each finding a loss bound over an explicit time window, with the formula, so you can substitute real prices;
- labels every claim as measured fact, code fact, config fact, inference, or unknown.

## What it checks

Eleven audit categories, each reported as a finding or `not found`:

1. Recursive work
2. Per-request fan-out
3. Fallback becomes the hot path
4. Denial of wallet
5. Retry storms and partial failure
6. Bandwidth and hot-object attacks
7. Storage accumulation
8. Query and analytics amplification
9. Test and automation leakage
10. Control-plane traps
11. Detection and stop controls

Checks that ordinary reviews often miss:

- guest quotas stored only in a cookie;
- a CAPTCHA that the client can report as "unavailable" to skip it;
- server actions that are public endpoints without their own auth or rate limit;
- WAF and rate-limit rules scoped to one hostname while a preview host uses the same paid keys;
- a successful handler that re-triggers itself, so retry limits never apply;
- a budget alert mistaken for a hard cap, and a removed schedule that is still deployed.

Findings are ranked P0–P3 by who can trigger them (anonymous, cheap account, paying tenant, leaked credential, internal) and by the spend rate that actor can actually reach.

## What the skill produces

- a billing graph from triggers to metered operations;
- prioritized findings with evidence and a loss bound;
- preventive, containment, detection, and stop controls;
- immediate and durable actions;
- explicit unknowns that require provider-dashboard verification.

During an incident it gives containment steps for you to run, and runs a step only when you authorize that step.

## Example prompts

```text
Use $serverless-cost-guardrails to audit this architecture before launch.
```

```text
Compare our queue, AI, storage, and email paths with public runaway-billing incidents. Separate measured facts, code and config facts, inference, and unknown dashboard settings.
```

```text
We received a sudden cloud bill. Find the amplification path, estimate the maximum remaining exposure, and give immediate containment actions before long-term fixes.
```

## Sources and reference date

- Incident patterns come from [ServerlessHorrors](https://serverlesshorrors.com/); case notes were checked against the original pages on 2026-10-09.
- The skill does not freeze prices, quotas, or provider features. Each audit verifies current official documentation and records the retrieval date.
- The workflow is vendor-neutral. Only Cloudflare (Workers, storage, R2, and AI products) has a product-specific checklist so far; other providers use the general patterns and controls.

`evals/evals.json` holds three behavior scenarios (pre-launch audit, active incident, and an out-of-scope request) in the format from Anthropic's [skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices#build-evaluations-first). The skill does not load this file.

This project summarizes incident patterns and links original sources. It is not affiliated with ServerlessHorrors or any cloud provider.

## 中文说明

一个 agent skill，帮你在账单爆掉之前，找出 serverless 和按量计费系统里的失控计费路径和"刷钱"漏洞。规则来自 [ServerlessHorrors](https://serverlesshorrors.com/) 上的真实事故，适用于 Cloudflare、Vercel、AWS、Firebase 等平台。

### 安装

```sh
npx skills add https://skills.sh/p/CeCib20mkAR8I0HC
```

### 一句话就能用

```text
用 $serverless-cost-guardrails 检查这个项目的账单风险
```

默认行为：

- **只读。** 不改基础设施，不清空队列，不轮换密钥，不跑压测，也不动被审仓库的分支。任何改动都要你针对这个具体操作单独授权。
- **只问一次。** 问你能承受多少损失或月支出，然后按默认值继续审计，不停下来等你回答。
- **先看入口。** 先从环境变量和依赖找出付费服务，再列出所有入口，包括路由、server action、webhook、定时任务和队列，最后说明哪些代码没读。
- **算出钱数。** 每条发现都给出损失上限公式和时间窗口，你可以代入真实单价。
- **标明证据。** 每个结论都标为实测、代码事实、配置事实、推测或未知。

### 检查什么

11 个类别：递归任务、单次请求扇出、兜底逻辑变成常走的路径、刷钱攻击、重试风暴、带宽和热点对象、存储累积、查询放大、测试和自动化泄漏、停掉服务后仍在计费、检测和止损。

常被忽略的检查项：

- 游客额度只存在 cookie 里；
- 客户端可以声明"验证码不可用"来跳过人机验证；
- server action 是公开入口，却没有自己的鉴权和限流；
- WAF 和限速规则只覆盖主域名，而预览域名用的是同一套付费 key；
- 处理成功后又触发自己的循环，重试上限挡不住；
- 把预算提醒当成硬上限，或者以为删掉配置就停掉了定时任务。

发现按 P0–P3 分级，依据是谁能触发（匿名、免费注册、付费用户、泄露的凭据、内部）以及这个人实际能刷多快。

### 资料来源

事故模式来自 ServerlessHorrors，案例于 2026-10-09 对照原文核对过。价格、配额和服务商功能不写死在 skill 里，每次审计都以当天的官方文档为准，并记录查询日期。

## Author

Bole Chen ([@avenger on X](https://x.com/avenger))

## License

MIT
