# Realistic AI Avatar — research + build log

Date: 2026-10-30
Goal: photorealistic AI avatar of a mid-range US woman (30s, average attractiveness), 3 renders per model, no visible "AI look".

---

## 1. Model facts (verified)

| Claim | Status | Source |
|---|---|---|
| `MAI-Image-2.6-Flash` exists | **True** — preview 2026-07-31, GA announced **2026-10-27** | Microsoft Learn "Deploy and use MAI image models in Microsoft Foundry"; Microsoft Community Hub GA post |
| MAI-Image-2.6 family | Diffusion + flow-matching, 20B non-embedding params, 32K context, text+image in, max **2,359,296 px** (1536x1536) | MAI-Image-2.6 Model Card (4 Sep 2026) |
| Where it runs | Azure AI Foundry, Global Standard, regions: West Central US, East US, West US, West Europe, Sweden Central, South India, UAE North | Microsoft Learn |
| API shape | MAI image generations API + MAI image edits API, Entra ID or API key | Microsoft Learn |
| "Nano Banana 2 Lite" | **True** — API id `gemini-3.1-flash-lite-image`; "Nano Banana 2" is `gemini-3.1-flash-image` | Gemini model list + treg catalog |
| NB2 Lite limits | text→image, **1K output only** (`imageSize: 2K` rejected), aspect ratios incl. 3:4, 4:5, 9:16, 16:9 | live API test |
| Watermarks — Gemini API | no visible mark, **invisible SynthID** always on | Google/OpenAI provenance docs, SeamUI 2026 guide |
| Watermarks — OpenAI API | **C2PA Content Credentials + SynthID** (OpenAI adopted SynthID May 2026) | OpenAI Help Center "Provenance signals" |

### Blocker
No Azure OpenAI / Foundry credentials exist on this machine (searched env, all `.env` files, `~/.azure`, opencode config). Present keys: `GEMINI_API_KEY`, `OPENAI_API_KEY`.
So: **the MAI-Image-2.6-Flash set was not produced.** The second set is **OpenAI `gpt-image-2.5-flare`** as the nearest available stand-in. Give me `AZURE_OPENAI_ENDPOINT` + key + deployment name and the identical prompts re-run on MAI in ~2 minutes.

---

## 2. What the research actually says

### 2.1 The consensus (Gemini docs + Reddit + X)

1. **Describe the scene as a paragraph, not keyword soup.** Google's own image-gen guide: "A narrative, descriptive paragraph will almost always produce a better, more coherent image than a list of disconnected words."
2. **Name the light's physical source and direction.** "Single window light from the left, uneven exposure, soft ambient shadows, realistic falloff." Generic "natural lighting", "perfectly lit", "professional portrait lighting" pull the model to studio gloss.
3. **Delete the hype tokens.** `8k`, `hyperrealistic`, `ultra-detailed`, `cinematic lighting`, `volumetric light`, `octane render`, `masterpiece`, `flawless` — in diffusion weights these map to "over-rendered CGI". Reddit's most repeated fix.
4. **Prompt for epidermal flaws explicitly.** visible pores of varying size, peach fuzz/vellus hair, uneven tone, mild erythema around nostrils, under-eye darkness, chapped lips with vertical fissures, a couple of small blemishes, asymmetric eyebrows.
5. **Camera language beats art language.** Phone model, focal length, "front camera at arm's length", "someone else holding the phone at chest height". Real amateur photos have wide-ish lenses, deep DOF, blown highlights, JPEG artifacts.
6. **Imperfection in framing.** Off-center, tilted horizon, cropped head/feet, subject mid-blink or mid-word, clutter in frame, background in focus.
7. **"Too attractive" is a tell.** Northwestern's Groh et al. (Oct 2026) list "faces that seem a little too attractive", waxy sheen and oversaturation as top stylistic artifacts — alongside anatomy, functional and physics errors.

### 2.2 X / Twitter (the prompt-engineer crowd)

