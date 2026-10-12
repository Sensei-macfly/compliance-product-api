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
- optional Amazon/FBA facts explicitly known from Solenda, Amazon or another reliable source;
- optional existing seller EPR/REP/IDU registrations.

Existing seller EPR registrations are not a central UX requirement, but the engine must support and use them when available to determine whether a new registration is actually needed.

Per-analysis facts include:
- EAN/GTIN/UPC/ASIN;
- supplier/source stock country;
- target marketplace;
- optional supplier/product/Amazon enrichment.

Do not ask the client to declare which FBA countries are "possible" as a guess. Do not require the client to manually construct a route. Generate reasonable candidate routes from facts and documented rules.

Keep supplier stock, prep/transit, FBA inventory country and target marketplace as separate route facts.

VAT-registration countries are context only. They do not prove FBA storage eligibility, EPR compliance, or upstream coverage.

Amazon SP-API is optional. The engine must work without a Professional Amazon account.

The dashboard and Solenda must call the same business API. Solenda may provide richer facts, but the compliance engine must not read the Solenda database directly.

User-facing output is uniformly **OUI/NON**. Internal states may be VERIFIED, UNVERIFIED, NON_COMPLIANT or CONTRADICTORY.

Results must be represented at least at **route × target market × EPR stream** granularity. A global verdict may exist only as a summary.

UNKNOWN is never treated as compliant.

A NON caused by missing evidence means the engine cannot prove that the route avoids a new registration. It does not automatically mean the sale is legally prohibited or that seller registration is definitively required.

An upstream EPR/IDU number is never enough by itself to prove coverage of the seller, product/SKU, category, market or route.


Additional V1 contract rules:
- one analysis request targets one marketplace; multi-market analysis is orchestrated through multiple calls;
- each candidate route uses ordered typed segments: SUPPLIER_STOCK, TRANSIT, PREP_CENTER, FBA_INVENTORY, MARKETPLACE;
- PREP_CENTER segments must reference a real client-profile prep center by id;
- every route must include all V1 EPR streams (PACKAGING, EEE/WEEE, BATTERY) with applicability APPLICABLE, NOT_APPLICABLE or UNRESOLVED;
- any unresolved stream applicability or unresolved overall stream scope blocks OUI;
- a route cannot become OUI by omitting an uncertain stream.
