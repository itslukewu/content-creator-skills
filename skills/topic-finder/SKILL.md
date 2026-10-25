---
name: topic-finder
description: "[English version, for English-speaking users; the Chinese version is topic-finder-zh in zh-CN/] Finds content topics for creators. Reads a config; if no reusable audience profile exists, first infers initial audience profiles from benchmark accounts' outperforming posts. Then searches for topics per fixed content pillar and openly across several sources, screens and de-duplicates them, and outputs a candidate topic table with source links and a readiness status. Writes to a topic library only after the user confirms. Use when the user asks to 'find topics', 'what should I post this week', or 'give me a batch of content ideas'."
---

# topic-finder

This is the English version, for English-speaking users. A Chinese version with the same content is in `zh-CN/` (skill name `topic-finder-zh`), for Chinese-speaking users. Install only one of the two.

## Inputs and outputs

- **Input**: a config (format in `config.example.yaml`); optionally, an existing topic library.
- **Output**: a candidate topic table. Every row carries clickable source links and a readiness status. Nothing is written to an external system until the user confirms.

## Workflow

```text
Read config
    ↓
Is there a reusable audience profile?
    ├─ No  → Step 1: infer initial audience profiles from benchmark content
    └─ Yes → reuse it and skip Step 1
    ↓
Step 2: collect candidate topics (pillar-directed search + open search)
    ↓
Step 3: screen and de-duplicate
    ↓
Step 4: output the topic table → user confirms → write to the topic library
```

## Before you start

1. Read the config.
2. If there is no config, confirm three things with the user:
   - What audience or benchmark leads they have (benchmark accounts or sample posts both work)
   - What fixed content pillars they have (none is fine)
   - Which sources are available, and how to access them

   An undecided audience does not block the run: as long as there are benchmark leads, go to Step 1. Never invent any of this information for the user.
3. Keep three time-related concepts separate; none replaces another:
   - **Target audiences the user explicitly chose** (`source: user_defined`): never expire with time and are never replaced by inferred results. New evidence only adds observations.
   - **Profiles inferred from data** (`source: inferred`): prompt a review when `review_every_days` is reached; may be updated with new evidence.
   - **Benchmark lookback window** (`lookback_days`, default 90): only sets how far back posts are fetched. It has nothing to do with how long a profile stays valid.

## Shared rule: relative performance (RP)

Step 1 and Source A both use this rule.

- **RP** = a post's likes ÷ the median visible likes of that account within the same window.
- Hidden or missing likes are not treated as 0: they are excluded from the baseline and get no RP.
- If an account has fewer than `min_baseline_posts` (default 5) posts with visible likes, or its baseline is 0, compute no RP for that account.
- A post with RP ≥ `outlier_rp` (default 2) is an outlier post.
- **Topic outlier rate** = outlier posts in a topic ÷ all RP-eligible posts in that topic. Never compute it from the outlier posts alone.
- Record on every run: the time window, the fetch date, the number of valid posts, each account's baseline, and which accounts were excluded and why.
- Comment counts are inflated by "comment a keyword to get the link" tactics. Judge audience response mainly by likes and report comments separately.
- These numbers only describe how a topic performed in this run's benchmark sample. Average performance does not mean the market is oversupplied, and it is not grounds for rejecting a new angle.

## Step 1: infer initial audience profiles

Runs by default whenever there is no reusable profile.

Goal: build initial profiles from observations grounded in content, rather than assuming an audience and then looking for evidence. A profile describes who this content serves; it does not describe the real demographics of the benchmark accounts' followers.

1. Get the benchmark data. Share the same batch with Source A; do not fetch twice.
2. Apply the shared RP rule: find outlier posts, group them by topic, and compute each topic's outlier rate.
3. From the content of the outlier posts, infer: who the content helps, what problem it solves, and what outcome it promises.
4. Write each profile in two layers:
   - **Observations**: what can be read directly from the posts. Attach links to the evidence posts and note how many accounts are involved.
   - **Hypotheses to verify**: age, income, willingness to pay, occupation, and also motivation and life or business stage. Mark a hypothesis "self-reported" (with a link) when a post contains the person's own direct statement; otherwise mark it "inferred".
5. Flag any conclusion that rests on only one or two accounts.

**Output**: 3 to 6 initial profiles, written back to the `audience` section of the config with `source: inferred`, the generation date, and the data window. Once the user confirms them, they can be switched to `user_defined`.

## Step 2: collect candidate topics

Run two kinds of search side by side. These are two ways of searching, not two kinds of source:

- **Pillar-directed search**: for each fixed pillar in the config, actively look for material that fits its `brief`, such as tools, use cases, and examples. Skip this when there are no fixed pillars.
- **Open search**: find anything else worth covering across the sources. These topics do not need to fit a fixed pillar; their pillar can be "open topic".

By default, use every available source in the config together. If the user limits the sources, or a source cannot be reached, run with the remaining ones and state at the top of the output which sources this run covered, what is missing, and why. Never fabricate material to make up for a missing source.

### Source A: outlier content from benchmark accounts

