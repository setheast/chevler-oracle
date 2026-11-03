# Skills

Five skills copied from github.com/mattpocock/skills at commit 4ae2317 (31 Oct 2026), MIT licence (`LICENSE-mattpocock-skills`). Copied rather than installed as the plugin, so they can't change under us and the other 22 skills (a competing `code-review`, `pr`, issue-tracker triage) stay out. They don't update automatically; re-copy deliberately if wanted. Seth chose them on 3 Nov 2026.

| Skill | Invoked by | Use in Oracle |
|---|---|---|
| `grill-me` | Seth (`/grill-me`) | Runs `grilling`. |
| `grilling` | Seth or Claude | Decision sessions: rounds of numbered questions, each with a recommended answer. First use: the spec's §7 decisions + hosting. |
| `domain-modeling` | Claude | `GLOSSARY.md` at the root and ADRs in `docs/adr/`. **Local edit:** ADRs are for architecture only; production rulings stay in the spec's register. |
| `codebase-design` | Claude | Design vocabulary (module, interface, depth, seam, adapter). "Design it twice" starts 3+ subagents, so only for big interface decisions. |
| `tdd` | Claude | Red → green in vertical slices. **Local edits:** the engine's seams are the spec §3 pipeline stages (don't ask Seth to pick them); expected values from rulings / worked examples, with sandbox fixtures as divergence references; refactoring review = the built-in `/code-review`. |

Not taken: `to-questionnaire` (Stuart works by phone; question lists go by email) and the rest of the repo.
