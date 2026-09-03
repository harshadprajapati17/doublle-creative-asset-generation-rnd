---
layout: ../layouts/Layout.astro
title: Research Report
---

# AI Image & Video Generation — Research Report

Verified: Sep 2, 2026. Prices are provider list rates from the official pages linked inside each table row.
Currency: **1 USD = ₹95.00** (Sep 2, 2026). Rupee figures are rounded.

## Pipeline Context

Our existing **Model DNA generator** outputs a structured prompt. This report covers the stage after that: prompt → image (→ optional video).

Implications for model choice:

- **Prompt adherence is the primary selection criterion** — DNA prompts are long and attribute-dense, so the model must honour every attribute, not just the overall vibe.
- **Reference-image conditioning matters more than fine-tuning** — subject/product consistency across a DNA set comes from passing the approved still as a reference, not from training.
- **Batch tiers may halve the cost on Google and OpenAI**, but only if a generation can wait rather than returning immediately. *Assumption to confirm:* whether the DNA generator's output can tolerate a delayed image. If every image must come back in real time, batch pricing does not apply and all Google/OpenAI figures below should be read at the Standard rate, not the Batch rate.
- **Text input cost is negligible.** A ~300-token DNA prompt costs ~₹0.14 on GPT Image 2 and ~₹0.06 on Gemini 3 Pro Image. Image output tokens dominate; ignore prompt cost in budgeting.

---

# Image

## Capability Shortlist

| Model | Provider | Max res | Reference images | Editing | Notes |
|---|---|---|---|---|---|
| Gemini 3 Pro Image (Nano Banana Pro) | Google | 4K | Up to 14 per prompt | Yes, multi-turn | Best all-round for reference-driven consistency |
| GPT Image 2 | OpenAI | 1536px | Yes | Yes | Best prompt adherence and text rendering; slowest (~4.2s) |
| FLUX.2 [pro] | Black Forest Labs | 2K | Yes | Yes | Open weights; self-hostable and fine-tunable |
| Nano Banana 2 | Google | 4K | Yes | Yes | Fastest (~0.85s); official rate not published |
| Seedream 5.0 | ByteDance / BytePlus | 4K | Yes | Yes | Cheap photorealism; mostly via aggregators |

## Cost — Image

**Scope note:** only high-quality / high-resolution tiers are listed. Low-quality and sub-1K tiers (GPT Image 2 low at $0.006, FLUX.1 [schnell] at $0.003) are excluded — they do not meet our output bar and are only worth using for throwaway prototyping.

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Tier</th>
      <th>$ / image</th>
      <th>₹ / image</th>
      <th>Images per ₹100</th>
      <th>Monthly ₹ @ 10,000 images</th>
      <th>Official price link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="4"><b>Gemini 3 Pro Image</b><br/>(Nano Banana Pro)</td>
      <td>2K — Standard</td>
      <td>$0.134</td>
      <td>₹12.73</td>
      <td><b>7</b></td>
      <td>₹1,27,300</td>
      <td rowspan="4"><a href="https://ai.google.dev/gemini-api/docs/pricing">ai.google.dev/gemini-api/docs/pricing</a></td>
    </tr>
    <tr>
      <td>2K — Batch</td>
      <td>$0.067</td>
      <td>₹6.37</td>
      <td><b>15</b></td>
      <td>₹63,650</td>
    </tr>
    <tr>
      <td>4K — Standard</td>
      <td>$0.240</td>
      <td>₹22.80</td>
      <td><b>4</b></td>
      <td>₹2,28,000</td>
    </tr>
    <tr>
      <td>4K — Batch</td>
      <td>$0.120</td>
      <td>₹11.40</td>
      <td><b>8</b></td>
      <td>₹1,14,000</td>
    </tr>
    <tr>
      <td rowspan="4"><b>GPT Image 2</b></td>
      <td>1024×1024 High — Standard</td>
      <td>$0.211</td>
      <td>₹20.05</td>
      <td><b>4</b></td>
      <td>₹2,00,450</td>
      <td rowspan="4"><a href="https://platform.openai.com/docs/pricing">platform.openai.com/docs/pricing</a></td>
    </tr>
    <tr>
      <td>1024×1024 High — Batch</td>
      <td>$0.106</td>
      <td>₹10.07</td>
      <td><b>9</b></td>
      <td>₹1,00,700</td>
    </tr>
    <tr>
      <td>1024×1536 High — Standard</td>
      <td>$0.165</td>
      <td>₹15.68</td>
      <td><b>6</b></td>
      <td>₹1,56,750</td>
    </tr>
    <tr>
      <td>1024×1536 High — Batch</td>
      <td>$0.083</td>
      <td>₹7.89</td>
      <td><b>12</b></td>
      <td>₹78,850</td>
    </tr>
    <tr>
      <td><b>FLUX.2 [pro]</b></td>
      <td>up to 2K</td>
      <td>$0.030</td>
      <td>₹2.85</td>
      <td><b>35</b></td>
      <td>₹28,500</td>
      <td><a href="https://docs.bfl.ai/quick_start/pricing">docs.bfl.ai/quick_start/pricing</a></td>
    </tr>
    <tr>
      <td><b>Nano Banana 2</b><br/><i>(rate unverified)</i></td>
      <td>up to 2K</td>
      <td>~$0.075</td>
      <td>~₹7.13</td>
      <td><b>~14</b></td>
      <td>~₹71,250</td>
      <td><a href="https://ai.google.dev/gemini-api/docs/pricing">ai.google.dev/gemini-api/docs/pricing</a></td>
    </tr>
    <tr>
      <td><b>Seedream 5.0</b><br/><i>(rate unverified)</i></td>
      <td>up to 2K</td>
      <td>~$0.030</td>
      <td>~₹2.85</td>
      <td><b>~35</b></td>
      <td>~₹28,500</td>
      <td><a href="https://docs.byteplus.com/en/docs/ModelArk/Pricing">docs.byteplus.com/en/docs/ModelArk/Pricing</a></td>
    </tr>
  </tbody>
