# Research Angle: Profile Page & Site Structure

_Raw agent output. Direct fetch of homepage + profile URL blocked (403 egress + login gate); reconstructed from third-party sources._

## Key findings
- Niche education-gated matrimony for alumni of premier institutes across many disciplines (Engineering, Management, Law, Medicine, Fashion, etc.). "Niche portal with focus on education." [TechStory](https://techstory.in/iitiimshaadi/)
- Broad stream coverage listed in app: Engineering, Management, Architecture, Medicine, Finance, Law, Fashion Design, Theater, Media, Psychology, Sociology, Music, Dance, Culinary Arts, etc. [Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_IN)
- **Profile URL structure:** `/Members/userProfile/{token}` where token is Base64 of a **sequential numeric user ID** — `OTMxODA=` → `93180`. Enumerable IDs = scraping/privacy weakness. [target URL](https://iitiimshaadi.com/Members/userProfile/OTMxODA=)
- **Verification model:** mandatory educational-document authentication (UG/PG certs) before database access — the core differentiator. [Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_US)
- Marketed scale: "1,50,000 users, 30,000 active/authenticated, ~500 new profiles/week, 5000 families." [Google Play](https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_US)
- **Confirmed fields:** education/institute/discipline; contact info (phone+email) gated behind payment. [fee page](https://iitiimshaadi.com/fee)
- **Inferred fields (typical, unconfirmed on live page):** name/ID, age/DOB, height, religion/community/caste, mother tongue, marital status, profession, income, location, family details, horoscope/manglik, photos. [ScoopWhoop](https://www.scoopwhoop.com/news/wedding-bells/)
- Photos/full details blurred for free users, unlock on payment. [loving.is](https://www.loving.is/en/review/iitiimshaadi-com/)
- **Features:** specialised search filters (paid unlock more), quick search by ID/keyword, messages + friend requests, shortlist, "video and wallet features," profile-boost. [fee page](https://iitiimshaadi.com/fee) · [NewsBytes](https://www.newsbytesapp.com/news/india/matrimonial-website-to-find-iit-iim-groom-or-bride/story)
- Site sections: `/fee`, `/register`, `/faqs`, `/success-stories`, `/androidapp`. Native apps on Play + App Store (`com.tycho.iitiimshadi`, id `1626456060`).
- **Free vs paid:** free = register + browse ~10 profiles only. Paid = full details, contacts, messaging, filters, boost.
- Pricing (App Store USD): Testing $0.99, 3mo $249, 6mo $299, Yearly $349, Lifetime $499. Website tiers = contact unlocks for 20/50/150/500 profiles. Lifetime ₹33,900+18% GST (~₹40,002). [loving.is](https://www.loving.is/en/review/iitiimshaadi-com/)

## Gaps / Pain points
- Enumerable profile IDs (Base64 of sequential int) = scraping/privacy risk.
- Small active pool vs marketed size; hard to make meaningful contact.
- App "extremely slow"; no way to identify newly added profiles.
- Value-for-money skepticism on hefty subscription.
- Founder-credibility meme (not IIT/IIM alum).
- Could not confirm horoscope/manglik/gotra/complexion/photo-verified-badge from live page.

## Sources
- https://iitiimshaadi.com/Members/userProfile/OTMxODA= (403/login-gated)
- https://iitiimshaadi.com/fee
- https://iitiimshaadi.com/success-stories
- https://play.google.com/store/apps/details?id=com.tycho.iitiimshadi&hl=en_US
- https://apps.apple.com/in/app/iitiimshaadi-matchmaking-app/id1626456060
- https://techstory.in/iitiimshaadi/
- https://www.loving.is/en/review/iitiimshaadi-com/
- https://www.scoopwhoop.com/news/wedding-bells/
- https://www.vervemagazine.in/arts-and-culture/a-suitable-degree
- https://www.newsbytesapp.com/news/india/matrimonial-website-to-find-iit-iim-groom-or-bride/story
- https://www.teamblind.com/post/How-was-your-experience-with-iitiimshaadicom-o4d7fXwi
- https://chrome-stats.com/d/com.tycho.iitiimshadi
