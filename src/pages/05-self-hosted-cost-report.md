---
layout: ../layouts/Layout.astro
title: Self-Hosted Cost Report
---

# Self-Hosted Image/Video — Cost Research Report

Verified: Sep 3, 2026. Follows `04-self-hosted-research-harness.json`.
Currency: **1 USD = ₹95.00** (Sep 3 rate ≈ ₹94.89, rounded to match `03-research-report.md`).
Baseline hosted costs from `03-research-report.md`: Gemini 3 Pro Image 2K batch **₹6.37/img**, Veo 3.1 Fast 1080p **₹71.25 / 5s clip**.

> **Revision note (Sep 3, 2026):** the first draft of this report priced GPU **Pods** but divided by images-per-hour as though billing were per-image. Pods bill wall-clock time, not generations. All cost sections below are corrected, and pod vs serverless are now priced separately. See §5.

## Verdict (read this first)

| Decision | Recommendation | Why |
|---|---|---|
| Image | **Hybrid** — hosted primary, self-host bulk/draft **on serverless only** | Serverless self-host is ~60× cheaper per image than hosted; but open-weight quality on DNA prompts still trails Gemini 3 Pro Image |
| Video | **Stay hosted** | Open weights stop at Wan 2.2 (720p, no native audio, minutes per clip). Corrected costs show no meaningful saving vs Veo 3.1 Fast |
| **Billing model** | **Serverless (Modal / RunPod Serverless), not 24×7 Pods** | At 10k images/month a pod is **0.76% utilised** — you'd pay 730 hrs to use 5.6 |
| Best free commercial image weight | **FLUX.2 [klein] 4B** (Apache 2.0) | Only FLUX.2 weight with free commercial use; 9B / [dev] need a paid BFL licence |
| Best free commercial video weight | **Wan 2.2 TI2V-5B** (Apache 2.0) | Latest downloadable Wan; T2V+I2V, 720p@24fps |
| Cloud fit | **Modal or RunPod Serverless**; AWS Mumbai only for data residency | AWS Mumbai H100 Capacity Blocks are ~5× RunPod and are prepaid reservations |

---

# 1. Model shortlist

## Image

