# Market Research Dossier — iitiimshaadi.com Teardown & Opportunity Map

**Prepared for:** iitiimrishte.com (a broader, international, verified premium-professional matchmaking platform)
**Date:** 2026-07-22
**Method:** 12 parallel research agents, each covering a distinct angle. Every claim below links to the original source. Raw per-angle outputs live in [`docs/research/raw/`](./raw/).

> **Evidence caveat (read first).** The target site (`iitiimshaadi.com`), the app stores, Reddit, Quora, and most review aggregators were **not directly fetchable** from the research environment (HTTP 403 at the egress proxy; several are additionally login-gated or bot-blocked). Findings therefore rest on **search-engine-indexed snapshots** of those real pages, not first-hand page loads. Company scale/pricing numbers originate largely from **company press releases and third-party data aggregators and are unaudited**. Reddit specifically returned **zero** usable data (network-blocked, not absent) — TeamBlind and Quora serve as the professional-audience analog. Treat specific figures as *reported, not independently verified*, and re-confirm the load-bearing ones (pricing, active-member counts, refund terms) against the live pages before betting on them.

---

## 1. Executive Summary

iitiimshaadi.com is a niche, **education-gated** Indian matrimony platform for alumni of premier institutes (IITs, IIMs, ISB, BITS, NITs, Ivy/global top schools), operated by **Alma Mater Matters Private Limited** (Gurugram; CIN U74140HR2015PTC081038), launched April 2014 by father-son founders **Ajay Kumar Gupta** and **Taksh Gupta** (Founder-CEO) ([ZaubaCorp](https://www.zaubacorp.com/ALMA-MATER-MATTERS-PRIVATE-LIMITED-U74140HR2015PTC081038); [Modern Diplomacy](https://moderndiplomacy.eu/2022/04/04/taksh-gupta-founder-of-iitiimshaadi-matrimonial-website/)). Its single differentiator is **mandatory manual verification of educational documents** before a member can access the database ([Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_US)).

The brand is bootstrapped, small (~35–42 staff, ~₹5 Cr revenue band), and leans hard on a celebrity anchor — **Karan Johar, brand ambassador since 2022, renewed for a third term in 2025** ([Tracxn](https://tracxn.com/d/companies/iitiim-shaadi/__cT3hCA88BeEPKbcmWYZyNog-muG6TRqPwo5XQGouRS4); [ThePrint](https://theprint.in/ani-press-releases/karan-johar-continues-as-brand-ambassador-of-iitiimshaadi-com-for-third-term/2572250/)). It **claims 5 lakh (500,000) members as of Nov 2025**, but its own earlier materials describe only **~150,000 in the database of which ~30,000 are active/authenticated** — a large, undefined gap between the marketing headline and the reachable pool ([Business Standard](https://www.business-standard.com/content/press-releases-ani/iitiimshaadi-com-crosses-5-lakh-members-strengthening-its-position-as-india-s-premier-matrimonial-platform-for-educated-professionals-125112500803_1.html); [AppAdvice](https://appadvice.com/app/iitiimshaadi-matchmaking-app/1626456060)).

**The five biggest opportunities for a better product:**

1. **Fix the liquidity/inventory problem.** The #1 complaint everywhere is a thin, mostly-inactive pool where "many members are on free accounts" so shortlisted people never reply ([App Store](https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060)). A better product must guarantee a *reachable, active* pool and publish honest active-user metrics.
2. **Solve the gender-imbalance response collapse.** ~60% male / 40% female means paying men get almost no responses — Quartz wrote an entire piece on it being "harder on men than women" ([Quartz](https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women)).
3. **Upgrade verification from "spoofable document upload" to real, multi-signal identity + credential + intent trust.** Manual marksheet review verifies a degree, not a person, marital status, or intent — and no Aadhaar/KYC backend exists ([PTC News](https://www.ptcnews.tv/iit-iim-shaadi-dot-com-bizarre-matrimonial-site-screening-degrees-id-cards-and-what-not); [India TV govt advisory](https://www.indiatvnews.com/news/india-user-verification-with-valid-ids-must-on-matrimonial-sites-govt-advisory-332427)).
4. **Rebuild the value equation.** ~₹26,000–₹33,000 for a small, unresponsive pool with a slow app reads as "not worth it"; the canonical Quora review is a 4-month paid member reporting results "null and void" ([Quora](https://www.quora.com/Is-IITIIMShaadi-com-legit)).
5. **Broaden the moat from "which school" to "verified premium achievement," internationally.** The strict institute gate is simultaneously the product's identity and its inventory ceiling and reputational liability (elitism/caste/gender backlash) ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony)). Broadening to all verified premium professionals across geographies grows liquidity *and* softens the brand risk — while keeping rigorous verification as the actual moat.

---

## 2. Product & Profile Anatomy

- **Positioning:** a "niche portal with focus on education" for premier-institution alumni across many disciplines — Engineering, Management, Law, Medicine, Finance, Fashion, and more ([TechStory](https://techstory.in/iitiimshaadi/); [Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_IN)).
- **Profile URL structure (confirmed):** member profiles live at `/Members/userProfile/{token}`, where the token is **Base64 of a sequential integer ID** — the example `OTMxODA=` decodes to `93180`. Enumerable IDs behind the login wall are a **scraping/privacy weakness** ([target URL](https://iitiimshaadi.com/Members/userProfile/OTMxODA=)).
- **Confirmed profile fields:** education/institute/discipline, plus contact info (phone + email) gated behind payment ([fee page](https://iitiimshaadi.com/fee)). **Inferred (typical, unconfirmed on the live login-gated page):** name, age/DOB, height, religion/community/caste, mother tongue, marital status, profession, income, location, family details, horoscope/manglik, photos ([ScoopWhoop](https://www.scoopwhoop.com/news/wedding-bells/)).
- **Free vs paid gating:** free = register + browse ~10 profiles; photos and full details are blurred. Paid unlocks full details, contacts, messaging, extra filters, and profile boost ([loving.is](https://www.loving.is/en/review/iitiimshaadi-com/)).
- **Site sections:** `/fee`, `/register`, `/faqs`, `/success-stories`, `/androidapp`, `/users/privacy`. Native iOS + Android apps (`com.tycho.iitiimshadi`, App Store `id1626456060`).

---

## 3. Pricing & Monetization

- **Hard pay-to-contact wall** — all communication requires payment ([fee page](https://iitiimshaadi.com/fee)).
- **Web (INR):** Yearly **₹29,500**, Six-Month **₹26,000** — a suspiciously small gap that undercuts the annual upsell. One report cites a **₹32,922 "until marriage"** fee ([loving.is](https://www.loving.is/en/review/iitiimshaadi-com/); [PTC News](https://www.ptcnews.tv/iit-iim-shaadi-dot-com-bizarre-matrimonial-site-screening-degrees-id-cards-and-what-not)).
- **App-store (USD, sources conflict):** ~$249 (3mo), ~$299 (6mo), ~$349–399 (yearly), ~$499 (lifetime); another source says "from $199" ([App Store](https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060)).
- **Tiers gated by contact-unlock volume, not features** — unlock contacts of **20 / 50 / 150 / 500** profiles ([fee page](https://iitiimshaadi.com/fee)).
- **Concierge/assisted matchmaking** exists as a CEO-led "quote-on-request" upsell (contact the founder directly) — pricing entirely opaque ([fee page](https://iitiimshaadi.com/fee)).
- **Renewal** described as manual on web (app-store subscriptions may auto-renew — unconfirmed). A "Refund & Cancellation" page exists but its terms were not retrievable.

**Takeaway:** premium pricing anchored to a small, unresponsive pool is the core value complaint. There is no outcome-linked or lower-commitment entry point, and refund terms are opaque.

---

## 4. Reputation & Sentiment

**Quora (the richest public vein):** polarized and skeptical. Consensus is it is *genuine, not a scam*, but *ineffective and overpriced*. The canonical negative datapoint recurs across threads — a paying member of ~4 months: **"my experience has been nothing, just null and void"** ([Quora — legit?](https://www.quora.com/Is-IITIIMShaadi-com-legit); [Quora — success?](https://www.quora.com/How-successful-has-the-site-iitiimshadi-com-been-till-now-Have-many-people-been-able-to-find-educated-matches-for-themselves-on-it)). A recurring critique: elite grads "finalize their partners within the institute itself" ([Quora — views](https://www.quora.com/What-are-your-views-on-the-website-IITIIMShaadi-com)).

**TeamBlind (professional-audience analog to Reddit):** high-CTC men describe a "reverse dowry" dynamic and extreme interest-imbalance — female master's/FAANG profiles reportedly "receive thousands of interests per week, while male profiles typically receive 1-2" ([Blind — expectations vs reality](https://www.teamblind.com/post/shaadicom-expectations-vs-reality-evsz2u41); [Blind — experiment](https://www.teamblind.com/post/experiment-to-settle-down-in-life-kkuyqrmm)). "Tier 1 college only" filters draw open elitism criticism ([Blind — tier 1 only](https://www.teamblind.com/post/matrimonial-profile-filters-tier-1-college-only-2lnsw9vl)).

**App stores:** low volume, bimodal. Google Play ~29 reviews, **~42% at 1–2★**; iOS essentially a single rating. Positives praise verification/"no fake profiles"; negatives cite a slow app, small database, and poor value ([chrome-stats reviews](https://chrome-stats.com/d/com.tycho.iitiimshadi/reviews); [App Store reviews](https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060?see-all=reviews&platform=iphone)). A year-long user on Medium ranks it #3 and advises: "don't buy the subscription unless they offer lifetime in 10k" ([Medium](https://medium.com/@abhishekb913/best-matrimonial-apps-in-india-eef8c5fe5055)).

**JustDial:** ~4/5 across ~147 ratings — praise for verified/educated profiles, recurring negative = "lack of response from other users" ([Justdial](https://www.justdial.com/Delhi/Iitiimshaadi-Dlf-City-Phase-4/011PXX11-XX11-220521175402-C1X3_BZDET)).

**Complaint boards:** essentially empty for this brand — **no dedicated MouthShut/Trustpilot/Sitejabber/Scamadviser page found**. Important: matrimonial-fraud FIR/police stories that surface under this query actually belong to **Shaadi.com**, a different platform — do not conflate ([ConsumerComplaints — Shaadi.com](https://www.consumercomplaints.in/complaints/shaadi-com-c564729.html)).

**Press:** the independent (non-PR) coverage is overwhelmingly *critical* — "systemic elitism" ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony)), "harder on men than women" ([Quartz](https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women)), a 2014 feminist open letter on sexist eligibility copy ([Feminism in India](https://feminisminindia.com/2014/08/21/iitiimshaadi-com/)), and the Karan Johar "nepotism/segregation" backlash cycle ([India TV](https://www.indiatvnews.com/entertainment/celebrities/karan-johar-trolled-iit-iim-shaadi-matrimonial-ad-instagram-netizens-say-concept-is-offensive-2022-03-31-766847); [Onmanorama](https://www.onmanorama.com/news/india/2022/04/08/karan-johar-centre-of-debate-for-endorsing-matrimony-ad-iit-iim-shaadi.html)). Most *positive* coverage is ANI-syndicated press-release wire copy, not investigative.

---

## 5. Trust, Safety & Verification

- **Verification verifies the wrong thing.** Manual screening of degree/ID/marksheet confirms a *credential*, not identity, marital status, or intent — the vectors fraud actually exploits. There is **no Aadhaar/KYC backend**, unlike Shaadi.com (added Aadhaar) or LoveVivah (UIDAI backend) ([PTC News](https://www.ptcnews.tv/iit-iim-shaadi-dot-com-bizarre-matrimonial-site-screening-degrees-id-cards-and-what-not); [Forbes India](https://www.forbesindia.com/article/special/aadhaar-latest-tool-against-fake-matrimonial-profiles/50215/1)).
- **Spoofable by forged documents** — manual image review is not registrar/email/API source-authentication ([Quora — legit?](https://www.quora.com/Is-IITIIMShaadi-com-legit)).
- **Gender-asymmetric onboarding** ("harder on men," "biased towards girls") — looser female-side verification is exactly the seam honeytrap/romance-scam operators exploit in Indian matrimony ([Quartz](https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women); [Quora — biased](https://www.quora.com/Why-is-iitiimshaadi-com-biased-towards-girls)).
- **Concentrated sensitive-PII collection** (degrees, government IDs, marksheets) with **no public data-safety disclosure, breach history, or security audit** retrievable ([privacy page](https://iitiimshaadi.com/users/privacy)).
- **No documented platform-specific scam/breach** was found — but that likely reflects low index coverage/under-reporting, not a clean bill of health.
- **"Verified elite enclave" branding can breed false security** — a verified degree is not verified character or intent ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony)).

---

## 6. Feature Inventory

- **Matching is filter/preference-driven, NOT algorithmic** — "filters and preferences make it easy to find compatible profiles" ([loving.is](https://www.loving.is/en/review/iitiimshaadi-com/)).
- **Communication:** messaging + friend-request (interest) system, shortlist, ID/keyword quick-search, paid profile-boost — all paywalled ([Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_US)).
- **Baseline visibility** closed to the public (only registered users see profiles) ([loving.is](https://www.loving.is/en/review/iitiimshaadi-com/)).
- **Likely-absent features (evidence gap — present on Shaadi.com, NOT confirmed here):** incognito/hidden browsing, "who viewed me," photo-blur/photo-password privacy, granular per-member blocking, horoscope/Kundli (Guna Milan) matching, and any algorithmic compatibility scoring. These are **whitespace** for a better product.
- **App defects:** "extremely slow," "screens stuck on loading," and **no way to surface newly added profiles** — users re-scroll the same faces despite ~500 new/week ([chrome-stats reviews](https://chrome-stats.com/d/com.tycho.iitiimshadi/reviews)).

---

## 7. Company & Scale

| Attribute | Finding | Source |
|---|---|---|
| Operating entity | Alma Mater Matters Pvt Ltd (CIN U74140HR2015PTC081038), Gurugram, inc. Dec 2015 | [ZaubaCorp](https://www.zaubacorp.com/ALMA-MATER-MATTERS-PRIVATE-LIMITED-U74140HR2015PTC081038) |
| Founders | Ajay Kumar Gupta (IRMA) + Taksh Gupta (Founder-CEO, SP Jain) | [Tofler](https://www.tofler.in/alma-mater-matters-private-limited/company/U74140HR2015PTC081038) |
| Launched | April 2014 (website); entity 2015 | [Modern Diplomacy](https://moderndiplomacy.eu/2022/04/04/taksh-gupta-founder-of-iitiimshaadi-matrimonial-website/) |
| Claimed members | 5 lakh (Nov 2025 PR) — undefined metric | [Business Standard](https://www.business-standard.com/content/press-releases-ani/iitiimshaadi-com-crosses-5-lakh-members-strengthening-its-position-as-india-s-premier-matrimonial-platform-for-educated-professionals-125112500803_1.html) |
| Active/authenticated | ~30,000 of ~150,000 (~20%) | [AppAdvice](https://appadvice.com/app/iitiimshaadi-matchmaking-app/1626456060) |
| Gender ratio | ~60% M / 40% F | [Deccan Chronicle](https://www.deccanchronicle.com/tabloid/hyderabad-chronicle/iim-lovin-iit-matrimonials-1810881) |
| Web traffic | ~234k monthly visits (Dec 2024) | [Similarweb](https://www.similarweb.com/website/my.shaadi.com/competitors/) |
| Funding | Bootstrapped/unfunded (Startup Haryana) | [Tracxn](https://tracxn.com/d/companies/iitiim-shaadi/__cT3hCA88BeEPKbcmWYZyNog-muG6TRqPwo5XQGouRS4) |
| Revenue | ~₹5 Cr band (conflicting "$5.3M" aggregator = low confidence) | [PitchBook](https://pitchbook.com/profiles/company/590801-59) |
| Marketing | Karan Johar brand ambassador, 3rd term 2025 | [ThePrint](https://theprint.in/ani-press-releases/karan-johar-continues-as-brand-ambassador-of-iitiimshaadi-com-for-third-term/2572250/) |

**Reputational vulnerability:** the founder is **not himself an IIT/IIM alumnus**, a viral, repeatedly-quoted attack on a brand whose entire pitch is verified eliteness ([Zee News](https://zeenews.india.com/technology/ye-sab-dogalapan-hai-twitterati-trolls-iit-iim-shaadi-founder-for-not-passing-out-from-iit-or-iim-2449824.html)).

---

## 8. What Users Say Is Missing — Gap Analysis (the heart of the opportunity)

Ranked by frequency/severity across all sources:

1. **Thin, mostly-inactive inventory** — shortlisted people can't be contacted; only ~20% active ([App Store](https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060); [FAQs](https://iitiimshaadi.com/faqs)).
2. **Gender-imbalance response collapse** — paying men get near-zero replies ([Quartz](https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women)).
3. **Poor value-for-money** — high fee, weak outcomes, no outcome-linked pricing or refund clarity ([Quora](https://www.quora.com/What-do-you-think-about-the-IITIIM-Shaadi-matrimonial-site)).
4. **No new-profile discovery** — endless recycling of the same faces ([chrome-stats](https://chrome-stats.com/d/com.tycho.iitiimshadi/reviews)).
5. **Slow, poorly-rated mobile app** ([Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi)).
6. **Over-narrow eligibility gate** shrinks inventory and invites elitism backlash ([Homegrown](https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony)).
7. **Trust deficit** around the "verified elite" promise (non-alumnus founder, opaque data handling) ([Onmanorama](https://www.onmanorama.com/news/india/2022/04/08/karan-johar-centre-of-debate-for-endorsing-matrimony-ad-iit-iim-shaadi.html)).
8. **Opaque concierge upsell** with unproven ROI ([Quora](https://www.quora.com/How-effective-is-IITIIM-Shaadi-in-helping-me-get-the-right-partner-Is-the-hefty-subscription-justified)).
9. **Weak search results** even for qualified users ([ScoopWhoop](https://www.scoopwhoop.com/news/wedding-bells/)).
10. **"Concept good, execution disappointing"** — the classic churn signature ([Blind](https://www.teamblind.com/post/How-was-your-experience-with-iitiimshaadicom-o4d7fXwi)).

---

## 9. International / Premium-Professional Expansion Signals

- **Verification is the product, not a badge.** Every premium-priced competitor leads with rigorous verification — The League (LinkedIn+Facebook, manual vetting), Raya (Instagram + committee, ~8% acceptance), Betterhalf (selfie + AI ID), EliteMatrimony (invite-only concierge) ([The League](https://tinderprofile.ai/blog/the-league-dating-app/); [Raya](https://roast.dating/blog/raya-statistics); [Betterhalf](https://www.datingscout.com/betterhalf/review); [EliteMatrimony](https://www.elitematrimony.com/)).
- **"Premium professional" is a proven, ownable middle ground** — EliteSingles reports ~85% of members hold a degree ([EliteSingles](https://www.top10.com/dating/reviews/elitesingles)).
- **Diaspora-first is validated and underserved at the premium tier** — Dil Mil (2M+ across US/UK/Canada/Australia/India) proved diaspora demand but skews dating ([Dil Mil](https://healthyframework.com/dil-mil-review/)).
- **Pricing benchmarks:** self-serve premium ~$20–50/mo (EliteSingles, Raya, Dil Mil, Aisle) + optional concierge/success-fee ($300+/mo or one-time) ([EliteSingles cost](https://www.datingscout.com/elite-singles/cost); [Aisle](https://onlineforlove.com/aisle-review/)).
- **Curated scarcity drives demand** — Raya's 2.5M waitlist against a six-figure user base ([Raya](https://www.vidaselect.com/raya-dating-app-review)).
- **Compliance is table stakes and a differentiator:** DPDP Act 2023 (India) demands unbundled, unconditional consent (penalties to ₹250 cr); GDPR is the EU/UK baseline; cross-border sensitive matrimony data needs dual compliance ([Bar & Bench](https://www.barandbench.com/columns/ghosts-in-the-machine-dpdp-act); [EY](https://www.ey.com/en_in/insights/cybersecurity/decoding-the-digital-personal-data-protection-act-2023); [CookieYes](https://www.cookieyes.com/blog/india-digital-personal-data-protection-act-dpdpa/)).
- **Learn from Betterhalf's wind-down** — manual verification + AI matching are capital-intensive; premium pricing is what makes rigorous verification sustainable ([startupintros](https://startupintros.com/orgs/betterhalf)).

---

## 10. Master Source List

### Target site & app
- https://iitiimshaadi.com/Members/userProfile/OTMxODA=
- https://iitiimshaadi.com/fee
- https://iitiimshaadi.com/faqs
- https://iitiimshaadi.com/success-stories
- https://iitiimshaadi.com/users/privacy
- https://iitiimshaadi.com/blog/a_sweet_wedding_story
- https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_US
- https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060
- https://appadvice.com/app/iitiimshaadi-matchmaking-app/1626456060
- https://chrome-stats.com/d/com.tycho.iitiimshadi/reviews
- https://www.appbrain.com/dev/IITIIMShaadi/
- https://apkpure.com/iitiimshaadi-exclusively-for-the-highly-educated/com.iitiimshadi.almamater

### Company / financial
- https://www.zaubacorp.com/ALMA-MATER-MATTERS-PRIVATE-LIMITED-U74140HR2015PTC081038
- https://www.tofler.in/alma-mater-matters-private-limited/company/U74140HR2015PTC081038
- https://tracxn.com/d/companies/iitiim-shaadi/__cT3hCA88BeEPKbcmWYZyNog-muG6TRqPwo5XQGouRS4
- https://pitchbook.com/profiles/company/590801-59
- https://rocketreach.co/iitiimshaadicom-profile_b40e6989ffe62940
- https://in.linkedin.com/in/taksh-gupta-84ba0682
- https://www.similarweb.com/website/my.shaadi.com/competitors/

### Reviews & sentiment
- https://www.quora.com/Is-IITIIMShaadi-com-legit
- https://www.quora.com/How-successful-has-the-site-iitiimshadi-com-been-till-now-Have-many-people-been-able-to-find-educated-matches-for-themselves-on-it
- https://www.quora.com/How-effective-is-IITIIM-Shaadi-in-helping-me-get-the-right-partner-Is-the-hefty-subscription-justified
- https://www.quora.com/Why-is-iitiimshaadi-com-biased-towards-girls
- https://www.quora.com/What-do-you-think-about-the-IITIIM-Shaadi-matrimonial-site
- https://www.quora.com/What-are-your-views-on-the-website-IITIIMShaadi-com
- https://www.teamblind.com/post/shaadicom-expectations-vs-reality-evsz2u41
- https://www.teamblind.com/post/experiment-to-settle-down-in-life-kkuyqrmm
- https://www.teamblind.com/post/matrimonial-profile-filters-tier-1-college-only-2lnsw9vl
- https://www.teamblind.com/post/matrimonial-profile-elitism-32cuhlg6
- https://www.teamblind.com/post/How-was-your-experience-with-iitiimshaadicom-o4d7fXwi
- https://www.justdial.com/Delhi/Iitiimshaadi-Dlf-City-Phase-4/011PXX11-XX11-220521175402-C1X3_BZDET
- https://www.loving.is/en/review/iitiimshaadi-com/
- https://medium.com/@abhishekb913/best-matrimonial-apps-in-india-eef8c5fe5055
- https://www.scoopwhoop.com/news/wedding-bells/

### Press & critique
- https://qz.com/india/2148968/elitist-iitiimshaadi-com-is-harder-on-men-than-women
- https://homegrown.co.in/homegrown-explore/iit-iim-shaadi-is-emblematic-of-the-systemic-elitism-in-indian-matrimony
- https://feminisminindia.com/2014/08/21/iitiimshaadi-com/
- https://www.indiatvnews.com/entertainment/celebrities/karan-johar-trolled-iit-iim-shaadi-matrimonial-ad-instagram-netizens-say-concept-is-offensive-2022-03-31-766847
- https://www.dnaindia.com/bollywood/report-karan-johar-gets-brutally-trolled-for-promoting-matrimonial-site-for-iit-and-iim-alumni-2942964
- https://www.onmanorama.com/news/india/2022/04/08/karan-johar-centre-of-debate-for-endorsing-matrimony-ad-iit-iim-shaadi.html
- https://thefederal.com/business/iit-iim-shaadi-website-founder-karan-johar-get-trolled-heres-why
- https://zeenews.india.com/technology/ye-sab-dogalapan-hai-twitterati-trolls-iit-iim-shaadi-founder-for-not-passing-out-from-iit-or-iim-2449824.html
- https://storypick.com/iitiimshaadi-com-ceo/
- https://tfipost.com/2022/04/now-karan-johar-brings-nepotism-to-indian-marriages/
- https://moderndiplomacy.eu/2022/04/04/taksh-gupta-founder-of-iitiimshaadi-matrimonial-website/
- https://techstory.in/iitiimshaadi/
- https://www.vervemagazine.in/love/a-suitable-degree
- https://www.deccanchronicle.com/tabloid/hyderabad-chronicle/iim-lovin-iit-matrimonials-1810881
- https://theprint.in/ani-press-releases/karan-johar-continues-as-brand-ambassador-of-iitiimshaadi-com-for-third-term/2572250/
- https://www.business-standard.com/content/press-releases-ani/iitiimshaadi-com-crosses-5-lakh-members-strengthening-its-position-as-india-s-premier-matrimonial-platform-for-educated-professionals-125112500803_1.html

### Safety / verification / regulatory
- https://www.ptcnews.tv/iit-iim-shaadi-dot-com-bizarre-matrimonial-site-screening-degrees-id-cards-and-what-not
- https://www.indiatvnews.com/news/india-user-verification-with-valid-ids-must-on-matrimonial-sites-govt-advisory-332427
- https://www.forbesindia.com/article/special/aadhaar-latest-tool-against-fake-matrimonial-profiles/50215/1
- https://www.deccanherald.com/amp/story/india%2Fsix-held-for-duping-prospective-brides-through-fake-matrimonial-profiles-1218219.html
- https://www.barandbench.com/columns/ghosts-in-the-machine-dpdp-act
- https://www.ey.com/en_in/insights/cybersecurity/decoding-the-digital-personal-data-protection-act-2023
- https://www.cookieyes.com/blog/india-digital-personal-data-protection-act-dpdpa/

### International / competitive
- https://tinderprofile.ai/blog/the-league-dating-app/
- https://www.bestdatingapps.com/reviews/the-league/
- https://roast.dating/blog/raya-statistics
- https://www.vidaselect.com/raya-dating-app-review
- https://www.top10.com/dating/reviews/elitesingles
- https://www.datingscout.com/elite-singles/cost
- https://healthyframework.com/dil-mil-review/
- https://justuseapp.com/en/app/879383901/dil-mil-south-asian-dating
- https://onlineforlove.com/aisle-review/
- https://play.google.com/store/apps/details?id=com.freeelective.jodi365&hl=en_US
- https://www.datingscout.com/betterhalf/review
- https://startupintros.com/orgs/betterhalf
- https://www.elitematrimony.com/
- https://blog.nrimb.com/tips-for-finding-nri-matches-for-arranged-marriage-through-matrimony-services-in-2026/
