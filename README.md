# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> The starter begins with all three tools as stubs. This Unit 3 build now implements the three tools and the planning loop.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr is an agent that helps a user find thrift listings based on what they want, including an optional size and maximum price. It searches the listings data, selects a matching item, and uses the user's wardrobe to suggest an outfit. It then creates a short fit card for the thrift find. If the search finds nothing, the agent stops early and tells the user what they can change in their search.

---

## Tool Inventory

### `search_listings`

- **What it does:** Searches the thrift listings for items matching the user's description, optional size, and optional maximum price.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None)
- **Returns:** A list of matching listing dictionaries, ordered best match first. Each listing contains fields such as `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list `[]`. Size matching will be case-insensitive and based on size tokens rather than plain substring matching, so a request for `M` can match `S/M` without incorrectly matching unrelated sizes.

### `suggest_outfit`

- **What it does:** Takes the selected thrift listing and the user's wardrobe and creates one or two outfit ideas using items the user already owns.
- **Inputs:** `new_item` (dict), `wardrobe` (dict)
- **Returns:** A non-empty string containing outfit suggestions for the selected item.
- **When it has nothing:** If the wardrobe contains no items, it returns general styling advice for the new item instead of failing or returning an empty string.

### `create_fit_card`

- **What it does:** Turns the selected item and outfit suggestion into a short caption that could be posted with the thrift find.
- **Inputs:** `outfit` (str), `new_item` (dict)
- **Returns:** A two-to-four sentence caption that mentions the item, its price, its platform, and the overall outfit vibe.
- **When it has nothing:** If `outfit` is empty or only whitespace, it returns a descriptive message explaining that a fit card cannot be created without an outfit suggestion instead of raising an error.

---

## Planning Loop

**Branch rule:** If `search_listings` returns an empty list, put a useful message in the session telling the user to try a broader description, another size, or a higher maximum price, then stop. Otherwise, select the first search result and continue to `suggest_outfit`, then `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** The query is parsed with regular expressions. The agent extracts an optional maximum price and size, removes those parts from the query, and uses the remaining text as the item description.

**What moves through the session:** The parsed search values go into `session["parsed"]`. The results from `search_listings` go into `session["search_results"]`, and the first result goes into `session["selected_item"]`. That selected item is read back from the session by `suggest_outfit`, whose result is stored in `session["outfit_suggestion"]`. Finally, `create_fit_card` reads the outfit suggestion and selected item from the session and stores the result in `session["fit_card"]`.

---

## Sample Run

**One full query**

```text
$ python app.py ask 'vintage graphic tee under $30'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here is a wearable outfit combining the new Y2K baby tee with pieces from your wardrobe:

**Y2K Streetwear Look**
*   **Top:** Y2K Baby Tee (Butterfly Print)
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

  Fit card: Channeling total Y2K streetwear energy with this butterfly print baby tee. Pair it with dark wash baggy jeans, chunky sneakers, and a vintage black denim jacket for the ultimate nostalgic fit. Grab this cute find for just $18.00 live on Depop now!

2 model calls this session, 597 prompt + 146 output tokens
```

**The three tools, tested one at a time**

### `search_listings`

```text
$ python -c "from tools import search_listings; r=search_listings('graphic tee', max_price=30); print([(x['title'], x['price']) for x in r])"

[('Y2K Baby Tee — Butterfly Print', 18.0), ('Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('Mesh Long-Sleeve Top — Black', 15.0), ('Vintage Band Tee — Faded Grey', 19.0), ('Low-Rise Cargo Pants — Khaki', 27.0), ('Vintage Graphic Hoodie — Faded Black', 26.0)]
```

### `suggest_outfit`

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Here are two practical outfit ideas using the new Vintage Levi's 501 Jeans and items from your current wardrobe:

