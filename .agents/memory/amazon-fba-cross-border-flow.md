---
name: Amazon FBA cross-border flow
description: Product-model decisions for the EPR compliance engine.
---

The product is a standalone, API-first compliance engine that may later be integrated into Solenda. Solenda remains the sourcing platform.

The seller buys products from suppliers/wholesalers across Europe. Stock may transit through prep centers before Amazon FBA and sale on European marketplaces.

Core product rule: **clients provide facts; the engine produces deductions**.

Persistent client facts can include:
- seller legal entity and establishment country;
- VAT-registration countries;
- prep centers actually available;
- optional Amazon/FBA facts explicitly known from Solenda, Amazon or another reliable source.

Per-analysis facts include:
- EAN/GTIN/UPC/ASIN;
- supplier/source stock country;
- target marketplace;
- optional supplier/product/Amazon enrichment.

Do not ask the client to declare which FBA countries are "possible" as a guess. Do not require the client to manually construct a route. Generate reasonable candidate routes from facts and documented rules.

Existing seller EPR registrations are not a central V1 input because the product's primary goal is to identify routes that avoid requiring a new EPR/REP/IDU registration. The model may remain extensible for existing registrations later.

VAT-registration countries are context only. They do not prove FBA storage eligibility, EPR compliance, or upstream coverage.

Amazon SP-API is optional. The engine must work without a Professional Amazon account.

The dashboard and Solenda must call the same business API. Solenda may provide richer facts, but the compliance engine must not read the Solenda database directly.

User-facing output is binary YES/NO. Internal states may be VERIFIED, UNVERIFIED, NON_COMPLIANT or CONTRADICTORY. UNKNOWN is never treated as compliant.

An upstream EPR/IDU number is never enough by itself to prove coverage of the seller, product/SKU, category, market or route.
