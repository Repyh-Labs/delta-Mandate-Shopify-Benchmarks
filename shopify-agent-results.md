---
title: "Shopify MCP Benchmark — Full Results (100 intents)"
date: 2026-06-22
tags: [benchmark, shopify, mcp, delta, agent-shopping]
---

# Shopify MCP Benchmark — Full Results

## Headline

**27.3% of products the agent claimed to find were wrong** (purchase error rate, aka false discovery rate) — the agent presented products as matching all constraints when the catalog data did not contain the evidence. Error rate scales with constraint count.

| Tier | Total | Found | Pass | Correct | False Positives | Purchase Error Rate |
|---|---|---|---|---|---|---|
| Easy (1–2 constraints) | 30 | 27 | 3 | 23 | 4 | **14.8%** |
| Medium (3–4 constraints) | 40 | 32 | 8 | 21 | 11 | **34.4%** |
| Hard (5+ constraints) | 30 | 7 | 23 | 4 | 3 | **42.9%** |
| **Overall** | **100** | **66** | **34** | **48** | **18** | **27.3%** |

## Methodology

- **100 taxonomy-aligned purchase intents** (every constraint expressible in delta's product taxonomy)
- Agent selected products from Shopify UCP Global Catalog search results (structured product data via `ucp catalog search`)
- Agent instructed to return PASS if no product satisfies ALL constraints
- 66 of 100 intents resulted in FOUND; 34 resulted in PASS
- Each FOUND product verified by:
  1. Fetching actual product page via curl
  2. Checking every constraint against catalog data (title + description from Shopify structured fields)
  3. Cross-checking against delta's enforcement engine results where available
- **Correct** = all constraints confirmed in catalog data; **Error** = one or more constraints not found in evidence
- **Pass** = agent returned no product (gave up). Pass outcomes are classified as:
  - **False negative (FN)** — a valid product exists but the agent didn't find it (delta's pipeline found and verified one)
  - **True negative (TN)** — no valid product exists in the catalog; agent was correct to give up
- Evaluation corrected using delta's engine as ground truth — 3 cases where my text-based eval was too strict (features in structured data not visible in page text) were corrected to "correct"; 5 cases where my eval used intuition ("by definition slip-on") were corrected to "false positive" after confirming the terms do NOT appear in the product's catalog data

## Key Finding: Every Failure is Missing Evidence, Not Wrong Attribute

Every single error is a feature the agent *claimed* was satisfied but the catalog data doesn't confirm. Not wrong color, not wrong size, not wrong price — **missing evidence**.

**Two failure patterns:**

1. **Feature not in catalog data** (13 cases): The product page/catalog data simply doesn't mention "dishwasher safe", "non-insulated", "machine washable", "reusable", "USB-C" etc. The agent asserted the constraint was satisfied without evidence.

2. **Agent used intuition instead of evidence** (5 cases): The agent inferred a property from the product type rather than extracting it from catalog data:
   - 041: Claimed "slip-on" because Chelsea boots are *by definition* slip-on — but the term doesn't appear in the catalog data
   - 046: Claimed "reusable" because silicone baking mats are *by definition* reusable — but not in the catalog data
   - 052: Claimed "dishwasher safe" because stainless steel is *inherently* dishwasher safe — but not in the catalog data
   - 066: Claimed "machine washable" because cotton pajamas are *typically* machine washable — but not in the catalog data
   - 081: Claimed "single origin" because Pinhead Gunpowder is a specific tea varietal — but not in the catalog data

   All 5 terms **do exist** in the Shopify taxonomy (other products in the same search results have them explicitly). But the specific products the agent selected **do not** have them. The agent used product-type knowledge, not catalog evidence.

This is exactly the failure mode delta's enforcement engine is designed to catch: the agent *claims* a constraint is satisfied, but the product evidence doesn't confirm it. A deterministic verification layer requiring taxonomy-grounded evidence for each constraint would block all 18 false positives.

## Full Results Table (100 items)

