USE /caveman ULTRA skill during every session and type 0 words, 0 charachters and 0 text on chat until a task is done. For security and pentests use /security-review skill. THESE TWO SKILLS ARE MANDATORY COMBINED WITH /ponytail ultra skill. During pentests it is mandatory to use the below rules and checklists for each finding, and every single finding must pass all of those rules and it's also mandatory to use the /humanize and /caveman ULTRA skills and stay with no text output on chat unless something mayor happens or task/plan/goal is finished.


# MANDATORY OPERATING DIRECTIVE — BACKWARD-CHAIN HUNTING

This is the primary hunting methodology for the entire engagement. **Work from the end state
backwards. Do not start by collecting random bugs and hoping they become impactful. Pick one credible
attack chain, lock onto it, and work every prerequisite in reverse until the chain is proven or killed.**

## THE BACKWARD-CHAIN LOOP — STRICT, ONE CHAIN AT A TIME

**A. Define the finish objective before touching a candidate.**

Start with the highest-credibility, highest-impact **in-scope** security consequence the target could
realistically expose based on recon, architecture, functionality, trust relationships, privilege
models, and program rules. Examples include unauthorized money movement, cross-principal data access,
account takeover, privileged action, arbitrary code execution, or another concrete boundary break.

Write the finish objective in one sentence:

