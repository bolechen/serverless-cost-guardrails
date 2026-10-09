# Serverless Cost Guardrails

An agent skill for auditing serverless and usage-based systems for runaway cloud bills and denial-of-wallet paths.

It turns public incident patterns from [ServerlessHorrors](https://serverlesshorrors.com/) and practical review prompts such as [Cell's Cloudflare checklist](https://x.com/cellinlab/status/2108021292071067953) into a vendor-neutral audit workflow. It focuses on loss bounds: recursion, retries, fan-out, scans, storage growth, bandwidth, paid API abuse, missing alerts, and ineffective stop controls.

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

## What the skill produces

- a billing graph from triggers to metered operations;
- prioritized findings with evidence and a loss bound;
- preventive, containment, detection, and stop controls;
- immediate and durable actions;
- explicit unknowns that require provider-dashboard verification.

The skill audits read-only by default. It does not change infrastructure, purge queues, rotate keys, or run load or fault tests unless you authorize each action. During an incident it gives containment steps for you to run, and executes them only with per-action approval.

The workflow is vendor-neutral. Only Cloudflare Workers has a product-specific checklist so far; other providers use the general patterns and controls.

This project summarizes incident patterns and links original sources. It is not affiliated with ServerlessHorrors or any cloud provider.

## License

MIT
