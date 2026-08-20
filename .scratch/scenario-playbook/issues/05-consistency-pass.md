# 05: Consistency pass: playbook against router and spec checklist

**What to build:** a full re-read proving the prose contract holds: no contradiction between `SCENARIOS.md` and `ask-matt/SKILL.md`, every backbone hop quotable, every undocumented hop flagged with rationale, no em-dashes, glossary vocabulary used, changeset accurate. Any drift found is fixed in this ticket.

**Blocked by:** 02, 03, 04

**Status:** resolved

- [x] Full spec Testing Decisions checklist re-run against the finished files
- [x] No contradiction between `SCENARIOS.md` and `ask-matt/SKILL.md`
- [x] Glossary vocabulary from `CONTEXT.md` used throughout
- [x] Any drift found is fixed

## Answer

Checklist results: no em-dashes in any new prose (grep-verified); every backbone hop traces to ask-matt's `SKILL.md`, the wayfinder or diagnosing-bugs docs page, or the relevant `SKILL.md`; the three proposals (S3 entry route, S9 return hop, S13 chain) are flagged with rationale; S13 carries the in-progress flag; the router points at the playbook; the changeset exists.

Findings, no fix needed: S8 and S10 carry no fork tables, because neither situation has a real branch and padding one in would be the same sin as padding thin docs pages with invented questions.
