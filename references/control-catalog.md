# Cost Control Catalog

Choose controls that bound the specific billing meter. Prefer multiple independent layers.

## Prevent

- Authenticate mutation and generation endpoints.
- Validate cost-bearing inputs on the server.
- Allowlist source domains, models, formats, sizes, and batch counts.
- Separate public reads from privileged cache-fill or generation writes.
- Make workflow state transitions acyclic and explicit.
- Use least-privilege credentials with service and resource scope.

## Contain

- Apply per-request work limits and streaming byte limits.
- Apply per-IP, principal, tenant, feature, and global rate limits.
- Use daily application limits on tokens, events, bytes, messages, and objects.
- Use distributed idempotency and single-flight control.
- Cap concurrency at the expensive side effect.
- Configure maximum queue deliveries and dead-letter handling.
- Add circuit breakers (automatic stops) and kill switches for each paid provider.
- Serve a degraded cached or static mode when a paid dependency is disabled.

Rate limits and total-usage limits solve different problems. A steady request rate can still exceed a monthly application limit. A low monthly limit can still allow a damaging one-minute burst.

## Detect

- Alert on spend rate and usage units, not only invoice total.
- Alert on derivative signals: calls per user action, writes per job, retry ratio, fallback ratio, cache-miss ratio, bytes per request, and unique objects per hour.
- Compare short and long windows to catch step changes.
- Attribute usage by feature, tenant, route, job type, model, and environment.
- Send alerts through a channel that does not depend on the failing service.

An alert without a named responder, a response deadline, and a stop control is not a control.

## Stop

- Verify whether the provider offers a true hard cap, delayed action, notification only, or no cap.
- Test automatic stops in a test environment, with authorization.
- Protect stop and resume operations with strong authentication and audit logs.
- Define who can accept downtime versus continued spend.
- Make recovery explicit: reconcile queues, deduplicate side effects, rotate compromised keys, and re-enable features gradually.

Provider controls change. Verify current official documentation during every audit. Examples of control classes include Vercel Spend Management actions, AWS Budgets actions, GCP budgets and programmatic notifications, and provider-specific AI or messaging quotas. Do not assume that the word “budget” means automatic shutdown.

## Calculate the loss bound

Every bound needs an explicit window. The default window runs from first exploitation through detection and shutdown. Also state the per-billing-cycle bound when it differs.

Use the first applicable bound:

1. hard provider cap, minus charge classes it excludes;
2. global application limit;
3. global rate limit multiplied by detection and shutdown time;
4. tenant/principal count multiplied by its application limit, only when principal creation is itself bounded (with free self-service signup, this bound does not apply);
5. queue depth multiplied by maximum attempts and cost per attempt;
6. storage/object limit multiplied by unit cost and retention.

A rate limit that is enforced per region, colo, or instance is not a global limit. Multiply it by the number of enforcement points. A bound that depends on a human stop holds only if a named responder can execute it within the window.

Include delayed metering and in-flight work. State uncertainty as a range. If a control has not been tested, discount it rather than treating it as certain.

## Verify controls

Use observable behavior:

- the N+1 request is rejected before the paid call;
- duplicate delivery produces one paid side effect;
- a poison job reaches the dead-letter destination;
- the circuit breaker opens and serves degraded mode;
- a provider budget notification or action fires in a test environment;
- usage dashboards and internal counters agree within an explained delay;
- stopping service A does not leave storage, replicas, or queues growing.

Before you run fault injection, a load test, or a poison-message test, follow the test procedure in the Boundaries section of `SKILL.md`: get authorization, use a test environment, and write the abort condition. A test account that is billed at production prices still needs a spend limit.

Exercise failure paths, not only successful requests. In the test environment, make the downstream dependency fail repeatedly. Observe whether retry count, delay, dead-letter routing, circuit breaking, and billing side effects match the design. A retry that stops during a normal run has not proved that it stops under persistent failure.
