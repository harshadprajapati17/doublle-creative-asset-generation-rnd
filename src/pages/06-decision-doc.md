---
layout: ../layouts/Layout.astro
title: Decision Doc
---

# Decision Doc — Image & Video Generation

Verified Sep 3, 2026 · 1 USD = ₹95.00 · Two tables, one decision each.
Detail and workings: `03-research-report.md` (API/hosted) and `05-self-hosted-cost-report.md` (self-host).
**What-if volume:** [Cost calculator](/calculator) — drag images/month to recalculate.

## The two approaches, side by side

| | **A. API / HOSTED** | **B. SELF-HOSTED** |
|---|---|---|
| Who owns the GPU | Provider | **We do (rented cloud GPU)** |
| What we pay for | **Each generated image/clip** | **GPU time** — per active second (serverless) or 24×7 (pod) |
| Infrastructure work | None | We set up and maintain the server, scaling, safety filters and model updates |
| Model weights | Closed, not downloadable | **Open weights we download and run** |
| Licence | Commercial rights included in the price | Depends on the weight — Apache 2.0 free, or paid licence, or revenue-capped |
| Quality today | **Higher** (Gemini 3 Pro Image, Veo 3.1) | Lower — open weights trail on prompt adherence |
| Fine-tuning / LoRA | Not available | **Available** |
| Data residency | Data leaves our network | **Stays in our network** |
| Cost at 10k images/mo | ₹63,650 | **₹364** (serverless) or ₹54,787 (24×7 pod) |

## Inside Group B: Serverless vs Pod

If we self-host, we still choose **how** we rent the GPU. This changes the bill by up to 150×.

> Full plain-language explanation, worked examples and the model-load-time calculation: **[`07-explainer-pod-vs-serverless.md`](./07-explainer-pod-vs-serverless)**

| | **B1. SERVERLESS** (RunPod, Modal) | **B2. POD / VM** (RunPod Pod, HF Endpoints, AWS, Alibaba EAS) |
|---|---|---|
| We pay for | Only the seconds the GPU is generating | Every second the machine exists, 24×7 |
| Bill shape | Varies with volume | **Fixed** — same at 10 images or 500,000 |
| Hourly rate | Higher (L40S $1.75/hr RunPod) | Lower (L40S $0.79/hr RunPod Community) |
| Cheaper when | GPU busy **under ~45%** of the month | GPU busy **over ~45%** of the month |
| Our situation | **Under 1% busy** → this one | Not close |

### Platform rates (validated Sep 3, 2026)

Same GPU class, different landlords. Only public rates we opened and confirmed are used in the calculator.