**Outfit 1: Casual Streetwear**
*   **Bottoms:** Vintage Levi's 501 Jeans (Medium Wash)
*   **Tops:** White ribbed tank top
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Outfit 2: Cozy & Classic**
*   **Bottoms:** Vintage Levi's 501 Jeans (Medium Wash)
*   **Tops:** Oversized grey crewneck sweatshirt
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt, Black crossbody bag
```

### `create_fit_card`

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('Pair these jeans with a white tank, black denim jacket, and chunky white sneakers for a casual streetwear look.', load_listings()[0]))"

Nothing beats the fit of broken-in vintage Levi's 501 jeans, and this medium wash pair is a total closet staple. Grab them on Depop for just $38.00 before they're gone. Throw them on with a simple white tank, a black denim jacket, and chunky sneakers for the ultimate casual streetwear vibe.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked AI for help designing `search_listings`, especially how to handle sizes without using unsafe substring matching.
- *What came back:* It suggested splitting sizes into tokens so a request for `M` can match `S/M` without making `L` automatically match `XL`.
- *What I changed:* I used token-based size matching and then tested the search against the real listing data, including the maximum-price filter and an impossible query.

**Moment 2**

- *What I asked for:* I asked AI for help wiring the three tools into a planning loop while keeping the tool results visible in session state.
- *What came back:* It suggested storing each result in the session and using a branch after `search_listings` so an empty result stops the run before the model tools are called.
- *What I changed:* I implemented the branch in `agent.py::run_agent`, kept the selected listing and later outputs in the session, and tested both a successful query and an impossible query. The impossible query stopped with `fit_card` still set to `None`.

**Unit 4 assistance**

I asked Codex to inspect the current repository, preserve Unit 3, and finish testing. The MCP wiring and failure handling/tracing were already implemented in commits `de0aca8` and `50e7ffd`; this assistance inspected and verified them rather than rewriting them. Earlier AI help with those steps is not independently documented in this session, so I do not attribute their authorship to this run. Codex verified the MCP schema and fresh traces, retained the existing failure handlers, adopted the suggested session ID and complete search-price instrumentation, and retained my current evaluation scenarios. It collected real uncached BEFORE evidence, scored each criterion, and diagnosed the provider's 503 errors. I adopted one suggested improvement: bounded retries for temporary 503 responses. Codex ran the same AFTER evaluation and helped prepare this evidence and documentation. Existing Unit 3 AI entries above are preserved. Human review should confirm the scoring and that this reflects my understanding before I submit.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. matching query completes | 4/5 | PASS | PASS | PASS | PASS | FAIL | MET (4/5) |
| 2. impossible query stops early | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. selected item stays the same through state | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. fit card includes listing details | 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. search respects maximum price | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Full evidence: [run_2026-10-13_0230_before.md](results/run_2026-10-13_0230_before.md). Produced by `run_eval.py::main` / `write_report`, loop `agent.py::run_agent`, model tools `tools.py`, search through `mcp_client.call_tool` → `mcp_server.py::search_listings` → `tools.py::search_listings`. Caching off, five tries, unchanged model/data/temperature (0.9). Report timestamps use the host clock; this work occurred October 13, 2026 in America/New_York.

### Criterion 1: real Try 1 output

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
- selected_item_id: lst_002
- suggest_outfit_item_id: lst_002
- search_result_prices: [18.0, 24.0, 19.0, 26.0, 15.0, 22.0, 27.0, 20.0, 30.0, 12.0]

Outfit suggestion:

```
Here is an outfit combining the Y2K baby tee with pieces from your wardrobe:

**Y2K Streetwear Look**
* **Top:** Y2K Baby Tee — Butterfly Print
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Outerwear:** Vintage black denim jacket
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

This look balances the fitted, feminine silhouette of the baby tee with the relaxed, baggy fit of the dark-wash jeans, leaning into a classic Y2K streetwear aesthetic.
```

Fit card:

```
Channeling ultimate Y2K streetwear energy with this butterfly print baby tee. Pair it with baggy dark-wash jeans and a vintage denim jacket for the complete look. Grab this Y2K baby tee for $18.00 on Depop before it's gone!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here is an outfit combining the Y2K baby tee with pieces from your wardrobe:  **Y2K Streetwear Look** * **Top:…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling ultimate Y2K streetwear energy with this butterfly print baby tee. Pair it with baggy dark-wash jea…
```

### Criterion 2: real Try 1 output

- stopped early: yes — I couldn't find a matching listing. Try a broader description, another size, or a higher maximum price.
- selected_item: (none)
- search_results: 0
- selected_item_id: None
- suggest_outfit_item_id: None
- search_result_prices: []

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
```

### Criterion 3: real Try 1 output

