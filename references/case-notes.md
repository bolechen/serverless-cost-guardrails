# Public Incident Pattern Notes

Source hub: [ServerlessHorrors](https://serverlesshorrors.com/)

These are concise pattern notes, not reproductions of the original stories. Open the current source before citing details because posts and provider conclusions can change.

The Cloudflare code-review checklist was also informed by [Cell's post on X](https://x.com/cellinlab/status/2108021292071067953), which called out alarms that always reschedule, polling without stop conditions, cron idempotency, D1 query shape, cyclic Queue/Worker calls, hot-path writes, unlimited retries, and AI loops without iteration caps. A reply adds an important verification point: persistent downstream failure is the test that reveals whether retries truly stop.

## Recursive queues, storage writes, and fallback scans

[RetainDB, reported $36,000](https://serverlesshorrors.com/all/cloudflare-36k/)

The published summary attributes the bill to three compounding paths: a queue workflow that re-enqueued async work, multiple unbatched storage writes per logical write, and a key-list fallback that ran after index misses. The reusable lesson is multiplicative: fix recursion, operations per item, and fallback frequency separately.

## Infinite stateful-object work without a stop control

[Cloudflare Durable Objects, reported $8,846.78](https://serverlesshorrors.com/all/cloudflare-88k/)

The summary says two stateful objects entered an infinite loop and the team learned from the bill. The reusable lesson is to bound internal work even when no public request remains active, and to verify whether the provider offers an enforceable spend stop.

## Large-scale bandwidth

[Jmail on Vercel, reported $46,485.99](https://serverlesshorrors.com/all/vercel-46k/)

The report highlights that caching mitigations did not make extreme pageview volume free. The reusable lesson is to model every billed transfer layer and request class, then test the provider's spend action rather than assuming a configured amount automatically stops usage.

[Netlify DDoS, reported $104,500](https://serverlesshorrors.com/all/netlify-104k/)

The reported attack focused on a multi-megabyte static file and generated extreme transfer. The reusable lesson is that “static” does not mean costless. Bound hot-object bytes and place denial-of-wallet protection in front of storage and CDN delivery.

## Email and account-creation abuse

[Mailgun during a DoS, reported $11,000](https://serverlesshorrors.com/all/mailgun-11k/)

The reusable lesson is to rate-limit at the paid send, make flows idempotent, and apply a global daily message ceiling. Per-IP controls alone are weak against distributed account creation.

## Analytics event explosions

[PostHog cases](https://serverlesshorrors.com/tags/posthog/)

The reusable lesson is to treat telemetry as a paid write path. Cap event production, sample high-volume events, prevent autonomous code changes from inventing unbounded instrumentation, and monitor events per user action.

## Provider documentation to verify live

- [Vercel Spend Management](https://vercel.com/docs/spend-management)
- [AWS budget actions](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-controls.html)
- [Google Cloud budgets](https://cloud.google.com/billing/docs/how-to/budgets)
- [Cloudflare billing documentation](https://developers.cloudflare.com/billing/)

Do not freeze provider pricing, quotas, or feature availability in the skill. Record the retrieval date in each audit instead.