</table>

**Reading the table:** "Images per ₹100" and "Monthly ₹" count **generated** images, not accepted ones. Multiply by the retry factor for real cost — at a 50% usable rate you generate 2 images per accepted one, so double the monthly figure. That rate is the single biggest lever in the budget and must be measured against our DNA prompts in `02-model-evaluation.md`.

**Token math behind the numbers:** Google bills image output at $120 / 1M tokens — a 1K/2K image is 1,120 tokens ($0.134), a 4K image is 2,000 tokens ($0.24). OpenAI bills image output at $30 / 1M tokens; the per-image figures above are OpenAI's own published output examples. Batch mode is a flat 50% discount on both.

---

# Video

## Capability Shortlist

| Model | Provider | Max clip | Res | Native audio | Reference support |
|---|---|---|---|---|---|
| Veo 3.1 | Google | 8s, extendable | 4K | Yes | Image-to-video, storyboard interpolation |
| Kling 3.0 Pro | Kuaishou | 15s | 1080p / 4K | Yes | Image-to-video, multi-shot |
| Seedance 2.0 | ByteDance | 15s | 720p | Yes (always on) | Up to 9 images + 3 videos + 3 audio |
| Runway Gen-4.5 | Runway | ~10s, extendable | 4K export | Yes | Character references, motion brush |

Excluded: **Sora 2** — API sunsets Sep 24, 2026. Do not build on it.

## Cost — Video

**Scope note:** all rows are **5-second clips with audio on**, at 1080p or above. Sub-1080p tiers are excluded except Seedance 2.0, which caps at 720p natively. Veo 3.1 Lite (720p, $0.05/s) is excluded for the same quality reason but remains the cheapest fallback for internal drafts. Clip affordability is shown per ₹1,000 because a single clip already costs more than ₹100.

<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>Tier</th>
      <th>$ / sec</th>
      <th>$ / 5s clip</th>
      <th>₹ / 5s clip</th>
      <th>Clips per ₹1,000</th>
      <th>Monthly ₹ @ 1,000 clips</th>
      <th>Official price link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="4"><b>Veo 3.1</b><br/>(Google)</td>
      <td>Standard 4K</td>
      <td>$0.600</td>
      <td>$3.00</td>
      <td>₹285.00</td>
      <td><b>3</b></td>
      <td>₹2,85,000</td>
      <td rowspan="4"><a href="https://fal.ai/models/fal-ai/veo3.1">fal.ai/models/fal-ai/veo3.1</a></td>
    </tr>
    <tr>
      <td>Standard 1080p</td>
      <td>$0.400</td>
      <td>$2.00</td>
      <td>₹190.00</td>
      <td><b>5</b></td>
      <td>₹1,90,000</td>
    </tr>
    <tr>
      <td>Fast 4K</td>
      <td>$0.350</td>
      <td>$1.75</td>
      <td>₹166.25</td>
      <td><b>6</b></td>
      <td>₹1,66,250</td>
    </tr>
    <tr>
      <td>Fast 1080p</td>
      <td>$0.150</td>
      <td>$0.75</td>
      <td>₹71.25</td>
      <td><b>14</b></td>
      <td>₹71,250</td>
    </tr>
    <tr>
      <td rowspan="2"><b>Kling 3.0 Pro</b><br/>(Kuaishou)</td>
      <td>1080p + audio</td>
      <td>$0.168</td>
      <td>$0.84</td>
      <td>₹79.80</td>
      <td><b>12</b></td>
      <td>₹79,800</td>
      <td rowspan="2"><a href="https://fal.ai/models/fal-ai/kling-video/v3/pro/image-to-video">fal.ai/models/fal-ai/kling-video/v3/pro</a></td>
    </tr>
    <tr>
      <td>1080p + voice control</td>
      <td>$0.196</td>
      <td>$0.98</td>
      <td>₹93.10</td>
      <td><b>10</b></td>
      <td>₹93,100</td>
    </tr>
    <tr>
      <td rowspan="2"><b>Seedance 2.0</b><br/>(ByteDance)<br/><i>720p max</i></td>
      <td>Standard 720p</td>
      <td>$0.3024</td>
      <td>$1.51</td>
      <td>₹143.64</td>
      <td><b>6</b></td>
      <td>₹1,43,640</td>
      <td rowspan="2"><a href="https://fal.ai/pricing">fal.ai/pricing</a></td>
    </tr>
    <tr>
      <td>Fast 720p</td>
      <td>$0.2419</td>
      <td>$1.21</td>
      <td>₹114.90</td>
      <td><b>8</b></td>
      <td>₹1,14,900</td>
    </tr>
    <tr>
      <td><b>Runway Gen-4.5</b></td>
      <td>Subscription (Standard)</td>
      <td colspan="2">$12 / month</td>
      <td>₹1,140 / month</td>
      <td>—</td>
      <td>Credit-based, not per-second</td>
      <td><a href="https://runwayml.com/pricing">runwayml.com/pricing</a></td>
    </tr>
  </tbody>
