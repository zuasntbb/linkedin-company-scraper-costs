# LinkedIn Company Scraper: What 1,000 Companies Cost in Credits, Which Fields Come Back Empty, and How to Batch 10,000 Profiles

ScraperAPI fetches a LinkedIn company page without a LinkedIn login, and it bills the request in credits. LinkedIn sits in the expensive tier at about 30 credits per page, so the plan that fits depends on how many companies you actually need per month, not on how big the headline credit number looks. A new account gets 5,000 credits over 7 days, and there's a standing free plan at 1,000 credits a month, enough to check whether the pages you care about come back complete before you enter a card.

This piece covers the credit math for LinkedIn specifically, the current plan lineup, the fields that arrive blank often enough to plan around, the throughput each tier gives you on a 10,000-company list, and the point where a pre-built company extractor is the better buy than writing your own parser.

> **Quick pick:** take the 7-day trial first, measure your real credit cost per company, then buy Hobby ($49/month, 100,000 credits) if you need fewer than roughly 3,000 LinkedIn pages a month, or Startup ($149/month, 1,000,000 credits) once you're past that. 👉 [Check current ScraperAPI plans and what the trial includes](https://www.scraperapi.com/?fp_ref=coupons)

## All ScraperAPI plans at a glance

| Plan | Price | Credits/month | Concurrent threads | Start here |
| --- | --- | --- | --- | --- |
| Free | $0 | 1,000 | 5 | [Create a free account](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49/month | 100,000 | 20 | [Open Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | $149/month | 1,000,000 | 50 | [Open Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | $299/month | 3,000,000 | 100 | [Open Business](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | $475/month | 5,000,000 | Higher, set per account | [Open Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | Custom | Custom | Custom | [Ask for an Enterprise quote](https://www.scraperapi.com/?fp_ref=coupons) |

Two things about that table matter more than the prices. First, the 5,000 credits people mention on review sites is a 7-day trial, not a monthly allowance; the permanent free plan is 1,000 credits a month, which at 30 credits per LinkedIn page is about 33 companies. Second, ScraperAPI has changed allowances and thread limits between tiers more than once, so treat the credit column as a sizing guide and confirm the numbers on the pricing page before checkout. Annual billing lowers the effective monthly rate, and there's a 7-day refund window if you size the plan wrong.

## What comes back: HTML, not a filled-in company record

Send ScraperAPI a LinkedIn company URL and you get the page back, either as raw HTML or as JSON wrapping that HTML. Pulling out the name, industry, headcount, office addresses and follower count is your job, which means a parser, a selector for each field, and maintenance when LinkedIn changes its markup.

ScraperAPI does sell structured data endpoints that return parsed JSON directly, but they cover Amazon, Google Search, Google Shopping, Maps, News, Jobs, Walmart, eBay and Redfin. LinkedIn is not among them. That single fact decides the buy for a lot of teams: if nobody on your side wants to own a LinkedIn parser, the credit price is the smaller half of the total cost.

## The fields that exist, and the ones that arrive empty

A public LinkedIn company page carries more than most people expect: name, tagline, about text, website, industry, company size band, employees on LinkedIn, headquarters, every office address, founding year, specialties, company type, follower count, logo, cover image, products, similar pages, affiliated pages, and the latest few posts with like and comment counts.

Three limits are worth knowing before you build a schema around them.

**Funding rounds, phone numbers and employee names are not on the page.** No scraper can return them from a company page, because LinkedIn doesn't publish them there. Employee names require a separate people scraper and a separate compliance conversation.

**Fields the company didn't fill in come back blank.** A published sample of Google's company page returned an empty founding date and an empty slogan. Airbnb's page shows 2007, San Francisco as headquarters, a second office in Singapore and 3,495,245 followers, so the data is there when the company maintains it. Smaller companies routinely leave specialties, products and the about section empty, and a null in your spreadsheet usually means the company skipped the field, not that your scraper failed.

**"Employees on LinkedIn" and "company size" are two different measurements.** Airbnb's page shows a size band of 5,001–10,000 employees and 72,846 employees on LinkedIn. The band is a range the company selects from a fixed list. The other number counts LinkedIn accounts that list Airbnb as their employer, which includes people who left years ago and never updated their profile. Building a headcount column out of the band and then comparing it to the account count will produce nonsense. Pick one, label the column clearly, and never compute growth from the band, since a company can jump from one band to the next without hiring anyone.

## What 1,000 LinkedIn company pages actually cost

ScraperAPI prices by credits, and the credit cost per request depends on what you ask it to do and which site you target. The domain multiplier is applied automatically, whether or not you thought about it.

| Request type | Credits per request |
| --- | --- |
| Standard HTML page | 1 |
| E-commerce endpoints (Amazon, Walmart, eBay) | 5 |
| Google SERP | 25 |
| LinkedIn company page | 30 |
| JavaScript rendering (`render=true`) | +10 |
| Anti-bot bypass when Cloudflare, DataDome or PerimeterX is detected | +10 each |
| Ultra-premium proxy plus JS rendering | up to 75 |

Run those multipliers against the plan prices and the LinkedIn cost per 1,000 pages looks like this:

| Plan | Price | Credits | LinkedIn pages/month | Cost per 1,000 pages |
| --- | --- | --- | --- | --- |
| Hobby | $49 | 100,000 | ~3,333 | $14.70 |
| Startup | $149 | 1,000,000 | ~33,333 | $4.47 |
| Business | $299 | 3,000,000 | ~100,000 | $2.99 |
| Scaling | $475 | 5,000,000 | ~166,666 | $2.85 |

The steep drop happens between Hobby and Startup, where the per-page cost falls by two thirds. Business triples your headroom for a modest further saving, which makes it a volume purchase rather than a value purchase. Scaling saves another 14 cents per 1,000 pages, so buying it for the discount alone doesn't pay; buy it because you need 100,000 or more pages a month and want the higher thread ceiling.

Add `render=true` and every request goes from 30 credits to 40, which pushes Business to about $3.99 per 1,000 pages and cuts the monthly page ceiling by a quarter. LinkedIn company pages usually render server-side, so test without the flag first. 👉 [Size a plan against your own monthly page count](https://www.scraperapi.com/?fp_ref=coupons)

## Batching 10,000 companies: the three things that slow you down

**Input has to be a URL.** ScraperAPI takes a target URL per request. Turning a list of company names into `linkedin.com/company/...` URLs is your work, and name collisions are common enough that a fuzzy match will quietly pull the wrong company. Some pre-built extractors accept company names directly and return the closest matches when a name doesn't resolve; with a raw API, that lookup is on you.

**Duplicates cost full price.** The same company appears under country subdomains (`uk.linkedin.com`, `de.linkedin.com`), and each variant is a separate 30-credit request. Deduplicate on the company slug before you send anything.

**Thread count sets the wall clock.** Third-party benchmarks put ScraperAPI's LinkedIn success rate around 95% at roughly 18 seconds per request. On that basis, 10,000 pages takes about 2.5 hours at 20 threads on Hobby, about an hour at 50 threads on Startup, and about 30 minutes at 100 threads on Business. The same 95% figure means budgeting for around 500 retries on a 10,000-company run.

One structural risk of doing your own parsing: you pay for the request, not for a clean record. A selector that returns a 200 with the wrong text still spends its 30 credits, and you find out three days later when the industry column is full of nulls. Validate on 100 companies before you point a 10,000-company list at the endpoint. 👉 [Test the endpoint on a small list with the free trial credits](https://www.scraperapi.com/?fp_ref=coupons)

## Login, account safety, and what you're responsible for

There is no LinkedIn account involved. ScraperAPI proxies the request from its own infrastructure, so no cookie, session or profile of yours is at risk, and there's no account of yours for LinkedIn to restrict. That is a real advantage over browser-extension scrapers that run inside a logged-in session.

It isn't a legal shield. You're collecting data from a site whose terms restrict automated access, and if you store records on people in the EU or California, data protection rules apply to what you keep and how long you keep it. Stick to pages that anyone can see without signing in, don't touch anything behind a login, and keep a record of why you collected what you collected.

## When a pre-built company scraper is the better buy

If you want named fields today and have nobody to maintain a parser, two other options do that work for you.

Bright Data's LinkedIn company scraper returns parsed fields (company ID, industry, size, employees on LinkedIn, followers, locations, founded date) and charges only for records successfully delivered, with proxies, rendering and parsing included in the per-record price. New accounts get 5,000 free records a month, which the company values at roughly $7.50, and a single API call accepts up to 5,000 target URLs.

Apify's company details actor accepts company names, slugs or full URLs, defaults to 1,000 companies per run, and doesn't charge for companies it can't find. It returns up to 10 recent posts plus most of the fields above, and states plainly that it does not collect funding rounds, phone numbers or employee names. A second actor from a different developer takes only URL lists, deduplicates input and advertises no per-run cap.

Those tools cost more per record than ScraperAPI at Business volume, and they hand you a schema you didn't design. Above roughly 30,000 pages a month, the parser becomes worth writing.

## How to choose

- **Under ~3,000 LinkedIn pages a month, with a developer who can write selectors:** Hobby at $49.
- **3,000 to 33,000 pages a month:** Startup at $149. This is where most teams running a quarterly account refresh land.
- **33,000 to 100,000 pages a month:** Business at $299, mostly for the 100 threads and the threefold headroom.
- **Past 100,000 pages, or you need contractual guarantees:** Enterprise.
- **No parser and no engineer:** buy a field-level extractor now, and revisit ScraperAPI when volume makes the parser pay for itself.

Before you enter a card, confirm two things on the pricing page: the current credit multiplier for LinkedIn, since that single number sets your plan size, and whether your target pages need `render=true`, which adds 10 credits to every request and quietly changes the answer. The 7-day refund window means a wrong size costs you a support ticket rather than a month's fee. 👉 [Start on the trial and price out your first 1,000 companies](https://www.scraperapi.com/?fp_ref=coupons)
