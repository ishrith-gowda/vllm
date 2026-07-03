# vLLM Weighted Fair Queuing (WFQ) Scheduler — Maintained Fork

This branch (`wfq-scheduler`) maintains the Weighted Fair Queuing scheduler
proposed upstream in [vllm-project/vllm#31909](https://github.com/vllm-project/vllm/pull/31909)
(+4,632 LOC, 13 files). Upstream review concluded that priority/FCFS policies cover most
current use cases and a non-standard scheduling API carries maintenance overhead, so per
that discussion the feature lives here as a maintained fork.

- **Base commit:** `f1531d9f2` (upstream main, Dec 2025). The scheduler integrates with the
  vLLM v1 engine core (`SchedulingPolicy` enum + request-queue factory).
- **Status:** feature-complete as reviewed; validated by the test/benchmark suite below.

## What it does

Proportional GPU-time sharing across requests via classic Weighted Fair Queuing
(GPS-approximating virtual-time scheduling; Demers et al. 1989, Parekh–Gallager 1993),
adapted to continuous batching:

- `WFQRequestQueue` — O(log n) heap-based virtual-time queue (`vllm/v1/core/sched/wfq_queue.py`)
- Per-request `weight` (default 1.0; uniform weights reproduce FCFS exactly, preserving
  backward compatibility)
- `--scheduling-policy wfq` config plumbing (`vllm/config/scheduler.py`)

Related literature: fairness in LLM serving is an active research area — cf. Virtual Token
Counter, "Fairness in Serving Large Language Models" (Sheng et al., OSDI '24) and
locality-aware fair scheduling (arXiv:2501.14312). WFQ provides weight-proportional
allocation, complementary to VTC's token-cost fairness.

## Measured results (from the upstream PR's benchmark suite)

| Metric | Result |
|---|---|
| Throughput | 950 req/s — 95% of FCFS baseline (1,000 req/s) |
| P99 latency | 1,400 ms (1.17× FCFS) |
| Queue operations | 0.08–0.15 ms add/pop at 10,000+ concurrent requests |
| Weighted fairness | completion-time ratios within 15% of configured weight ratios |
| Starvation | none — 100% completion across all stress scenarios |
| Test suite | 120 tests across 5 categories (unit / integration / e2e / stress / fairness) |

Reproduce: `tests/v1/core/benchmark_wfq_performance.py`, `benchmark_wfq_fairness.py`,
`stress_test_wfq.py`.

## Docs

- [Design](docs/DESIGN_WFQ_SCHEDULER.md) · [Config spec](docs/WFQ_CONFIG_SPEC.md) ·
  [Usage guide](docs/WFQ_USAGE_GUIDE.md)

## Roadmap

- [ ] Periodic rebase onto upstream main (tracked against v1 scheduler API changes)
- [ ] Head-to-head fairness benchmark vs. VTC-style token-cost scheduling on shared traces
- [ ] Write-up targeting an ML-systems workshop (methodology per OSDI '24 fairness evaluation)

Maintainer: [@ishrith-gowda](https://github.com/ishrith-gowda) · ishrithgowda@berkeley.edu
