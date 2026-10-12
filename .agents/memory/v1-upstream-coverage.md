---
name: V1 upstream coverage model
description: Current product-model decisions for the compliance engine.
---

The V1 is a standalone, API-first REP/EPR compliance engine. Solenda remains the sourcing platform.

Core goal: determine whether sufficient upstream REP/EPR coverage can be demonstrated for the product in the target market so the seller does not have to carry a new own REP/EPR obligation for applicable streams.

Minimal per-analysis facts:
- EAN/GTIN/UPC/ASIN;
- supplier_stock_country = physical country where the product is stocked at sourcing time;
- target_marketplace = one of FR/DE/IT/ES/GB.

Persistent seller facts:
- seller legal entity;
- establishment country;
- optional VAT countries as context.

V1 explicitly does NOT model:
- prep centers;
- transit;
- logistics routes;
- FBA inventory country;
- Amazon redistribution;
- bundles;
- repackaging or packaging modification.

Do not guess Amazon storage or logistics.

V1 product assumption: original product and original commercial packaging, no bundle/repackaging.

Streams: PACKAGING, EEE/WEEE, BATTERY. Every analysis includes all streams with applicability APPLICABLE, NOT_APPLICABLE, or UNRESOLVED.

An applicable stream is OUI only when upstream coverage is verified. A seller-owned registration is not the favorable path targeted by V1.

Never equate registration number found with product coverage verified. Return numbers even when coverage remains unproven.

User-facing verdicts are OUI/NON. Technical states and reason codes stay in English; user explanations are in French.

On a NON result, still surface all verified registrations and successful checks, then highlight the blocking or unresolved point.

UNKNOWN is never compliant.