> **Finish objective:** `<named victim/principal> loses `<specific asset/authority>` when `<mechanical event>`.

No finish objective = no hunting.

**B. Model the chain backwards as a dependency graph.**

Always reason in this order:

`FINAL IMPACT → REQUIRED PRIVILEGE/CAPABILITY → TRUST BOUNDARY → INTERMEDIATE CAPABILITY → INITIAL ACCESS`

For every transition, write exactly what must already be true immediately before that step. Treat every
transition as **UNVERIFIED** until independently measured.

Use this template:

| Stage | Required precondition | Security property that must fail | Observable test | Evidence required | Status |
|---|---|---|---|---|---|
| Final impact | ... | ... | ... | victim-side oracle + browser/system oracle | UNVERIFIED |
| Required capability | ... | ... | ... | ... | UNVERIFIED |
| Trust-boundary crossing | ... | ... | ... | ... | UNVERIFIED |
| Intermediate capability | ... | ... | ... | ... | UNVERIFIED |
| Initial access | ... | ... | ... | ... | UNVERIFIED |

**C. Research the exact chain family before designing probes.**

Ground the chain in documented security research: public bounty disclosures, CVEs, vendor advisories,
incident reports, security writeups, comparable architectures, exploitation prerequisites, historical
failure modes, and vendor fixes. Prefer recognisable vulnerability classes and attack patterns that
have repeatedly produced concrete security impact in real programs.

Prefer classes and chain patterns that are repeatedly documented as high-severity and rewarded in public
bug-bounty reports when comparable prerequisites and impact exist. Do not treat reward history as proof that
the target is vulnerable; use it only to prioritize a credible, recognisable hypothesis.

Research is not permission to assume the target is vulnerable. It is only used to establish a credible
hypothesis and to identify the prerequisites that must be tested.

**D. Pick ONE chain. Lock it.**

After recon and research, choose the single most credible chain to investigate first. Write the join
before spending serious testing time:

> **Chain:** `A-x gives X + A-y removes Y + A-z crosses Z → FINISH OBJECTIVE`

For each link, record:
- what capability it must create;
- which trust boundary it must cross;
- which documented/security-relevant property it would violate;
- why the link is needed by the final objective;
- how the link can be disproved quickly.

**Do not hunt 100 different things at once. Do not maintain a grab-bag of unrelated primitives.**
A new anomaly is worth pursuing only when it satisfies a specific unmet prerequisite in the locked
chain. Everything else is background noise and belongs in the register.

**E. Hunt backwards, one prerequisite at a time.**

For the current UNVERIFIED link, ask:

> **“What security property would have to fail for this transition to occur?”**

Search specifically for that failure mode. Prioritise:
- authorization inconsistencies;
- privilege mismatches;
- trust-boundary confusion;
- identity/session mix-ups;
- flawed state transitions;
- race conditions;
- inconsistent policy enforcement;
- version/cache discrepancies;
- unintended data flows;
- exposed internal functionality;
- parser/validation mismatches;
- cross-service assumptions that disagree.

Do not pivot forward merely because a surface looks interesting. Every probe must answer a question
about the current prerequisite.

**F. Attack the defenses on the chosen path, not the entire application.**

For each chain transition, enumerate the controls that are supposed to stop it. Then test those controls
one by one, within authorization, looking for the smallest discrepancy that defeats the required
security property.

Record the result as one of:
- **BLOCKED** — the control works and this transition is disproven;
- **PARTIAL** — a primitive exists, but the required capability is not yet gained;
- **OPEN** — the required capability is demonstrated;
- **UNKNOWN** — evidence is insufficient.

Never treat **PARTIAL** as **OPEN**.

**G. Prove composition; never assume composition.**

A chain exists only when each transition works in sequence under the same documented attacker model.
Validate every edge independently and then replay the complete path from initial access to final impact.

Do not claim “A + B = impact” because A and B are individually real. Demonstrate that A actually
produces the input/state/capability that B consumes, under the same scope and trust assumptions.

**H. Enforce the fix-survival question at every link.**

For every candidate weakness, write:

> **“If the obvious fix for this shallow issue shipped tomorrow, would the locked chain still work?”**

If the answer is no, the candidate is likely only the shallow bug or a variant of it. Do not inflate it
into a larger chain. If the answer is yes, explain the surviving architectural seam and continue testing
that seam.

**I. Use a hard kill-and-reselect loop.**

When a transition is disproven, the chain is **KILLED**, not “maybe later.” Run one bounded attempt to
falsify the blocker using fresh evidence. If it still fails, close the chain and return to reconnaissance
and the chain register.

Then select a **different, better-supported chain** whose prerequisites fit the actual target
architecture and measured behavior. Carry forward the evidence, including closures; do not repeatedly
re-test the same dead path without a new reason.

The loop is:

`RECON → RESEARCH → PICK ONE CHAIN → WORK BACKWARD → TEST ONE PREREQUISITE → PROVE/FAIL →`
`E2E PROOF OR KILL → RESELECT FROM TARGET EVIDENCE → REPEAT`

**J. Do not submit primitives as high severity.**

A primitive is only a chain input. A high-severity claim requires a **research-grounded, reproducible,
end-to-end demonstrated consequence** that crosses the relevant security boundary.

The required proof is:
1. the attacker starts with the documented initial capability;
2. each intermediate capability is obtained through a measured transition;
3. the final victim/principal is affected;
4. the impact is observable through an oracle outside the attacker-controlled component;
5. the state change or authority gain is reversible where practical and authorized;
6. the entire sequence can be replayed from clean state.

No end-to-end proof = no high-severity conclusion.

**K. Evidence is part of the chain, not an afterthought.**

Every load-bearing transition must have:
- the exact request/action that triggered it;
- the resulting state/capability;
- the oracle that proves the result;
- timestamps/identifiers needed to replay it;
- scope/authorization justification;
- screenshots/video when they materially establish the victim-side or boundary-crossing effect.

Keep attacker-controlled observations separate from product-side evidence. A message displayed only
on the attacker page is not proof of victim impact.

**L. Stop only when the chain is proven or the credible search space is exhausted.**

Do not stop at “interesting,” “likely,” “theoretically possible,” or “impact inferred.” Continue
through the dependency graph until the finish objective is demonstrated, or until the current chain
has been killed and the target evidence justifies selecting a different chain.

The goal is **not maximum finding count**. The goal is the **smallest authorized, reproducible,
research-backed chain that produces the largest substantiated security consequence**.

**NON-NEGOTIABLE FOCUS RULE:** one objective, one active chain, one current prerequisite. Everything
else waits unless it directly helps complete or falsify that chain.

For Skroutz, all testing remains within the program scope and rules of engagement. Never use destructive
actions, unnecessary data access, persistence, or activity outside authorization. Distinguish genuine
unintended security behavior from documented functionality, intended administrative behavior, and
explicitly accepted program behavior.


## Workspace map — read in this order

1. `ANOMALIES.md` — the chain register. Targets are picked from the **joins**, never from a single
   anomaly. Write the join before spending time (Gate 10, Rule 29).
2. `PROVEN.md` — the measurements behind every anomaly, including the closures. Nothing here is
   dead; a closure is a chain input.
3. `SUBMISSIONS.md` — what has been filed, its state, and the pre-written answers to what triage
   may ask next. Filed bundles are frozen (Rule 34). It also carries the **prior-filings ledger**:
   every report this workspace has sent and what came back. Read that table with `ANOMALIES.md`,
   not after it (Rule 35).
4. `SCOPE.md` and `knownreports_1win.md` — asset list and the disclosed-report corpus for Rule 4.
5. `recon-checklist.md` — surface enumeration.

**Gate order at target-selection time: Gate 8, then Gate 10, then Gate 5, then the rest.** Gate 10
is the one with a triaged report behind it; where it disagrees with Gate 3, it wins.

### RULES LIST :

**Run both gates before you type a single word of a report. If anything fails, close the tab — don't submit. No exceptions, no "but this one's different."**

## GATE 1 — Duplicate Gate (all 4 must pass)

**1. Front Door Rule**
Never submit anything found by clicking the obvious flow — signup, login, search box. If a normal user could stumble into it, assume they already did.

**2. Source Rule**
Never submit without checking: is this in code or a feature that changed in the last ~90 days? Old, stable, unchanged code has already been hammered by everyone.

**3. Scanner Rule**
Never submit something a default Nuclei/sqlmap/Burp scan would output as-is. If a tool finds it in one click, a hundred people already ran that tool first.

**4. Graveyard Rule**
Never submit before checking Hacktivity/disclosed reports on that program. Already public = already paid to someone else.

## GATE 2 — Impact Gate (all 5 must pass)

**5. Victim Rule**
Never submit without writing one sentence naming who loses and what they lose. No named victim = no report.

**6. Boundary Rule**
Never submit anything that only affects your own account, browser, or session. If it doesn't cross from attacker to a real victim (or low-priv to high-priv), it's self-inflicted — this is the rule that kills self-XSS, self-DoS, and "I broke my own thing" instantly, every time.

**7. Three "So What"s Rule**
Ask "so what" three times. If you run out before landing on a concrete real-world action — stolen data, stolen money, stolen account, code execution — you're not done, you're just annoyed at the app.

**8. Five-Minute Rule**
Never submit a report a total stranger couldn't reproduce in under 5 minutes using only what's written. If it needs you to walk them through it, it needs more work, not a submit button.

**9. Scope Rule**
Never submit without re-checking the scope page the same day you submit. Scope changes. Memory doesn't.

## The tie-breaker (Rule Zero)

**When in doubt, the answer is no.** If you're not sure whether a rule passes, that uncertainty *is* your answer — you haven't finished proving uniqueness or impact yet. Go find one more piece of evidence before you decide, not after you submit.

**Both gates fully green → submit. One red box anywhere → don't.** That's the whole algorithm — it doesn't get more sophisticated than this, elite hunters just run it faster and without skipping steps.
---

## GATE 3 — Depth Gate (MANDATORY, all sessions, all findings)

Added 2026-09-01 after four closures. Two were Informative ("hardening", "not a boundary") and two
were Duplicate — real bugs, found by someone else first. The Duplicates share one property: their
root cause fits in a single sentence and is repaired by a single line. That is the property to stop
producing.

Gate 3 runs **in addition to** Gates 1 and 2. All four rules must pass. Run it before spending time
on an idea, not after.

**10. One-Line-Fix Rule.** *(SUPERSEDED 2026-09-14 by Gate 10. Read the precedence note there
before applying this rule. It is now a dupe-risk signal, not a veto: the one report out of this
workspace that cleared triage has a one-line root cause.)*
Never submit a finding whose root cause is repaired by editing a single expression — an escape, a
regex, a comparison operator, a missing `if`. Write down the fix you believe the vendor would ship.
If that fix is one line, the finding carries elevated dupe risk and needs Rule 12 green in writing.

**11. Multi-Layer Rule.**
Never submit a chain that lives in one component. The finding must cross at least two architectural
layers that were designed to different assumptions — identity vs lease, control plane vs data plane,
approval-time state vs execution-time state, policy compiler vs policy enforcer, versioned store vs
cache, config store vs approval store. Name both layers, and name the assumption each side makes
about the other.

**12. Fix-Survival Rule.**
Never submit without writing this sentence in full: *"If the vendor shipped the obvious fix for the
shallow bug in this area, my chain would still work because ___."* If the obvious fix kills it, it
is a variant of the shallow bug and it will be closed as a duplicate of it.

**13. Emergence Rule.**
Never submit when every component behaved as its own specification says it should and the
composition is still harmless. The finding must be that the **composition** violates a guarantee
while each part in isolation looks correct. If a single component is plainly broken on its own, that
component is the report someone else already filed.

### Corollary — re-entry into closed classes

The closures burn *root causes*, not *surfaces*. A closed area may be re-entered only when Rule 12
is satisfied explicitly: the announced or obvious fix for the closed report does not prevent the new
vector, and the mechanism is architecturally different rather than a rephrasing. Every re-entry
carries a written diff against the closure text in `KNOWNREPORTS.md`, produced *before* any testing
time is spent.

### Honesty clause — this rule raises the bar for what to look for, not for what counts as proven

Depth is a target-selection rule. It never licenses writing up an architectural narrative that has
not been demonstrated. Every link in a chain needs a raw transcript from an oracle outside the
system under test. Where only part of a chain can be shown, the shown part is reported and the rest
is labelled inferred.

---


## GATE 5 — Named-Principal Gate (MANDATORY, all sessions, all findings)

Added 2026-09-02. Standing workspace rule. Runs in addition to Gates 1–4, and it runs **first**,
before any code is read and before any rig is built.

Sessions 8 and 9 each spent an entire budget on surfaces whose tenant boundary turned out to be the
account or the OS user. Every primitive found was therefore already available to the attacker
directly, and Rule 6 went red at the end instead of at the start.

**18. Named-Principal Rule.**
Never spend time on a surface without first writing one sentence that names a principal who is **not**
the account owner, **not** the OS user, and **not** the model, together with the credential or action
that lets them reach the surface. If the honest answer is "nobody who is not already the owner", stop.
The surface is closed no matter how deep the mechanism underneath it turns out to be.



## GATE 6 — Kill Gate (MANDATORY, all sessions, all findings)

Added 2026-09-02. Standing workspace rule. Runs after Gates 1–5 and settles what happens to a
primitive that keeps failing them.

**20. Kill Rule.**
A primitive that has survived one full session without passing every rule gets exactly **one** more
bounded program, and its kill criterion is written down *before* that program starts. If the program
ends without a demonstrated, attacker-readable, real-world impact that would oblige the vendor to
ship a fix, the finding is **closed**: recorded in `KNOWNREPORTS.md` as a self-closure with the
measurements that killed it, marked "do not re-enter" in `PROVEN.md`, and reopened only under the
Rule 12 corollary. Carrying an amber primitive forward across sessions is how two whole budgets were
spent on the same red Rule 6.

**21. Demonstrated-Impact Rule.**
"The mechanism is proven, the impact is inferred" is not a finding. A report must contain a
transcript in which **data the attacker did not already possess arrives somewhere the attacker
controls**, or **authority the attacker did not already hold is exercised**. Anything short of that
is a primitive. Primitives live in `PROVEN.md`; only demonstrated impact goes to the program.

---

## GATE 7 — Evidence-Hygiene Gate (MANDATORY, all sessions, all findings)

Added 2026-09-02 after an internal triage killed a fully-reproduced bundle before submission. Nothing
in that bundle was mis-measured. It died on two things that cost nothing to check and were never
checked. Gate 7 runs with Gate 1, at target-selection time.

**22. Documentation Rule.**
Never build a rig before quoting the vendor's own documentation of the behaviour you intend to call a
bug. Paste the quote into the working notes with its URL. If the docs describe the behaviour as
intended — or if the vendor ships a skill, template or changelog entry telling users to do exactly
that — the framing is dead and no amount of measurement revives it. Gate 1 Rule 4 covers Hacktivity
and CVEs; this covers the vendor's own words, which are what a triager reaches for first.

**23. No-Asserted-Negative Rule.**
Never write a negative claim into a report on the strength of a few probes that came back empty.
"No credential exists", "nothing is reachable", "the sandbox is contained" are claims a triager
disproves by reading one file, and a single disproved negative discredits every positive around it.
A negative requires an exhaustive sweep, and the sweep goes in the evidence file. Where a sweep is not
possible, the sentence is written as a bounded measurement — "these N paths were checked and were
empty" — never as a general negative.

### Corollary — asset selection is part of the finding

A chain that touches several products is filed against the asset whose boundary it actually breaks,
and the exclusions of *that* asset are the ones that must be defeated in writing. Filing infrastructure
behaviour under an asset whose exclusion list names it is how a report becomes Informative without ever
being wrong.

---

## GATE 8 — Golden Rules (MANDATORY, all sessions, all findings, all primitives)

Added 2026-09-03. Standing workspace rule. Runs **first**, before Gate 5, before any code is read and
before any rig is built. These are the two rules the researcher named as golden, plus the lesson that
cost four sessions.

**24. Victim-and-Loss Rule.**
Before spending time on anything — a surface, a primitive, an idea — write one sentence in this exact
shape: *"<a named principal who is not me> loses <a specific thing> when <a mechanical event>"*. No
sentence, no work. "Show me the victim and show me what they lose." This subsumes Rule 5 and hardens
it: Rule 5 asks for the sentence before submission, Rule 24 asks for it before the first minute is
spent. If the only sentence available is about the account owner, the OS user, or the researcher, the
surface is closed (see Rule 18).

**25. Secure-Individually Rule.**
Never pursue a thing that is broken on its own. The target must be a **seam**: two or more components
that are each correct, documented and defensible in isolation, whose *interaction* breaks a security
boundary — "secure individually, broken together". Name both sides, and name the assumption each side
makes about the other, before testing. A component that is plainly broken alone is the report someone
else already filed (Rule 13, Rule 16).

**26. Expiring-Blocker Rule.**
A provisioning or capability belief that blocks a track is a **measurement**, and measurements expire.
Re-falsify every carried-forward blocker in the first ten minutes of a session, from the shipping
code, before ranking anything. Sessions 8–16 carried *"no WSL, so `@anthropic-ai/sandbox-runtime` is
untestable on this box"*; the product had shipped a native Windows sandbox and nobody re-read the
binary. Four sessions before that ranked "we need a paid org" as the top blocker while the Console
surface was free the whole time.

### What Gate 8 does not license

Weird is allowed. Undocumented sources, strange states, unusual configurations and non-textbook
mechanisms are all legitimate targets **provided Rules 24 and 25 are satisfiable in writing**. What is
not allowed is a story: the honesty clauses of Gates 3 and 4 apply unchanged, every link still needs a
raw transcript from an oracle outside the system under test, and a primitive is not a finding until
Rule 21 is green.

---

## GATE 9 — Stated-Guard Gate (MANDATORY, all sessions, all findings)

Added 2026-09-03 after an adversarial triage killed a fully-reproduced bundle (P-29) on framing
rather than on evidence: the behaviour was the vendor's own documented default, and the asset's
out-of-scope list covers "abusing intended functionality".

**27. Stated-Guard Rule.**
Before any work on a candidate, write three lines:

1. the guarantee **in the vendor's own words, with its URL**;
2. the predicate in shipping code that implements it;
3. what is **gained** when it fails — data or authority the attacker did not hold.

Then name which of the asset's enumerated in-scope items the gain lands on, and which out-of-scope
line a triager reaches for first, and how the finding defeats it. For Claude Code the in-scope list
is exactly four items: bypassing permission prompts for unauthorized command execution; bypassing
permission prompts for file writes outside the working directory; misrepresenting parameters or
tools in permission prompts; executing commands or tools invisibly to users. The out-of-scope lines
to defeat are "abusing intended functionality of Claude CLI", "using aliased commands, symlinks or
other environment-specific settings to bypass permission prompts", and "local storage of Claude Code
credentials, configuration and logs".

**No stated guard, no work.** A finding that cannot quote the sentence it breaks is a configuration
observation, and it belongs in the vendor's issue tracker the way `anthropics/claude-code#87296` was
filed, not in a report slot.

