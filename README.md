# claude api credits: how the prepaid balance really works, and how to top up when your card keeps getting declined

You open the Claude Console, generate a key, fire off a test request, and get back `400 credit_balance_too_low`. No trial run, no grace period. That single line is where most people's first hour with the Claude API ends, and it's also where a lot of confusing advice starts, because "claude api credits" gets used to mean at least three different things.

Here's the version that saves you an afternoon: the API is prepaid, the balance has no relationship to your Claude Pro subscription, and the thing that usually stops people isn't the $5 top-up. It's the card.

## Claude Pro does not come with API credits

This trips up more people than any rate limit. Anthropic runs separate billing systems that don't share balances:

| What you're paying for | Where it's billed | How it's charged |
| --- | --- | --- |
| Claude API, Claude Code with an API key, your own scripts | Claude Console → Settings → Billing | Prepaid credits, deducted per token |
| Claude Pro / Max in the claude.ai app | claude.ai → Settings → Billing | Monthly subscription |
| Claude Team / Enterprise | Admin console or a contract | Per seat, or invoiced usage at API rates |

Max does not refill your Console balance. Credits in the Console do not unlock the app. Anthropic's own help centre says this outright, and it cuts both ways: a Pro or Max subscription gives you no API allowance at all, and the free plan gives you none either.

Practical consequence: if you bought a Claude Pro subscription through a reseller because your card wouldn't clear, you still have zero API credits. If your actual goal is to run code against Sonnet or Opus, you're shopping for a different product.

## What buying credits actually does

The mechanics are short and worth knowing before you spend anything.

- You buy credits in advance, and the minimum first purchase is small, single-digit dollars.
- Credits attach to the organization, not to an individual API key. Rotating keys doesn't reset anything.
- Every call deducts input tokens and output tokens at different rates. Cache reads cost roughly 10% of the input price, and batch requests are about 50% off.
- List prices span a wide range. As of the pricing documentation I checked in September 2026, the cheap end sits around $1 in / $5 out per million tokens and the expensive end around $10 in / $50 out. That 10x spread matters more than your top-up size: defaulting to a small model on a summarisation job can save more than hunting for a discount code ever will.
- Credits expire. Twelve months from purchase at the time of writing, and the expiry isn't extendable.
- Purchases are non-refundable. This is the one line people skip.
- Auto-reload exists and is the single most useful switch on the page. Set a threshold, let it top up, stop discovering the problem from a production error at 2am.
- Spend caps are the only hard ceiling. You can set a monthly limit per workspace, and the workspace limit can't be raised above your organisation's tier cap.

The usage tiers are worth a glance if you're planning a launch. The standard tiers consolidated into Start, Build and Scale, with monthly spend caps of roughly $500, $1,000 and $200,000 respectively; anything above that is a conversation with an account team. New organizations sometimes start in an Evaluation tier with lower limits as an anti-fraud measure, which lifts on its own. Buying a large credit package doesn't buy you a higher tier.

## The errors that tell you which wall you hit

Most "my credits aren't working" threads are one of five problems, and the error message already names it:

| What you see | What it means | What to do |
| --- | --- | --- |
| `401 invalid x-api-key` | Key is copied wrong, revoked, or expired | Check for stray whitespace, confirm the key is active |
| `400 credit balance too low` | The organization is unfunded, or your tool is authenticating with a key from a different org | Check the key's organization on the Billing page, then check the tool's auth path |
| `400 specified usage limit` | Your own spend cap, not your balance | Raise or remove the cap, or wait for the reset. Topping up won't help |
| `400 workspace header required` | Unscoped key | Send `anthropic-workspace-id`, or create a scoped key |
| `429` | Rate limit, separate from balance entirely | Check the Limits page; bursts can exhaust capacity even when the per-minute average looks fine |

That last one deserves emphasis. A request can fail with money sitting in the account. Balance, spend cap, tier cap and throughput are four separate controls, and topping up fixes exactly one of them.

## Why the card gets declined

If your balance is the problem, the fix isn't a bigger top-up. It's the payment method. In rough order of how often it's the cause:

1. **Billing address mismatch.** The address on the card has to match what you type. A single wrong postal code is enough.
2. **The issuer blocks the merchant category.** Plenty of banks decline AI and SaaS merchants by default. A two-minute call usually clears it.
3. **Region.** Anthropic doesn't serve every country, and prepaid cards issued in unsupported regions fail regardless of balance.
4. **Virtual cards that worked once.** This is a known pattern: the first charge clears, the renewal doesn't. If you're relying on a virtual card, expect to maintain it rather than set and forget.