- **@Marcos de Castro — "Anti-Plastic / Hyper-Realism Protocol"** (most-copied structure): force *stochastic noise* (non-uniform pore structure), *vellus hair* on jawline/forehead catching rim light, *melanin variance* and subdermal vascularity, *sebaceous sheen* on the T-zone only, dry lips with micro-cracks, and an explicit fail state: *"If skin appears smooth, doll-like, or airbrushed, GENERATION FAILS."*
- **@tiki1803**: "EDIT REFERENCE IMAGE, DO NOT CHANGE ANYTHING ELSE EXCEPT: SKIN REALISM (ENHANCED – PRIORITY)" + a negative prompt list: over-smooth skin, plastic skin, waxy texture, CGI look, unrealistic symmetry, beauty filter, airbrushed skin.
- **@Aqsahere_**: identity locking — "Keep the exact facial identity from the reference image with no facial modifications, no identity blending."
- **@Kashberg_0**: the older style — "wet dewy skin, visible water droplets… 85mm f/1.4… photorealistic, 8k, RAW photo." Still works on close-ups, but it is the style that produces the "AI Instagram" look.

### 2.3 Reddit (what practitioners say after the glow wears off)

- r/StableDiffusion, "Nano Banana alternatives that don't look AI-generated": *"Nano Banana is very good at general prompt adherence"* — i.e. it gives you exactly what you ask for, including the plastic look, if your prompt asks for polish. Another: *"to get photorealistic output you have to use a heavily finetuned model"* — the honest ceiling of general models. Also flags **SynthID** as an invisible watermark on all Nano Banana output.
- r/generativeAI thread: *"AI image tends to be… absolutely, statistically 'Average'. The whole background looks average, clean and tidy. RL tends to not be like that."* → the mid-range-woman strategy is right, but the **environment** must also be un-styled.
- Same thread, the other half: *"the big problem is creating really attractive people that look real"* — mid-range faces are the easy mode; 10/10 faces are where models break.
- Continuity: *"A lot of AI tells are not just inside one image… Does the person still look like the same person from another angle? Another outfit? Different lighting?"* — the #1 tell for an avatar that has to survive multiple posts.
- Post-processing that actually moves detectors: **1.5–3% monochromatic grain**, high-pass 1.0–1.5 px at ~40% soft light.

### 2.4 YouTube

- **metricsmule, "The Prompt Formula That Makes AI Images Look Too Real"** — the opposite camp: stack "meta tokens" (`Canon EOS R5`, `Leica Summilux-M 35mm f/1.4`, `Sony Venice`, `8k, HDR, cinematic color grading, film grain`). Works well in Midjourney; on Gemini/GPT it reintroduces the glossy look. **I tested against this and it loses.**
- **AI Master, "Ultimate Nano Banana Tutorial: 15 Prompt Techniques"** (347k views) — technique list, no per-model evidence.
- **AI2Play, "Nano Banana 2 Realistic Photography Prompts"** — 10 photography "Gems"; the useful part is the admission that *some photo genres are simply easier for AI than others* (product/food easy, candid people hard).
- **Paul J Lipsky, "Nano Banana 2.1 vs GPT Image 2.5 vs Nano Banana Pro"** — GPT Image wins on props/text/instruction following, NB wins on skin and speed. Matches what I saw.

### 2.5 Where the sources disagree
| Question | "Gear/buzzword" camp | "Narrative/anti-buzzword" camp | My test |
|---|---|---|---|
| `8k, hyperrealistic, cinematic` | include them | delete them | **delete** — they made skin waxier on both models |
| Camera specs | brand + lens + film stock | only how the photo was taken | middle: *phone model + focal length + who held it* worked best |
| "Professional lighting" | yes | never | **never** — it produced studio gloss every time |

---

## 3. The prompt system I used

Five blocks, in this order. This is the reusable part.

1. **Provenance of the photo** — who took it, on what, where, at what time. *"A photo her husband took on his phone from across the kitchen without warning."*
2. **Character block, verbatim across all images** — age, region, and 8–10 specific asymmetries. "Mid-range" is written explicitly: *"looks like an ordinary person rather than a model"* + a named flaw list.
3. **Light with a physical source and a direction** — *"hard low sun through the windshield from her left"*, *"cool morning window light from the right over the sink, falling off into the room"*, *"flat shadowless overcast light with no direction"*.
4. **Framing defects** — tilted horizon, cropped feet/head, off-center, mid-blink, hand out of frame.
5. **Anti-beautify clause** — *"Nothing is styled, nothing is cleaned up, no beauty retouching."* On GPT Image also: *"Do not beautify or smooth skin, do not make the person look like a model."*