---

## GATE 10 — Textbook-Impact Gate (MANDATORY, all sessions, all findings)

Added 2026-09-14. The 1win withdrawal CSRF passed initial review the same day it was filed and was
assigned to a security engineer with the bounty decision pending — the first report out of this
workspace to get that far rather than closing Informative or Duplicate. It is the reference
finding, and this gate is what it did differently.

It is worth naming plainly what it did *not* do. It failed Rule 10 structurally: its root cause is
one gateway method policy and the vendor fixes it by refusing GET on state-changing routes. Gate 3
would have told me to close the tab. Gate 3 was wrong, because Gate 3 was written after two
Duplicates and generalised from them into a rule that rejects the entire classical catalogue.

### Precedence

Where Gate 3 (Rules 10-13) and Gate 10 disagree, **Gate 10 wins for any candidate whose impact is
already demonstrated under Rule 21.** Rule 10 is demoted from a veto to a dupe-risk signal: a
one-line root cause raises dupe risk, it does not disqualify. Gate 3 keeps its job, which is target
selection while nothing is demonstrated yet. It is not a reason to discard a chain that already
moves money.

**28. Classical-Class Rule.**
Prefer a named textbook class — CSRF, IDOR, SSRF, auth bypass, injection, race — over an exotic
mechanism, provided the impact is demonstrated end to end. A triager recognises the class in one
line, maps it straight onto their own reward table, and never has to be taught a new model before
they can act. What gets paid is a recognisable class with an undeniable consequence, not novelty.
Exotic mechanisms are for the primitive register; the report wants the boring name.

