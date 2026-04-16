# Skills

Alla skills ligger i `.opencode/skills/<name>/SKILL.md`.

## Kvalitetsskala

| Nivå | Beskrivning |
|------|-------------|
| ⬜ Saknas | Filen finns men saknar meningsfullt innehåll |
| 🟥 Grund | Har steg men saknar regler, output-format eller edge cases |
| 🟨 Godkänd | Täcker happy path, har regler, men saknar djup i felscenarier eller exempel |
| 🟩 Bra | Tydliga steg, regler, output-format och edge cases täckta |
| 🟦 Utmärkt | Allt i Bra + konkreta exempel, verifierade mot verkliga verktyg/konventioner |

---

## Översikt

| Skill | Kategori | Kvalitet | Anteckningar |
|-------|----------|----------|--------------|
| [commit](skills/commit/SKILL.md) | Git | 🟦 Utmärkt | Goda/dåliga exempel, scope-guide, breaking change-footer, HEREDOC-instruktion |
| [pr-create](skills/pr-create/SKILL.md) | Git | 🟦 Utmärkt | Fullständigt gh-kommando-exempel, draft-PR, reviewer-tilldelning, rebase-check |
| [changelog](skills/changelog/SKILL.md) | Git | 🟦 Utmärkt | Monorepo-hantering, inga-taggar-fallback, bra/dåliga entryexempel, Keep a Changelog |
| [review-pr](skills/review-pr/SKILL.md) | Review | 🟦 Utmärkt | Språkspecifika checklistor (TS/Python/Go/SQL), konkreta blocker/suggestion/nit-exempel |
| [security-review](skills/security-review/SKILL.md) | Review | 🟦 Utmärkt | Sårbara kodmönster med fix, falska positiver dokumenterade, CVE-scanning inkluderat |
| [test](skills/test/SKILL.md) | Kvalitet | 🟦 Utmärkt | Unit/integration/e2e-distinktion, bra/dåliga testexempel, namnkonventioner |
| [debug](skills/debug/SKILL.md) | Kvalitet | 🟦 Utmärkt | Debugger-tabell per språk, konkret hypotes/verify-cykel-exempel |
| [simplify](skills/simplify/SKILL.md) | Kvalitet | 🟦 Utmärkt | Before/after-kodexempel för prematur abstraktion, defensiv kod, duplicering |
| [scaffold](skills/scaffold/SKILL.md) | Kodgen | 🟦 Utmärkt | Framework-specifika wiring-exempel, monorepo-hantering, kompileringskontroll |
| [document](skills/document/SKILL.md) | Docs | 🟦 Utmärkt | Fullständiga exempel i TS/Python/Go/Rust, bra/dåliga kommentarsexempel |
| [init](skills/init/SKILL.md) | Docs | 🟦 Utmärkt | Hanterar befintlig AGENTS.md, fullständig mall med verkliga kommandon, TODO-markers |
| [explore](skills/explore/SKILL.md) | Docs | 🟦 Utmärkt | Monorepo-strategi, fullständigt output-exempel med fil:rad-refs, grep-kommandon |
| [deps-update](skills/deps-update/SKILL.md) | Underhåll | 🟦 Utmärkt | Peer deps, monorepo, konkret changelog-läsningsexempel, express 4→5 case |
| [migrate](skills/migrate/SKILL.md) | Databas | 🟦 Utmärkt | ORM-exempel (Prisma/Alembic/Knex), zero-downtime deploy-ordning, batched data migration |
| [profile](skills/profile/SKILL.md) | Prestanda | 🟦 Utmärkt | Flame graph-läsning, EXPLAIN ANALYZE-exempel med före/efter-mätning |
| [deploy](skills/deploy/SKILL.md) | DevOps | 🟦 Utmärkt | Rolling/blue-green/canary-jämförelse, rollback-kommandon per plattform (k8s/Fly/ECS/Lambda) |
| [web-search](skills/web-search/SKILL.md) | Research | 🟦 Utmärkt | Källkvalitetstabell, bra/dåliga query-exempel, site:-operator, versionskänslighet |
| [research](skills/research/SKILL.md) | Research | 🟦 Utmärkt | web-search vs research-distinktion, konflikthantering med konkret exempel, stop-kriterier |
| [angular](skills/angular/SKILL.md) | Frontend | 🟦 Utmärkt | Signals, signal inputs/outputs, inject(), ny control flow-syntax (@if/@for), lazy loading, TestBed |
| [dotnet](skills/dotnet/SKILL.md) | Backend | 🟦 Utmärkt | Records, primary constructors, cancellation tokens, IHttpClientFactory, Options-pattern, global exception handler |
