---
name: designing-software-specs
description: Conduct a design conversation and produce an RFC 2119 specification ready for TDD implementation. Use when designing a new component or a substantial refactor.
---

# Designing Software Specs

A design conversation has a shape. Follow it. The output is an RFC 2119 spec — precise enough that a coding agent can implement it test-first without asking questions.

## 1. Understand the Domain First

Before proposing anything, establish:

- What problem does this component solve?
- What does it own, and what does it delegate?
- Who are its callers, and what do they need from it?
- What already exists that this must compose with?

Do not propose names or interfaces until the domain is clear. Premature naming forecloses options.

## 2. Design the Interfaces

Work interface-first. For each component:

- Name the interface after the domain concept, not the mechanism.
- Identify the minimal contract — what must every implementation honour?
- Identify the null object — what does absence look like?
- Identify the wrapping structure — what decorates, what composes?
- Confirm dependency direction — foundational types must not depend on domain types that use them.

Surface smells early: if a component needs a "has this already run" flag, it is not a pure decorator. If a synchronous interface needs a suspending implementation, the interface boundary is wrong.

## 3. NFR Sweep (do this before drafting the spec)

Ask explicitly — do not skip:

- **Latency** — are there timing guarantees? Must any operation complete within a frame budget, startup window, or user-perceptible threshold?
- **Memory** — are there allocation constraints? Hot paths? Objects that must not hold references beyond a bounded scope?
- **Threading** — which operations are safe to call from which threads? Are there synchronisation requirements?
- **Failure modes** — what happens when a dependency fails? Must errors be swallowed, surfaced, or logged? Are there OTel spec constraints (e.g., MUST NOT throw)?
- **API / platform constraints** — are there minimum API level gates, platform-specific behaviour, or version compatibility requirements?

NFRs that are resolved become MUSTs. NFRs with implementation latitude become SHOULDs. NFRs that are optional become MAYs. If an NFR is deferred, say so explicitly — do not silently omit it.

## 4. Draft the Spec

Write RFC 2119. One section per component, ordered from foundational to dependent.

- **MUST** — required for conformance. No latitude.
- **SHOULD** — recommended; valid reasons to deviate exist but must be understood.
- **MAY** — optional. Implementer's choice.
- **MUST NOT** / **SHOULD NOT** — prohibitions, strong and advisory respectively.

Each section covers:
1. What the component is (one sentence)
2. Behavioural MUSTs
3. NFR MUSTs/SHOULDs derived from the sweep
4. If the design involves a composition chain, close the spec with a **Composition** section describing the wrapping order and any components that stand outside it.

## 5. OOP Review

Before handing off, load `writing-organism-oriented-code` and review the spec against it. Flag violations and resolve them before implementation begins.

## 6. Handoff

The spec is ready when:

- Every component has a clear interface and null object (where applicable)
- NFR sweep is complete and findings are in the spec
- OOP review is done and violations resolved
- A coding agent could implement it TDD without asking design questions

Do not hand off a spec that defers design decisions to the implementer. That is not a spec — it is a wish list.

## 7. What NOT to Do

- Do not name interfaces or propose structure before the domain is understood.
- Do not skip the NFR sweep — omitting it produces specs that are silent on failure modes, threading, and performance expectations.
- Do not use MUST for requirements that have valid implementation latitude — that is what SHOULD is for.
- Do not silently omit deferred NFRs — note them explicitly in the spec.
- Do not include a Composition section unless composition is actually part of the design.
- Do not hand off a spec with open design questions — resolve them first.
- Do not skip the OOP review — violations caught before implementation are cheaper than violations caught in review.
