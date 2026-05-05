# CLAUDE.md — saas-playbook (meta-projekt önmagáról)

> Ezt Claude Code minden session elején olvassa el. Soha ne töröld.
> Utolsó frissítés: **2026-05-04** — (1) első kiadás: mátrix-filozófia
> + gap-workflow + self-update szabály rögzítve. (2) `ai/claude/`
> template-kit frissítve ugyanezekkel a mintákkal derived SaaS
> projektekhez (CLAUDE.template.md + DOC-GAP-AUDIT.template.md +
> END-OF-SESSION-CHECKLIST.md + README.md).

---

## TL;DR — Mit kell tudni első session-ben

Ez **NEM egy SaaS app**, hanem egy stack-független **public playbook**
solo founderoknek. 24 topic, mindegyik egy mappa (`NN-topic/`) és egy
sor a `tasks/MATRIX.md`-ben.

A három legfontosabb minta:

1. **Mátrix-filozófia** — minden topic = egy sor; egy sor kibontása
   három output-ot ad: tartalom + gap analysis + sprint-pickable taskok.
2. **Két helyen tartott gap-lista** — gap-ek a topic doc végén ÉS a
   központi `tasks/GAPS.md`-ben. A duplikáció szándékos.
3. **Doksi-szinkron self-update** — bármilyen változás ami egy topic
   doksit érint, a CLAUDE.md-t is frissíti, ugyanabban a sessionben.

A repo **public English content**, de a `CLAUDE.md` és a Claude-facing
internal doksik (pl. `ai/claude/`) **magyarul** vannak — ezt tartsd.

---

## Projekt áttekintés

- **Cél:** stack-független SaaS knowledge base solo foundereknek.
- **Felépítés:** 24 topic-mappa (`01-infrastructure/` … `24-launch-playbook/`),
  mindegyik egy `README.md` + `checklist.md` + `best-practices.md` +
  `gotchas.md` + `examples/` szerkezettel.
- **Lifecycle:** Stage 0–4, lásd [`ROADMAP.md`](./ROADMAP.md).
- **Sprint planning forrás:** [`tasks/MATRIX.md`](./tasks/MATRIX.md)
  (topic-szintű) + [`tasks/GAPS.md`](./tasks/GAPS.md) (item-szintű
  backlog).
- **Referencia-implementációk:** Grabit (`bookolj/` monorepo) — lásd
  user-memory `reference_grabit.md`. A topic 01 + 03 már Grabit-mintákból
  kibontott; ők a depth-bar.
- **Derived SaaS projektekhez** külön sablonok vannak `ai/claude/`-ban
  (CLAUDE.template.md, DOC-GAP-AUDIT.template.md, END-OF-SESSION-
  CHECKLIST.md, README.md). Azokat **ne keverd** ezzel a fájllal — ez
  a **playbook saját** alkotmánya, azok a leszármazott projekteké.
  Ugyanazt a mátrix-filozófiát + két helyes gap-workflow-t kódolják,
  csak más fájl-leképezéssel: a derived projektekben a központi gap-
  aggregator a `DOC-GAP-AUDIT.md` 5. szekciója (nem külön `GAPS.md`),
  és a topic-fájlok `internal-docs/<topic>.md` formában léteznek (nem
  `NN-topic/README.md` mappastruktúrában).

---

## A mátrix-filozófia (ez a központi minta)

Minden topic = egy sor a `tasks/MATRIX.md`-ben. Egy topic kibontása
**nem csak ismertető tartalmat** termel, hanem **három párhuzamos
outputot**:

1. **Tartalom** — a topic mappa fájljai (`README.md`, `checklist.md`,
   `best-practices.md`, `gotchas.md`, `examples/*`). Diátaxis-szerű
   felosztás: koncepció / how-to / reference / gotchák.
2. **Gap analysis** — mit NEM fed le a topic, mi maradt homályos, mi
   hiányzik a referencia-implementációkhoz (Grabit) képest, milyen
   konvenciók nincsenek dokumentálva.
3. **Sprint-pickable taskok** — a gap-ekből származó konkrét, becsülhető
   (S/M/L), prioritizált (P0/P1/P2) feladatok. Ezek azok, amik egy
   sprint-fájlba (`tasks/sprints/YYYY-WNN-*.md`) bemásolhatók.

