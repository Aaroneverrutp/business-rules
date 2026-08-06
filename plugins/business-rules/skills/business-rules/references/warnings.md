# Example of a well-built warning

Scenario: `BUSINESS_RULES.md` has rule 2 — *"A coupon discount can never exceed 50% of the total."* The user asks to implement a `LAUNCH70` coupon at 70%.

---

> **This breaks rule 2 of `BUSINESS_RULES.md`:** *"A coupon discount can never exceed 50% of the total."*
>
> **How it breaks:** adding the coupon to the `lookupCoupon()` map wouldn't throw an error, and that's the problem. `applyDiscount()` does `Math.min(70, 50)` and silently caps it at 50% before computing the total. The coupon would be advertised as 70% and checkout would charge 50%: the rule holds in the final number, but the system ends up lying, which is worse than an explicit rejection. There's already precedent in the code — `MEGA80: 80` suffers exactly the same today.
>
> **What it means for the design:** the campaign wouldn't deliver the 70% you planned, so your conversion and margin projections don't match what will actually happen. It opens a gap between what marketing communicates and what the system charges, which is the kind of thing that ends in customer complaints or a false-advertising problem. And if the coupon combines with volume discounts, the cap becomes unpredictable: today `applyDiscount()` only caps the coupon percentage, not the accumulated discount, so rule 2 may already be breaking through another path.
>
> **Options:**
> 1. Respect the rule and rework the campaign: `LAUNCH50` at 50% plus a non-monetary incentive (free shipping, a gift). Comparable perceived value without touching the contract. I can implement this now.
> 2. Change rule 2 — your call, not mine. Suggested wording: *"...except campaign coupons explicitly flagged as such, capped at 70% and with a mandatory expiration date."* We update `BUSINESS_RULES.md` first, with the before → after, and only then I write the code. That way there's a record of who authorized the exception and under what limits.
> 3. Separate "coupon" from "promotional discount" in the model, each with its own cap. Cleaner long term but expensive; I wouldn't recommend it just to enable one coupon.

---

## What makes this example work

- **The rule is quoted verbatim and by number.** The user can open the file and read the same line. "This might conflict with the rules" is useless.
- **The mechanism is concrete and verifiable**, naming the line of code responsible. Not "could cause problems" but "`Math.min` trims it to 50 without warning".
- **The implications are real consequences**, not generic caution: projections that won't hold, a specific legal risk, and a downstream effect the user hadn't asked you to evaluate.
- **The options carry explicit trade-offs**, and one of them is changing the rule, clearly marked as the human's decision and routed through update mode — not something you do in passing.
- **There's no pre-written code** implementing the shortcut. Handing over a ready-to-paste block pushes the decision, even when it comes wrapped in warnings.

## The most common mistake

Detecting the conflict, flagging it, and still delivering the implemented code "just in case". That defeats the purpose: the user already has the implementation in front of them and the warning reads as a formality. Actually stop, and wait for the decision.

## Indirect conflicts

When the conflict isn't in the text of the rule but in its consequences, spend the "how it breaks" block tracing the chain step by step, because the user can't see it. Example structure:

1. Today `order.total` is computed like this and frozen on save.
2. `markAsPaid` validates `order.total !== amountReceived` and rejects on mismatch.
3. With a tip, `amountReceived` becomes `total + tip` → every tipped order gets blocked with `AMOUNT_MISMATCH`.
4. The intuitive "fix" of adding the tip to the total passes validation but hollows out the rule: `total` stops being the order's price and the rule no longer protects anything.

That fourth step — showing why the obvious shortcut is worse than the problem — is usually the most useful part of the whole warning.
