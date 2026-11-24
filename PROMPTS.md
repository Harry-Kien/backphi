# BACKPHI - 3 prompt để dán vào Claude Code

Dán từng prompt vào khung chat của Claude Code (chạy `claude` trong thư mục backphi).
Chỉ dán phần nằm giữa hai dòng gạch ngang.

## PROMPT 1 - Kiểm tra repo, nghiên cứu đối thủ, lập kế hoạch (dán 1 lần)

---

Read docs/SPEC.md completely. You are the lead engineer for BACKPHI.

Do Phase 1 only. Do not write application code yet.

1. Audit this repository: stack, dependencies, structure, existing features, risks.
   If it is empty, say so and propose the default stack from the spec.
2. Do the competitor research in section 13 of the spec with the REA skill and write
   docs/COMPETITOR.md. Stay inside the limits of that section. Take notes
   on features only. Do not copy text, design or assets.
3. Write these files:
   - CLAUDE.md: short project rules that every agent must follow (money rules,
     honesty and compliance rules, commands for lint / typecheck / tests, the file
     ownership rule, the definition of done). Keep it under 150 lines and point to
     docs/SPEC.md for detail.
   - docs/PLAN.md: the 12 phases broken into tasks sized for one agent each, with
     dependencies, the files or directories each task owns, and acceptance criteria.
   - docs/DECISIONS.md, docs/PROGRESS.md, docs/OWNER_ACTIONS.md.
   - .claude/agents/: one definition file for each role: architect (read-only
     planning and design review), backend (API, database, jobs, broker adapters),
     frontend (UI, design system, i18n), ledger (rebate engine, ledger, withdrawals,
     reconciliation), qa (tests, end-to-end, bug reproduction), security-reviewer
     (read-only review of auth, money paths, uploads, secrets). Each definition says
     what the role owns, what it must not touch, and its definition of done.
4. Give me one list of the questions you cannot answer from the repository or the
   spec.

Stop after this and wait for my approval of docs/PLAN.md.

---

## PROMPT 2 - Chạy một phase bằng đội agent (dán lại cho mỗi phase, N = 2 đến 12)

Thay <N> bằng số phase trước khi dán.

---

Read CLAUDE.md, docs/SPEC.md, docs/PLAN.md and docs/PROGRESS.md.
Run Phase <N> from docs/PLAN.md as an agent team.

- Create an agent team. Spawn only the teammates this phase needs (3 to 5), using
  the agent types in .claude/agents/. Name each teammate by its role.
- Give every teammate a spawn prompt with: the phase goal, its tasks, the
  directories it owns, the interfaces it must respect, and the definition of done.
  Teammates do not see this conversation.
- No two teammates edit the same file. Shared contracts (database schema, API
  types) have one owner, who announces every change to the others.
- Write tests first on money paths.
- When the implementation tasks are done, have the qa teammate run the full test
  suite and the security-reviewer teammate review the diff. Fix what they find.
- Wait for your teammates to complete their tasks before proceeding. Do not
  implement their tasks yourself.
- Then: update docs/PROGRESS.md, write docs/reports/phase-<N>.md with the test
  output as evidence, add anything that needs the owner to docs/OWNER_ACTIONS.md,
  commit, shut down the teammates, and stop. Do not start the next phase.

---

## PROMPT 3 - Chạy tiếp khi phiên bị ngắt

Thay <N> bằng số phase đang làm dở.

---

Read CLAUDE.md, docs/PLAN.md and docs/PROGRESS.md. A previous session stopped in the
middle of Phase <N>. Check git status and run the tests to see the real state. Tell
me what is done and what is left. Then continue Phase <N> with a new agent team
under the same rules as before. Earlier teammates no longer exist; spawn new ones.

---
