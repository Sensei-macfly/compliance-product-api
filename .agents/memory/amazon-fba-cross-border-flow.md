---
name: Amazon FBA cross-border flow
description: The user's sales and supply-chain model relevant to this EPR compliance project.
---

The user's company is the seller on Amazon. It buys products from suppliers and wholesalers in France, Germany, Italy, Spain and more broadly the EU. Products can transit through a prep center in another European country before Amazon FBA and resale across European markets. The product should analyze each supported source/storage-to-target-market combination to determine whether the user's company needs its own EPR registration or can rely on a specifically identified upstream entity, conservatively when the link is not proven.

**Why:** the user stated this business model; the answer depends on the full seller and fulfillment route, not just a product identifier or one supplier-country field.

**How to apply:** represent supplier establishment, physical stock, prep-center locations, FBA program and known inventory countries, target market, seller entity and claimed upstream responsible entity separately. Do not infer that an upstream registration covers the seller, product/SKU, category or route without applicable rules and evidence.

For the user-facing decision, expose only OUI/NON for whether the route can proceed without a new seller registration. Keep detailed states such as UNVERIFIED and their reason codes internal; explain a NON due to missing proof without presenting it as a legal prohibition or a proven duty to register.

**Why:** the user explicitly wants internal uncertainty states preserved without turning them into a third commercial decision.

**How to apply:** return OUI only when the required evidence establishes a favorable result; return NON for all other internal states, with plain-language explanation and evidence details.