If you've worked through that list and the Console still won't take payment, you're in the group that third-party payment bridges exist for.

## Where a payment bridge fits when the card simply won't clear

A payment bridge does one job: it completes an overseas AI subscription on your behalf, and you pay it in a currency and through a channel you already have.

### What BeWild is, and what it isn't

BeWild, also written WildAI, runs at bewild.ai and is operated by Mudanjiang Limited. It's the successor to WildCard, which was a virtual-card product; after the 2024 regulatory squeeze on that model, the team repositioned around direct subscription handling instead of issuing cards. The catalogue is built around consumer AI subscriptions, Claude Pro and ChatGPT Plus first, with third-party write-ups also listing developer-side options such as Claude Code and OpenAI Codex, plus ChatGPT Pro and Gemini Pro. You pay with Alipay or WeChat Pay, or a domestic card.

Be it Claude, ChatGPT or Gemini, one thing needs saying plainly: this is a third-party service, not an Anthropic or OpenAI channel. Independent Chinese-language reviews say the same thing, and it's the right way to read it. Whatever you buy here is a subscription on your own account, arranged by someone else.

That distinction also answers the keyword. BeWild's catalogue is subscriptions. Anthropic's Console credits are a different product, and the platform does not market itself as a credit reseller. What people actually use it for on the API side is narrower and simpler: as the payment leg. In a developer forum thread about topping up Claude API, BeWild turns up with one-line verdicts along the lines of "it just adds a service fee." Whether that fee is worth it depends on how much your time costs versus how much the card fight costs.

For Claude specifically, the flow has a hard constraint worth knowing before you pay. You register, install the browser extension, and it captures the login session from your signed-in Claude tab. It never asks for your password. The platform then checks your network environment: a residential or ISP-class IP rather than a datacenter one, DNS that doesn't leak outside your exit IP, and a browser fingerprint that matches the region. Datacenter IPs from typical proxy pools get flagged, which is why some users report failures that have nothing to do with payment. Once the order goes through, activation is usually minutes, not hours.

