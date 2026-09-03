# Decision Doc — Image & Video Generation

Verified Sep 3, 2026 · 1 USD = ₹95.00 · Two tables, one decision each.
Detail and workings: `03-research-report.md` (API/hosted) and `05-self-hosted-cost-report.md` (self-host).

## The two approaches, side by side

| | **A. API / HOSTED** | **B. SELF-HOSTED** |
|---|---|---|
| Who owns the GPU | Provider | **We do (rented cloud GPU)** |
| What we pay for | **Each generated image/clip** | **GPU time** — per active second (serverless) or 24×7 (pod) |
| Infrastructure work | None | Serving stack, queue, autoscale, safety filters, upgrades |
| Model weights | Closed, not downloadable | **Open weights we download and run** |
| Licence | Commercial rights included in the price | Depends on the weight — Apache 2.0 free, or paid licence, or revenue-capped |
| Quality today | **Higher** (Gemini 3 Pro Image, Veo 3.1) | Lower — open weights trail on prompt adherence |
| Fine-tuning / LoRA | Not available | **Available** |
| Data residency | Data leaves our network | **Stays in our network** |
| Cost at 10k images/mo | ₹63,650 | **₹364** (serverless) or ₹54,787 (24×7 pod) |

**Billing rule that decides everything in group B:** *Serverless* bills only active generation seconds. *Pods/VMs* bill wall-clock 24×7 whether or not you generate. At 10k images/month a pod sits **0.76% utilised** — 5.6 GPU-hours of real work against 730 hours billed.

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
      <td>CONDITIONAL — only above <b>8,600 img/mo</b>, else hosted is cheaper</td>
    </tr>
    <tr style="background:#fdf3e6">
      <td>AWS <b>g6.xlarge</b> L4<br/><b>24×7 wall-clock</b></td>
      <td>$0.8048 / hr</td>
      <td>₹5.58</td>
      <td>18</td>
      <td>₹55,813 <i>fixed</i></td>
      <td>CONDITIONAL — above <b>8,800 img/mo</b>; Mumbai region available</td>
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

**Image decision:** stay on **Gemini 3 Pro Image batch (Group A)** for production. Pilot **FLUX.2 klein 4B on RunPod Serverless L4 (Group B)** for bulk and draft volume. Do not buy a 24×7 pod below ~8,600 images/month. Blocking unknown: klein 4B's usable-output rate on our DNA prompts.

---

# TABLE 2 — VIDEO

Volume basis: **1,000 generated 5-second clips/month**. Self-host assumes Wan 2.2 TI2V-5B.

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Licence / commercial</th>
      <th>Hardware we must run</th>
      <th>Deploy &amp; billing unit</th>
      <th>Rate</th>
      <th>Output quality</th>
      <th>₹ / 5s clip</th>
      <th>Clips per ₹1,000</th>
      <th>₹ / month @ 1k</th>
      <th>Verdict</th>
    </tr>
  </thead>
  <tbody>
    <tr><td colspan="10" style="background:#0b5394;color:#ffffff"><b>GROUP A — API / HOSTED</b> · no GPUs, no ops · billed <b>per second of video</b> · commercial rights included</td></tr>
    <tr style="background:#eaf2fb">
      <td><b>Veo 3.1 Fast</b> (Google)</td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per video second</b></td>
      <td>$0.15 / s</td>
      <td><b>1080p + native audio</b><br/>seconds latency</td>
      <td><b>₹71.25</b></td>
      <td><b>14</b></td>
      <td><b>₹71,250</b></td>
      <td><b>PRIMARY</b> — best quality/price with audio</td>
    </tr>
    <tr style="background:#eaf2fb">
      <td><b>Kling 3.0 Pro</b> (Kuaishou)</td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per video second</b></td>
      <td>$0.168 / s</td>
      <td>1080p + audio, up to 15s</td>
      <td>₹79.80</td>
      <td>12</td>
      <td>₹79,800</td>
      <td>Alternate — longer clips, strong motion</td>
    </tr>
    <tr style="background:#eaf2fb">
      <td><b>Veo 3.1 Standard</b></td>
      <td>Hosted API — full commercial</td>
      <td><b>None</b> — provider's GPUs</td>
      <td>API · <b>per video second</b></td>
      <td>$0.40 / s</td>
      <td>1080p + audio, top fidelity</td>
      <td>₹190.00</td>
      <td>5</td>
      <td>₹1,90,000</td>
      <td>Hero / final assets only</td>
    </tr>
    <tr><td colspan="10" style="background:#38761d;color:#ffffff"><b>GROUP B — SELF-HOSTED (open weights on our rented GPUs)</b> · billed <b>per GPU-time</b>, not per clip · we own the serving stack</td></tr>
    <tr style="background:#fdf3e6">
      <td rowspan="3"><b>Wan 2.2 TI2V-5B</b><br/><i>open weights, 5B params</i></td>
      <td rowspan="3"><b>Apache 2.0</b><br/>free commercial<br/>we own outputs</td>
      <td rowspan="3">≥24 GB VRAM w/ offload<br/><b>L40S 48GB</b> recommended<br/>~20–40 GB weights</td>
      <td>RunPod <b>Serverless</b> L40S<br/><b>per active second</b></td>
      <td>$1.75 / hr</td>
      <td><b>720p, NO audio</b><br/>~5 min latency</td>
      <td>₹13.9 <i>(300s)</i></td>
      <td>72</td>
      <td>₹13,854</td>
      <td>CONDITIONAL — cheapest, but far below Veo on quality</td>
    </tr>
    <tr style="background:#fdf3e6">
      <td>RunPod <b>Serverless</b> L40S<br/><b>per active second</b></td>
      <td>$1.75 / hr</td>
      <td>720p, no audio<br/>~9 min latency</td>
      <td>₹24.9 <i>(540s)</i></td>
      <td>40</td>
      <td>₹24,938</td>
      <td>CONDITIONAL — conservative speed case</td>
    </tr>
    <tr style="background:#fdf3e6">
      <td>Modal <b>Serverless</b> L40S<br/><b>per active second</b></td>
      <td>$1.95 / hr</td>
      <td>720p, no audio</td>
      <td>₹27.8 <i>(540s)</i></td>
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
      <td>720p, no audio</td>
      <td>₹54.8</td>
      <td>18</td>
      <td>₹54,787 <i>fixed</i></td>
      <td>REJECTED — near Veo cost for far worse output</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Wan 2.2 A14B</b></td>
      <td>Apache 2.0</td>
      <td>~80 GB → H100 class</td>
      <td>Serverless H100 $4.79/hr</td>
      <td>—</td>
      <td>720p, no audio</td>
      <td>—</td><td>—</td><td>—</td>
      <td>REJECTED — not benchmarked, H100 cost kills it</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Wan 2.6 / 2.7</b></td>
      <td>API-only — <b>no open weights published</b></td>
      <td><i>Cannot self-host</i></td>
      <td>Not available as weights</td>
      <td>—</td>
      <td>up to 15s</td>
      <td>—</td><td>—</td><td>—</td>
      <td>REJECTED — self-hosting impossible</td>
    </tr>
    <tr style="background:#fdeeee">
      <td><b>Sora 2</b> (Group A)</td>
      <td>Hosted API</td>
      <td><b>None</b></td>
      <td>API</td>
      <td>—</td><td>—</td><td>—</td><td>—</td><td>—</td>
      <td>REJECTED — API sunsets <b>Sep 24, 2026</b></td>
    </tr>
  </tbody>
</table>

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
