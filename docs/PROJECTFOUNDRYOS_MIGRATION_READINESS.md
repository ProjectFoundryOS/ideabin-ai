# IdeaBin.ai → ProjectFoundryOS Migration Readiness

Mission: `PFORG-MIG-IDEABIN-001` (Agent Mission Control, worker `claude-seat-1`).

Scope: audit `TheRealSynth/ideabin-ai` for repository-transfer hazards ahead of
an owner-executed GitHub repository transfer to `ProjectFoundryOS/ideabin-ai`,
fix reversible repository-side hazards, and leave the transfer itself and
other owner-gated actions for explicit owner execution. No transfer,
production deploy, DNS change, billing change, or credential rotation was
performed by this mission.

Audited base: `origin/main` @ `d166788c33c90ad7e78332c38de402c7104c1100`.

## Verdict

**READY_AFTER_OWNER_ACTION**

All reversible, repository-content hazards found in this audit have been
fixed on this mission's branch/PR. Nothing found requires new engineering
work before a transfer can happen. What remains is exclusively outside a git
repository's content: GitHub App/service reauthorization, Vercel project
reconnection, and updating clone/remote URLs on machines other than this one
— all owner-gated or owner-executed by nature. See "Remaining owner actions"
below. This verdict covers `ideabin-ai` only; the portfolio-wide cutover
(all five repos, ordering, control-plane pointer updates) is tracked in
`agent-mission-control`'s own `PFORG-MIG-AMC-005` mission and its resulting
`docs/PROJECTFOUNDRYOS_MIGRATION_READINESS.md` in that repository.

## Hazard inventory and fixes

| # | Hazard | File | Risk if unfixed | Fix applied |
|---|---|---|---|---|
| 1 | Agent launcher prompt hardcodes `Repository: TheRealSynth/ideabin-ai` as a literal fact fed to autonomous coding agents | `docs/AGENT_LAUNCHER_PROMPT.md` | An agent launched after transfer would open PRs/branches against the wrong owner, or assert a false repository identity in its output/completion report | Replaced the hardcoded string with an instruction to resolve the current owner/repo at runtime (`git remote get-url origin` / invoking GitHub context) instead of hardcoding an org, plus a pointer to this document |
| 2 | Vercel deployment contract hardcodes `TheRealSynth/ideabin-ai` in both the "current state" note and the one-time setup instruction | `docs/DEPLOYMENT_VERCEL.md` | A future reader would import/re-import the wrong (stale) repository into Vercel, or not realize the Vercel Git integration needs reconnecting after an org-to-org transfer | Reworded the current-state note to be point-in-time ("at the time of that check"), reworded the setup instruction to reference "the canonical IdeaBin GitHub repository" via this document rather than a fixed string, and added an explicit note that a cross-org GitHub transfer does not automatically carry the Vercel Git integration — it must be reconnected and the Vercel GitHub App re-confirmed against the new owner |
| 3 | Agent product roadmap hardcodes `TheRealSynth/agent-mission-control` as the control-plane repository agents must defer to | `docs/AGENT_QUEUE.md` | Agents reading this file after `agent-mission-control` itself transfers (tracked separately, see below) would look for control state in the wrong repository | Reworded to "the Agent Mission Control control-plane repository (currently `TheRealSynth/agent-mission-control`...)" and pointed to this document and to that repository's own migration tracking, rather than asserting a permanent literal path |
| 4 | Promotion contract's canonical JSON example hardcodes `"existing_repo": "TheRealSynth/agent-mission-control"` | `docs/PROMOTION_CONTRACT_V1.md` | A reader could copy the example literally, or infer the schema itself pins that org, when `existing_repo` is a runtime-supplied field | Replaced the example value with an explicit `<owner>/agent-mission-control` placeholder and added a one-line note explaining why, so the schema/contract itself is never confused with a fixed default |
| 5 | GitHub Actions CI workflow (`.github/workflows/ci.yml`) was already removed on `main` (commit `d166788`, "Disable GitHub-hosted Actions on dormant repo") while the project is `PAUSED` in the portfolio | `.github/workflows/` (absent) | Not a transfer hazard by itself (no org-specific content) and not something this mission's objective authorizes reversing — it was a deliberate, unrelated decision tied to the project's paused status, not to the pending migration | No change. Recorded here only as context: CI will not automatically re-run on `main`/PRs after transfer either, exactly as it does not today. This mission's own acceptance commands (`pnpm test`, `pnpm typecheck`, `pnpm build`) were run directly instead of relying on CI (see "Validation performed") |
| 6 | Historical receipt references `https://github.com/TheRealSynth/ideabin-ai/pull/12` | `.agent-results/IDEA-REV-005.json` | None — this is a `forbidden_paths` file for this mission and a truthful historical record | Left untouched, by mission contract. GitHub preserves old URLs as redirects after a repository transfer/rename, so the link remains resolvable after a future transfer; it is correctly a historical fact ("this PR was here when this record was written"), not live configuration |

No other `TheRealSynth`, `github.com/TheRealSynth`, `vercel.com`, or
`supabase.co` literals were found anywhere else in the repository (full
recursive search across all tracked files at the audited base SHA).
`package.json` carries no `repository` field and no org-specific data.
`.env.example` carries no org-specific data. No hardcoded Supabase project
ref, project URL, or API host was found anywhere in `apps/**`, `packages/**`,
or `supabase/**`.

## Cross-repo dependencies (not fixed here — out of this mission's ownership boundary)

These are real hazards for the *portfolio*, but they live in a different
repository than the one this mission (`owns_paths`) is authorized to touch,
or they require the GitHub-side (not git-content) actions listed below:

- `TheRealSynth/agent-mission-control`'s `control/PROJECTS.yaml` pins
  `repo: TheRealSynth/ideabin-ai` for the `ideabin` project entry, and
  `control/WORKERS.yaml`/routine wiring assumes reachability of specific
  repos. Making that control-plane repository itself portable — including a
  canonical repository registry/alias mechanism for the staged, mixed-owner
  window while the five-repo cohort transfers on different schedules — is
  the explicit objective of the sibling mission `PFORG-MIG-AMC-005`
  (`agent-mission-control` repo, state `READY`, unassigned at time of
  writing). This mission intentionally does not duplicate or race that work.
- The Agent Mission Control dispatcher's GitHub access (the GitHub App/token
  used to fetch this repository, open PRs, and post receipts) is scoped to a
  specific set of repositories under `TheRealSynth`. That access scope must
  be extended to `ProjectFoundryOS/ideabin-ai` (via org owner action) before
  or as part of the transfer, or the dispatcher will lose the ability to
  read/write this repository immediately after the org changes. This is a
  platform/owner action, not a file in this repository.

## Remaining owner actions (not repository-content, cannot be fixed by editing files)

1. Perform the actual GitHub repository transfer `TheRealSynth/ideabin-ai` →
   `ProjectFoundryOS/ideabin-ai` (owner-gated; out of scope for this mission).
2. Re-authorize/re-install any GitHub Apps this repository depends on
   (Mission Control's dispatcher app/token, Vercel's GitHub App once a
   project exists, any other installed app) against the `ProjectFoundryOS`
   org, since GitHub App installation scope does not automatically follow a
   cross-org transfer.
3. If/when a Vercel project is created for this repository (see
   `docs/DEPLOYMENT_VERCEL.md` — none exists yet as of this audit), reconnect
   its Git integration after any subsequent owner change; it will not follow
   an org-to-org transfer automatically.
4. Update `origin` remotes on any existing local or VPS clones
   (`git remote set-url origin https://github.com/ProjectFoundryOS/ideabin-ai.git`)
   after the transfer. GitHub's automatic redirect for the old URL is not a
   substitute for updating a clone's remote long-term.
5. Land and apply `PFORG-MIG-AMC-005` in `agent-mission-control` (or an
   equivalent control-plane pointer update) so the dispatcher resolves this
   repository's new owner correctly; coordinate the order of that mission
   relative to this repository's actual transfer so there is never a window
   where the dispatcher points at a repository it can no longer reach.
6. Re-verify secrets/environment variables required by
   `docs/DEPLOYMENT_VERCEL.md` are (re)configured on whatever hosting project
   is connected post-transfer; none are stored in this repository today.

## Post-transfer validation checklist

After the owner completes the transfer, verify before considering the
migration for this repository closed:

- [ ] `git clone https://github.com/ProjectFoundryOS/ideabin-ai.git` succeeds
      from a machine with no prior local state.
- [ ] The old URL `https://github.com/TheRealSynth/ideabin-ai` redirects to
      the new location (confirms GitHub's transfer redirect is active).
- [ ] `pnpm install --frozen-lockfile=false && pnpm typecheck && pnpm test && pnpm build`
      pass on a fresh clone of the transferred repository, unchanged from the
      results recorded in "Validation performed" below.
- [ ] Mission Control's dispatcher can read this repository, open a branch,
      and open a PR against it post-transfer (proves GitHub App/token scope
      was correctly extended).
- [ ] `docs/AGENT_LAUNCHER_PROMPT.md`'s runtime-discovery instruction
      actually resolves to `ProjectFoundryOS/ideabin-ai` when exercised by a
      real agent session.
- [ ] If a Vercel project exists by then, it deploys `main` successfully
      after reconnecting its Git integration.
- [ ] Historical PR links (e.g. `.agent-results/IDEA-REV-005.json`'s
      `pull/12` URL) still resolve via GitHub's redirect.

## Rollback

Repository-content changes in this mission are a small, additive doc edit
set (see diff) with no schema, code, or CI behavior change — reverting the
PR (or the specific commits) fully rolls back this mission's changes with no
data-loss or migration risk. If the owner performs the actual GitHub
transfer and needs to roll back, GitHub itself supports transferring a
repository back to its original owner; this document's "Remaining owner
actions" and "Post-transfer validation checklist" apply symmetrically in
reverse (re-point clones/Vercel/GitHub App access back to `TheRealSynth`).
No repository-side change made here is a one-way door.

## Validation performed

Ran the mission's required acceptance commands directly against this
branch (no CI workflow currently exists on `main` — see hazard #5 above):

- `pnpm test`
- `pnpm typecheck`
- `pnpm build`

Results are recorded in `.agent-results/PFORG-MIG-IDEABIN-001.json` on this
branch.

## Challenge Pass summary

See `.agent-results/PFORG-MIG-IDEABIN-001.json`'s `challenge_pass` object for
the full record (real objective, hidden assumptions, obvious/optimized/
non-obvious paths, route comparison, validation result, and evidence refs).
In short: the obvious route (global string-replace `TheRealSynth` →
`ProjectFoundryOS`) was rejected because the transfer has not happened yet —
writing the new org as present fact would be false today, and performing the
transfer itself is explicitly out of this mission's authority. The chosen
route instead removes the hardcoded org from live/launcher/deployment
prose in favor of runtime-resolved identity plus a single pointer to this
document, consistent with the canonical-registry approach the sibling
`PFORG-MIG-AMC-005` mission is building for the control-plane repository —
so no second edit pass across these files is needed at actual cutover time.