👉 [See what BeWild currently offers for Claude](https://bewild.ai?code=ACCPAY)

Also: the link above already carries the invite code `ACCPAY`. Published invite codes on this platform have been documented to knock about a dollar off an order, and the discount shows up at checkout rather than being applied invisibly. Check the total before you pay.

### What the risk record actually says

Ignoring this part would be dishonest. Independent blogs that track the platform have logged real failures in 2026: one user's three-month ChatGPT Plus plan didn't auto-renew on schedule, and a separate two-month order showed as complete while the account stayed on the free tier. Both ended in refunds or refund arrangements after support picked up the thread, with a delay before anyone answered. A later entry records a renewal failing outright when the underlying payment method died, a partial refund of ¥150.69, and a re-purchase that only completed after a retry.

In the Chinese developer forums, opinions split. Long-term users describe repeated orders and refunds for failed subscriptions, including cases where an account ban was refunded minus a fee. Others report a 25% fee on refunds. One thread claims the platform had stopped listing Claude and showed everything as sold out. I couldn't verify the current refund rule or the current Claude availability from those posts, and forum screenshots go stale fast.

So the honest summary is: a mature operator with a public site, real support, and a documented history of both refunds and failed renewals. If that risk profile doesn't suit you and your card works, buy from Anthropic directly. If your card has never once cleared on an overseas AI merchant, this is the trade you're making.

### Current terms, as documented

Prices on this platform track FX and promotions, so treat the table as a starting point and confirm the total at checkout. The figures below come from third-party write-ups and purchase records dated January and August 2026.

| Service | Term | Price as documented | Billing | Get it |
| --- | --- | --- | --- | --- |
| Claude Pro (your own account) | 1 / 3 / 12 months | Live price at checkout; Anthropic's list is $20/mo | One payment per term | [Check the Claude Pro plan](https://bewild.ai?code=ACCPAY) |
| Claude Code / developer tooling | Varies by term | Live price at checkout | One payment per term | [Check developer plans](https://bewild.ai?code=ACCPAY) |
| ChatGPT Plus | 1 month | $24.99 to $25.99 across two sources | One payment per month | [Check ChatGPT Plus monthly](https://bewild.ai?code=ACCPAY) |
| ChatGPT Plus | 2 months | $46.99, about $23.50/mo | One payment per term | [Check the 2-month plan](https://bewild.ai?code=ACCPAY) |
| ChatGPT Plus | 3 months | $66.09 to $66.99, about $22/mo | One payment per term | [Check the 3-month plan](https://bewild.ai?code=ACCPAY) |
| ChatGPT Pro | 1 month | Live price at checkout; OpenAI's list is $200/mo | One payment per month | [Check ChatGPT Pro](https://bewild.ai?code=ACCPAY) |
| Gemini Pro | 1 month | Live price at checkout; Google's list is $19.99/mo | One payment per month | [Check Gemini Pro](https://bewild.ai?code=ACCPAY) |

Two structural notes. Longer terms lower the effective monthly cost on the ChatGPT plans, in one case by roughly $3.50 a month. And renewal behaviour differs by term: you can switch renewals off in the order page, and a subscription that isn't renewed simply stops rather than charging you silently.

## Credits you can get without paying anything

Before buying credits anywhere, exhaust these. They exist, they're documented, and they're worth more than any coupon.

- **Claude for Startups.** Expanded on 6 October 2026. Approved companies get a one-time $1,000 in API credits plus a free year of Claude Team covering up to five Premium seats, which Anthropic values at $6,000. Eligibility is a startup founded within five years or funded within two. Bootstrapped companies qualify. Applications need a Console account and a company email matching your website's domain, and most decisions come back within minutes, with manual reviews taking two to three business days.
- **The fine print on that $1,000.** It expires six months after it's granted, it works only on Anthropic's first-party API through the Console, and it can't be spent via AWS Bedrock, Google Cloud Vertex AI or other third-party platforms. Recipients also get higher rate limits.
- **Partner routes.** Startups backed by a VC in Anthropic's partner network can access up to $100,000 in additional credits through that investor. The Claude Startup Stack adds partner offers advertised at up to $45,000 in list-price value; that number depends entirely on which offers you actually redeem.
- **Research programmes.** AI for Science has granted up to $20,000 in API credits over a six-month period for academic and nonprofit researchers on high-priority topics. External Researcher Access grants $1,000 for AI safety and alignment work. Both are reviewed on the first Monday of each month.
- **Cloud credits you already have.** Claude is available through Bedrock, Vertex AI and Microsoft Foundry. If your AWS Activate or Google Cloud startup credits are already sitting there, that's a legitimate way to pay for Claude inference without touching the Console billing page at all.

One thing to stop expecting: the old advice about a fixed free credit for every new account. Anthropic's current documentation describes the Console as running on prepaid purchase credits, so don't plan your project around a trial that may not appear.

## A top-up sequence that avoids the usual mistakes

1. Confirm which product you actually need. Running your own code means Console credits. Chatting in the app means a subscription. These do not overlap.
2. Fund the organization, not a key. Credits live at the org level.
3. Buy small first. The minimum first purchase is a few dollars, and that's enough to confirm your integration works end to end.
4. Set a workspace spend cap before you deploy, not after the first surprising invoice. A runaway loop against an expensive model can burn real money overnight.
5. Turn on auto-reload with a threshold.
6. Set a calendar reminder for the expiry date. Twelve months, not extendable, and non-refundable if you forget.

If step 2 is where you're stuck, that's a payment-method problem, and it's worth solving once properly rather than repeating every month.

👉 [Open BeWild and check the invite code is applied](https://bewild.ai?code=ACCPAY)

## Quick answers

**Do credits carry over if I upgrade to Max?** There's no upgrade path between the two systems. A Max subscription doesn't add API credit, and API credit doesn't change your app plan.

**Do unused credits expire?** Yes, twelve months from purchase, and the date can't be extended. Don't stockpile a year of usage.

**Can I get a refund on credits?** Purchases are non-refundable. Failed API calls aren't charged, with one documented exception: a call that was on track to succeed but got cut off by a client-side timeout or disconnect.

**How much should I buy first?** Single-digit dollars is enough to validate a working integration. Then look at your actual token consumption per model before committing more. The gap between the cheapest and most expensive model is roughly 10x, and that's where the real cost decision lives.

**Is a third-party subscription service the same as buying API credits?** No. A subscription service like BeWild arranges a Claude Pro plan on your own account. Console credits are a separate prepaid balance sold by Anthropic, and if the Console won't accept your card, most people's better options are a properly issued international card, an existing cloud credit balance through Bedrock or Vertex AI, or one of the startup programmes above.
