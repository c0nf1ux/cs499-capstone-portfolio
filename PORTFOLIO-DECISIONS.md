# Enhancement Decisions & Trade-Offs

Each finding in [PORTFOLIO-CODE-REVIEW.md](PORTFOLIO-CODE-REVIEW.md) gets a specific fix here — what it is, why it's the right call over the alternatives, what it costs, and which CS-499 course outcome it serves.

---

## ABCU Advising Program (CS-300)

### 1. Benchmark `std::map` against a hash table

**Decision:** Implement the same course lookup with a hash table, then benchmark load and search time against the existing `std::map` version — on the real catalog CSV, not synthetic data.

**Why:** The current O(log n) claim has never been tested against the O(1)-average alternative. A complexity analysis that was never measured is a guess wearing a citation.

**Trade-off:** A second implementation to build and maintain for comparison, vs. an actual measured answer instead of a textbook one.

**Result:** Replaces an assumed complexity trade-off with a measured one — directly the kind of evidence a trade-off analysis needs.

### 2. Model prerequisites as a graph, add cycle detection

**Decision:** Replace the plain-string prerequisite list with a directed graph and implement cycle detection plus topological sort.

**Why:** Right now, two courses requiring each other, or a prerequisite chain that never resolves, gets printed as if it were valid — the program has no way to notice. Cycle detection turns an associative-array lookup into a real graph algorithm that can actually flag an unsatisfiable or circular chain instead of silently printing nonsense.

**Trade-off:** A meaningfully larger data-structure change than the other fixes here, but it's the only way to turn "this would print a broken chain" into "this catches a broken chain."

**Result:** The program can validate its own data instead of trusting it blindly.

### 3. Unit tests around parsing, lookup, and graph validation

**Decision:** Add automated tests, so correctness stops depending on a human reading console output against a sample-output PDF.

**Why:** The original verification method (eyeballing output) doesn't scale and doesn't catch regressions.

**Trade-off:** Upfront test-writing time against a program that's "worked so far" by manual inspection.

**Result:** Correctness becomes something the build can prove, not something I have to remember to re-check by hand.

### 4. Named enum for the menu + real `cin` failure handling

**Decision:** Replace the raw menu literals (`case 1`, `case 2`...) with a named enum, and add `cin.clear()`/`cin.ignore()` around the input read.

**Why:** This is the actual bug, not a style complaint — a non-numeric menu input currently locks the program into repeating "not a valid option" forever, because the failed `cin` state is never cleared.

**Trade-off:** Small, contained fix — low cost, removes a real infinite-loop defect.

**Result:** Bad input gets rejected cleanly instead of hanging the program.

### 5. Replace the hand-rolled CSV parser with the reusable one

**Decision:** Swap the from-scratch CSV splitting in this file for the parser class already built in an earlier assignment in the same course.

**Why:** Maintaining two copies of the same parsing logic in two places is exactly the kind of duplication the checklist flags, and this project already has a better answer sitting unused elsewhere in my own coursework.

**Trade-off:** None, really — pure removal of duplicated code.

**Result:** One parser to maintain instead of two.

**Outcome mapping:** the ordered-map-vs-hash-table trade-off and graph validation are the primary evidence for *Outcome 3*; the benchmarking methodology supports *Outcome 4*; the complexity write-up supports *Outcome 2*.

---

## Secure REST Service (CS-305)

### 1. Merge the two projects behind one TLS boundary

**Decision:** Run the CRUD service over the existing TLS configuration instead of leaving it as a second, unconnected Maven project.

**Why:** Right now the "secure" half of this artifact and the service half have never touched. A TLS server that doesn't protect anything isn't a security boundary, it's a demo.

**Trade-off:** Real integration work merging two previously independent build configs, for a security guarantee that currently doesn't exist at all.

**Result:** The service the TLS layer is supposed to protect actually runs behind it.

### 2. Real auth + role-based authorization, remove dead scaffolding

**Decision:** Replace the skeleton-level auth on `CRUDController` with real authentication and role-based authorization, and delete `customer.java` outright instead of leaving unused, half-initialized code in the artifact.

**Why:** There's currently no security annotation anywhere on the controller, and `customer.java` is scaffolding nothing calls — dead code that opens a database connection (via the related `DocData` issue below) is its own risk, not a neutral leftover.

**Trade-off:** Standard auth-integration effort; the removal side costs nothing but discipline.

**Result:** The endpoint is actually gated, and the artifact stops carrying code that does nothing but create confusion.

### 3. Rewrite `DocData`: kill the hardcoded credential, fix the dead `read_document` path, fix `toString`

**Decision:** Remove the hardcoded `root`/`root` JDBC credential and raw connection string, replace it with a repository pattern over externalized configuration, give `DocData` a real `toString()` (or a proper DTO), and either wire `read_document` into the actual request path or delete it.

**Why:** This is the highest-severity finding in the whole artifact. The credential is a textbook default-admin pair in source control; the method that uses it is never called at all; and the one endpoint that's supposed to return data returns a default `Object.toString()` hash instead — it doesn't work, today, as written.

