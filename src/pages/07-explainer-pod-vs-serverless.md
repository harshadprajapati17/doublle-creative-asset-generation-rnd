# Explainer — Pod vs Serverless, and What Model Load Time Costs Us

Companion to `06-decision-doc.md`. This doc explains the concepts in plain language and shows the full cost calculation. The decision doc carries only the conclusions.

Verified Sep 3, 2026 · 1 USD = ₹95.00

---

# Part 1 — What "self-hosting" actually means

If we use an **API** (Gemini, Veo, GPT Image), we send a prompt and get an image back. We pay per image. Someone else owns the computer.

If we **self-host**, we download a free, open model (like FLUX.2 klein 4B) and run it ourselves on a rented graphics card (GPU) in the cloud. Nobody charges us per image — instead we **rent the GPU by time**.

That single difference — paying per image versus paying per unit of time — is where all the confusion in this project came from.

---

# Part 2 — Pod vs Serverless, in plain English

Once we decide to rent a GPU, there are two ways to rent it — and **every cloud product falls into one of these two buckets**, even when the marketing name sounds different.

| Product the team suggested | Which bucket? | Why |
|---|---|---|
| RunPod Serverless, Modal | **Serverless** | Meter runs only while generating; scale to zero |
| RunPod Pod, AWS EC2, Hugging Face Inference Endpoints (min 1 replica), Alibaba PAI-EAS | **Pod** | Meter runs while the machine/replica exists |
| Alibaba “Serverless” | **Not usable for us** | Official docs: serverless is **SDWebUI only**, not custom FLUX |

Think of renting a car.

- **A Pod is renting a car for the whole month.** You pay the full monthly rental whether you drive every day or leave it parked outside. The daily rate is cheap, but you pay for every day it sits idle.
- **Serverless is taking a taxi.** The per-kilometre rate is higher, but you only pay while you are actually moving. Park it and you pay nothing.

A GPU works the same way:

- A **Pod** is a machine we rent that stays switched on 24 hours a day, every day. The meter runs even when nobody is generating anything.
- **Serverless** switches the machine on when a request arrives, generates the image, then switches off and stops charging.

## The price example

Say we generate **10,000 images in a month**, and each image takes the GPU **2 seconds**.

| Step | Result |
|---|---|
| Total GPU work needed | 10,000 × 2 seconds = **5.6 hours** |
| Hours in a month | **730 hours** |
| So the GPU is actually working | **5.6 out of 730 hours — under 1% of the time** |

Two bills for that identical output:

| Option | What we pay for | The bill |
|---|---|---|
| **Pod** (rent the machine all month) | All **730 hours**, including the ~724 hours it sits idle | **₹54,787** |
| **Serverless** (pay only while generating) | Just the **5.6 hours** of real work | **₹364** |

Same 10,000 images. **₹364 versus ₹54,787.**

## When does a Pod become cheaper?

A Pod's hourly rate is lower, so it wins *if* we keep it busy. The tipping point is simply the pod's hourly rate divided by the serverless rate:

| GPU | Pod rate | Serverless rate | Pod becomes cheaper above | In images per month |
|---|---|---|---|---|
| L40S 48GB | $0.79/hr (Community) | $1.75/hr | 45% busy = 330 hours/month | **~593,000** |
| L40S 48GB | $0.99/hr (Secure) | $1.75/hr | 57% busy = 413 hours/month | **~743,000** |
| L4 24GB | $0.8048/hr (AWS g6) | $0.69/hr (RunPod) | **Never** — serverless is cheaper per hour *as well* | no volume works |

We are at roughly 1% busy. A Pod is not close to worthwhile.

---

# Part 3 — Model load time, and why it changes the price

**This is the part the ₹364 figure ignores, and it matters.**

## What model load time is

The FLUX model file is around 10–20 GB. Before the GPU can generate anything, that file has to be read off disk and loaded into the graphics card's memory, and the software has to warm up. This is called a **cold start**.

Back to the taxi analogy: the taxi doesn't teleport to your door. It has to drive to you first — **and on serverless, the meter is already running during that drive.**

## Serverless charges us for it

Both platforms bill this. RunPod's own pricing documentation states that workers are billed across **three phases**:

| Phase | What happens | Billed? |
|---|---|---|
| 1. Start time | Container starts, model loads into GPU memory | **Yes** |
| 2. Execution | The image is actually generated | **Yes** |
| 3. Idle timeout | Worker stays awake briefly waiting for another request (RunPod default: **5 seconds**) | **Yes** |

