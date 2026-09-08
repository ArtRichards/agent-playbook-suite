# Agentic testing: failure modes and proposed suite improvements

Lifecycle: draft
Role: notes
Project: agent-playbook-suite
Updated: 2026-09-08

The most useful lesson from Dan Luu’s [“How well do agents use test/verification techniques?”](https://danluu.com/agentic-testing/) is that **agents can identify a risky behavior, write tests for it, get those tests passing, and still never distinguish the correct implementation from a plausible incorrect one**. More tests, more iterations, and a more sophisticated tool do not necessarily repair that gap.

My recommendation for [Agent Playbook Suite](https://github.com/ArtRichards/agent-playbook-suite) is a small refinement of its existing testing guidance: use trustworthy expectations, choose cases that expose likely mistakes, and diagnose failures before changing expectations or implementation. The central principle is **before counting a check as evidence, establish what wrong answer it can reject**. Apply it as a quick design or review question; ordinary inspection or existing RED feedback will often answer it. Reuse existing checks and add deeper testing only for a specific remaining gap or an existing project requirement.

**Recommended scope:** a few wording changes within the existing workflow. No additional default phase, readiness gate, document, matrix column, dependency, or reviewer. The article examples and methodology catalogue below explain the reasoning; they are reference material, not a checklist of work for each milestone.

This assessment reviews the article as retrieved on 8 September 2026 and the suite’s **0.8.1** payload at [commit 975d545](https://github.com/ArtRichards/agent-playbook-suite/commit/975d545d61bf7722fdbbb8095c788e9074ebb9fb). Article observations are linked below. Proposed remedies, examples explicitly marked illustrative, and suggested workflow changes are my analysis; the article does not establish that they will improve this suite.

The minimal scope below was subsequently authorized and applied to the working
skill payload and public documentation for version **0.8.2**. The
[implementation record](../feedback-integration/feedback-log.md) tracks the
changes and validation; the broader proposals remain deferred.

## What the experiment establishes

Luu compared 26 prompt conditions and four skills, primarily on implementing Zstd in Rust, using GPT-5.6 Sol at medium and xhigh effort, with 80 runs per condition per effort. The main correctness measure was the **fraction of runs passing every hidden test**, not the fraction of individual tests passing. He also tried IMAP and a few other RFC tasks. [Experiment and results](https://danluu.com/agentic-testing/#overall-results).

No instruction produced an overwhelming improvement. Default behavior performed above average. Several established skills added ineffective work; a short custom skill targeting specific failure modes achieved the highest observed score, although important instructions were usually ignored. The author explicitly cautions against treating the ordering as a reliable ranking of techniques. In particular, agents frequently did not implement the named technique faithfully. These results concern agents’ use of techniques under these conditions, not the intrinsic effectiveness of TDD, differential testing, or formal verification.

The evidence has practical limits. The tasks are concentrated in RFC implementation; the IMAP perfect-pass measure had a severe floor effect; some ACL2 out-of-memory runs were excluded; detailed experimental procedures are incomplete; and skill exposure, model effort, and tool use varied. Selecting a winner among many conditions also warrants replication. The author’s reports that interactive guidance and a good harness help are valuable experience, but not a controlled comparison in this experiment. There is no population-wide frequency estimate for the failure modes below. [Caveats](https://danluu.com/agentic-testing/#appendix-experimental-details), [IMAP](https://danluu.com/agentic-testing/#fn:I), [ACL2](https://danluu.com/agentic-testing/#acl2), [practical experience](https://danluu.com/agentic-testing/#how-do-you-get-agents-to-write-good-tests).

## Failure examples and what should change

These examples are the strongest basis for changes to the suite. The practical lessons apply when the behavior or technique is relevant. Most can be addressed while choosing or reviewing an ordinary test; sophisticated techniques remain optional unless already selected by project policy.

| Article example | Why the testing was ineffective | Practical lesson when relevant |
|---|---|---|
| **Reversal tested with palindromic data.** Even a run using the custom skill identified reversal as risky and still chose a palindrome. [Source](https://danluu.com/agentic-testing/#skill) | Correct and reversed interpretations produce the same observation. Naming the risk did not produce a useful test. | Choose non-palindromic input. Illustratively, `001011` differs from its reversal; a complete codec fixture must also be valid. Inspection usually establishes this distinction without building a faulty implementation. |
| **Four-stream decoding tested with four identical, trivial streams.** TDD and Verus conditions repeatedly missed the jump-table behavior. [TDD](https://danluu.com/agentic-testing/#tdd), [Verus](https://danluu.com/agentic-testing/#verus) | Swapping streams can preserve the expected output. A test can cover a feature without distinguishing its incorrect implementations. | Use distinguishable valid contents and check output order. Different lengths are useful when offset handling is in scope. This is a fixture choice within the existing test. |
| **Differential testing repeated the same algorithm and bug.** Agents did not build two complete independent implementations; many comparisons were trivial or duplicated logic. [Source](https://danluu.com/agentic-testing/#differential-testing) | Agreement between correlated implementations is weak evidence. A “reference” can reproduce exactly the assumption under examination. | When comparison is selected, use an independent reference within its supported domain. Avoid sharing the helper that computes the disputed answer. Resolve disagreements against the contract. |
| **Random inputs mostly exercised rejection.** Fuzzing and QuickCheck runs concentrated on invalid inputs. Even “audit and fuzz risky areas” often did this. [Fuzzing](https://danluu.com/agentic-testing/#fuzzing), [QuickCheck](https://danluu.com/agentic-testing/#quickcheck), [targeted fuzzing](https://danluu.com/agentic-testing/#audit-and-fuzz-risky-areas) | Randomness supplied volume without reaching the behavior where bugs lived. | When generated tests are selected, inspect a few samples and confirm that they exercise the intended behavior. Adjust a rejection-only generator when successful processing is the target. Use existing coverage output if needed; no new dashboard is necessary. |
| **The oracle was only “does not crash.”** Risk-targeted fuzzing often did not check decoded output. [Source](https://danluu.com/agentic-testing/#audit-and-fuzz-risky-areas) | A decoder that returns wrong bytes without crashing can pass. Safety and functional correctness are different claims. | Assert the relevant output or effect when testing correctness. Keep crash checks for robustness; they do not establish correct output. |
| **Properties and round trips targeted easy behavior.** Hegel checked malformed-input handling and simple round trips; metamorphic tests sometimes used legitimate relations that missed frequent faults. [Hegel](https://danluu.com/agentic-testing/#hegel), [metamorphic](https://danluu.com/agentic-testing/#metamorphic-testing) | A true property can be insufficient or unrelated to the likely defect. Two mutually wrong encoder/decoder functions can also round-trip successfully. | Choose a property relevant to the changed behavior. If a round trip could preserve the suspected mistake, add a trustworthy example that distinguishes it. |
| **Proofs were vacuous or disconnected from implementation.** Verus proofs included assumptions restated as conclusions; SMT work did not prevent addition being implemented as bitwise OR. [Verus](https://danluu.com/agentic-testing/#verus), [SMT](https://danluu.com/agentic-testing/#smt) | A proof can be valid while proving little about the shipped code. An assumed precondition can exclude the defect. | When formal verification is selected, check what the proof establishes about the actual code. An abstract proof alone does not verify the implementation. |
| **A model found an irrelevant overflow.** Alloy’s 8-bit counterexample motivated extra Rust code even though the implementation used 64-bit `usize` and the overflow could not occur for the inputs. TLA+ model fixes did not visibly lead to Rust fixes. [Alloy](https://danluu.com/agentic-testing/#alloy), [TLA+](https://danluu.com/agentic-testing/#tla) | The modeled state space did not adequately correspond to the runtime system. Counterexamples can cause unnecessary complexity. | Before changing code for a model counterexample, check that its assumptions match the runtime and that the failure is possible in the supported domain. |
| **TDD generated more tests and more incorrect expectations.** Test counts increased across broad classes, including integration and E2E, without preventing important errors. [Source](https://danluu.com/agentic-testing/#tdd) | Test-first order and frequent GREEN feedback can entrench a mistaken interpretation. “Too many unit tests” alone does not explain this result. | Keep behavior contracts and meaningful tests before implementation; adjust cases when implementation reveals a gap. Diagnose failures against the contract before changing code or expectations. |
| **A framework name substituted for its defining operation.** Agents put ordinary tests inside rstest or Insta; mutation instructions rarely produced actual mutation testing. [rstest](https://danluu.com/agentic-testing/#rstest), [Insta](https://danluu.com/agentic-testing/#insta), [mutation](https://danluu.com/agentic-testing/#mutation-testing) | Tool import, invocation, or a prose claim was mistaken for method execution. | Check the normal tool output when relying on a selected technique. Distinguish input mutation from implementation mutation. Additional evidence formats are unnecessary. |
| **Freshness instructions and skills were inconsistently followed.** The custom skill rarely obtained fresh derivations; some agents never opened available skills. One skill’s dependency-approval instruction impeded its intended approach in autonomous runs. [Custom skill](https://danluu.com/agentic-testing/#skill), [Trail of Bits skill](https://danluu.com/agentic-testing/#tob-skill) | Availability is not execution. A prompt cannot itself create an independent context or make a missing capability available. | Verify already-selected actions through their outputs and report unavailable capabilities honestly. The suite already orchestrates reviewers; this observation does not justify more reviewers by default. |
| **Long instructions increased cost without useful evidence.** Hegel’s skill and Rust reference exceeded 20,000 tokens and were repeatedly read; more structured work did not improve correctness. [Source](https://danluu.com/agentic-testing/#hegel) | Context and execution budgets were spent on low-value activity. | Use a short operational core and load a recipe only when selected. Bound runs, summarize machine output, retain reproducible artifacts, and stop repeating a method when it supplies no new evidence. Measure cost alongside correctness. |

One particularly useful arithmetic witness follows directly from the article’s SMT example. The intended computation was `byte1 + (byte2 << 8) + 0x7F00`; a recurring error used OR instead of the last addition. This illustrative check separates those alternatives:

```sh
python3 - <<'PY'
byte1, byte2 = 0, 1
correct = byte1 + (byte2 << 8) + 0x7F00
incorrect = (byte1 + (byte2 << 8)) | 0x7F00
assert correct == 0x8000
assert incorrect == 0x7F00
assert correct != incorrect
print("The witness distinguishes addition from OR.")
PY
```

This is an arithmetic demonstration, not a complete Zstd conformance test. It illustrates the central principle: **before counting a check as evidence, establish what wrong answer it can reject**. Here that answer is the OR result. In ordinary work, the distinction can be evident from the test itself; constructing a second implementation is unnecessary.

Mocking needs a similarly precise treatment. The article quotes criticism of excessive mocks, but also describes successful E2E tests with mocked I/O after tests were placed in a separate crate with interface constraints. The useful distinction is whether a mock removes the behavior being tested. Controlled external I/O can help; mocking away persistence in a persistence test defeats the claim. A separate test package improves structural separation but does not make tests private or immutable when the agent can edit it. [Discussion](https://danluu.com/agentic-testing/#how-do-you-get-agents-to-write-good-tests).

## Testing methodologies mentioned

These belong to different dimensions. Unit versus E2E describes execution scope; properties and references provide ways to judge results; fuzzing generates/searches inputs; mutation examines test sensitivity; TDD specifies work order. A stateful property test can also be an integration test, use a differential oracle, and participate in TDD.

| Method or family | Tools and forms discussed | Useful application and article limitation |
|---|---|---|
| Example-based unit, integration, and E2E testing | Rust’s built-in test framework; mocked I/O; separate test crates | Concrete observable examples and system boundaries. Fixed expected values can simply encode a mistaken implementation. |
| Fixture-based and parameterized testing | rstest | Organize reusable setup and varied cases. Merely wrapping an ordinary test in the framework adds little. |
| Snapshot or golden testing | Insta; discussion of Jamie Brandon’s experience | Compare output with a reviewed baseline. Generating a snapshot from the implementation does not establish that the snapshot is correct. |
| Test-driven development | TDD prompt; ECC Rust skill | Use failing behavioral checks to guide development. The experiment generally did not demonstrate faithful fine-grained TDD, and test-first timing did not ensure sound expectations. |
| Property-based testing | QuickCheck, proptest, Hegel; Trail of Bits and Hegel skills | Assert justified properties across generated inputs. Most use was weak, but proptest’s shrinking sometimes produced useful smaller counterexamples. [Observation](https://danluu.com/agentic-testing/#proptest). |
| Fuzzing and randomized testing | Random bytes, mutated examples, structured generation, risk-targeted fuzzing | Explore inputs and execution paths. Structured generation occurred in only 10 of 160 fuzzing runs and found real bugs in five; this is a promising observation, not a causal estimate of its success rate. [Observation](https://danluu.com/agentic-testing/#fuzzing). |
| Metamorphic and round-trip testing | Relations between transformed inputs/outputs; encode/decode round trips | Useful when exact expected output is difficult to compute. The relation must be justified and sensitive to relevant faults. |
| Differential testing | Comparing implementations or independent derivations | Particularly useful for compatibility. Shared algorithms, helpers, assumptions, or reference defects weaken independence. |
| Mutation testing | Deliberately changing implementation behavior and checking whether tests fail | Tests the tests’ sensitivity. Agents rarely did the defining operation in this experiment. |
| SMT-based reasoning | Z3, cvc5, Yices | Check arithmetic, constraints, and bounded alternatives. Scratchpad calculations alone did not bind the result to the code. |
| Deductive verification and theorem proving | Verus, Creusot, Lean 4, ACL2 | Prove relevant claims under stated assumptions; Verus/Creusot support verification tied to code. Agents mostly proved peripheral or disconnected claims. |
| Model checking and behavioral modeling | Alloy, Spin, TLA+, Kani | Examine states, transitions, and counterexamples within the tool’s model/bounds. Kani sometimes checked actual Rust and caught one nontrivial bug; abstraction correspondence remained a central issue elsewhere. |
| Auditing and risk-guided review | Post-implementation audit; audit plus fuzzing | Examine difficult mechanisms and feature interactions. Risk selection was often sensible, but independent or useful checking did not reliably follow. |
| Black-box and white-box testing | Specification-led versus implementation-informed test design | Kreinin hypothesizes that examining the implementation can reveal difficult cases missed by upfront black-box tests. This was a proposed explanation, not a separately tested condition. [Discussion](https://danluu.com/agentic-testing/#tdd). |

“Default,” “Make no mistakes,” and “Judgement” were experimental controls or prompting conditions, not separate testing methodologies. The four skill conditions were Hegel’s official skill, ECC Rust testing, Trail of Bits property-based testing, and Luu’s custom skill. The article’s opening inventory and linked method sections distinguish these conditions.

## Minimal recommended changes

The suite’s [shared quality model](../../../plugins/agent-playbook-suite/skills/_shared/references/agentic-quality-model.md) already asks for semantic, structure-insensitive tests, proportional gates, realistic paths, and restraint with mocks. Those rules are sound. The most useful additions are three habits within ordinary test writing and review:

1. **Use a trustworthy expected result.** Derive it from the agreed behavior, specification, or an established reference. Do not simply capture the implementation’s current output and call it correct. Existing acceptance criteria normally suffice; add a source note only when the expectation is non-obvious or disputed.
2. **Choose inputs that expose a likely mistake.** Use distinct values for ordering or routing, and relevant boundary cases. Exercise the actual behavior the test claims to cover. Usually the test itself and ordinary RED feedback show this; there is no routine requirement to build mutants, write a second implementation, or document a separate proof of sensitivity.
3. **Diagnose failures before changing expectations or code.** A failure may be a product bug, a wrong expectation, or a fixture/environment problem. Check against the agreed behavior and fix the responsible part. Correcting a demonstrably wrong test does not require inventing a contract change; weakening a valid test to accommodate a bug remains unacceptable.

These habits improve the choice of checks rather than prescribe a larger test suite. A straightforward change may already have adequate tests. An existing test can often be improved by changing an uninformative fixture or assertion instead of adding another test.

A suitable short instruction for the existing test-writing guidance would be:

> Use the project’s existing checks to test the changed observable behavior. Take expected results from the contract or a trustworthy reference, not solely from the implementation. Before counting a check as evidence, establish what wrong answer it can reject; ordinary inspection or RED feedback usually suffices. Use distinguishable values when order or identity matters, and avoid asserting incidental internal structure. When a check fails, determine whether the behavior, expectation, or setup is wrong before changing it. Add a deeper technique only for a specific remaining gap or an existing project requirement. Once selected checks pass and identified concerns are resolved, stop.

This should replace or tighten overlapping guidance, not become another checklist repeated in every phase.

## Small changes to the existing suite

| Location | Keep in the proposal | Cost control |
|---|---|---|
| [Project foundation](../../../plugins/agent-playbook-suite/skills/project-foundation/references/foundation-playbook.md) and its generated context | Make existing acceptance criteria state an observable outcome and clarify consequential ambiguity. Reuse the selected test commands and existing contract-to-test mapping. | No new behavior inventory, mandatory use-case document, expanded matrix fields, or calibration gate. A greenfield harness can remain a milestone deliverable. |
| [Shared quality guidance](../../../plugins/agent-playbook-suite/skills/_shared/references/agentic-quality-model.md) and [test-writing phases](../../../plugins/agent-playbook-suite/skills/create-milestones/references/tdd-phases.md) | Incorporate the three habits above, with a short asymmetric-input example. Review tests for both weak assertions and constraints on irrelevant implementation details. | Apply during existing writing/review; no additional reviewer or per-test record. New tests remain driven by changed behavior and actual gaps. |
| TDD phase wording | Keep meaningful RED evidence for new or corrected behavior. Allow a behavior-preserving refactor to use an adequate GREEN baseline. Permit integration checks as soon as useful, without treating Phase 9 as the first allowed end-to-end run. | No manufactured failures, mandatory mutant demonstrations, phase reordering, or separate early-integration gate. Add tests only if the refactor’s relevant behavior lacks adequate coverage. |
| [Simplify](../../../plugins/agent-playbook-suite/skills/simplify/SKILL.md) and [sync-and-commit](../../../plugins/agent-playbook-suite/skills/sync-and-commit/SKILL.md) | Describe a commit as an accepted baseline, not proof of correctness. Diagnose failures and meaningful protection loss rather than equating every test-count or metric decrease with a regression. Apply the same selected-gate policy across consumers. | Reuse existing results where their relevant inputs are unchanged. Preserve existing approval requirements for actual adequacy reductions; add no new approvals or gate categories. |

The current foundation template and DoR also disagree about the presence of a matrix template. Resolve that by pointing to the existing [milestone matrix](../../../plugins/agent-playbook-suite/skills/create-milestones/references/milestone-playbook.md) or providing that same minimal shape. It does not justify inventing a richer testing schema.

## Proposals to remove or defer

The earlier proposal combined too many individually reasonable measures. The following are outside the recommended default change:

| Proposal | Disposition |
|---|---|
| A six-part evidence record for each consequential behavior; separate oracle inventories and expanded quality metrics | **Remove.** Existing contracts, tests, and normal result logs are sufficient for routine work. Record only a non-obvious assumption or unresolved concern. |
| Deliberately breaking code to validate every important test or refactor baseline | **Remove as a routine obligation.** Use targeted mutation when a particular assertion’s effectiveness is uncertain or project policy already selects it. |
| New foundation calibration gates and expanded instructions for all ten phases | **Remove.** Keep the targeted wording corrections above. |
| Additional independent derivation agents or extra review passes | **Defer.** Use existing review. A specific difficult ambiguity may justify independent investigation; the article does not establish that more reviewers improve correctness. |
| New evidence-collection infrastructure, private acceptance infrastructure, dashboards, or a catalogue of executable testing recipes | **Defer until a recurring practical gap justifies the work.** Reuse project tools first. |
| A broad benchmark programme with multiple experimental arms and ablations | **Defer.** Begin with a small maintainer comparison using existing scenario checks. |

Advanced testing remains available under existing project policy. When fuzzing or property tests are selected, inspect whether generated cases exercise meaningful behavior. When differential testing is selected, check that its reference does not reproduce the same disputed logic. These are checks on a chosen technique, not reasons to introduce it everywhere.

## When to stop or deepen testing

**Stop when the selected checks pass and the specific concerns identified for the change are resolved.** Passing tests do not prove universal correctness, but that limitation is not itself a reason to keep adding tests, reviewers, or tools. Broaden testing when a new failure, a concrete untested behavior, or an existing risk requirement warrants it. Use the cheapest check that addresses that gap.

This does not waive already-selected gates or authorize deleting valid regression coverage. A required but unavailable check remains an unresolved or explicitly deferred item under existing policy, not a passing result. Documentation and other low-impact work should keep proportionate explicit checks instead of gaining artificial product tests.

## A small example

For a settings store that promises persistence across process restart and independent keys, one focused scenario can save two distinct values, overwrite one key, restart, and read both through the public interface using a temporary real store. It checks the relevant outcomes without specifying internal helper calls or storage layout.

There is no automatic need to add a dictionary model, randomized histories, deliberate mutants, or another reviewer. A concrete concurrency or partial-failure requirement could justify additional checks. For an unrelated change already covered by this scenario, running the existing test may be sufficient.

## Check that the refinement earns its cost

The suite already has [CI validation](https://github.com/ArtRichards/agent-playbook-suite/blob/975d545d61bf7722fdbbb8095c788e9074ebb9fb/.github/workflows/validate.yml) and recorded [agent scenario checks](../feedback-integration/feedback-log.md). Maintainers can compare the current and revised guidance on a few representative tasks using those facilities, comparable budgets, and independently checked acceptance outcomes. Include both a weak-test trap from the article and a simple change where extra process would be wasteful. Observe useful defect detection, incorrect expectations, time/cost, and unnecessary work.

A small trial can reveal regressions or excessive overhead; it cannot establish a general performance advantage. Repeat or expand it only if a release decision needs stronger evidence. This is suite-maintainer validation, not an extra phase for every adopting project. The proposed refinement should earn its place by improving the quality of ordinary tests at modest cost.