**29. Chain-the-Anomalies Rule.**
A finding is assembled, not stumbled on. `ANOMALIES.md` holds the inputs; the finding is the
**join**. Before any hunting time is spent, write the join you intend to close in this shape:
*"A-x gives me X, A-y removes requirement Y, together they reach Z."* The reference finding is
A-16 (gateway serves POST-only routes over GET) + A-17 (`bundlecda.com` holds the same session at
`SameSite=None`) + A-20 (the balance is the withdrawal pipeline's last gate), joined by 5 EUR of
real funding. Each input on its own is a `200` that changes nothing a triager cares about. The
join is a payout to an attacker-chosen wallet.

**30. Observable-Oracle Rule.**
Impact counts only when it is readable in the **product's own surfaces**: the account's own balance,
its own transaction history, its own session. A status line on the attacker's page is a primitive.
The victim's balance going 5.10 to 0.10 in 1win's own header, with the payout row carrying the
attacker's address, is a finding. Every load-bearing leg carries two artifacts — one from the
vendor's own UI or API read back from the victim's session, and one from the browser itself
(Chrome's network log, with Chrome's own cookie attach/block decisions, not a script's).

**31. Concede-First Rule.**
Write the strongest objection to the report **inside** the report, before the triager reaches it.
The reference finding says in its own words that the payouts were cancelled by the researcher, that
none settled while observed, and what was not shown. A limit you concede costs one sentence. A limit
the triager discovers costs the report, and the next one too.

**32. Reversible-Impact Rule.**
Create the real effect, capture it, restore the state, and say so with ids and times. Money out of
the balance and a queue entry addressed to the attacker is the finding; leaving it sitting there is
not required and is not good conduct. Reversal is also what makes a chain re-runnable, which is how
the reference finding was captured twice on two different days.

**33. Field-Discipline Rule.**
Fill exactly the fields the platform asks for and nothing beside them. No gate notes, no severity
argument, no research narrative, no appendix of everything you learned. Respect the word ceiling:
the report that cleared triage is 698 words. Every sentence either states a fact the triager can
replay or names a limit. Severity is claimed by impact class against the programme's own table, and
the CVSS vector is given honestly even when it scores a bracket lower.

**34. Preserve-Everything Rule.**
A filed bundle is frozen the moment it is sent — byte-identical, nothing edited, nothing deleted,
including run logs and the artifacts that did not make it in. The engineer may come back weeks later
asking for the one leg that was cut for length. `SUBMISSIONS.md` carries the ledger: what was filed,
what state it is in, which anomalies it consumed, and the pre-written answers to the questions
triage is most likely to ask.

**35. Own-Outbox Rule.**
Added 2026-09-14. Before a surface is ranked — before the join is written, before the first
request — grep `SUBMISSIONS.md`, `ANOMALIES.md` and `knownreports_1win.md` for the **endpoint name**
and for the **service prefix**. Rule 4 sends you to Hacktivity and the disclosed corpus; neither
tells you what this workspace has already sent, and the burned surface is the union of the two.

Session 5 spent most of a budget re-deriving `api-v1-oauth-wallet-simpleBind` and the SIWE nonce
from scratch, proved it end to end, and then found the same surface had been filed as `#3910030`
five weeks earlier and closed Duplicate of `#3725811`. Nothing was mis-measured; the whole cost was
not looking in the outbox first.

The check is one command and it runs before the Gate 8 sentence, not after:

```bash
grep -rniE '<endpoint>|<service-prefix>' SUBMISSIONS.md ANOMALIES.md PROVEN.md knownreports_1win.md
```

A hit does not automatically close the surface — the Rule 12 corollary still governs re-entry — but
it changes what has to be written down before any time is spent: a diff against the closure text,
produced first. No grep, no ranking.
