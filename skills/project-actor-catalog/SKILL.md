---
name: "project-actor-catalog"
description: "Build or update a project actor catalog (people, teams and systems that touch the product) in ASD-STE100 style, as a shared doc that feeds the C4 diagrams."
---

# Project actor catalog

An actor is a person, a team or a system that sends data to the product, gets data from the product, or changes the product. The catalog lists every actor. It is the single input for the C4 diagrams (see the `c4-figjam-diagrams` skill), so the names in the catalog and in the diagrams must be the same.

## 1. Collect the sources

Read all the sources before you write anything. Record which source supports each actor.
- Scoping packet, SOW, proposal, or estimate sheet.
- PMO or delivery plan (for example a Google Sheet launch plan).
- Meeting notes (kickoff, discovery). These often change earlier decisions.
- The live site or product. Check what is really there (forms, cart, login, domains, image hosts, tags, consent tool).
- Project memory and earlier threads, for decisions that are already made.

If two sources do not agree, write down both values and add an open question. Do not choose one value without a reason.

## 2. Use this structure

Create a Claude Doc (or update the existing one) titled `<Product> actor catalog`. Give it these sections:

1. **Scope**: what is in scope and what is not (other brands, other sites). Name the sources and the date you checked them. Say that release timing is not in the catalog.
2. **A. External human actors**: visitors, leads, customers, dealers or partners, owners, account holders.
3. **B. Client team actors**: sponsors, CTO, content editors, translation approvers, data owners (PIM, DAM, CRM per org, dealer data), legal/privacy, analytics, SEO, paid media, IT/DNS, legacy platform owner, UAT owner.
4. **C. Agency/delivery team actors**: PM, development lead, developers by type, DevOps, designer, QA, SEO, analytics.
5. **D. System actors in the target architecture**: browser, DNS/CDN, edge router, redirect map, CMS/site, localization, middleware and its parts, PIM, DAM, CRM orgs, feeds, maps, consent, tag manager, analytics, testing, search console, monitoring, secrets, repository and CI.
6. **E. Legacy, adjacent and automated actors**: legacy platform and its features, old image hosts, old DAM, country or partner sites, out-of-scope stores, search and AI crawlers, external link consumers, bots and spam senders.
7. **Open questions about actors**: one question in each line.
8. **Sources**.

Each group is a table with these columns: `ID | Actor | Description (STE) | Interfaces | Source`.
- IDs use the group letter and a number (A1, B7, D14). Do not reuse an ID after you delete an actor. Mark the actor as removed.
- **Interfaces** names the actors or systems that this actor touches. Use the exact names from the other rows.
- **Source** values: `Live site`, `Packet`, `Plan`, `Kickoff`, `Research notes`, `Plan comments`, or `Assumed`. `Assumed` means that no source confirms the actor.

## 3. Writing rules (ASD-STE100)

- Short sentences, with a maximum of 20 words. Use the active voice.
- Write one fact or one instruction in each sentence.
- Always use the same word for the same thing. Do not write "lead" in one row and "inquiry" in another.
- Product and system names are technical names. You can use them as they are.
- Describe what the actor does: "The lead service records each form submission first. Then the service sends the lead to Salesforce NA or EU."
- When a decision is not made, say so: "The team must decide if VWO stays."
- Add a line in Scope which says that you did not check each word against the STE dictionary, unless you did.

## 4. Keep the catalog current

- When a decision changes an actor (for example "no nightly sync, Akeneo events trigger the middleware"), update that row and every row that names it in **Interfaces**. Remove or update the open question that the decision answers.
- When you add a system actor, also check if the C4 L1 or L2 diagram needs the same change. Tell the user which diagram changes, or run the `c4-figjam-diagrams` skill if they ask.
- Read the doc again before you edit it, because other people can edit it too.
- Save the doc link and the key decisions in project memory.

## 5. Reply to the user

Give the doc link, the number of actors in each group, the new or changed actors, and the open questions that need the client. Do not paste the tables into the chat.