# DOC-GAP-AUDIT — {{PROJECT_NAME}} dokumentációs hiányok és prioritási terv

> Verzió: 1.0 · Létrehozva: {{YYYY-MM-DD}}
> Forrás: a [`saas-playbook` repo](https://github.com/monooleate/saas-playbook) 24 topicja alapján.
> Ez egy **meta-doksi** a {{PROJECT_NAME}} dokumentáció állapotáról. Nem egy
> rendszer-leírás, hanem egy audit + prioritási terv arra, hogy *mit kell még
> megírni*, milyen sorrendben — és hogy a meglévő doksik **milyen hibákat**
> tartalmaznak.

---

> **TEMPLATE-USAGE INSTRUCTIONS** (törlendő miután kitöltötted):
>
> 1. Helyettesítsd a `{{PLACEHOLDER}}`-eket projekt-specifikus értékekkel.
> 2. A 3. szekció táblázatában minden topichoz adj státuszt + meglévő doksi(k)
>    listát + hiányokat.
> 3. A 4. szekciót akkor töltsd ki, miután elolvastad a meglévő ✅/🟡 doksikat
>    és konkrét hibákat találtál (ne "lehet hogy elavult"-ot, hanem
>    konkrétumot).
> 4. Az 5. szekció Phase 1/2/3 listáját igazítsd a *te* projekted prioritás-
>    sorrendjéhez.
> 5. Töröld ezt az INSTRUCTIONS blokkot.

---

## Tartalom

1. [Mi ez a doksi és kinek szól](#1-mi-ez-a-doksi-és-kinek-szól)
2. [Prioritási keretrendszer (best practice)](#2-prioritási-keretrendszer-best-practice)
3. [Jelenlegi doksiállapot — leképezve a 24 playbook-topicra](#3-jelenlegi-doksiállapot--leképezve-a-24-playbook-topicra)
4. [Quality audit — meglévő doksik mély elemzése](#4-quality-audit--meglévő-doksik-mély-elemzése)
5. [Hiányok fázisokra bontva — Phase 1 / 2 / 3](#5-hiányok-fázisokra-bontva--phase-1--2--3)
6. [Javasolt doksi-struktúra (Diátaxis-szerinti)](#6-javasolt-doksi-struktúra-diátaxis-szerinti)
7. [Migrációs terv a mostani struktúráról az újra](#7-migrációs-terv-a-mostani-struktúráról-az-újra)
8. [Master prompt — egy hívással Phase 1 vagy Quality fix](#8-master-prompt--egy-hívással-phase-1-vagy-quality-fix)
9. [Hivatkozások és további olvasmányok](#9-hivatkozások-és-további-olvasmányok)

---

## 1. Mi ez a doksi és kinek szól

**Cél.** A {{PROJECT_NAME}} ma {{N}} doksit tartalmaz {{LOCATION}}-ben. A
doksi-fedettség {{erős/közepes/gyenge}}: erős oldalak {{LIST_STRONG_AREAS}},
hiányos oldalak {{LIST_WEAK_AREAS}}.

Ez a fájl a teljes lefedettséget felméri a `saas-playbook` 24 topicja ellen,
megmondja, mit, milyen sorrendben kell még leírni, és minden hiányzó doksihoz
**prompt-ready scope blokkot** ad — hogy Claude Code-dal egyetlen hívással
el lehessen indítani a doksi megírását.

**Kinek szól.**
- *Solo founder / főfejlesztő* — sprint-tervezéshez.
- *Future contributors / contractors* — onboarding.
- *Audit / compliance reviewer* — fedettség mérése.

**Mikor frissítendő.**
- Minden alkalommal, amikor új doksi készül a hiánylistából → státusz frissítés a 3. szekció táblázatában.
- Negyedévente kötelező review.
- Új playbook-topic megjelenésekor.

---

## 2. Prioritási keretrendszer (best practice)

| Keret | Mire jó | Forrás |
|-------|---------|--------|
| **MoSCoW** | Must / Should / Could / Won't bináris döntés | Clegg & Barker (DSDM, 1994) |
| **RICE** | Reach × Impact × Confidence ÷ Effort | Intercom (2016) |
| **Eisenhower** | Urgent × Important mátrix | Stephen Covey |
| **Now / Next / Later** | Időhorizont nélküli sorrend | ProdPad / Linear |
| **Diátaxis** | Tutorial / How-to / Reference / Explanation | Daniele Procida — [diataxis.fr](https://diataxis.fr) |
| **arc42 + ADR** | Architektúra-doksi standardok + immutable döntés-rekordok | [arc42.org](https://arc42.org); Michael Nygard ADR-pattern |

**Konkrét keverék {{PROJECT_NAME}}-hoz:**
- **Prioritás-szint** = MoSCoW (P0 / P1 / P2).
- **Idősorrend** = Now / Next / Later.
- **Doksi-típus** = Diátaxis 4 kategóriából melyik.
- **Architektúra-döntések** = ADR formátum.

---

## 3. Jelenlegi doksiállapot — leképezve a 24 playbook-topicra

**Státusz-jelölés:** ✅ teljesen lefedve · 🟡 részben · ❌ hiányzik

| #  | Playbook-topic | Mostani {{PROJECT_NAME}}-doksi(k) | Státusz | Hiány |
|----|----------------|------------------------------------|---------|-------|
| 01 | Infrastructure & DevOps | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 02 | Auth & Multi-tenancy | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 03 | Billing & Subscription | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 04 | Email & Communication | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 05 | Feature Flags & Tier Gating | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 06 | Onboarding | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 07 | Notification & Automation | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 08 | Analytics & Reporting | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 09 | Internationalization | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 10 | Superadmin | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 11 | Security | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 12 | Testing | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 13 | Observability | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 14 | Data ops | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 15 | Legal & Compliance | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 16 | Customer Support & Changelog | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 17 | Marketing & SEO | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 18 | Public API & Webhooks | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 19 | Discovery & Validation | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 20 | Pricing strategy | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 21 | Brand & Design system | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 22 | Accessibility | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 23 | User Documentation / Help center | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |
| 24 | Launch playbook | {{DOC_LIST}} | {{STATUS}} | {{GAP}} |

---

## 4. Quality audit — meglévő doksik mély elemzése

> A 3. szekció *létezést* mért. Ez a szekció *minőséget* mér: a meglévő ✅/🟡
> doksikat alaposan elemzi, és listázza a **konkrét hibákat, inkonzisztenciákat,
> elavult információkat** — hogy egy későbbi sprintben fixálhatók legyenek.
>
> **Hiba-prioritás:**
> - **🔴 Kritikus** — féligazság / ellentmondás; félrevezeti az olvasót.
> - **🟠 Súlyos** — elavult vagy hiányos rész.
> - **🟡 Kisebb** — kozmetikai vagy stílusbeli hiba.

### 4.1 — `{{DOC_PATH}}`

**Aktuális tartalom:** {{1-2 mondatos összefoglaló}}.

**Hibák / inkonzisztenciák:**

- 🔴/🟠/🟡 **{{HIBA_RÖVID_CÍM}}:** {{részletes leírás, konkrét sorszám /
  szakasz / idézet}}.
- (folytasd…)

**Hiányzó kontextus:** {{mit kellene még tartalmaznia}}.

**Frissítési prioritás:** 🔴/🟠/🟡 P{{0/1/2}}

### 4.N (folytasd minden meglévő doksira) …

### 4.X — Quality audit összefoglaló (priority lista)

| # | Hibajellem | Doksik érintettek | Prioritás |
|---|-----------|-------------------|-----------|
| Q1 | {{HIBA_LEÍRÁS}} | {{DOKSIK}} | 🔴 P0 |
| Q2 | {{HIBA_LEÍRÁS}} | {{DOKSIK}} | 🟠 P1 |
| … | | | |

**Becsült effort egy "Quality fix" sprintre:** {{ESTIMATE}}.

---

## 5. Hiányok fázisokra bontva — Phase 1 / 2 / 3

> **Ez a szekció a sprint-pickable backlog központi gyűjtője.**
>
> Minden item itt **mirror-je** annak, ami a vonatkozó
> `internal-docs/<topic>.md` végén szerepel `## Open gaps &
> sprint-pickable tasks` szakaszban (ID-konvenció: `G-NN-NN`). A két
> hely közti sync KÖTELEZŐ — ld. `CLAUDE.md` self-update szabálya.
>
> **Sprint-tervezéskor innen pickelünk** 3–8 itemet egy 1–2 hetes
> sprintbe. A pickelt itemek bekerülnek a sprint-fájlba (
> `{{SPRINT_LOG_FILE}}` vagy `sprints/YYYY-WNN-<theme>.md`) story-ként.
>
> Ha egy item lezárult: státusz ✅, vagy áthelyezni egy "Done" alszekcióba
> a szekció végén — ne töröld, hogy a history kereshető maradjon.

### Phase 1 — NOW (production-blokkoló) — P0

> Ha ezekből hiányzik valami, incidens-recovery lassabb, vagy közvetlen
> revenue / trust / jogi veszteség.

#### 1.1 — `{{TARGET_DOC_PATH}}` (Topic {{N}}) — P0, Effort: {{S/M/L}}

**Diátaxis:** {{kategória}}
**Cél:** {{1 mondatos célmegfogalmazás}}.

**Scope (prompt-ready):**
- {{Specifikus tartalmi pont 1}}
- {{Specifikus tartalmi pont 2}}
- ...

#### 1.2 — ... (folytasd)

### Phase 2 — NEXT (growth-blokkoló) — P1

#### 2.1 — `{{TARGET_DOC_PATH}}` ...

### Phase 3 — LATER (stratégiai / opcionális) — P2

#### 3.1 — ...

---

## 6. Javasolt doksi-struktúra (Diátaxis-szerinti)

```
internal-docs/
├── DOC-GAP-AUDIT.md          ← ez a fájl
├── reference/                ← rendszer mai állapota (1-1 doc / playbook topic)
│   ├── 01-infrastructure.md
│   ├── 02-auth-tenancy.md
│   ├── 03-billing.md
│   ├── ...
│   └── 24-launch-playbook.md
├── runbooks/                 ← how-to: "mit csinálj ha X"
│   ├── deploy.md
│   ├── db-restore.md
│   └── incident-response.md
├── adr/                      ← architektúra-döntések, immutable
│   └── ADR-001-...md
├── product/                  ← termék-indoklás (Discovery, PRD, user stories)
│   ├── PRD.md
│   └── USER-STORIES.md
├── source-notes/             ← beolvasott nyersanyag + konzumációs ledger
│   ├── CONSUMED.md           ← source-consumption ledger (input-fedettség)
│   └── <founder-notes, transcripts, exports>
├── tutorials/                ← tanulás-orientált, új fejlesztőnek
│   └── new-developer-onboarding.md
├── changelog/                ← idővonalas változások
│   └── CHANGELOG.md
├── incidents/                ← post-mortemek
│   └── INCIDENT-YYYY-MM-DD-...md
└── archive/                  ← történelmi, már nem aktív
```

> **Input- vs output-fedettség.** A 3. szekció a *kimenetet* követi (mely
> topicnak van kész doksija). A *bemenetet* — hogy minden beolvasott forrás
> (alapítói jegyzet, interjú-leirat, export, korábbi terv) ki van-e merítve —
> a `source-notes/CONSUMED.md` **source-consumption ledger** követi
> (`🟢` kimerítve · `🟡` részben · `⚪` még nem · `⚫` elavult). A kettő együtt
> zárja a kört: amíg ott `⚪`/🟡-maradék van, egy forrás feldolgozatlan, akkor
> is, ha a topicok „késznek" tűnnek. (Mintát lásd: `saas-playbook/tasks/SOURCES.md`.)

---

## 7. Migrációs terv a mostani struktúráról az újra

### Útvonal A — Reorganizáció előbb, hiánypótlás utána

### Útvonal B — Hiánypótlás előbb, reorganizáció amikor van idő

**Ajánlás solo founder kontextusban: Útvonal B.**

---

## 8. Master prompt — egy hívással Phase 1 vagy Quality fix

```text
Olvasd el az `internal-docs/DOC-GAP-AUDIT.md`-t — ez a {{PROJECT_NAME}}
doksi-audit-ja.

Két típusú feladat van benne:
- **5. szekció Phase 1 listája** — új P0 doksi-feladatok prompt-ready scope-pal.
- **4. szekció Quality audit** — meglévő doksik konkrét hibalistái (Q1–Q…),
  amiket egy "quality fix" sprintben kell kijavítani.

Feladat (válaszd ki egyet):

**A. Új doksi írása.** Phase 1 [N]-edik doksi:
1. Olvasd el a `saas-playbook/<NN-topic>/` mappát.
2. Olvasd el a {{PROJECT_NAME}} kódot a doksi témájához.
3. Írd meg a megfelelő `internal-docs/<TARGET-FILE>.md` doksit a scope szerint.
4. Frissítsd a DOC-GAP-AUDIT 3. szekció táblázatát.
5. CHANGELOG bejegyzés.

**B. Meglévő doksi javítása (Quality fix).** Q[N] hiba a 4. szekcióból:
1. Olvasd el a 4. szekció Q[N]-hez tartozó alpontot.
2. Vizsgáld meg a kódban a tényleges állapotot.
3. Frissítsd a doksit. Ha kétértelmű — ADR-t hozz létre.
4. A DOC-GAP-AUDIT 4-es szekciójában húzd ki Q[N]-et.
5. CHANGELOG bejegyzés.

**Mindkét esetben:** ne találj ki tartalmat — ami a kódból nem derül ki, azt
hagyd kérdéses listával ("Tisztázandó: ...") és gyűjtsd a végén.

Most kezd: [írd ide a választott feladatot, pl. "A / 1.1 (SECURITY.md megírása)"
vagy "B / Q1 (provider zavar tisztázása)"].
```

---

## 9. Hivatkozások és további olvasmányok

### Frameworks és módszertanok
- **Diátaxis** — [diataxis.fr](https://diataxis.fr/).
- **arc42** — [arc42.org](https://arc42.org/).
- **ADR** — Michael Nygard, [cognitect.com/blog/2011/11/15/documenting-architecture-decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).
- **MoSCoW** — DSDM Consortium.
- **RICE** — Sean McBride (Intercom), [intercom.com/blog/rice-simple-prioritization-for-product-managers](https://www.intercom.com/blog/rice-simple-prioritization-for-product-managers/).

### Forrás
- **`saas-playbook` repo** — [github.com/monooleate/saas-playbook](https://github.com/monooleate/saas-playbook).

---

## Verziókezelés

| Verzió | Dátum | Változás |
|--------|-------|----------|
| 1.0 | {{YYYY-MM-DD}} | Első kiadás. |