A mátrix **nem csak listázás** — egy aktív, élő backlog-generátor.

---

## Hol élnek a gap-ek és a taskok

**Két helyen, duplikáció szándékos:**

### 1) A topic doc végén — kontextusban

Minden topic `README.md` (ahol már van mit gyűjteni) végén legyen egy
szakasz:

```markdown
## Open gaps & sprint-pickable tasks

- **G-NN-01** — <rövid leírás>. Effort: S/M/L. Priority: P0/P1/P2.
- **G-NN-02** — <rövid leírás>. Effort: M. Priority: P1.
```

Az ID konvenció: **`G-<topic-number>-<sequence>`** (pl. `G-04-01` az
első email-gap). Így az ID a forrásra mutat vissza.

**Miért a topic doc végén:** aki a topicot olvassa egy görgetésre lássa
"mi még nincs lefedve" — döntéshez kontextus kell.

### 2) Központilag `tasks/GAPS.md`-ben — sprint-pickup-hoz

**Ugyanazok az itemek** kerüljenek be a `tasks/GAPS.md` központi
táblázatba (egy sor per item, source-topic-linkkel, sprint-oszloppal).

**Miért központilag:** sprint-tervezéskor egy fájlt szkennelsz, nem 24-et.
A `tasks/sprints/YYYY-WNN-*.md` innen szed sorokat.

### Mit gyűjt mi

| Fájl | Mit gyűjt | Granularitás |
|------|-----------|--------------|
| `tasks/MATRIX.md` | Topic-szintű "Remaining work" összefoglaló | Egy mondat / topic |
| `tasks/GAPS.md` | Item-szintű backlog (összes gap egyben) | Egy sor / konkrét task |
| `NN-topic/README.md` (vége) | A topic saját gap-jei | Item-lista, kontextusban |
| `tasks/sprints/*.md` | Aktív sprint-tervek | Story-szintű, GAPS.md-ből pickelt |

A négy szint redundáns — szándékosan. Sprint-tervező a MATRIX-ot szűri
topicra, a GAPS.md-ből pickel itemeket, a sprint-fájlba bemásolja, a
topic doc-ban látja a kontextust.

---

## Workflow — egy topic kibontásakor

1. **Olvasd** a topic `README.md`-t (és ha van, a többi fájlt is).
2. **Mine** a referencia-implementációt (Grabit `bookolj/`, lásd
   user-memory `reference_grabit.md`). Topic 01 + 03 a depth-bar.
3. **Frissítsd** a topic doksijait — content first.
4. **Gyűjts gap-eket** munka közben — ne a végén próbáld emlékezetből.
   Ha valami homályos, hiányzó, vagy "ezt később ki kéne fejteni" típusú,
   azt **gap-jelölt**.
5. **Add be a gap-eket két helyre** (sorrendben: topic doc → GAPS.md):
   a. **Topic doc vége:** `## Open gaps & sprint-pickable tasks` szakasz,
      item-listában. ID konvenció: `G-NN-NN`.
   b. **`tasks/GAPS.md`** központi táblázat: ugyanazok az itemek, +
      source-link a topic-doc szakaszra.
6. **Topic state-váltás esetén** (📋 → 🟡 → ✅):
   a. `tasks/MATRIX.md` Status oszlop + Remaining work cella.
   b. Gyökér `README.md` mátrixa (ha van).
   c. `CHANGELOG.md` ha jelentős.
7. **CLAUDE.md self-update** — lásd köv. szakasz.

---

## CLAUDE.md self-update szabály (KÖTELEZŐ)

> **Szabály:** Bármilyen változás ami **érint egy topic doksit vagy a
> tasks/ struktúrát**, a CLAUDE.md is frissül **ugyanabban a sessionben**.
> Nem hagyjuk a következő munkamenetre.

### Minden session vége előtt fusd át

