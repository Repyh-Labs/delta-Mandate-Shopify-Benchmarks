# Agent Shopping Benchmark — without vs with delta Mandate

Benchmark comparing AI agent purchase accuracy on Shopify with and without delta Mandate's intent enforcement layer.

## Headline

When given 100 purchase intents across the Shopify catalog, a Shopify UCP CLI-equipped agent bought the wrong product **27.3% of the time** (42.9% on hard, multi-constraint intents). With delta Mandate's verification layer: **0%**.

## Repository contents

| File | Description |
|------|-------------|
| [`intents.md`](intents.md) | The 100 taxonomy-aligned purchase intents, graded by difficulty (easy / medium / hard), with full change log documenting how each intent was adapted to use Shopify taxonomy attributes |
| [`shopify-agent-results.md`](shopify-agent-results.md) | Full results for the Shopify CLI control agent: all 66 found products verified against catalog data, with 18 errors identified and categorized |
| [`mandate-results.md`](mandate-results.md) | Full results for the delta Mandate agent across all 100 intents, showing per-intent verification evidence and pass/fail/error status |
| [`comparison-analysis.md`](comparison-analysis.md) | Side-by-side comparison: confusion matrix, error analysis, discovery vs enforcement distinction, and the 14 disagreement cases |

## Methodology

- **100 purchase intents** generated to span the full range of difficulty, from single-constraint lookups ("Bamboo cutting board, under $30") to multi-constraint combinations that may not exist ("Leather journal cover, solid pattern, not black, under $60, with elastic closure, leather book cover material")
- Every constraint mapped to a Shopify product taxonomy attribute (see change log in `intents.md`)
- Both agents used Shopify's UCP CLI (Merchant Tools Protocol + UCP catalog search) for product discovery
- The delta Mandate agent additionally ran a policy engine that verifies each candidate product's extracted evidence against the user's constraints before allowing purchase
- Both agents were instructed to pass if no suitable product was found, rather than picking a product that doesn't fit

## Results

### Confusion matrix

|  | Shopify MCP agent | delta Mandate agent |
|--------|-------------------|---------------------|
| **True positive** (valid product found and purchased) | 48 | 56 |
| **True negative** (no valid product exists, correctly passed) | 23 | 44 |
| **False positive** (purchased a product that violated constraints) | 18 | 0 |
| **False negative** (passed when a valid product existed) | 11 | 0 |
| **Total** | 100 | 100 |

> **Read the negative rows with care.** True/false *negatives* reflect **discovery** — whether each system's search surfaced a valid product — not enforcement, and "no valid product exists" was judged from each pipeline's own results rather than an independent label. The two columns therefore aren't a clean apples-to-apples comparison (delta's pipeline also searches differently). 5 of delta's 44 negatives were intents that timed out or errored under concurrency stress: no purchase was made (the safe outcome), but the engine returned no verified verdict — and the Shopify control wasn't run under the same concurrency, so that reliability dimension isn't comparable. The clean enforcement number is the purchase error rate below.

### Error rates

The metric that matters for delegated spending is the **purchase error rate** — of the products the agent claimed to find, what fraction violated at least one constraint:

| Metric | Formula | Shopify agent | delta Mandate |
|--------|---------|---------------|---------------|
| **Purchase error rate** | FP / (TP + FP) | **27.3%** (18/66) | **0.0%** (0/56) |

It assumes nothing about whether a valid product *exists* — it asks only whether what the agent approved was actually valid — so it is a clean, apples-to-apples enforcement comparison. 27.3% of the time the Shopify agent said "I found it," the purchase violated a constraint. With delta Mandate's verification in front of the same agent: **0%**.

On hard intents (5+ constraints), the purchase error rate hit **42.9%**.

### What the Shopify agent got wrong

Every false positive was a case where the agent asserted a constraint was satisfied without evidence in the catalog data:
- **13 cases:** the constraint wasn't present anywhere in the product data. The agent claimed it was satisfied anyway.
- **5 cases:** the agent used product-type intuition ("stainless steel is inherently dishwasher safe") rather than extracting explicit evidence from the catalog.

## About delta Mandate

[delta Mandate](https://delta.network/mandate) is an intent enforcement layer for agentic commerce. It sits between an agent's discovery step and the payment, extracting structured evidence from product data and evaluating it against the user's constraints. If the verification passes when it shouldn't have, delta reimburses the user through the Mandate Guarantee.

## License

Benchmark data is provided for verification and reproducibility. Contact [delta](https://delta.network) for questions.