**Trade-off:** The most invasive change in this category, but it's fixing the thing that's actually broken, not a hypothetical.

**Result:** The endpoint returns real content, and there's no hardcoded admin credential sitting in source.

### 4. Validation, centralized exception handling, and an enforced dependency gate

**Decision:** Add real input validation and centralized exception handling (fixing the `printStackTrace` leak directly), fix the dead `business_name` filter parameter, rename `content`/`content2` to fields that describe what they hold, and make the OWASP Dependency Check plugin a hard build gate instead of a silent report.

**Why:** Each of these is a checklist item the code currently fails outright — not stylistic — with the dependency-check gate being the one that prevents the *next* vulnerable dependency from landing unnoticed.

**Trade-off:** Incremental, low-risk changes; the build-gate change has the side effect of occasionally blocking a build on a dependency bump, which is the point.

**Result:** Errors stop leaking internals, the filter parameter actually filters, and a known-vulnerable dependency fails the build instead of just getting logged.

### 5. Integration tests across the hardened paths

**Decision:** Add integration tests for rejected auth, invalid input, the TLS handshake itself, and an assertion that the read endpoint returns real content instead of a default object hash.

**Why:** Every fix above needs a test proving it actually holds, especially the content-return fix — that's the one a human skimming the code could easily believe is already working.

**Trade-off:** Test-writing time against an artifact that currently has none.

**Result:** The hardening is provable, not just asserted.

**Outcome mapping:** the security mindset (anticipate / expose / mitigate) across these five is the primary evidence for *Outcome 5*; the enforced OWASP gate supports *Outcome 4*; documenting each vulnerability and its remediation supports *Outcome 2*.

---

## Grazioso Salvare Dashboard (CS-340)

### 1. Complete the CRUD module

**Decision:** Add `update` and `delete`, held to the same validation and error-handling standard as the existing `create`/`read`.

**Why:** Half the acronym the class name promises isn't implemented.

**Trade-off:** Straightforward addition; the main cost is holding the new methods to the same error-handling bar already set.

**Result:** The module actually does what its name says.

### 2. One credential, in one place, read from the environment

**Decision:** Move the connection string and all credentials into environment variables — not just the password, but `HOST`, `PORT`, `DB`, and `COL` too — and, just as importantly, make the dashboard notebook read that *same* environment variable instead of carrying its own separate hardcoded copy.

**Why:** The password currently exists hardcoded in two files that can only be kept in sync by a human remembering both exist. They've already drifted (the notebook's constructor call doesn't even match the class's actual signature anymore). Fixing only the field that "looks worst" leaves the duplication — the real risk — untouched.

**Trade-off:** This is the one enhancement here that's genuinely about discipline over difficulty: the technical change is simple, but it only works if both files are actually updated to the same source of truth.

**Result:** One credential to rotate, not two files to remember.

### 3. Index the dashboard's actual filter/sort fields, and measure it

**Decision:** Add indexes on the fields the dashboard filters and sorts on, then measure query time before and after on the full ~10,000-record dataset.

**Why:** Every query currently runs against the raw collection with no indexing — flagged in the project's own README previously and never acted on.

**Trade-off:** Index maintenance overhead on writes, in exchange for query performance that's measured rather than assumed.

**Result:** A performance claim backed by a before/after number, not an assumption.

### 4. A real aggregation pipeline for summary analytics

**Decision:** Build a MongoDB aggregation pipeline for outcome rate by breed and rescue type, and intake-to-outcome time distribution.

**Why:** This is the enhancement that maps most directly to the analytics-focused roles I'm targeting — it turns a read-only wrapper into something that produces the kind of aggregated insight a rescue organization would actually use to make decisions, not just a database exercise for its own sake.

**Trade-off:** The most analytically ambitious piece here; it's also the one with the clearest real-world payoff.

**Result:** The dashboard moves from showing raw records to answering an actual operational question.

### 5. Logging around every database operation

**Decision:** Add an audit trail for every database read/write — who, what, when.

**Why:** There's currently no record of any database operation at all.

**Trade-off:** Minor overhead, standard practice.

**Result:** Operations become traceable after the fact instead of invisible.

**Outcome mapping:** the aggregation pipeline and index-driven tuning are the primary evidence for *Outcome 4*; indexing as an efficiency decision supports *Outcome 3*; credential hygiene and audit logging support *Outcome 5*; and because the dashboard already exists to support one organization's real decisions, the analytics work deepens *Outcome 1*.

---

## The throughline

All three artifacts move the same direction: from read-only lookups toward code that's validated, measured, secured, and analytically richer. In the algorithms artifact, that means proving a claimed complexity with an actual measurement and replacing silent string comparison with a real graph structure. In the software design artifact, it means closing a security boundary that doesn't actually connect to the service it's supposed to protect, and making the one working endpoint return real data instead of a default object hash. In the database artifact, it means finishing an incomplete CRUD implementation, collapsing a credential that lives in two unsynced files down to one, and turning a read-only wrapper into something that produces the kind of analytics this organization would actually use. One throughline across all five course outcomes, not three disconnected fixes.
