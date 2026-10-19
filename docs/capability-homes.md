# Capability homes in the `true-line` organisation

**Status:** Accepted
**Date:** 2026-10-19
**Issue:** true-line/trueline-agent-ci#1

## Context

The `qwts` personal account's repositories were transferred into the
`true-line` organisation. Eight of them — `cartograph`, `overlook`,
`image-trail`, `bookmarkit`, `localnotes`, `quorum`, `agentic-code-analysis`
and `codex-rules-editor` — now live in an organisation, while the shared CI
capabilities they consume lived in a personal account:
`qwts/qwts-agent-ci` and `qwts/qwts-agent-docs-gov`.

Two facts about that arrangement had already cost real work.

**A transferred repository's self-hosted runner does not travel with it.** The
`dml-arm64-qualification` job in `overlook` runs on
`[self-hosted, Windows, ARM64]`, and the runner that serves it, `SURFACE-13`,
stayed behind in the personal account. The job sat queued for 35 minutes with
every other job in the run green, because the runner was offline — and that
runner is not visible in the organisation's runner list at all.

**The policy home was in the wrong organisation.** The CI policy roster has to
list the `true-line` agent identities, and the release-lifecycle catalog has
to carry an entry for every `true-line/*` repository, in a repository that
belonged to somebody else's account. Both are now in place, but the
arrangement inverts the ownership: the organisation that owns the governed
repositories did not own the rules that govern them.

The fleet pin itself was never in question. `qwts-agent-ci` is the shared CI
home for the `qwts` fleet, it stays there, and the `qwts` fleet continues to
consume it by SHA.

## Decision

1. **`qwts-agent-ci` does not move.** It stays in the `qwts` personal account
   and continues to serve the `qwts` fleet. Moving it would relocate a shared
   capability home that works, and would change the pinned path for every
   repository in the `qwts` fleet for no gain.
2. **The `true-line` organisation holds copies, named `trueline-agent-*`.**
   `true-line/trueline-agent-ci` and `true-line/trueline-agent-docs-gov` are
   the organisation's own capability repositories.
3. **A copy is made when the two are likely to diverge**, not merely because a
   copy is possible. The copies start as exact mirrors — the same git history,
   the same pinned commits — and diverge deliberately, one reviewed change at
   a time, in whichever organisation needs the difference.
4. **The copies are not a second source of truth.** A capability repository
   has one home. While a copy is an exact mirror it may be re-synced from its
   source; once it has diverged, it is its own repository and the source no
   longer constrains it.
5. **Consumers move by reviewed pull request**, with the pinned commit
   unchanged, as ENG-0282 requires. A copy carries the same history, so the
   same 40-hex commit stays reachable from the copy's default branch; the
   reference remains immutable and the change is a reviewed one in the
   consumer.

## Consequences

- The `true-line` organisation can change its own CI policy — its roster, its
  release-lifecycle catalog, its action set — without a change to a personal
  account, and without a change to the `qwts` fleet.
- **The copies double the maintenance surface of the capability repositories.**
  Two repositories now carry the same composite actions, the same policy
  classifier and the same pin checks, and a security fix in the shared code
  has to be applied in both. That is the cost of two homes, and it is why
  rule 3 makes divergence — not duplication — the trigger for keeping a copy.
- **The pin-reachability check now has two targets.** A pinned commit must be
  reachable from the default branch of the repository that is actually
  consumed, which after this change is the copy, not the original.
- The `true-line` organisation's allowed-actions list must permit the
  organisation's own capability repositories. The bot cannot read or write the
  organisation-level setting, so the pattern is set per repository; the
  organisation-level list is the better home and remains an owner action.
- Until the copies diverge, they are exact mirrors and carry the entries for
  repositories they do not govern. Trimming the catalog and the roster to the
  `true-line` identities is a deliberate follow-up, and it is a *policy*
  change: it makes the copies' own content differ from the source.
- The `qwts/agent-sop` decision series, which held the org-wide records, is
  archived and read-only, so this decision is recorded here rather than as an
  `ENG-NNNN` record: that series' numbering is derived from issue numbers in
  a repository this organisation does not own, and reusing it here would
  collide with that space.