| Platform | Billing shape | Scale to 0? | L4 $/hr | L40S $/hr | ₹/mo @ 10k imgs (gen-only / fixed) | Source |
|---|---|---|---|---|---|---|
| **RunPod** | Serverless + Pods | Yes (flex) | **$0.69** serverless | **$1.75** serverless · $0.79 pod | **₹364** / ₹54,787 | [runpod.io/pricing](https://www.runpod.io/pricing) · browser OK |
| **Modal** | Serverless only | Yes | **$0.80** ($0.000222/s) | **$1.95** ($0.000542/s) | **₹422** / — | [modal.com/pricing](https://modal.com/pricing) · browser OK |
| **Hugging Face Endpoints** | Dedicated replica (pod) | Optional | **$0.70** GCP · $0.80 AWS | **$1.80** AWS | **₹48,545** (GCP L4 min=1) | [HF pricing](https://huggingface.co/docs/inference-endpoints/en/pricing) · browser OK |
| **AWS EC2** | Always-on VM | No | **$0.80** g6.xlarge | — | **₹55,813** | [AWS on-demand](https://aws.amazon.com/ec2/pricing/on-demand/) |
| **Alibaba PAI-EAS** | Always-on pay-as-you-go | No* | **console only** | console only | — not in calc | [EAS billing](https://help.aliyun.com/en/pai/product-overview/billing-of-eas) |

\*Alibaba serverless exists **only for SDWebUI**, not custom FLUX. GPU unit prices are region-specific and shown in the PAI console — we refuse to invent a $/hr.

**Team takeaway:** Hugging Face and Alibaba are both **pod-class** for our use case. At 10k images/month they cost roughly the same order as a RunPod Pod (~₹50k), not RunPod Serverless (~₹400). Live filter: [calculator](/calculator).

**At our volumes:**

| Volume | GPU actually working | Serverless L4 | Pod L40S (fixed) | Hosted API |
|---|---|---|---|---|
| 10,000 images/month | 5.6 of 730 hours (under 1%) | **₹364** * | ₹54,787 | ₹63,650 |
| 100,000 images/month | 55.6 hours (8%) | **₹3,642** * | ₹54,787 | ₹6,36,500 |
| 1,000 video clips/month | 150 hours (21%) | ₹9,832 * | ₹54,787 | ₹71,250 |

**\* Warning — these serverless figures assume zero model load time.** Serverless also bills the time the GPU spends loading the 10–20 GB model file before it can generate anything, plus a short idle wait afterwards. Loaded once per wake-up, this is cheap if we generate many images per wake-up and expensive if we generate one at a time.

| 10,000 images/month, RunPod Serverless L4 | ₹ / month |
|---|---|
| Ignoring model load time (the figure above) | ₹364 |
| **Best case** — snapshot restore, batches of 100, 60 s idle window | **₹480** |
| **Well tuned** — weights cached, batches of 50, 60 s idle window | **₹656** |
| **Naive** — platform defaults, one image per wake-up, weights downloaded each boot | **₹14,020** |

Self-hosting still beats the hosted API even in the naive case, but the margin drops from ~175× to about 4.5×. The fixes are standard and mostly free — bake the weights into the container image, turn on FlashBoot or Modal GPU snapshots, and set a 60 s idle timeout. **These are mandatory if we pilot self-hosting, not optimisations.** Seven techniques with costs, and a live what-if: [`07-explainer-pod-vs-serverless.md`](./07-explainer-pod-vs-serverless) · [calculator](/calculator).

**Conclusion:** serverless wins at every volume we plan for. We would need roughly **half a million images a month** before an always-on Pod saves money.

---

# TABLE 1 — IMAGE

Volume basis: **10,000 generated images/month**, 1024²-class high quality. Self-host assumes FLUX.2 klein 4B at 2 s/image.

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Licence / commercial</th>
      <th>Hardware we must run</th>
      <th>Deploy &amp; billing unit</th>
      <th>Rate</th>
      <th>₹ / image</th>
      <th>Images per ₹100</th>
      <th>₹ / month @ 10k</th>
      <th>Verdict</th>
    </tr>
  </thead>
  <tbody>
    <tr><td colspan="9" style="background:#0b5394;color:#ffffff"><b>GROUP A — API / HOSTED</b> · no GPUs, no ops · billed <b>per generated image</b> · commercial rights included</td></tr>
    <tr style="background:#eaf2fb">
      <td><b>Gemini 3 Pro Image</b> 2K<br/>(Nano Banana Pro)</td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per image</b> (Batch tier)</td>
      <td>$0.067 / img</td>
      <td><b>₹6.37</b></td>
      <td><b>15</b></td>
      <td><b>₹63,650</b></td>
      <td><b>PRIMARY</b> — best prompt adherence, 14 reference images</td>
    </tr>
    <tr style="background:#eaf2fb">
      <td><b>GPT Image 2</b> 1024² High</td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per image</b> (Batch tier)</td>
      <td>$0.106 / img</td>
      <td>₹10.07</td>
      <td>9</td>
      <td>₹1,00,700</td>
      <td>Alternate — text-in-image, hardest prompts</td>
    </tr>
    <tr style="background:#eaf2fb">
      <td><b>FLUX.2 [pro]</b></td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per image</b></td>
      <td>$0.030 / img</td>
      <td>₹2.85</td>
      <td>35</td>
      <td>₹28,500</td>
      <td>Alternate — cheapest hosted, zero infra work</td>
    </tr>
    <tr><td colspan="9" style="background:#38761d;color:#ffffff"><b>GROUP B — SELF-HOSTED (open weights on our rented GPUs)</b> · billed <b>per GPU-time</b>, not per image · we own the serving stack</td></tr>
    <tr style="background:#eefaee">
      <td rowspan="3"><b>FLUX.2 [klein] 4B</b><br/><i>open weights, 4B params</i></td>
      <td rowspan="3"><b>Apache 2.0</b><br/>free commercial<br/>fine-tune + LoRA allowed</td>
      <td rowspan="3">~13 GB VRAM<br/><b>L4 24GB</b> minimum<br/>L40S 48GB comfortable<br/>~10–20 GB weights</td>
      <td>RunPod <b>Serverless</b> L4<br/><b>per active second</b>, scale-to-zero</td>
      <td>$0.69 / hr</td>
      <td><b>₹0.036</b></td>
      <td><b>~2,750</b></td>
      <td><b>₹364</b></td>
      <td><b>BEST SELF-HOST</b> — 175× cheaper than hosted <i>if</i> quality clears bar</td>
    </tr>
    <tr style="background:#eefaee">
      <td>Modal <b>Serverless</b> L4<br/><b>per active second</b>, scale-to-zero</td>
      <td>$0.80 / hr</td>
      <td>₹0.042</td>
      <td>~2,370</td>
      <td>₹422</td>
      <td>Equivalent — better developer experience</td>
    </tr>
    <tr style="background:#eefaee">
      <td>RunPod <b>Serverless</b> L40S<br/><b>per active second</b>, scale-to-zero</td>
      <td>$1.75 / hr</td>
      <td>₹0.092</td>
      <td>~1,080</td>
      <td>₹924</td>
      <td>Use if L4 too slow, or for 9B later</td>
    </tr>
    <tr style="background:#fdf3e6">
      <td rowspan="2"><b>FLUX.2 [klein] 4B</b><br/><i>same model, always-on</i></td>
      <td rowspan="2">Apache 2.0</td>
      <td rowspan="2">L40S 48GB / L4 24GB</td>
      <td>RunPod <b>Pod</b> L40S Community<br/><b>24×7 wall-clock</b></td>
      <td>$0.79 / hr</td>
      <td>₹5.48</td>
      <td>18</td>
      <td>₹54,787 <i>fixed</i></td>
      <td>REJECTED at our volume — beats hosted only above 8,600 img/mo, but <b>serverless beats it until ~593,000 img/mo</b></td>
    </tr>
    <tr style="background:#fdf3e6">
      <td>AWS <b>g6.xlarge</b> L4<br/><b>24×7 wall-clock</b></td>
      <td>$0.8048 / hr</td>
      <td>₹5.58</td>
      <td>18</td>
      <td>₹55,813 <i>fixed</i></td>
      <td>REJECTED — L4 serverless is cheaper <b>per hour too</b> ($0.69 vs $0.8048), so this pod never wins; keep only for Mumbai data residency</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>FLUX.2 [klein] 9B</b><br/><b>FLUX.2 [dev]</b> 32B</td>
      <td><b>FLUX Non-Commercial</b><br/>paid BFL licence required<br/><b>price not published</b></td>
      <td>9B: ~29 GB (15 GB FP8)<br/>dev: ~64 GB (32 GB FP8) → H100</td>
      <td>Blocked until licensed</td>
      <td>—</td><td>—</td><td>—</td><td>—</td>
      <td>REJECTED for now — get BFL quote before any GPU spend</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Stable Diffusion 3.5 Large</b></td>
      <td>Stability Community<br/>free only under <b>$1M/yr revenue</b></td>
      <td>~16–18 GB FP16</td>
      <td>Serverless or pod</td>
      <td>—</td><td>—</td><td>—</td><td>—</td>
      <td>REJECTED — below FLUX.2 quality, licence risk as we grow</td>
    </tr>
  </tbody>
</table>

**Image decision:** stay on **Gemini 3 Pro Image batch (Group A)** for production. Pilot **FLUX.2 klein 4B on RunPod Serverless L4 (Group B1)** for bulk and draft volume. **Do not provision a 24×7 pod (B2)** — serverless is cheaper until roughly 593,000 images/month, far beyond our plan. Blocking unknown: klein 4B's usable-output rate on our DNA prompts.

---

# TABLE 2 — VIDEO

**Every row below delivers the same thing: one clip that is 5 seconds long.** Costs are normalised to that 5-second output so the rows are directly comparable. Volume basis: **1,000 such clips per month**.

Two different durations appear in this table — do not confuse them:

- **Clip length = 5 s** — the length of the video we get. Fixed for every row.
- **Generation time** — how long the GPU works to produce those 5 seconds. Seconds on a hosted API, **5–9 minutes** on self-hosted Wan 2.2. This is what self-host cost is billed against.

Every cost column is anchored to that same 5-second clip: `₹ per clip` is one 5 s clip, `5 s clips per ₹1,000` counts 5 s clips, and `₹ / month` is **1,000 × the 5 s clip price**. The only exception is the always-on Pod row, where the monthly figure is a fixed rental and the per-clip price is derived by dividing it across 1,000 clips.

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Licence / commercial</th>
      <th>Hardware we must run</th>
      <th>Deploy &amp; billing unit</th>
      <th>Rate</th>
      <th>Output<br/><b>(clip length 5 s)</b></th>
      <th>Generation<br/>time per clip</th>
      <th>₹ per clip<br/><b>(5 s of video)</b></th>
      <th>5 s clips<br/>per ₹1,000</th>
      <th>₹ / month<br/>@ 1,000 clips<br/><b>(5 s each)</b></th>
      <th>Verdict</th>
    </tr>
  </thead>
  <tbody>
    <tr><td colspan="11" style="background:#0b5394;color:#ffffff"><b>GROUP A — API / HOSTED</b> · no GPUs, no ops · billed <b>per second of finished video</b> · commercial rights included</td></tr>
    <tr style="background:#eaf2fb">
      <td><b>Veo 3.1 Fast</b> (Google)</td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per second of video</b></td>
      <td>$0.15 / video-sec</td>
      <td><b>1080p + native audio</b><br/>max clip 8 s, extendable</td>
      <td>seconds</td>
      <td><b>₹71.25</b><br/><i>5 × $0.15</i></td>
      <td><b>14</b></td>
      <td><b>₹71,250</b></td>
      <td><b>PRIMARY</b> — best quality/price with audio</td>
    </tr>
    <tr style="background:#eaf2fb">
      <td><b>Kling 3.0 Pro</b> (Kuaishou)</td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per second of video</b></td>
      <td>$0.168 / video-sec</td>
      <td>1080p + audio<br/>max clip 15 s</td>
      <td>seconds</td>
      <td>₹79.80<br/><i>5 × $0.168</i></td>
      <td>12</td>
      <td>₹79,800</td>
      <td>Alternate — longer clips available, strong motion</td>
    </tr>
    <tr style="background:#eaf2fb">
      <td><b>Veo 3.1 Standard</b></td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per second of video</b></td>
      <td>$0.40 / video-sec</td>
      <td>1080p + audio, top fidelity<br/>max clip 8 s</td>
      <td>seconds</td>
      <td>₹190.00<br/><i>5 × $0.40</i></td>
      <td>5</td>
      <td>₹1,90,000</td>
      <td>Hero / final assets only</td>
    </tr>
    <tr><td colspan="11" style="background:#38761d;color:#ffffff"><b>GROUP B — SELF-HOSTED (open weights on our rented GPUs)</b> · billed <b>per GPU-second of generation time</b>, not per second of video · we own the serving stack</td></tr>
    <tr style="background:#fdf3e6">
      <td rowspan="3"><b>Wan 2.2 TI2V-5B</b><br/><i>open weights, 5B params</i><br/>native output <b>5 s @ 720p 24fps</b></td>
      <td rowspan="3"><b>Apache 2.0</b><br/>free commercial<br/>we own outputs</td>
      <td rowspan="3">≥24 GB VRAM w/ offload<br/><b>L40S 48GB</b> recommended<br/>~20–40 GB weights</td>
      <td>RunPod <b>Serverless</b> L40S<br/><b>per active GPU-second</b></td>
      <td>$1.75 / GPU-hr</td>
      <td><b>720p, NO audio</b><br/>max clip 5 s</td>
      <td><b>~300 s</b> (5 min)<br/><i>optimistic</i></td>
      <td>₹13.9<br/><i>300 GPU-s × $1.75/hr</i></td>
      <td>72</td>
      <td>₹13,854</td>
      <td>CONDITIONAL — cheapest, but far below Veo on quality</td>
    </tr>
    <tr style="background:#fdf3e6">
      <td>RunPod <b>Serverless</b> L40S<br/><b>per active GPU-second</b></td>
      <td>$1.75 / GPU-hr</td>
      <td>720p, no audio<br/>max clip 5 s</td>
      <td><b>~540 s</b> (9 min)<br/><i>conservative</i></td>
      <td>₹24.9<br/><i>540 GPU-s × $1.75/hr</i></td>
      <td>40</td>
      <td>₹24,938</td>
      <td>CONDITIONAL — conservative speed case</td>
    </tr>
    <tr style="background:#fdf3e6">
      <td>Modal <b>Serverless</b> L40S<br/><b>per active GPU-second</b></td>
      <td>$1.95 / GPU-hr</td>
      <td>720p, no audio<br/>max clip 5 s</td>
      <td><b>~540 s</b> (9 min)</td>
      <td>₹27.8<br/><i>540 GPU-s × $1.95/hr</i></td>
      <td>36</td>
      <td>₹27,788</td>
      <td>CONDITIONAL — same trade-off</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Wan 2.2 TI2V-5B</b><br/><i>same model, always-on</i></td>
      <td>Apache 2.0</td>
      <td>L40S 48GB</td>
      <td>RunPod <b>Pod</b> L40S Community<br/><b>24×7 wall-clock</b></td>
      <td>$0.79 / hr</td>
      <td>720p, no audio<br/>max clip 5 s</td>
      <td>~540 s</td>
      <td>₹54.8<br/><i>fixed ÷ 1,000 clips</i></td>
      <td>18</td>
      <td>₹54,787 <i>fixed</i></td>
      <td>REJECTED — near Veo cost for far worse output</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Wan 2.2 A14B</b></td>
      <td>Apache 2.0</td>
      <td>~80 GB → H100 class</td>
      <td>Serverless H100 $4.79/GPU-hr</td>
      <td>—</td>
      <td>720p, no audio</td>
      <td>not benchmarked</td>
      <td>—</td><td>—</td><td>—</td>
      <td>REJECTED — not benchmarked, H100 cost kills it</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Wan 2.6 / 2.7</b></td>
      <td>API-only — <b>no open weights published</b></td>
      <td><i>Cannot self-host</i></td>
      <td>Not available as weights</td>
      <td>—</td>
      <td>max clip 15 s (via API only)</td>
      <td>—</td>
      <td>—</td><td>—</td><td>—</td>
      <td>REJECTED — self-hosting impossible</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Sora 2</b> (Group A)</td>
      <td>Hosted API</td>
      <td><b>None</b></td>
      <td>API</td>
      <td>—</td><td>—</td><td>—</td><td>—</td><td>—</td><td>—</td>
      <td>REJECTED — API sunsets <b>Sep 24, 2026</b></td>
    </tr>
  </tbody>
</table>

**How to read a Group A vs Group B cost:** Veo 3.1 Fast bills the 5 seconds of video we receive → 5 × $0.15 = $0.75 = ₹71.25. Wan 2.2 bills the ~300–540 seconds the GPU spent producing those same 5 seconds → 300 × ($1.75 ÷ 3600) = $0.146 = ₹13.9. Same 5-second deliverable, two completely different billing units.

**If we needed 10-second clips instead:** every Group A figure doubles (billed per video second). Group B does not simply double — Wan 2.2 TI2V-5B caps at 5 s natively, so a 10 s output needs two generations plus stitching, or a different model.

**Video decision:** stay fully on **Group A / Veo 3.1 Fast**. Self-hosting saves money on paper but delivers 720p without audio at minutes-per-clip latency, and on a 24×7 pod the saving disappears entirely. Revisit only when an open model ships 1080p with native audio.

---

## Open blockers before spend is committed

| Blocker | Blocks | Action |
|---|---|---|
| klein 4B usable-output rate on DNA prompts | The entire Group B image case | Run in `02-model-evaluation.md` |
| Measured seconds/image on our stack | Every Group B ₹ figure scales with it | Benchmark L4 and L40S |
| BFL licence price for klein 9B / [dev] | Any Group B quality upgrade | Request quote from BFL sales |
| Cold-start seconds on serverless | Adds ~5–15% to Group B cost | Measure after first deploy |

## Price sources

**Group A:** Gemini [ai.google.dev/gemini-api/docs/pricing](https://ai.google.dev/gemini-api/docs/pricing) · OpenAI [platform.openai.com/docs/pricing](https://platform.openai.com/docs/pricing) · BFL [bfl.ai/pricing](https://bfl.ai/pricing) · Veo/Kling [fal.ai/pricing](https://fal.ai/pricing)

**Group B:** RunPod [runpod.io/pricing](https://www.runpod.io/pricing) · Modal [modal.com/pricing](https://modal.com/pricing) · AWS [on-demand](https://aws.amazon.com/ec2/pricing/on-demand/) · Licences: [bfl.ai/licensing](https://bfl.ai/licensing) · [stability.ai/license](https://stability.ai/license) · [github.com/Wan-Video/Wan2.2](https://github.com/Wan-Video/Wan2.2)