Source: [docs.runpod.io/serverless/pricing](https://docs.runpod.io/serverless/pricing)

## The correct formula

The naive calculation we used earlier was just:

```
cost per image = generation seconds × rate per second
```

The real one is:

```
billed seconds per image = (cold start + idle timeout) ÷ images per wake-up  +  generation seconds

cost per image           = billed seconds per image × rate per second
```

The critical term is **images per wake-up** — how many images the GPU generates before it goes back to sleep. Load time is paid **once per wake-up**, so the more images we generate per wake-up, the more thinly that cost is spread.

## How bad can it get?

At **10,000 images/month on RunPod Serverless L4** ($0.69/hr), 2 seconds per image, FLUX.2 klein 4B (12 GB):

| Cold start | Images per wake-up | Billed sec/image | ₹ / image | ₹ / month | vs the ₹364 estimate |
|---|---|---|---|---|---|
| 70 s (weights downloaded fresh) | **1** | 77.0 | ₹1.402 | **₹14,020** | **38× worse** |
| 70 s | 10 | 9.50 | ₹0.173 | ₹1,730 | 4.8× worse |
| 70 s | 100 | 2.75 | ₹0.050 | ₹501 | 1.4× worse |
| 20 s (weights cached on disk) | **1** | 27.0 | ₹0.492 | **₹4,916** | **13.5× worse** |
| 20 s | 10 | 4.50 | ₹0.082 | ₹819 | 2.2× worse |
| 20 s | 100 | 2.25 | ₹0.041 | ₹410 | 1.1× worse |
| 3.5 s (snapshot restore) | **1** | 10.50 | ₹0.191 | **₹1,912** | 5.2× worse |
| 3.5 s | 10 | 2.85 | ₹0.052 | ₹519 | 1.4× worse |
| 3.5 s | 100 | 2.08 | ₹0.038 | ₹380 | ~same |
| *ignoring cold start entirely* | — | 2.00 | ₹0.036 | *₹364* | *the original, optimistic figure* |

### Where the load-time numbers come from

Load time scales with model size — the weights have to be copied into GPU memory:

```
load seconds = fixed container overhead + model size in GB ÷ transfer speed
```

| Weights come from | Overhead | Speed | A 12 GB model | A 64 GB model |
|---|---|---|---|---|
| Downloaded on every boot | 10 s | 0.2 GB/s | **70 s** | 330 s |
| Cached on volume / baked into image | 8 s | 1 GB/s | **20 s** | 72 s |
| Memory snapshot restore | 2 s | 8 GB/s | **3.5 s** | 10 s |

These are estimates calibrated against published figures (FLUX cold starts of 60+ seconds without caching; Modal reporting up to 10× faster starts with GPU snapshots). **They need to be replaced with a measurement from our own deployment.**

## Why our traffic pattern is the worst case

10,000 images spread evenly across a month is **13.7 images per hour — one image roughly every 4.4 minutes**.

RunPod's default idle timeout is **5 seconds**. So the worker finishes an image, waits 5 seconds, sees nothing, and shuts down. Four minutes later the next image arrives and **pays the full load time again**.

Left at defaults with one-at-a-time requests, we would land in the **worst row of that table** — ₹12,200/month instead of ₹364.

## How much batching we need

To keep load time under a 10% cost overhead:

| Cold start | Images needed per wake-up |
|---|---|
| 3 s | ~40 |
| 20 s | ~125 |
| 60 s | ~325 |

---

# Part 4 — What teams do about it

This is a solved problem. Nobody running serverless image generation pays the naive price — there is a standard set of fixes, and all but the last are free.

## The seven techniques

| # | Technique | What it does | Effect | Cost to us |
|---|---|---|---|---|
| 1 | **Bake weights into the container image, or put them on a fast attached volume** | The model file is already on the machine when it boots, so nothing is downloaded | ~70 s → ~20 s | Storage, ~$0.07–0.10/GB/month |
| 2 | **Snapshot restore** — Modal GPU memory snapshots, RunPod FlashBoot | Saves a checkpoint of the model *already loaded into GPU memory* and restores it. Skips loading and compiling entirely | ~20 s → ~3 s | Free, built into both platforms |
| 3 | **Batch requests per wake-up** | Load is paid once per wake-up, so 50 images per wake-up costs 1/50th the load per image | Divides load cost by batch size | Free — needs our app to group work |
| 4 | **Raise the idle timeout** (5 s → 60–300 s) | The worker stays awake briefly, so a request arriving soon after reuses it instead of reloading | Removes repeat loads within a burst | The extra awake seconds |
| 5 | **Quantise the weights** (FP8 / INT8) | A 12 GB model becomes ~6 GB, so there is half as much to copy | Roughly halves load time | Small, testable quality risk |
| 6 | **Cache `torch.compile` artifacts** to a volume | Sets `TORCHINDUCTOR_CACHE_DIR` / `TRITON_CACHE_DIR` so compiled kernels persist. Without this, compilation can take *minutes* on every boot | Avoids re-compiling | Free |
| 7 | **Keep one worker always warm** (`keep_warm=1` / min workers > 0) | A worker never shuts down, so there is no cold start at all | Eliminates it | Real money — that worker bills like a pod, 24×7 |

Techniques 1–3 do most of the work. Technique 7 is the escape hatch when latency matters more than cost, and it is really a partial pod.

## What the combinations cost

10,000 images/month, RunPod Serverless L4, FLUX.2 klein 4B (12 GB), 2 s per image:

| Setup | Weights from | Batch | Idle | Extra billed per image | ₹ generating | ₹ model load | **₹ / month** |
|---|---|---|---|---|---|---|---|
| **Naive** — platform defaults | Downloaded each boot (70 s) | 1 | 5 s | 75.0 s | ₹364 | ₹13,656 | **₹14,020** |
| **Well-tuned** — 1 + 3 + 4 | Cached (20 s) | 50 | 60 s | 1.6 s | ₹364 | ₹291 | **₹656** |
| **Best case** — 1 + 2 + 3 + 4 | Snapshot (3.5 s) | 100 | 60 s | 0.64 s | ₹364 | ₹116 | **₹480** |

**Two afternoons of configuration is worth about ₹13,500 a month at this volume**, and the gap widens with scale.

Note that the techniques interact: once snapshot restore has load down to ~3.5 s, batching matters far less. We would not need all seven — techniques 1, 2 and 4 alone get us most of the way, and 3 only becomes important if snapshot restore turns out not to work for our model.

You can try these combinations on the [calculator](/calculator) — the **Naive / Well-tuned / Best case** buttons set all four inputs at once.

## What we would actually do

| Step | Why |
|---|---|
| Bake the weights into the container image | Easiest win, removes the largest single chunk of load time |
| Turn on FlashBoot (RunPod) or GPU memory snapshots (Modal) | Free, and does more than everything else combined |
| Set the idle timeout to ~60 s | One line of config; catches bursts cheaply |
| Measure the real cold start before tuning further | Everything above is estimated from model size — one deploy replaces the estimates with facts |
| Add batching only if the measured number is still high | Requires app changes, so do it last and only if needed |

---

# Part 5 — What this means for the decision

**The conclusion does not change, but the margin does.**

| Path (10,000 images/month) | Cost |
|---|---|
| Hosted API — Gemini 3 Pro Image batch | ₹63,650 |
| Self-host serverless, **best case** (snapshot + batching) | ₹480 |
| Self-host serverless, **well tuned** (cached weights + batching) | ₹656 |
| Self-host serverless, **naive** (defaults, one image per wake-up) | ₹14,020 |
| Self-host on a 24×7 Pod | ₹54,787 |

Self-hosting still wins on raw compute cost even in the naive case — but the advantage narrows from ~175× to about 4.5×. And that comparison still assumes the open model's output quality is acceptable, which is **not yet proven**.

**Practical guidance:** if we pilot self-hosting, caching the weights and turning on snapshot restore are mandatory, not optimisations. Otherwise most of the saving evaporates into paying the GPU to load the same file over and over.

---

# Open items to measure

| Item | Why it matters | How to get it |
|---|---|---|
| Actual cold start seconds for klein 4B on our container | Sets which row of the table above we live in | Deploy once and read the platform logs |
| Actual generation seconds per image | Every ₹ figure scales directly with it | Benchmark on L4 and L40S |
| How many images we can realistically batch per wake-up | Decides whether cold start is 1.1× or 33× | Depends on how the DNA generator hands off work |
| Whether images must return immediately or can wait | Decides if API Batch tiers (50% off) apply at all | Product decision |

# Sources

| Topic | Link | Validated |
|---|---|---|
| RunPod Serverless pricing — the three billed phases | https://docs.runpod.io/serverless/pricing | HTTP 200 |
| RunPod GPU rates | https://www.runpod.io/pricing | Browser OK · L4 serverless $0.69 · L40S $1.75 |
| Modal pricing | https://modal.com/pricing | Browser OK · L4 $0.000222/s · L40S $0.000542/s |
| Hugging Face Inference Endpoints | https://huggingface.co/docs/inference-endpoints/en/pricing | Browser OK · L4 GCP $0.70 · AWS $0.80 · L40S $1.80 |
| Alibaba PAI-EAS billing rules | https://help.aliyun.com/en/pai/product-overview/billing-of-eas | Docs fetch OK · GPU $/hr console-only |
| Modal GPU memory snapshots | https://modal.com/blog/gpu-mem-snapshots | |
| AWS EC2 on-demand | https://aws.amazon.com/ec2/pricing/on-demand/ | |