- stopped early: no
- selected_item: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
- search_results: 4
- selected_item_id: lst_004
- suggest_outfit_item_id: lst_004
- search_result_prices: [45.0, 75.0, 30.0, 33.0]

Outfit suggestion:

```
Here is a practical, streetwear-inspired outfit combining the 90s track jacket with pieces from your wardrobe:

**The 90s Streetwear Look**
*   **Outerwear:** 90s Track Jacket (Navy/White Stripe)
*   **Top:** White ribbed tank top
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** Layering the track jacket unzipped over the fitted white ribbed tank creates a balanced silhouette with the baggy dark wash jeans. Finishing the look with chunky white sneakers and the black crossbody bag leans right into the vintage athletic aesthetic.
```

Fit card:

```
Channeling major 90s streetwear energy with this navy and white stripe track jacket! Pair it with a ribbed tank, baggy jeans, and chunky sneakers for the ultimate vintage athletic vibe. Snag this piece over on Poshmark for just $45.00 before someone else does.
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 4 items: 90s Track Jacket — Navy/White Stripe, 90s Leather Bomber — Black, 90s Silk Slip Dress — Floral, Midi Length … +1 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here is a practical, streetwear-inspired outfit combining the 90s track jacket with pieces from your wardrobe:…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling major 90s streetwear energy with this navy and white stripe track jacket! Pair it with a ribbed tan…
```

### Criterion 4: real Try 1 output

- stopped early: no
- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- search_results: 7
- selected_item_id: lst_007
- suggest_outfit_item_id: lst_007
- search_result_prices: [42.0, 38.0, 45.0, 24.0, 33.0, 30.0, 27.0]

Outfit suggestion:

```
Here are two wearable outfit ideas combining the new light wash cropped denim jacket with pieces from your current wardrobe:

**Outfit 1: Casual Streetwear**
*   **Outerwear:** Denim Jacket — Light Wash, Cropped
*   **Top:** White ribbed tank top
*   **Bottoms:** Baggy straight-leg jeans, dark wash
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag
*   *Why it works:* A classic double-denim look with contrasting light and dark washes, balanced by a fitted tank and casual sneakers.

**Outfit 2: High-Low Contrast**
*   **Outerwear:** Denim Jacket — Light Wash, Cropped
*   **Top:** Oversized grey crewneck sweatshirt
*   **Bottoms:** Wide-leg khaki trousers
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt, black crossbody bag
*   *Why it works:* Layering the cropped jacket over the oversized sweatshirt creates a cool proportion play, paired with minimal trousers and edgy boots.
```

Fit card:

```
Scored this light wash cropped denim jacket for just $42.00 over on Poshmark! It’s giving effortless streetwear and high-low contrast vibes that go with literally everything in my closet. Grab it before I change my mind!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 7 items: Denim Jacket — Light Wash, Cropped, Vintage Levi's 501 Jeans — Medium Wash, 90s Track Jacket — Navy/White Stripe … +4 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two wearable outfit ideas combining the new light wash cropped denim jacket with pieces from your cur…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Scored this light wash cropped denim jacket for just $42.00 over on Poshmark! It’s giving effortless streetwea…
```

### Criterion 5: real Try 1 output

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
- selected_item_id: lst_002
- suggest_outfit_item_id: lst_002
- search_result_prices: [18.0, 22.0, 24.0, 20.0, 12.0, 24.0, 16.0, 18.0, 21.0, 25.0]

Outfit suggestion:

```
Here is a wearable outfit combining the new Y2K butterfly baby tee with pieces from your current wardrobe:

**Y2K Streetwear Casual**
*   **Top:** Y2K Baby Tee (Butterfly Print)
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Black cropped zip hoodie
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

This look balances the fitted, feminine silhouette of the baby tee with the relaxed, baggy fit of your dark wash jeans, tied together with the cropped hoodie and chunky sneakers for an easy Y2K-inspired streetwear vibe.
```

Fit card:

```
Serving major Y2K streetwear vibes with this adorable butterfly print baby tee. Pair it with baggy dark denim and chunky sneakers for the ultimate casual look. Grab this gem now on Depop for just $18.00!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Oversized Flannel Shirt — Plaid Red/Black, Graphic Tee — 2003 Tour Bootleg Style … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here is a wearable outfit combining the new Y2K butterfly baby tee with pieces from your current wardrobe:  **…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Serving major Y2K streetwear vibes with this adorable butterfly print baby tee. Pair it with baggy dark denim …
```