</table>

**Reading the table:** same caveat as images — these are **generated** clips. At a 50% usable rate, double the monthly figure. Turning audio off halves Veo 3.1's rate ($0.20/s instead of $0.40/s at 1080p), so generate silent when audio is added in post.

---

## Blended Production Estimate

10,000 generated images + 1,000 generated 5s clips per month, before retries.

| Stack | Image line | Video line | Monthly total |
|---|---|---|---|
| Budget | FLUX.2 [pro] — ₹28,500 | Veo 3.1 Fast 1080p — ₹71,250 | **₹99,750** |
| Balanced (recommended) | Gemini 3 Pro Image 2K batch — ₹63,650 | Veo 3.1 Fast 1080p — ₹71,250 | **₹1,34,900** |
| Premium | Gemini 3 Pro Image 4K std — ₹2,28,000 | Veo 3.1 Standard 1080p — ₹1,90,000 | **₹4,18,000** |

At a 50% usable rate, double each total. Not included: storage/CDN, aggregator markup, and human review time.

## Techniques to Evaluate

- Reference-image workflows for subject/product consistency (Gemini 3 Pro Image up to 14 refs, Seedance 2.0 up to 9, Runway References)
- Image-to-video: lock an approved still from the DNA prompt, then animate — better consistency and cheaper than text-to-video
- Batch tiers: flat 50% saving on Google and OpenAI, if delayed delivery of an image is acceptable
- Multi-candidate generation with automated selection, since real cost is driven by the usable-output rate
- Multi-model routing via fal.ai / Replicate for benchmarking; move winners to direct provider APIs
- Fine-tuning only on FLUX (open weights); closed models offer reference conditioning instead

## Recommendation → Evaluation

- **Image primary:** Gemini 3 Pro Image (Nano Banana Pro), 2K batch tier — ₹6.37 per image, 15 images per ₹100. Alternates: GPT Image 2 high (prompt adherence and text), FLUX.2 [pro] (cost, self-host, fine-tuning).
- **Video primary:** Veo 3.1 Fast 1080p — ₹71.25 per 5s clip. Alternates: Kling 3.0 Pro (cheapest at 1080p with audio), Seedance 2.0 (reference-heavy multi-shot, 720p ceiling).
- **Access layer:** fal.ai for benchmarking; direct provider APIs in production.

## Uncertain / To Verify

- Nano Banana 2 official per-image rate and model ID — not published on Google's pricing page
- Seedream 5.0 official BytePlus rate
- Kling 3.0 and Seedance 2.0 rate limits and enterprise/volume pricing
- Runway Gen-4.5 credit-to-clip conversion, needed to compare it per-clip
- Actual usable-output rate per model against our DNA prompts — required before any budget is final

## Additional References

Checked and returning HTTP 200 on Sep 2, 2026.

| Source | Link |
|---|---|
| Gemini 3 Pro Image model card | https://ai.google.dev/gemini-api/docs/models/gemini-3-pro-image |
| Google Vertex AI pricing | https://cloud.google.com/vertex-ai/generative-ai/pricing |
| OpenAI developer portal pricing | https://developers.openai.com/api/docs/pricing |
| fal.ai — Veo 3.1 Fast | https://fal.ai/models/fal-ai/veo3.1/fast |
| Replicate pricing | https://replicate.com/pricing |
| Kling API docs | https://app.klingai.com/global/dev/document-api/apiReference/commonInfo |