- Read or refresh the benchmark data independently, using `lookback_days`. If Step 1 already fetched the same batch in this run, reuse it.
- Find outlier posts with the shared RP rule. An outlier post can be used as long as it fits the target audience; the user does not need to have had the same experience.
- Borrow the topic, angle, structure, and format. Do not copy the other creator's script, opinions, case studies, or footage.

### Source B: audience and market signals

- **Surveys and reports**: record the disclosed sample size, respondent population, method, and fieldwork dates. Write "not disclosed" for anything not disclosed; never fill it in. Label secondary summaries "secondary source", and trace them back to the original report when it can be found.
- **Communities, forums, Q&A, comments, product reviews**: keep the original wording, context, link, and date. Distinguish genuine questions, experience reports, generic praise, and lead-magnet replies. Never inflate a single complaint into a group consensus.
- **Search suggestions**: use only to discover how people phrase things and to find leads, then go find the actual material.
- Write search queries around the audience's identity, activities, and goals. Do not put presumed pain points into queries. Tool names can be an entry point but are never required.

### Source C: the user's saved references

- Read the saved-references location set in the config. First decide what each saved item is for:
  - **Topic lead**: a topic, question, opinion, or case study
  - **Production reference**: structure, hook, visuals, or editing. Keep it as a reference; do not force a topic out of it
- When the user left a note, give it priority. When there is no note, do not guess why the item was saved.
- For videos, get a transcript where possible. For content that depends on visuals, also check the visual material (cover, screenshots, scene descriptions). If retrieval fails, state what was actually read and what is missing; never pretend to have watched it. The other sources carry on as normal. Fetching tools are listed under `tools` in the config.
- Flag any saved item whose title or caption does not match its actual content.

For every candidate, record:
- Where it was found, with links (there can be several)
- Its pillar
- Usable material (including the user's own experience, data, or test results, if any)

## Step 3: screen and de-duplicate

Answer five questions for every candidate:

1. **Who is it for?** It must map to at least one target audience: one primary audience, optionally secondary ones.
2. **Why is it worth covering for this audience?** Is the basis outlier response, a market signal, a concrete use, or an opinion or case study from the user's saved references? State each basis separately. "Already has high engagement" is not a universal threshold.
3. **What concrete value does it give the target audience, what angle will it take, and what material can strengthen it?** A clear explanation, a useful compilation, a case breakdown, or a demo can each carry a piece of content; it does not need a discovery nobody else has. The user's own experience, data, and test results are a bonus, never an entry requirement for any source.
4. **Do the key claims hold up?** Which facts, numbers, or features need checking? Label other people's case studies, official demos, and the user's own tests separately; never let one pass for another.
5. **What does production need?** Is the material ready, or does it need testing, filming, or more research? Judge production effort separately from topic value.

### De-duplication

- Compare candidates within this batch, and against the existing topic library, by "the question it answers + the angle". Do not judge by title or URL alone.
- Same topic, same angle: merge into one row and combine sources and links.
- Same topic, but a different audience or a different question: keep separate rows.
- The same material reshared in several places counts as one piece of evidence.

### Readiness

Give every candidate one status:

| Status | Meaning |
|---|---|
| Ready for brief | Enough basis to write a brief; the script comes later |
| Needs verification | Promising, but facts or numbers that affect the core point have not been checked. List what needs checking |
| Needs testing | The chosen treatment depends on real results (for example, claiming test results), so the claim can only be made after testing. Explaining an official feature or analyzing a public case study does not need testing |
| Drop | Does not fit the target audience, lacks concrete value, rests on a claim that cannot be supported, or substantially duplicates an existing topic's question and angle. State the reason |

"This topic performed averagely in the benchmark sample" is not a reason to drop.

## Step 4: output the topic table

Show the table to the user first. Each row includes:

- Topic / working title
- Primary audience (and secondary audiences, if any)
- Sources and links: one or more clickable links, each labelled with the source it came from
- Pillar: one of the fixed pillars, or "open topic"
- Why it's worth doing: basis, citation, limitations
- Angle: core point, case, or demo scenario; suggested format; any of the user's own material that can be used (only if there is some)
- Readiness + next step

Open the table with a short run note:
- The data window
- Coverage of each source, plus anything missing or degraded
- Whether an existing profile was reused this run

### Writing to the topic library

- Write only after the user confirms. Afterwards, report how many rows were written and where.
- Before writing, read the database's actual fields and map by field type. If a field cannot hold links, put the links in a text field and tell the user.
- Store **readiness** (Ready for brief / Needs verification / Needs testing) separately from **approval or workflow status** (for example "pending review"); neither overwrites the other. New rows get the default workflow status from the config. If the config sets `readiness_labels`, use those labels when writing.

## Don't

- Don't fabricate links, view counts, data, or report fields. If something can't be found, write "to be added" or "not disclosed".
- Don't present inferred audience traits as conclusions.
- Don't use benchmark sample performance to judge supply and demand in the whole market.
- Don't present other people's case studies or official demos as the user's own tests.
- Don't write to any external system (databases, GitHub, etc.) without the user's confirmation.