## Verdicts and Diagnoses

| # | Target | Verdict | Evidence and decision |
|---|---|---|---|
| 1 | 4/5 | MET (4/5) | Tries 1–4 completed all three tools and produced non-empty cards; try 5 stopped at a 503 before a card. Four passes meet 4/5. |
| 2 | 5/5 | MET (5/5) | All searches returned []; each trace stops after MCP search and the error names description, size, and price changes. Five passes meet 5/5. |
| 3 | 5/5 | MET (5/5) | All five selected IDs and actual outfit argument IDs equal lst_004, including runs whose model failed. Five passes meet 5/5; this criterion measures transfer, not completion. |
| 4 | 4/5 | MET (5/5) | All five successful cards identify the light wash cropped denim jacket, $42.00, and Poshmark. Natural wording and platform capitalization are allowed by the criterion. Five passes exceed 4/5. |
| 5 | 5/5 | MET (5/5) | All ten returned prices in each try are at most $25, including the $25 boundary. Five passes meet 5/5. |

**Diagnoses:** No criterion missed its original target. Nevertheless, Criterion 1 try 5 and Criterion 3 tries 2, 4, and 5 stopped at real Gemini 503 UNAVAILABLE responses. The failure location is the model call in `generate.py::generate`, reached from `suggest_outfit` or `create_fit_card`. Mechanism: the provider reports temporary high demand; the adapter only retries rate limits, so a transient 503 immediately becomes `ModelUnavailable`. Search and session transfer remained correct. This shared pattern supports one bounded 503 retry improvement.

**Were the targets too easy?** The deterministic state and budget tests repeat the same input, and matching-query completion only covers one query. In a future evaluation I would tighten Criterion 1 to require completion across five distinct matching queries, including size and price boundaries. The original criteria and this before/after comparison remain unchanged.

---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```text
$ python app.py ask 'vintage graphic tee under $30' --trace

[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here is a wearable outfit combining the new Y2K baby tee with pieces from your wardrobe:  **Y2K Streetwear Loo…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling total Y2K streetwear energy with this butterfly print baby tee. Pair it with dark wash baggy jeans,…

```

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5' --trace

[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)

I couldn't find a matching listing. Try a broader description, another size, or a higher maximum price.

0 model calls this session
```

**On the MCP move:** `agent.py::run_agent` now calls `mcp_client.call_tool("search_listings", arguments)` instead of the direct local function. `mcp_server.py` exposes exactly one tool, wrapping the existing `tools.py::search_listings` implementation. The typed protocol call changes the call shape; the returned listing dictionaries and downstream selection/branch behavior remain the same. `python mcp_client.py` verified the sole tool and description/string, optional size/string, and optional max_price/number inputs. Both traces above were already observed before this assistance; the fresh evaluation traces corroborate the same branches.

**Failure modes:** Empty search stops before either model tool (five fresh BEFORE tries); empty wardrobe produces general styling advice and fit cards (five fresh BEFORE tries). The invalid-key test was also triggered manually, then the real key was restored. The exact observed message was:

```text
The styling model couldn't be reached. The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com. Check your API key or internet connection and try again.
```

Real 503 failures in the BEFORE evidence also demonstrate the existing unavailable-model handlers.

---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:** One behavioral improvement in `generate.py::generate`: retry a provider 503 UNAVAILABLE response using the existing five-attempt budget, backoff, and request pacing. If all attempts fail, raise `ModelUnavailable` so the existing agent handler still stops cleanly. Other non-rate-limit errors still stop immediately. No changes to prompts, listing ranking, search filters, model, temperature, or data.

**Which failure it was meant to fix:** The BEFORE run had four early exits caused by temporary provider overload: Criterion 1 try 5 and Criterion 3 tries 2, 4, and 5. The adapter previously only retried 429/rate-limit errors. A short, bounded retry gives transient overload a chance to recover; persistent outages still fail honestly. All original targets were met, so this addresses observed failed runs without manufacturing a missed criterion.

**Verification:** Compilation passed. Mocked adapter checks passed for 503 then success (two calls), persistent 503 (exactly five calls then `ModelUnavailable`), and key rejection (one call). These diagnostic checks do not replace the real AFTER evaluation.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. matching query completes | 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. impossible query stops early | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. selected item stays the same through state | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. fit card includes listing details | 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. search respects maximum price | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Full evidence: [run_2026-10-13_0244_after.md](results/run_2026-10-13_0244_after.md). Same `scenarios.py`, five tries, original targets, model, data, temperature, and caching off. Provenance: `run_eval.py::main` / `write_report` → `agent.py::run_agent`; search via MCP wraps `tools.py::search_listings`, then `tools.py::suggest_outfit` and `create_fit_card`.

#### Criterion 1: real AFTER Try 1 output

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
- selected_item_id: lst_002
- suggest_outfit_item_id: lst_002
- search_result_prices: [18.0, 24.0, 19.0, 26.0, 15.0, 22.0, 27.0, 20.0, 30.0, 12.0]

Outfit suggestion:

```
Here are two ways to style the Y2K butterfly baby tee using items from your current wardrobe:

**Outfit 1: Casual Y2K Streetwear**
*   **Top:** Y2K Baby Tee — Butterfly Print
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Black cropped zip hoodie
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Outfit 2: Contrast Mix (Y2K meets Minimal)**
*   **Top:** Y2K Baby Tee — Butterfly Print
*   **Bottoms:** Wide-leg khaki trousers
*   **Accessories:** Brown leather belt, Black crossbody bag
*   **Shoes:** Chunky white sneakers
```

Fit card:

```
Channel your inner 2000s icon with this Y2K butterfly baby tee, available now on Depop for just $18. Style it with baggy denim and a zip hoodie for effortless streetwear vibes, or dress it down with wide-leg trousers for a cool contrast mix. Grab it before it's gone!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two ways to style the Y2K butterfly baby tee using items from your current wardrobe:  **Outfit 1: Cas…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channel your inner 2000s icon with this Y2K butterfly baby tee, available now on Depop for just $18. Style it …
```

#### Criterion 2: real AFTER Try 1 output

- stopped early: yes — I couldn't find a matching listing. Try a broader description, another size, or a higher maximum price.
- selected_item: (none)
- search_results: 0
- selected_item_id: None
- suggest_outfit_item_id: None
- search_result_prices: []

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
```

#### Criterion 3: real AFTER Try 1 output

- stopped early: no
- selected_item: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
- search_results: 4
- selected_item_id: lst_004
- suggest_outfit_item_id: lst_004
- search_result_prices: [45.0, 75.0, 30.0, 33.0]

Outfit suggestion:

```
Here are two practical outfit ideas using the 90s track jacket and pieces from your current wardrobe:

**Outfit 1: Casual Streetwear**
*   **Outerwear:** 90s Track Jacket (Navy/White Stripe)
*   **Top:** White ribbed tank top
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Accessories:** Black crossbody bag
*   **Shoes:** Chunky white sneakers

**Outfit 2: Sporty Minimalist**
*   **Outerwear:** 90s Track Jacket (Navy/White Stripe)
*   **Bottoms:** Wide-leg khaki trousers
*   **Accessories:** Brown leather belt, Black crossbody bag
*   **Shoes:** Chunky white sneakers
```

Fit card:

```
Channel ultimate casual streetwear or sporty minimalist vibes with this vintage 90s track jacket. It is super versatile for styling with your everyday denim or trousers. Grab this piece now on Poshmark for just $45.00!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 4 items: 90s Track Jacket — Navy/White Stripe, 90s Leather Bomber — Black, 90s Silk Slip Dress — Floral, Midi Length … +1 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two practical outfit ideas using the 90s track jacket and pieces from your current wardrobe:  **Outfi…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channel ultimate casual streetwear or sporty minimalist vibes with this vintage 90s track jacket. It is super …
```

#### Criterion 4: real AFTER Try 1 output

- stopped early: no
- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- search_results: 7
- selected_item_id: lst_007
- suggest_outfit_item_id: lst_007
- search_result_prices: [42.0, 38.0, 45.0, 24.0, 33.0, 30.0, 27.0]

Outfit suggestion:

```
Here are two wearable outfits using the new light wash cropped denim jacket and items from your current wardrobe:

**Outfit 1: Casual Streetwear (Double Denim)**
* **Top:** White ribbed tank top
* **Bottoms:** Baggy straight-leg jeans (dark wash)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag

**Outfit 2: High-Low Contrast**
* **Top:** Oversized grey crewneck sweatshirt
* **Bottoms:** Wide-leg khaki trousers
* **Shoes:** Black combat boots
* **Accessories:** Brown leather belt and black crossbody bag
```

Fit card:

```
Scored this light wash cropped denim jacket for just $42.00 on Poshmark! It’s giving effortless, everyday streetwear vibes that make double denim and high-low styling so easy. Can't wait to mix and match this piece with everything already in my closet.
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 7 items: Denim Jacket — Light Wash, Cropped, Vintage Levi's 501 Jeans — Medium Wash, 90s Track Jacket — Navy/White Stripe … +4 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two wearable outfits using the new light wash cropped denim jacket and items from your current wardro…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Scored this light wash cropped denim jacket for just $42.00 on Poshmark! It’s giving effortless, everyday stre…
```

#### Criterion 5: real AFTER Try 1 output

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 10
- selected_item_id: lst_002
- suggest_outfit_item_id: lst_002
- search_result_prices: [18.0, 22.0, 24.0, 20.0, 12.0, 24.0, 16.0, 18.0, 21.0, 25.0]

Outfit suggestion:

```
Here is a wearable outfit combining the Y2K butterfly baby tee with pieces from your wardrobe:

**Y2K Streetwear Look**
*   **Top:** Y2K Baby Tee — Butterfly Print
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Black cropped zip hoodie
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

This look balances the fitted, feminine silhouette of the baby tee with the baggy streetwear style of your dark wash jeans, tied together with the cropped hoodie and chunky sneakers.
```

Fit card:

```
Channeling major Y2K streetwear energy with this adorable butterfly print baby tee. Pair it with baggy dark-wash jeans and a cropped hoodie for the ultimate effortless fit. Grab this gem on Depop right now for just $18.
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Oversized Flannel Shirt — Plaid Red/Black, Graphic Tee — 2003 Tour Bootleg Style … +7 more
[2] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here is a wearable outfit combining the Y2K butterfly baby tee with pieces from your wardrobe:  **Y2K Streetwe…
[3] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling major Y2K streetwear energy with this adorable butterfly print baby tee. Pair it with baggy dark-wa…
```

**Did it help, and how do I know:** Criterion 1 improved from 4/5 to 5/5; Criteria 2–5 remained 5/5. Across the 25 matching-query runs (including the empty-wardrobe diagnostic), completion improved from 21/25 to 25/25. BEFORE had four early exits on 503 responses; AFTER had none. The AFTER adapter counted exactly 50 model calls for 25 two-model-call runs, with no 503 retry messages, so this run did not exercise live 503 recovery. Provider availability may simply have improved. The mocked recovery and exhaustion checks establish that the retry mechanism works; the real comparison establishes better observed completion, not proof that retries caused it. No criterion regressed. BEFORE used 49 model calls and 13,582 prompt + 4,552 output tokens; AFTER used 50 calls and 14,959 prompt + 4,871 output tokens. Failed requests can count as calls without reported tokens.

## What's Still Broken

No original criterion remains MISSED. The agent is not perfect: transient provider errors appeared in BEFORE, persistent outages can still exhaust the five-attempt limit, and retries add latency and request cost. Both evaluations repeat only one query per criterion. Search is keyword overlap, so it can return listings matching only some words; the observed denim-jacket search also returns jeans and a track jacket. Generated captions sometimes add unsupported sales urgency or first-person ownership claims (for example BEFORE Criterion 4 try 5 says to check out “my Poshmark closet”). Those claims are outside the five existing criteria and remain unfixed. Next I would test broader search relevance and factual caption grounding, then separately measure a targeted change. I stopped after one behavioral improvement to keep this Unit 4 comparison honest and within the assignment scope.

Human review: check the PASS/FAIL judgments, especially semantic item mentions and equivalent prices ($42 and $42.00), confirm the AI-use account and earlier user-reported invalid-key test, and review the unchanged Unit 3 criteria before submitting the repository URL. No assignment was submitted externally.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