| Model | Params | Licence | Commercial free? | VRAM (BF16) | Ref / edit | vs Gemini baseline | Source |
|---|---|---|---|---|---|---|---|
| **FLUX.2 [klein] 4B** | 4B | Apache 2.0 | **Yes** | ~13 GB | Yes (multi-ref + edit) | Trails on complex prompts/text; usable for drafts & volume | [HF card](https://huggingface.co/black-forest-labs/FLUX.2-klein-4B) |
| FLUX.2 [klein] 9B | 9B | FLUX Non-Commercial | **No** — paid BFL licence | ~29 GB (~15 GB FP8) | Yes | Closer quality; still below Gemini/GPT Image 2 | [HF card](https://huggingface.co/black-forest-labs/FLUX.2-klein-9B) |
| FLUX.2 [dev] | 32B | FLUX Non-Commercial | **No** — paid BFL licence | ~64 GB (~32 GB FP8) | Yes | Best open FLUX; H100-class | [BFL GitHub](https://github.com/black-forest-labs/flux2) |
| Stable Diffusion 3.5 Large | 8B | Stability Community | **Conditional** — free under $1M/yr org revenue | ~16–18 GB FP16 | Weaker than FLUX.2 | Older quality ceiling | [stability.ai/license](https://stability.ai/license) |

## Video

| Model | Open weights? | Licence | Max output | VRAM | vs Veo 3.1 | Source |
|---|---|---|---|---|---|---|
| **Wan 2.2 TI2V-5B** | **Yes** | Apache 2.0 | 5s @ 720p 24fps | ≥24 GB (official single-GPU path) | Large gap; no native audio | [GitHub Wan2.2](https://github.com/Wan-Video/Wan2.2) |
| Wan 2.2 T2V/I2V-A14B | Yes | Apache 2.0 | 720p | ~80 GB recommended | Better than 5B; still trails Veo | [HF T2V-A14B](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B) |
| Wan 2.6 / 2.7 | **No — API only** | Commercial API | up to 15s | n/a | Not self-hostable | Verified: no weights in Wan-AI org |
| HunyuanVideo | Yes | Community (regional exclusions — verify) | ~5s class | 14 GB+ with offload | Research option | [HF](https://huggingface.co/tencent/HunyuanVideo) |
| LTX-Video / LTX-2 | Yes | Check card | Short clips | Consumer–datacenter | Fast, lower fidelity | [HF LTX-Video](https://huggingface.co/Lightricks/LTX-Video) |

**Excluded:** Wan 2.6/2.7 (no weights), Sora 2 (API sunset Sep 24, 2026), hosted-only Veo/Kling/Seedance.

---

# 2. Licensing — is it free?

| Weight | Free commercial self-host? | Catch | Official link |
|---|---|---|---|
| FLUX.2 [klein] **4B** | **Yes** | None under Apache 2.0 | [HF](https://huggingface.co/black-forest-labs/FLUX.2-klein-4B) · [BFL licence FAQ](https://help.bfl.ai/articles/9272590838-self-serve-dev-license-overview-pricing) |
| FLUX.2 [klein] 9B / [dev] | **No** | Builder / Platform / Professional / Enterprise tiers; **dollar price not published** | [bfl.ai/licensing](https://bfl.ai/licensing) · [bfl.ai/pricing](https://bfl.ai/pricing) |
| SD 3.5 | Conditional | Free under **$1M annual revenue**; above that → Enterprise (custom quote) | [stability.ai/license](https://stability.ai/license) |
| Wan 2.2 | **Yes** | Apache 2.0; you own outputs; comply with AUP | [GitHub LICENSE](https://github.com/Wan-Video/Wan2.2) |

**Implication:** start commercial self-host image work on **klein 4B only**. Do not deploy 9B/[dev] without a signed BFL licence. Treat the licence fee as an unknown fixed monthly cost until quoted.

---

# 3. Hardware — how big?

| Workload | Min GPU | Recommended | Weights on disk | Notes |
|---|---|---|---|---|
| FLUX.2 klein 4B @ 1024² | L4 24GB / RTX 4070-class | **L4 or L40S** | ~10–20 GB | Official ~13 GB VRAM; 4-step distilled |
| FLUX.2 klein 9B FP8 | 16 GB card | L40S / A100 | ~20 GB | Needs commercial licence |
| FLUX.2 [dev] BF16 | H100 80GB | H100 / H200 | ~64 GB | Overkill unless quality gap proven |
| Wan 2.2 TI2V-5B 720p 5s | 24 GB (with offload) | **L40S 48GB / A100 80GB** | ~20–40 GB | Official: under 9 min on consumer GPU |
| Wan 2.2 A14B | 80 GB class | H100 | large | Multi-GPU FSDP path in README |

---

# 4. Throughput assumptions

| Job | Assumed | Confidence | Basis |
|---|---|---|---|
| klein 4B, 1024², 4 steps | **2.0 s/image** (conservative) · **1.0 s** (optimised) | Medium | BFL claims sub-second on modern HW |
| Wan 2.2 TI2V-5B, 5s 720p | **300 s** (optimistic) · **540 s** (conservative) | Medium | Official "under 9 min"; L40S community 265–331 s |

**Correct cost formulas — these differ by billing model:**

```
Serverless:  cost_per_output = seconds_per_output × $_per_second
             (utilisation is irrelevant — you pay only active seconds)

Pod / VM:    monthly_cost    = $_per_hour × 730          ← fixed, regardless of volume
             cost_per_output = monthly_cost / monthly_volume
```

The first draft wrongly applied a 70% utilisation divisor to serverless and applied the serverless formula to pods.

---

# 5. Billing model — the decisive factor

| Product | You pay for | Idle cost | Fits our usage pattern? |
|---|---|---|---|
| **RunPod Serverless** | active seconds, scale-to-zero | none | **Yes** |
| **Modal** | active seconds, scale-to-zero | none | **Yes** |
| RunPod Pod (Community/Secure) | wall-clock while pod exists | **full rate 24×7** | Only at high volume |
| AWS EC2 on-demand | wall-clock while instance runs | full rate | Only at high volume |
| AWS Capacity Blocks | **prepaid reservation block** | full block | No — worst fit |

### Duty cycle at our volumes

| Workload | Real GPU work | Hours billed on a 24×7 pod | Utilisation |
|---|---|---|---|
| 10,000 images @ 2 s | **5.6 GPU-hours/mo** | 730 | **0.76%** |
| 1,000 clips @ 540 s | **150 GPU-hours/mo** | 730 | **20.5%** |

A 24×7 pod at 0.76% utilisation means paying for 730 hours to use 5.6. This single fact reverses the image conclusion.

---

# 6. Cloud GPU rates (verified Sep 3, 2026)

## Serverless — per-second, scale-to-zero

| Provider | GPU | $ / hr | $ / sec | Official link |
|---|---|---|---|---|
| RunPod Serverless | L4 / A5000 / 3090 24GB | $0.69 | $0.000192 | [runpod.io/pricing](https://www.runpod.io/pricing) |
| RunPod Serverless | RTX 4090 24GB | $1.10 | $0.000306 | same |
| RunPod Serverless | L40 / L40S / 6000 Ada 48GB | $1.75 | $0.000486 | same |
| RunPod Serverless | A100 80GB | $2.72 | $0.000756 | same |
| RunPod Serverless | H100 80GB | $4.79 | $0.001331 | same |
| Modal | L4 | $0.80 | $0.000222 | [modal.com/pricing](https://modal.com/pricing) |
| Modal | L40S | $1.95 | $0.000542 | same |
| Modal | A100 80GB | $2.50 | $0.000694 | same |
| Modal | H100 SXM5 | $3.95 | $0.001097 | same |

## Pods / VMs — billed 24×7 while running

| Provider | GPU | $ / hr | ₹ / month (730 h) | India region | Official link |
|---|---|---|---|---|---|
| RunPod Pod Community | L40S 48GB | $0.79 | **₹54,787** | No | [runpod.io/pricing](https://www.runpod.io/pricing) |
| RunPod Pod Secure | L40S 48GB | $0.99 | **₹68,656** | No | same |
| RunPod Pod Secure | A100 PCIe 80GB | $1.39 | ₹96,396 | No | same |
| RunPod Pod | H100 PCIe 80GB | from $1.99 | ₹138,006 | No | same |
| AWS on-demand | g6.xlarge (1× L4) | $0.8048 | ₹55,813 | Mumbai g6 available | [AWS on-demand](https://aws.amazon.com/ec2/pricing/on-demand/) · [G6](https://aws.amazon.com/ec2/instance-types/g6/) |
| AWS Capacity Block | H100, Mumbai | $4.72 /GPU-hr | ₹327,332 | **Yes (ap-south-1)** | [AWS Capacity Blocks](https://aws.amazon.com/ec2/capacityblocks/pricing/) |
| GCP | GPU VMs | region-variable | — | limited | [GCP GPU pricing](https://cloud.google.com/compute/gpus-pricing) |

---

# 7. Cost — Image (FLUX.2 klein 4B, 1024², high quality)

## 7a. Serverless — recommended path

At 2.0 s/image. Halve every figure at 1.0 s/image.

<table>
  <thead>
    <tr>
      <th>Provider</th><th>GPU</th><th>$ / image</th><th>₹ / image</th>
      <th>Images per ₹100</th><th>Monthly ₹ @ 10k</th><th>Official link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3"><b>RunPod Serverless</b></td>
      <td>L4 24GB</td><td>$0.00038</td><td>₹0.036</td><td><b>~2,750</b></td><td>₹364</td>
      <td rowspan="3"><a href="https://www.runpod.io/pricing">runpod.io/pricing</a></td>
    </tr>
    <tr><td>L40S 48GB</td><td>$0.00097</td><td>₹0.092</td><td><b>~1,080</b></td><td>₹924</td></tr>
    <tr><td>H100 80GB</td><td>$0.00266</td><td>₹0.253</td><td><b>~400</b></td><td>₹2,528</td></tr>
    <tr>
      <td rowspan="3"><b>Modal</b></td>
      <td>L4</td><td>$0.00044</td><td>₹0.042</td><td><b>~2,370</b></td><td>₹422</td>
      <td rowspan="3"><a href="https://modal.com/pricing">modal.com/pricing</a></td>
    </tr>
    <tr><td>L40S</td><td>$0.00108</td><td>₹0.103</td><td><b>~970</b></td><td>₹1,029</td></tr>
    <tr><td>H100 SXM5</td><td>$0.00219</td><td>₹0.208</td><td><b>~480</b></td><td>₹2,085</td></tr>
    <tr>
      <td colspan="2"><b>Hosted baseline — Gemini 3 Pro Image 2K batch</b></td>
      <td>$0.067</td><td>₹6.37</td><td><b>15</b></td><td>₹63,650</td>
      <td><a href="https://ai.google.dev/gemini-api/docs/pricing">ai.google.dev pricing</a></td>
    </tr>
  </tbody>
</table>

Excludes cold-start seconds (model load billed as active time on both platforms) — add ~5–15% until measured.

## 7b. Pods — fixed cost regardless of volume

| Pod | ₹ / month (fixed) | ₹ / image @ 10k | ₹ / image @ 100k | Breakeven vs Gemini batch |
|---|---|---|---|---|
| RunPod L40S Community | ₹54,787 | ₹5.48 | ₹0.55 | **8,607 images/mo** |
| AWS g6.xlarge L4 | ₹55,813 | ₹5.58 | ₹0.56 | **8,769 images/mo** |
| RunPod L40S Secure | ₹68,656 | ₹6.87 | ₹0.69 | **10,787 images/mo** |
| RunPod H100 PCIe | ₹138,006 | ₹13.80 | ₹1.38 | 21,682 images/mo |
| AWS Mumbai H100 Capacity Block | ₹327,332 | ₹32.73 | ₹3.27 | 51,427 images/mo |

**At 10,000 images/month a RunPod L40S Secure pod (₹6.87/img) is more expensive than the hosted Gemini batch API (₹6.37/img).** Pods only win above their breakeven volume. A single L40S pod at 2 s/image and 70% utilisation caps at ~920,000 images/month, so headroom is not the constraint — sustained demand is.

---

# 8. Cost — Video (Wan 2.2 TI2V-5B, 5s @ 720p, no native audio)

## 8a. Serverless

<table>
  <thead>
    <tr>
      <th>Provider</th><th>GPU</th><th>Time/clip</th><th>₹ / clip</th>
      <th>Clips per ₹1,000</th><th>Monthly ₹ @ 1k clips</th><th>Official link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2"><b>RunPod Serverless</b></td>
      <td rowspan="2">L40S 48GB</td>
      <td>300 s</td><td>₹13.9</td><td><b>72</b></td><td>₹13,854</td>
      <td rowspan="2"><a href="https://www.runpod.io/pricing">runpod.io/pricing</a></td>
    </tr>
    <tr><td>540 s</td><td>₹24.9</td><td><b>40</b></td><td>₹24,938</td></tr>
    <tr>
      <td rowspan="2"><b>Modal</b></td>
      <td rowspan="2">L40S</td>
      <td>300 s</td><td>₹15.4</td><td><b>65</b></td><td>₹15,438</td>
      <td rowspan="2"><a href="https://modal.com/pricing">modal.com/pricing</a></td>
    </tr>
    <tr><td>540 s</td><td>₹27.8</td><td><b>36</b></td><td>₹27,788</td></tr>
    <tr>
      <td colspan="3"><b>Hosted baseline — Veo 3.1 Fast 1080p + audio</b></td>
      <td>₹71.25</td><td><b>14</b></td><td>₹71,250</td>
      <td><a href="https://fal.ai/models/fal-ai/veo3.1/fast">fal.ai Veo 3.1 Fast</a></td>
    </tr>
  </tbody>
</table>

## 8b. Pods

| Pod | ₹ / month | ₹ / clip @ 1k | Pod capacity @540 s | Breakeven vs Veo Fast |
|---|---|---|---|---|
| RunPod L40S Community | ₹54,787 | ₹54.8 | 4,867 clips/mo | 769 clips/mo |
| RunPod L40S Secure | ₹68,656 | ₹68.7 | 4,867 clips/mo | 964 clips/mo |

**The video saving is not worth taking.** Even the best serverless case (₹13.9/clip) buys 720p with no native audio and 5–9 minutes of latency, against Veo 3.1 Fast at ₹71.25 for 1080p with synchronised audio in seconds. On a pod, self-host costs effectively the same as Veo while delivering far less.

---

# 9. Hidden costs & quality tax

| Item | Impact | Notes |
|---|---|---|
| **Usable-output rate** | Dominates | If klein 4B needs 4 attempts where Gemini needs 2, self-host effective cost quadruples — still cheap on serverless, but only if output clears the bar at all |
| **Quality gap** | Deal-breaker risk | DNA prompts are long and attribute-dense; hosted models win prompt adherence |
| **BFL licence (9B/[dev])** | Unknown fixed $ | Needed to close the quality gap; price not public |
| **Cold starts** | 5–15% on serverless | Weight load is billed as active seconds |
| **Idle GPU** | Kills pods | The entire finding of §5 |
| **Engineering** | Fixed, recurring | Server setup, scaling, safety filters, model upgrades |
| **Safety filtering** | Extra | Hosted APIs include it; self-host must add Hive/Azure/etc. |
| **Egress + storage** | Small for images, real for video | Add CDN for delivery |

---

# 10. Breakeven summary

| Question | Answer |
|---|---|
| Serverless self-host vs hosted, images | Self-host wins at **any volume** on raw compute (₹0.04–0.10 vs ₹6.37) — decision rests entirely on quality |
| Pod self-host vs hosted, images | Hosted wins **below ~8,600–10,800 images/month** |
| Serverless self-host vs hosted, video | Self-host is cheaper (₹14–28 vs ₹71) but 720p, no audio, minutes of latency |
| Pod self-host vs hosted, video | Roughly a wash at 1,000 clips/month — not worth it |

---

# 11. Recommended strategy

1. **Production images:** keep **Gemini 3 Pro Image (batch)** as primary.
2. **Bulk/draft/A-B variants:** deploy **FLUX.2 klein 4B on RunPod Serverless L4 or Modal L4** — ₹0.04/image. **Never on a 24×7 pod at our current volume.**
3. **Measure first:** the whole image case hinges on klein 4B's usable-output rate against DNA prompts. Run that in `02-model-evaluation.md` before committing.
4. **Only move to a pod** if sustained volume exceeds ~10,000 images/month *and* quality clears the bar — then RunPod L40S Community is the cheapest always-on option.
5. **Video:** stay on Veo 3.1 Fast / Kling 3.0 Pro. Prototype Wan 2.2 only for privacy-constrained internal work.
6. **India residency:** AWS Mumbai H100 Capacity Blocks at $4.72/GPU-hr are prepaid and ~5× RunPod — use only if compliance mandates it.

---

# 12. Uncertain / next measurements

- Measured seconds/image for klein 4B on L4 and L40S in our stack (Diffusers vs ComfyUI vs TensorRT)
- **DNA-prompt usable-output rate** for klein 4B vs Gemini — blocks the real decision
- BFL Builder / Platform licence **dollar price** (quote required)
- Cold-start seconds per serverless invocation, which sets the 5–15% overhead
- Wan 2.2 A14B wall-clock on H100
- Live AWS ap-south-1 g6 on-demand quote (third-party ≈ $0.80–1.00/hr)

---

# Official sources (HTTP 200 on Sep 3, 2026)

| Topic | URL |
|---|---|
| FLUX.2 klein 4B | https://huggingface.co/black-forest-labs/FLUX.2-klein-4B |
| FLUX.2 klein 9B | https://huggingface.co/black-forest-labs/FLUX.2-klein-9B |
| BFL open-weights licensing | https://bfl.ai/licensing |
| BFL licence FAQ | https://help.bfl.ai/articles/9272590838-self-serve-dev-license-overview-pricing |
| BFL pricing | https://bfl.ai/pricing |
| Stability licence | https://stability.ai/license |
| Wan 2.2 | https://github.com/Wan-Video/Wan2.2 |
| Wan 2.2 TI2V-5B | https://huggingface.co/Wan-AI/Wan2.2-TI2V-5B |
| HunyuanVideo | https://huggingface.co/tencent/HunyuanVideo |
| LTX-Video | https://huggingface.co/Lightricks/LTX-Video |
| RunPod pricing (Pods + Serverless) | https://www.runpod.io/pricing |
| Modal pricing | https://modal.com/pricing |
| AWS Capacity Blocks | https://aws.amazon.com/ec2/capacityblocks/pricing/ |
| AWS On-Demand | https://aws.amazon.com/ec2/pricing/on-demand/ |
| AWS G6 (L4) | https://aws.amazon.com/ec2/instance-types/g6/ |
| GCP GPU pricing | https://cloud.google.com/compute/gpus-pricing |
| Gemini API pricing (baseline) | https://ai.google.dev/gemini-api/docs/pricing |
| fal.ai Veo 3.1 Fast (baseline) | https://fal.ai/models/fal-ai/veo3.1/fast |
