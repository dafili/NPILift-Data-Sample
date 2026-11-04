# US Dental Clinics — NPI Data Sample

**200 Texas dental practices, with street addresses and direct phone numbers.**
Exported from the US federal NPPES NPI Registry. Free to use, no attribution required.

**[→ Get the full dataset for any specialty and state](https://npilift.dafili.cn/)**

---

## What's in this repo

| File | What it is |
|---|---|
| `sample-200-texas-dental-clinics.csv` | 200 Texas dental practices, 11 columns, UTF-8 with BOM (opens cleanly in Excel) |

That's it. This repo exists to prove the data quality before you spend money on it.

### The sample, measured

| Metric | Value |
|---|---|
| Records | **200** |
| With a phone number | **200 / 200 (100%)** |
| Organisation records (a practice, not an individual) | **200 / 200** |
| Cities covered | **60** |
| Top cities | Dallas 29 · Fort Worth 20 · Austin 16 · Arlington 11 · Houston 10 |

### Columns

```
organisation, first_name, last_name, credential, specialty,
address_1, address_2, city, state, postcode, phone
```

---

## Where this data comes from

The **NPPES NPI Registry**, published by the US Centers for Medicare & Medicaid Services (CMS):

> https://npiregistry.cms.hhs.gov/

Every healthcare provider in the United States who wants to be paid by Medicare must register and obtain an NPI number. That legal requirement is what makes this dataset unusually complete — a practice that doesn't register cannot bill Medicare, so it cannot run a normal business. Coverage is close to total.

It is a **public federal dataset**, explicitly published for public use. Hospitals, insurers and researchers all query it. The data is not scraped, not purchased from a broker, and not behind a login.

---

## Why this is harder than it looks

The registry has an API, so exporting it sounds trivial. It is not, for one specific reason:

**The API silently caps every query at 1,200 records.**

Ask for more and it does not error, warn, or stop. It returns `skip`-offset pages until roughly 1,200, then **keeps returning the same final page over and over forever**.

I verified this directly:

```
skip=1000 → first record 1336316637
skip=1200 → first record 1336316637   ← identical
skip=1400 → first record 1336316637   ← still identical
```

So a naive exporter asks for "all dentists in Texas", receives about 1,200 rows, sees no error, and concludes it has everything.

**Texas alone has about 16,000 dentists.** Anyone selling you a Texas dentist list with ~1,200 rows exported it wrong.

### How to work around it

The cap applies per query, not per dataset. Split the query into independent slices that each fall under the ceiling:

- `enumeration_type` — `NPI-1` (individuals) and `NPI-2` (organisations) are counted separately, so each can return its own 1,200.
- `city` — each city in a state is its own query with its own ceiling.

Combined, a Texas dentist pull goes from 1,200 rows to roughly 16,000. That is the entire trick, and it is not a secret — the API is free and you can implement it yourself. This repo and the tool below exist because doing it properly takes an afternoon, and not everyone wants to spend that afternoon.

---

## The field most people miss: `enumeration_date`

If you are selling into practices or recruiting for them, **filter by `enumeration_date`**.

A practice that registered in the last six months has just opened. Which usually means:

- equipment is not fully purchased yet
- no incumbent supplier relationships
- the owner is still doing their own procurement
- decision chains are short

A list of practices open for twenty years is mostly noise. A list of practices open for six months is a queue of people who need things right now.

If you are doing cold outreach into dental, this one field is worth more than the rest of the row combined.

---

## Practical notes

**These are practice phone numbers, not consumer data.** For a large share of records the number is the clinic's own line, which is what you want if you are calling a practice about equipment, staffing or services.

**About 1% of records nationwide have no phone number** — the provider simply never supplied one. This sample happens to be 100%, because records without a phone were filtered out.

**Compliance is your responsibility.** The dataset is public and free to use. How you contact people is not covered by that. In the US, cold calls fall under the TCPA and commercial email under CAN-SPAM. Do not buy a list and blast it; that is a you problem, not a data problem.

---

## Getting more

The web tool does the same export for any specialty in any state, past the 1,200-row cap, and returns a CSV in about a minute:

### **[npilift.dafili.cn](https://npilift.dafili.cn/)**

- 18 specialties — dentists, physicians, chiropractors, physical therapists, optometrists, podiatrists, pharmacists, nurse practitioners, and more
- all 50 states plus DC and territories
- $29 for a full export, one time, no subscription, no account
- free to run — you see the first 25 rows before paying

If you only need a few hundred rows, **use the free registry and implement the sharding yourself** — the API costs nothing and the method is described above. This exists for people who would rather pay $29 than spend an afternoon on it.

---

## Licence

The **data** is public domain (US federal government work).

The sample CSV in this repo is released under **CC0** — copy it, resell it, put it in your product, no attribution needed.

---

*Questions: open an issue, or email support@dafili.cn*