| ID | Diff | Intent | Product | Verdict | What Failed |
|---|---|---|---|---|---|
| 001 | easy | White cotton t-shirt, size M | Versace White Cotton Crew Neck T-Shirt Size M | ✅ Correct | — |
| 002 | easy | Black leather wallet, under $50 | Black Leather Wallet ($20.00) | ✅ Correct | — |
| 003 | easy | Blue denim jeans, size 32, straight-leg fit | Diesel Straight Leg Denim Jeans Size 32 ($29) | ✅ Correct | — |
| 004 | easy | Stainless steel water bottle, vacuum-insulated | Stainless Steel Double Wall Vacuum Insulated Bottle ($14.99) | ✅ Correct | — |
| 005 | easy | Bamboo cutting board, under $30 | Undercut Bamboo Cutting Board ($23.99) | ✅ Correct | — |
| 006 | easy | Red ceramic mug, non-insulated | Red Ceramic Mug ($14.00) | ❌ Wrong | 'non-insulated' not in catalog data |
| 007 | easy | Black wireless mouse, under $40 | Logitech M325s Wireless Mouse Black ($22.99) | ✅ Correct | — |
| 008 | easy | Cotton tote bag, open top | Seaside Cotton Tote Bag ($4.48) | ✅ Correct | — |
| 009 | easy | Green terry bath towel, solid pattern, under $20 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 010 | easy | White athletic socks for running, size large, moisture-wicking | Feetures Elite Running Socks White Large ($16.19) | ✅ Correct | — |
| 011 | easy | Black beanie, acrylic, one size | AC/DC Logo Beanie Black Acrylic One Size ($23) | ✅ Correct | — |
| 012 | easy | White ceramic decorative bowl, round shape | 5in Wide White Ceramic Bowl ($7.99) | ✅ Correct | — |
| 013 | easy | Brown leather gloves, size M | Padded Leather Gloves Size M Brown ($15.47) | ✅ Correct | — |
| 014 | easy | Grey waterproof laptop backpack | Waterproof Laptop Backpack Grey ($79) | ✅ Correct | — |
| 015 | easy | Blue moisture-wicking shorts, size M | Blue Mesh Shorts M ($38) | ❌ Wrong | 'moisture-wicking' not confirmed in catalog data |
| 016 | easy | Wooden picture frame, 5x7 inch | 5x7 White and Black Wooden Picture Frame ($29) | ✅ Correct | — |
| 017 | easy | Black silicone battery phone case with USB type-C charging | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 018 | easy | Green enamel pin, under $10 | Green Mini Heart Enamel Pin ($7.50) | ✅ Correct | — |
| 019 | easy | White cotton bandana, large size, solid pattern | Plain Bandana White Large ($4.99) | ✅ Correct | — |
| 020 | easy | Stainless steel keychain, under $10 | Stainless Steel Keychain ($6.48) | ✅ Correct | — |
| 021 | easy | Grey sweatpants, size L | Polo RL Grey Sweatpants Size L ($37) | ✅ Correct | — |
| 022 | easy | Black leather belt for men | DUNDEE Mens Black Genuine Leather Belt ($13) | ✅ Correct | — |
| 023 | easy | Blue ceramic round planter | Blue Round Ceramic Planter ($325) | ✅ Correct | — |
| 024 | easy | White cotton pillowcase, standard size | Cotton Standard Pillowcase White ($8.99) | ✅ Correct | — |
| 025 | easy | Natural beeswax container candle | Beeswax Container Candle ($30) | ✅ Correct | — |
| 026 | easy | Black nylon duffel bag, under $40 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 027 | easy | Brown kraft notebook, lined pages | Brunnen Notebook Kraft Lined ($3.39) | ✅ Correct | — |
| 028 | easy | Clear glass tumbler, non-insulated, dishwasher safe | George Home Clear Glass Tumbler (£2.71) | ❌ Wrong | 'non-insulated' not in catalog data |
| 029 | easy | Grey fleece throw blanket, medium warmth | LURKA Checkered Sherpa Fleece Throw Smoke Grey ($32.39) | ❌ Wrong | 'medium warmth' not in catalog data |
| 030 | easy | Black solid-pattern headband, under $10 | Solid Black Headband ($5) | ✅ Correct | — |
| 031 | med | Green crewneck sweater, size M, wool, crew neckline, under $100 | Song and Soul Crew Sweater Wool Army Green M ($79.99) | ✅ Correct | — |
| 032 | med | Mechanical keyboard, TKL layout, tactile switches, under $150 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Vulcan TKL Tactile (ROCCAT)) |
| 033 | med | Organic cotton bed sheets, queen size, machine washable, white only | ORGANIC TEXTILES Cotton Sheets Queen White ($333) | ❌ Wrong | 'machine washable' not confirmed in catalog data |
| 034 | med | Leather crossbody bag, brown, under $200, with a magnetic closure | Genuine Leather Magnetic Snap Crossbody Bag Dark Brown ($36) | ✅ Correct | — |
| 035 | med | Cold brew coffee concentrate, dark roast, sugar-free, under $25 | Cold Brew Concentrate ($7) | ✅ Correct | — |
| 036 | med | Yoga mat, purple, TPE material, solid pattern, under $50 | Purple TPE Yoga Mat ($40.22) | ✅ Correct | — |
| 037 | med | Stainless steel French press, double wall insulated, under $40 | French Press 100% Stainless Steel Double Wall ($26) | ✅ Correct | — |
| 038 | med | Linen shirt, size L, blue, long sleeve, machine washable | Men's 100% Linen Long Sleeve Shirt Blue L ($43.98) | ❌ Wrong | 'machine washable' not in catalog data |
| 039 | med | Ceramic pour-over coffee dripper, manual, white, under $25 | White Ceramic Pour Over Dripper Manual ($8.99) | ✅ Correct | — |
| 040 | med | Resistance band set, medium resistance level, solid pattern, under $30 | Train Resistance Bands 5-Band Set ($29.99) | ❌ Wrong | 'medium resistance' not in catalog data |
| 041 | med | Suede Chelsea boots, size 9, Brown, slip-on, under $150 | Brown Clarks Suede Chelsea Boots Size 9 ($25) | ❌ Wrong | 'slip-on' not in catalog data — agent used intuition |
| 042 | med | Merino wool activewear top, long sleeves, size S, black, moisture-wicking | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 043 | med | Glass water bottle, BPA-free, double-walled, with screw-on lid | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 044 | med | Hemp protein powder, vanilla flavor, powder form, under $35 | Hemp Foods Australia Vanilla Hemp Protein ($31) | ✅ Correct | — |
| 045 | med | Cotton bath towel, grey, solid, terry weave | 2-Pack Gray Cotton Bath Towels Terry ($39.99) | ✅ Correct | — |
| 046 | med | Silicone baking mats, reusable, dishwasher safe, under $20 | Non-Stick Silicone Baking Mat Dishwasher Safe ($19.99) | ❌ Wrong | 'reusable' not in catalog data — agent used intuition |
| 047 | med | Leather card holder, black, compact design, under $40 | Double C Card Bag Black Leather ($38.80) | ❌ Wrong | 'compact design' not in catalog data |
| 048 | med | Cotton joggers, size M, heather grey, tapered leg fit | SKIMS Cotton Fleece Classic Jogger Heather Grey M ($88) | ❌ Wrong | fabric is Fleece not Cotton — Delta confirmed |
| 049 | med | Cast iron skillet, pre-seasoned, oven safe, under $40 | 3Pcs Pre-Seasoned Cast Iron Skillet Set Oven Safe ($17.49) | ✅ Correct | — |
| 050 | med | Canvas apron, unisex, solid pattern, with pockets, under $25 | Canvas Apron with Front Pocket ($10.50) | ✅ Correct | — |
| 051 | med | Bamboo utensil set, dishwasher safe, includes spatula, under $15 | 5-Piece Bamboo Cooking Utensil Set ($12.99) | ❌ Wrong | 'includes spatula' not confirmed in catalog data |
| 052 | med | Stainless steel lunch box, snap-on lid, dishwasher safe, under $35 | Lock & Go Stainless Lunch Box ($28) | ❌ Wrong | 'dishwasher safe' not in catalog data — agent used intuition |
| 053 | med | Felt laptop sleeve, grey, zippered closure, with internal padding, under $45 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Felt Laptop Sleeve w/ Pocket) |
| 054 | med | Cotton bandana, red, solid pattern, unisex | Solid Red Bandana ($4.99) | ✅ Correct | — |
| 055 | med | Leather belt, black, large size, men's | Size Large Black Woven Leather Belt Men's ($29) | ✅ Correct | — |
| 056 | med | Flannel shirt, size XL, plaid pattern, machine washable, under $50 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Long Sleeve Flannel Plaid Shirt (Haggar)) |
| 057 | med | Standard size pillow, hypoallergenic, synthetic fill, under $30 | Economical Hotel Pillows Synthetic Down ($28) | ✅ Correct | — |
| 058 | med | Beeswax pillar candles, unscented, reusable, under $20 | 2 Pack Beeswax Pillar Candles Unscented ($19.99) | ❌ Wrong | 'reusable' not in catalog data |
| 059 | med | Ceramic planter pot, round, white, for indoors, under $20 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 060 | med | Stainless steel cocktail shaker, satin finish, dishwasher safe, under $30 | Cuisinox Satin Finish Cocktail Shaker ($15.26) | ✅ Correct | — |
| 061 | med | Suede loafers, size 10, slip-on closure, dress occasion, under $120 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 062 | med | Cotton hoodie, size L, navy blue, long sleeve, under $50 | Cotton World Fleece Pullover Hoodie Navy Blue L ($8.99) | ✅ Correct | Delta confirmed — long sleeve in structured data |
| 063 | med | Glass food storage containers, leak-proof, dishwasher safe, under $35 | 5-Pack Glass Food Storage Leak-Proof Dishwasher Safe ($31.41) | ✅ Correct | — |
| 064 | med | Leather duffel bag, large size, brown, under $200 | Brown Leather Duffel Bag Large ($109.99) | ✅ Correct | — |
| 065 | med | Wool blend overcoat, size M, charcoal, insulated, under $250 | Jasper Men's Wool Blend Overcoat Charcoal Down Insulated ($163.44) | ✅ Correct | — |
| 066 | med | Cotton pajamas, size L, striped, long sleeve, machine washable, under $45 | Cotton Striped Printed Long-sleeved Pajamas Set ($32) | ❌ Wrong | 'machine washable' not in catalog data — agent used intuition |
| 067 | med | Ceramic teapot, gooseneck spout, dishwasher safe, under $30 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 068 | med | Cotton area rug, rectangular, braided, under $80, multi-color | Hand Made Woven Chindi Cotton Area Rugs Turquoise Multi ($19.99) | ✅ Correct | — |
| 069 | med | Leather messenger bag, black, with laptop compartment, under $100 | Black Mens Leather Laptop Messenger Bag ($0.01) | ✅ Correct | — |
| 070 | med | Stainless steel tongue scraper, under $10, reusable, for breath freshening | Zefiro Tongue Scraper ($5) | ✅ Correct | — |
| 071 | hard | Wireless mechanical keyboard, TKL layout, gasket mount, PBT keycaps, per-key RGB backlight, under $150 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 072 | hard | Cotton bed sheets, queen size, solid pattern, white, machine washable, includes pillowcases, hypoallergenic certified, under $80 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: 100% Natural Cotton Sheets Queen) |
| 073 | hard | Merino wool sweater, crewneck, size M, green, solid color, long sleeves, under $120, machine washable | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 074 | hard | Leather messenger bag, brown, solid pattern, laptop compartment, buckle closure, adjustable strap, under $300 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Oak Handmade Leather Messenger Bag) |
| 075 | hard | Whole bean coffee, medium roast, coarse grind, regular caffeine, under $20, single origin, unflavored | Single-Origin Whole Bean Coffee Colombia ($19.99) | ❌ Wrong | 'coarse grind' not in catalog data |
| 076 | hard | Puffer jacket, size M, black or navy, solid pattern, lightweight, machine washable, under $200 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 077 | hard | Stainless steel cookware set, induction compatible, oven safe, uncoated, includes saucepan, dishwasher safe, under $250 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 078 | hard | Linen duvet cover, king size, grey or green, solid pattern, plain weave, button closure, machine washable, under $150 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Linen Duvet Cover Set King) |
| 079 | hard | Wireless over-ear headphones, active noise cancellation, closed-back design, USB-C charging, built-in microphone, under $200 | Oneodio A Series ANC Wireless Headphones ($199) | ✅ Correct | Delta confirmed — USB-C etc. in structured data |
| 080 | hard | Cast iron Dutch oven, enameled, oven safe, induction compatible, dishwasher safe, under $100 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Cast Iron Enameled Dutch Oven ($59)) |
| 081 | hard | Organic green tea, loose leaf, unflavored, single origin, regular caffeine, no artificial flavors, under $15 | Organic Pinhead Gunpowder Loose Leaf Tea ($4.99) | ❌ Wrong | 'single origin' + 'unflavored' not in catalog data — agent used intuition |
| 082 | hard | Leather Chelsea boots, size 10, brown, slip-on closure, round toe, dress occasion, waterproof, under $250 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 083 | hard | Cotton flannel fitted sheets, queen size, plaid pattern, machine washable, hypoallergenic certified, under $100 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 084 | hard | Stainless steel water bottle, vacuum insulated, flip-top lid, BPA-free, silver, dishwasher safe, under $30 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Stainless Steel Water Bottle Flip Top Silver) |
| 085 | hard | Wool area rug, rectangular, hand-tufted, low pile, under $300, neutral color (grey or beige), hand wash care | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 086 | hard | Mechanical watch, automatic movement, anti-reflective crystal, silver case, black dial, leather strap, under $300 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Automatic Watch Silver Black) |
| 087 | hard | Cotton bath towels, terry weave, solid pattern, grey or white, under $80 | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Hays Cotton Bath Towel Set Gray) |
| 088 | hard | Bamboo cutting board, end-grain construction, rectangular shape, hand wash only, under $50, not acacia wood | — | ⬜ PASS (FN) | False negative — valid product exists, agent missed it (delta found: Riveira Bamboo End Grain Cutting Board) |
| 089 | hard | Leather wallet, cowhide leather, solid pattern, card organization, RFID blocking, under $75, brown or beige, not black | Genuine Cowhide Leather Wallet RFID Brown ($23) | ✅ Correct | Delta confirmed — my eval had false positive |
| 090 | hard | Cold brew maker, glass, clear, manual, works with coffee beans, under $40, not plastic | Glass Cold Brew Coffee Maker ($18.99) | ✅ Correct | — |
| 091 | hard | Wireless earbuds, IPX7 waterproof rating, USB-C charging, active noise cancellation, dynamic driver, under $100 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 092 | hard | Canvas tote bag, large size, beige, under $25, dual handles, lightweight, solid (unprinted) | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 093 | hard | Stainless steel French press, silver, double-wall insulated, manual (no power required), dishwasher safe, under $50, with metal mesh filter | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 094 | hard | Cotton t-shirt, size L, crew neck, solid pattern, machine washable, moisture wicking, under $30, not white, not black | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 095 | hard | Leather journal cover, solid pattern, brown, not black, under $60, with elastic closure, leather book cover material | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 096 | hard | Merino wool throw blanket, solid pattern, navy or gray, not acrylic, warm-rated, hand wash, under $100 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 097 | hard | Ceramic dinnerware set, matte finish, solid pattern, round shape, white, dishwasher and microwave safe, under $100 | 6 Piece Ceramic Dinnerware Set Matte White ($67.98) | ✅ Correct | — |
| 098 | hard | Stainless steel mixing bowl set, nesting, with lids, round shape, under $40, dishwasher safe | Stainless Steel Mixing Bowls with Lids Nesting ($37.48) | ❌ Wrong | 'round shape' not in catalog data |
| 099 | hard | Leather backpack, brown, solid pattern, laptop compartment, zipper closure, waterproof, chest/waist strap, under $250, not black | — | ⬜ PASS (TN) | True negative — no valid product found by either system |
| 100 | hard | Wireless mechanical keyboard, 75% layout, gasket mount, PBT keycaps, RGB backlight, N-key rollover, under $130 | — | ⬜ PASS (TN) | True negative — no valid product found by either system |

## Methodology Notes

- Product pages fetched via `curl --compressed` with mobile user agent
- Constraint checking is strict: a feature must appear in the product's catalog data (title or Shopify structured description) to be considered satisfied
- "By definition" reasoning (e.g., "Chelsea boots are inherently slip-on") is NOT accepted as evidence — the term must be extractable from the product's catalog data
- Evaluation was corrected using delta's enforcement engine as ground truth: 3 false positives in my text-based eval were corrected (062, 079, 089 — features were in structured data my text fetch missed), and 5 "ambiguous" cases were confirmed as false positives after verifying the terms do NOT appear in the product's catalog data
- 11 of 34 PASS intents are false negatives (a valid product exists but the agent couldn't find it); 23 are true negatives (no valid product exists)
- Raw data: `benchmark-v2/eval-shopify/all-66.json` and `benchmark-deltamandate-results-aligned.md`