```
[ ] Új gap-szakasz került egy topic doc végére?
    → tasks/GAPS.md is frissítve? (Ha csak egy helyen van, hiányos.)

[ ] Topic state változott (📋/🟡/✅)?
    → tasks/MATRIX.md Status + Remaining work + gyökér README.md mátrix
      frissítve?

[ ] Új konvenció / fájltípus / mappastruktúra a topicokban?
    → CLAUDE.md "Workflow" / "Hol élnek a gap-ek" szakasz frissítve?

[ ] Új mező / oszlop a MATRIX.md-ben vagy GAPS.md-ben?
    → CLAUDE.md "Mit gyűjt mi" táblázat frissítve?

[ ] Sprint indult / zárult?
    → tasks/MATRIX.md "Active sprints" tábla + lent "Aktuális állapot"
      frissítve?

[ ] Új ai/claude/ sablon vagy template-változás?
    → ai/claude/README.md + CLAUDE.md "Projekt áttekintés" frissítve?

[ ] Bármi tényleges változás történt?
    → fej-dátum ("Utolsó frissítés: ...") ELŐRE léptetve, rövid változás-
      leírással. Ne mozdítsd trivialis edit miatt.
```

### Mit NE frissíts

- **`README.md`** (public-facing) ne változzon trivialis kód-vagy-doksi-
  edit miatt. Csak nagyobb struktúra-változáskor.
- **CHANGELOG.md** csak a jelentős, kifelé érdekes változásokra.

### Ha a self-update elmarad

A következő Claude session **vakon dolgozik** — lát egy új gap-szakaszt
egy topic doc végén, de nem tudja hogy az van-e a központi GAPS.md-ben,
nem tudja melyik sprint-be pickelhető, nem tudja hogy a MATRIX-ban
frissíteni kellett-e a Remaining work cellát. Ez fragmentációhoz és
duplikált munkához vezet.

---

## Working style

- **Magyar nyelv** Claude-facing internal doksikhoz (CLAUDE.md, ai/claude/*).
- **Angol nyelv** public playbook tartalomhoz (topic README-k, MATRIX.md,
  GAPS.md, ROADMAP.md, gyökér README.md).
- **Shell** Windows-on PowerShell, de Bash is elérhető — a tool-kontextus
  dönt.
- **Smoke teszt** itt nincs build-step (csak markdown), de ha új linket
  adsz hozzá, ellenőrizd hogy létezik a target-fájl.
- **Ne találj ki tartalmat** — ami egy topichoz nem derül ki referenciából
  vagy felhasználói validálásból, az gap, nem kitöltendő mező.

---

## Aktuális állapot (2026-05)

**Befejezett milestone-ok:**
- 24 topic scaffolded (`01-…/` – `24-…/`).
- Topic 01 (Infrastructure) + 03 (Billing) fully written, Grabit-derived.
- `tasks/MATRIX.md` 24 sorral, sprint-planning matrix-szal.
- `ai/claude/` template-kit derived SaaS projektekhez — 2026-05-04
  frissítve a mátrix-filozófia + két helyes gap-workflow-val
  (CLAUDE.template.md + DOC-GAP-AUDIT.template.md 5. szekció +
  END-OF-SESSION-CHECKLIST.md + README.md egységesen).
- `CLAUDE.md` (ez a fájl) létrehozva 2026-05-04 — workflow rögzítve.
- `tasks/GAPS.md` skeleton létrehozva 2026-05-04 — első topic-deep-dive
  fogja tölteni.

**Folyamatban:**
- Sprint A — Discovery & Validation
  ([`tasks/sprints/2026-W19-discovery.md`](./tasks/sprints/2026-W19-discovery.md)),
  2026-05-04 → 2026-05-24.

**Tervezett:**
- Topic 04 (Email), 14 (Data ops), 02 (Auth) — full content, Grabit-mining.
- Per-topic gap-szakaszok feltöltése a `tasks/GAPS.md` workflow-val.

---

## Hivatkozások

- [`README.md`](./README.md) — public-facing áttekintés a playbook-ról.
- [`ROADMAP.md`](./ROADMAP.md) — Stage 0–4 lifecycle.
- [`tasks/MATRIX.md`](./tasks/MATRIX.md) — topic-szintű mátrix
  (sprint pickup forrása).
- [`tasks/GAPS.md`](./tasks/GAPS.md) — item-szintű backlog (gap-ek
  központi gyűjtése).
- [`tasks/sprints/`](./tasks/sprints/) — egy fájl / aktív sprint.
- [`tasks/sprint-template.md`](./tasks/sprint-template.md) — másold ezt
  új sprint induláskor.
- [`ai/claude/`](./ai/claude/) — sablonok **derived SaaS projektekhez**
  (NEM a playbook saját CLAUDE.md-jéhez — az ez a fájl).
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — külső kontribútoroknak.
