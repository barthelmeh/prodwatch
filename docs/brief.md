# Production E2E Monitor: Project Brief

_Working title. A base context document for agentic conversations. Treat the hypotheses here as starting assumptions to test, not fixed requirements._

## 1. One-line summary

Run your existing Playwright tests against production, on a schedule and after every deploy, with flat predictable pricing and alerts that only fire when something is really broken.

## 2. About the builder and constraints

- Junior software engineer (almost 2 years' experience), working full time, so evenings and weekends only.
- Target: shippable v1 in about 3 months of part-time work, using AI coding tools.
- Deploy on AWS. Tech stack is not prescribed; a typical modern web stack is fine.
- Previous side project lost momentum, so scope must stay small enough to finish and the problem must stay interesting.
- Primary goal is learning; the business outcome is secondary but should be plausible.

## 3. The problem and pain points

Teams often have a Playwright or other E2E suite that runs in CI before release, but nothing checks that the critical user journeys (login, checkout, signup) still work in production. Broken third-party integrations, config changes, expired credentials and bad deploys can break production flows without any unit or CI test noticing.

Existing tools address this, but have recurring pain points:

- **Unpredictable pricing.** Metered per-run billing means a misconfigured check or a higher frequency can become a surprise bill.
- **Alert noise.** Synthetic checks fail intermittently for reasons unrelated to real outages, so teams learn to ignore alerts. A false alarm should be treated as a defect in the monitor.
- **Rewrite and setup overhead.** Teams already own an E2E suite but have to port or re-author checks for a monitoring product.
- **Weak deploy awareness.** Linking a failing check to the deploy that caused it is often an afterthought.

## 4. Target customer

Small engineering teams of roughly 3 to 30 developers who already write Playwright, run E2E tests in CI, and either do not monitor production journeys or find existing tools expensive or heavy for their needs. Buyers are engineers or engineering leads.

## 5. Business position

- **Competition is real and expected.** Checkly is the reference product for code-first Playwright synthetic monitoring. Others include Hyperping (flat-rate), Better Stack, Datadog Synthetics, Grafana Synthetic Monitoring and Uptime.com. Checkly has also been adding Playwright suite support, so "run Playwright on a schedule" alone is not a differentiator.
- **Demand is proven.** The category exists, people pay for it, and switching is driven mostly by pricing predictability and complexity.
- **Wedge:** predictable flat pricing (hard cap on runs, so a monitor pauses rather than overbills) + flake-aware alerting + reuse of existing tests + post-deploy triggers.
- **Realistic ceiling:** a small, modest business serving a segment, not a Checkly replacement. The goal is a handful of paying customers and a strong CV project.
- **Validation still needed:** the flake-aware alerting angle is a hypothesis, not a proven gap. Talk to 5 to 10 developers who run Playwright in CI before committing heavily. Ask what they use for production checks, what annoys them, and whether anyone tried Checkly and left.

## 6. Differentiators (positioning)

1. **Flat, capped pricing.** Predictable monthly cost; hitting the cap pauses runs instead of billing overage.
2. **Flake-aware alerting.** Retry and confirm before paging. Keep per-check history so a long-stable check that suddenly fails is treated differently from one that fails 1 in 20 runs. Quarantine flaky checks and show a reliability score.
3. **No rewrite.** Point at an existing Playwright repo or upload a bundle, and select tests by tag (for example `@prod-safe`).
4. **Post-deploy trigger.** Run the tagged tests when a deploy completes and attribute results to that deploy.

## 7. v1 scope (3 months)

**In:**

- User accounts, auth and team/workspace basics
- Connect a repo or upload a test bundle; select tests by tag
- Scheduled runs and deploy-triggered runs (webhook or GitHub deployment event)
- Single region
- Run results with logs, screenshots and Playwright traces
- Retry-and-confirm logic and per-check run history
- Alerts to Slack and webhooks (PagerDuty if time allows)
- Flat-tier subscription billing with a run cap
- CI/CD for the project itself in a monorepo

**Out (deliberately):**

- Multi-region execution
- Visual or no-code test builder
- API and uptime checks
- Public status pages
- Terraform or other infrastructure-as-code providers
- Enterprise features (SSO, audit logs)

## 8. Event-driven design (sketch, to be refined)

Events and flow, with AWS services that fit naturally. This is a starting point for discussion, not a final architecture.

- **Schedule fires** (EventBridge Scheduler) → enqueues a run request (SQS).
- **Deploy completed** (webhook received by an API endpoint) → publishes a run request event.
- **Run worker** (Lambda container image or Fargate task running Playwright) consumes the queue, executes the tests, stores artifacts (S3) and emits a **run completed** event.
- **Result evaluator** consumes run completed events, applies retry/confirm logic and flake history, and emits **check failed (confirmed)** or **check recovered** events.
- **Notifier** consumes those events and fans out to Slack, webhooks and PagerDuty (SNS or EventBridge rules).
- **Billing metering** consumes run completed events to count usage against the plan cap.

## 9. Learning opportunities (mapped to goals)

- **Event-driven design on AWS:** EventBridge, SQS, SNS and Lambda used for genuine reasons (scheduling, fan-out, retries, dead-letter queues, idempotency).
- **Cloud programming:** running headless Chromium in Lambda or Fargate, timeouts and memory limits, container images, artifact storage.
- **Full stack:** user management and auth, team workspaces, a results dashboard.
- **Payments:** subscription tiers with Stripe plus usage metering against a cap.
- **Security and isolation:** running customer-supplied code, secret handling, network restrictions, per-tenant isolation. A strong interview talking point.
- **Monorepo and CI/CD:** shared packages, independent deploys for the web app, API and worker, infrastructure as code.
- **CV value:** a real multi-tenant, event-driven system with billing that can be discussed in depth.

## 10. Biggest risks

- **Security of running untrusted code.** Keep v1 tight: uploaded bundles only, strict timeouts, restricted networking, no access to the platform's own infrastructure.
- **Compute cost and margins.** Browser runs are relatively expensive, so run caps and tier pricing must be designed alongside the architecture.
- **Scope creep.** Resist matching Checkly's feature list; stay within the v1 scope above.
- **Distribution.** Finding the first paying customers matters more than features. Validate early.
- **Motivation.** Keep milestones small and demoable at each stage.

## 11. Open questions for the next conversations

- Repo connection (GitHub App) versus bundle upload for v1: which is simpler and safer?
- Lambda versus Fargate for Playwright runs, given duration, memory and cost?
- How should flake-aware alerting work in practice (retry count, history window, quarantine rules)?
- What pricing tiers and run caps make the economics work?
- Which stack and IaC tool for the monorepo?
- How to validate demand quickly with the first 5 to 10 conversations?