Deliberately **omitted**: 8k, hyperrealistic, cinematic, masterpiece, HDR, film grain overlays, "professional".

Character block used:
> a 34-year-old woman from central Ohio who looks like an ordinary person rather than a model: oval face with slightly full cheeks and a soft jaw, light brown hair with darker roots and a few gray strands, hazel eyes set a little close together, a small bump on the bridge of her nose, a thin upper lip, faint horizontal lines across her forehead, a small mole under her right jaw, mild acne scarring on her chin, visible pores across her nose and cheeks, fine peach fuzz catching the light along her jaw, a little darkness under her eyes, and no makeup except worn-off tinted lip balm.

Scenes: (1) driver's-seat selfie in a Kroger lot, 4pm, hard sun through the windshield; (2) kitchen counter, morning window light, unposed; (3) strip-mall parking lot, overcast, walking, mid-word.

---

## 4. Renders

Repo: https://github.com/rmanoj-11/ai-avatar-review

### Gemini 3.1 Flash Lite Image (Nano Banana 2 Lite) — 896x1200, ~3–4 s each
1. https://raw.githubusercontent.com/rmanoj-11/ai-avatar-review/main/final_gemini_1.png — car selfie
2. https://raw.githubusercontent.com/rmanoj-11/ai-avatar-review/main/final_gemini_2.png — kitchen
3. https://raw.githubusercontent.com/rmanoj-11/ai-avatar-review/main/final_gemini_3.png — parking lot

### OpenAI GPT Image 2.5 (stand-in for MAI-Image-2.6-Flash) — 1024x1536, ~15–18 s each
1. https://raw.githubusercontent.com/rmanoj-11/ai-avatar-review/main/final_gpt_1.png — car selfie
2. https://raw.githubusercontent.com/rmanoj-11/ai-avatar-review/main/final_gpt_2.png — kitchen
3. https://raw.githubusercontent.com/rmanoj-11/ai-avatar-review/main/final_gpt_3.png — parking lot

### Honest audit (what still gives them away at 100%)

**Gemini**
- Skin is the best of the two at close range (pores, shine, asymmetry) and the age reads correctly.
- Weak spot: **hands** (fingers merge into the phone / tote bag) and **readable background text** (one draft rendered "VAPE & SNAKE"; discarded).
- Adds a faint symmetrical "catalog pretty" if the prompt does not carry enough flaw language — the same face drifts ~10% younger between seeds.
- Car selfie: the interior mirror renders as a black rectangle.

**GPT**
- Better props, better signage, better instruction-following; the kitchen reads as someone's actual house.
- Weak spot: **over-textures skin** — it ages the subject 8–10 years unless you explicitly write "normal for a 34-year-old, not weathered".
- Eyes go glassy/vacant in front-facing shots; a slight squint or an off-camera glance fixes it.
- 1024x1536 is the max useful size on this model.

**Both**: no lens/sensor noise unless asked; no EXIF; provenance watermarks present (see §1). I did not strip them and would not.

---

## 5. If you want v2

1. **Lock identity first.** Pick the best face, then generate images 2 and 3 *from it* as a reference (Gemini: image input; GPT: `/v1/images/edits`). This kills the biggest avatar tell — a face that changes between posts.
2. **Two-pass refinement** instead of one-shot: pass 1 gets composition, pass 2 is an edit instruction that only touches skin/eyes (`@tiki1803` pattern, works on both).
3. **Kill text in frame** unless the scene needs it — garbled signage is the most common hard fail.
4. **Add 1.5–3% monochrome grain** in post only if the output looks plasticky; do not put "film grain" in the prompt, both models render it as an overlay.
5. **Run the same prompts on MAI-Image-2.6-Flash** once Azure creds exist — its model card claims stronger portrait fidelity than 2.5, which is the fair comparison you actually asked for.
