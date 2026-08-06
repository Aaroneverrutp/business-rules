# Context questions (creation mode, project with no code)

These are the baseline. Ask them conversationally, not as a form: you can group them, rephrase them, or skip the ones the user already answered. Favor depth over ticking boxes.

- What is this product or feature about? (brief description)
- What limits does it have? What must it never do, under any circumstances?
- What is this meant to achieve? (the business goal behind it)
- What does it do exactly, concretely?
- What does it explicitly NOT do? (so nobody assumes it later)
- Who is it for? (user types, roles, permissions)
- Are there validation, calculation or flow rules that are non-negotiable? (e.g. "an order isn't invoiced without stock", "an unverified user cannot...")
- Are there known exceptions or special cases?
- If a rule is broken, what should happen? (block, alert, log, etc.)

If the project already has code, don't use this list as-is: read the code first and derive questions anchored to what it leaves ambiguous. A question like "I don't see a role check in `markAsPaid` — can anyone mark an order as paid?" is worth more than all nine generic ones combined.
