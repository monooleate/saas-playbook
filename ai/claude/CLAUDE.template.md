# CLAUDE.md – {{PROJECT_NAME}} projekt alkotmány

> Ezt a fájlt Claude Code minden session elején olvassa el. Soha ne töröld.
> Utolsó frissítés: **{{YYYY-MM-DD}}** ({{rövid változás-leírás}})

---

> **TEMPLATE-USAGE INSTRUCTIONS** (törlendő miután kitöltötted):
>
> 1. Helyettesítsd a `{{PLACEHOLDER}}`-eket projekt-specifikus értékekkel.
> 2. Egészítsd ki a "Témakör → doksi leképzés" táblázatot a *te* projekted
>    doksi-szerkezetével.
> 3. Adj hozzá projekt-specifikus szakaszokat (pl. "Aktuális állapot",
>    "Kulcs dokumentumok").
> 4. Töröld ezt a TEMPLATE-USAGE INSTRUCTIONS blokkot.

---

## TL;DR — Mit kell tudni első session-ben

**{{PROJECT_NAME}}** = {{PROJECT_DESCRIPTION_ONE_SENTENCE}}.

Stack: {{STACK_BULLETS}} (pl. SvelteKit / Next.js / Astro + Supabase / Prisma / Drizzle + Stripe / Lemon Squeezy + Netlify / Hetzner).

Ha csak egy dolgot olvasol el, ez legyen:

1. **{{KEY_PATTERN_1}}** (pl. `Tenant = subdomain` / `Tier rendszer` / `i18n routing`).
2. **{{KEY_PATTERN_2}}** (pl. provider-választás, auth flow, RLS minta).
3. **{{KEY_PATTERN_3}}** (pl. árak forrása `pricing.json`-ban, addon registry).
4. **shell = bash** (git-bash Windows-on); ne keverj PowerShell-syntaxot.

---

## Projekt áttekintés

**{{PROJECT_NAME}}** – {{PROJECT_DESCRIPTION_LONGER}}.

- **Stack:** {{STACK_DETAILS}}
- **Hosting:** {{HOSTING}}
- **Multi-tenancy modell:** {{TENANCY_MODEL}}
- **Billing provider:** {{BILLING_PROVIDER}}
- **Auth provider:** {{AUTH_PROVIDER}}

**Referencia dokumentumok:**

- **`README.md`** — projekt áttekintés (kifelé orientált).
- **`SETUP.md`** vagy `internal-docs/SETUP.md` — production deployment guide.
- **`internal-docs/DOC-GAP-AUDIT.md`** — **single source of truth a doksi
  állapotra** (mit írtunk, mit nem, mi minőségileg rossz, milyen sorrendben
  fejlesszük). **Olvasd el negyedévente.**
- **`internal-docs/architecture.md`** vagy `ARCHITECTURE.md` — átfogó
  architektúra.
- **`internal-docs/CHANGELOG.md`** — időrendi változási log.
- {{PROJECT_SPECIFIC_KEY_DOCS_LIST}}

---

## A mátrix-filozófia (központi minta)

Minden dokumentált rendszer-elem (= egy `internal-docs/<topic>.md` fájl,
ami leképezi a `saas-playbook` 24-es topic egyikét) **három párhuzamos
outputot termel** session során:

1. **Tartalom** — a doksi maga (rendszer leírása, döntések, kapcsolatok,
   kód-kapcsolatok).
2. **Gap analysis** — mi maradt homályos, mi hiányzik, mi nincs még
   dokumentálva a kódhoz képest, hol van inkonzisztencia.
3. **Sprint-pickable taskok** — a gap-ekből származó konkrét, becsülhető
   (S/M/L), prioritizált (P0/P1/P2) feladatok.

A doksi-írás nem csak content-termelés — egyszerre felfedezés és backlog-
építés. Ha egy session csak az #1-est szállítja, a #2-#3 elveszik.

### Hol élnek a gap-ek és a taskok (két hely, szándékos duplikáció)

Output #2 + #3 **két helyen** él:

- **A doksi végén** (`internal-docs/<topic>.md` legutolsó szakasz:
  `## Open gaps & sprint-pickable tasks`) — kontextusban; aki a doksit
  olvassa, egy görgetésre lássa "mi még nincs lefedve, mi a következő
  lépés".
- **Centrálisan a `internal-docs/DOC-GAP-AUDIT.md` 5. szekciójában**
  (Phase 1 / 2 / 3 gap-lista, prompt-ready scope-pal) — sprint-pickup
  forrás; sprint-tervező egy fájlt szkennel, nem 24-et.

**ID konvenció:** `G-<topic-number>-<sequence>` (pl. `G-04-01` az első
email-gap, `G-14-03` a harmadik data-ops-gap). Az ID a forrásra mutat
vissza, és a két hely között hard-linkként működik.

A két hely **muszáj sync-ben** lenni — ha egy gap csak az egyik helyen
van, a self-update szabály (lent) hiányos. Sprint-tervezéskor
DOC-GAP-AUDIT.md 5. szekcióból pickelünk; doksi-olvasáskor a doksi
végén látjuk ugyanazt.

### Mit gyűjt mi (gyors referencia)

| Fájl | Mit gyűjt | Granularitás |
|------|-----------|--------------|
| `internal-docs/DOC-GAP-AUDIT.md` 3. szekció | Topic doksi-fedettség (mi van / nincs / félig) | Topic-szint |
| `internal-docs/DOC-GAP-AUDIT.md` 4. szekció | Quality audit — meglévő doksik konkrét hibái (Q1–Q…) | Hiba-szint |
| `internal-docs/DOC-GAP-AUDIT.md` 5. szekció | **Sprint-pickable backlog** — gap-derived taskok Phase 1/2/3-ba bontva | Item-szint |
| `internal-docs/<topic>.md` vége | Topic saját gap-jei és sprint-pickable taskjai | Item-lista, kontextusban |
| `{{SPRINT_LOG_FILE}}` | Aktív sprintek + retro | Story-szint |

A 4 + sprint-szint redundáns, szándékosan. Sprint-tervező a 3.
szekcióból szűr topicra, az 5. szekcióból pickel itemeket, a sprint-
fájlba bemásolja, a topic doc-ban látja a kontextust + a 4. szekcióból
látja a quality-fix-eket amik mellé összefűzhetők.

---

## Working style

- **Autonóm végrehajtás** — ne kérdezz a session minden lépése között.
- **Csak akkor állj meg**, ha tényleg blokkoló human input kell.
- **Smoke teszt minden kódváltozás után**: `{{BUILD_COMMAND}}` (pl.
  `pnpm --filter @app build`, `npm run build`).
- **TypeScript strict mode** — `tsc --noEmit` is checkold occasionally.
- **{{PROJECT_SPECIFIC_WORKING_STYLE_RULES}}** — pl. i18n minden user-facing
  szöveghez, dark mode default, mindig minden test-runnerrel egyezzen.

---

## Doksi szinkronizáció (KÖTELEZŐ minden session-ben)

> **Szabály:** Ha a session során **bármilyen kódváltozás érinti egy
> dokumentált rendszer-elem témakörét**, a vonatkozó `internal-docs/<doksi>.md`
> fájlt **MÉG A SESSION VÉGE ELŐTT frissíteni kell**. Nem hagyjuk a következő
> munkamenetre.

### Single source of truth: `DOC-GAP-AUDIT.md`

**Mielőtt új doksi-t írsz vagy meglévőt módosítasz, mindig nyisd meg az
`internal-docs/DOC-GAP-AUDIT.md`-t.**

- A 3. szekció táblázata: minden topichoz pontos doksi-státusz (✅/🟡/❌).
- A 4. szekció Quality audit: a meglévő doksik konkrét hibalistája (Q1–Q…).
  **Ha javítasz egy ott jelölt hibát, a 4-es szekció táblázatában húzd át
  a Q-bejegyzést.**
- Az 5. szekció Phase 1/2/3: új doksi-feladatok prompt-ready scope-pal.

### Témakör → doksi leképzés

| Mire vonatkozik a kódváltozás | Frissítendő doksi |
|---|---|
| DB séma változás (új tábla, oszlop, RLS, index) | `{{DB_DOC}}` (pl. `architecture.md` 3. szakasz) |
| Új migration | `{{MIGRATIONS_DOC}}` (pl. `MIGRATIONS.md` + `architecture.md`) |
| Auth flow | `{{AUTH_DOC}}` |
| Új API endpoint | `{{API_DOC}}` (pl. `architecture.md` 4. szakasz) |
| Új page / route | `{{ROUTING_DOC}}` |
| i18n string új kategória | `{{I18N_DOC}}` |
| Új env változó | `{{ENV_DOC}}` (pl. `SETUP.md` 4. szakasz + `ENV-VARIABLES.md`) |
| Új cron job | `{{CRON_DOC}}` |
| Billing flow változás | `{{BILLING_DOC}}` (provider-aware) |
| Email küldési flow | `{{EMAIL_DOC}}` |
| Tier / addon / limit | `{{TIER_DOC}}` |
| **Bármilyen jelentős fix vagy új feature** | **`internal-docs/CHANGELOG.md`** ÚJ bejegyzéssel |
| **Új gap / sprint-pickable task felfedezve** | **`internal-docs/<topic>.md` vége** (`## Open gaps & sprint-pickable tasks` szakasz, ID: `G-NN-NN`) **+ `internal-docs/DOC-GAP-AUDIT.md` 5. szekció** (Phase-megfelelő bucket) — **mindkét helyre, ugyanabban a sessionben** |

### Triggerek — automatikus felismerés

A session során ezek a minták KÖTELEZŐ doksi-frissítést jelentenek:

- **Új migration fájl** a `migrations/`-ben → `{{DB_DOC}}` + `{{MIGRATIONS_DOC}}`.
- **Új route** a `{{ROUTES_FOLDER}}`-ben → `{{ROUTING_DOC}}` + témaspecifikus.
- **Új env változó** (`process.env.<NAME>` vagy `import.meta.env.<NAME>`) → `{{ENV_DOC}}`.
- **Új cron endpoint** → `{{CRON_DOC}}`.
- **`pricing.json` / `limits.json` módosítás** → `{{PRICING_DOC}}` + `{{TIER_DOC}}`.
- **Új modul a `lib/`-ben** → `{{ARCH_DOC}}` (struktúra szakasz).
- **Doksi-írás session során "ezt később ki kéne fejteni" / "homályos" / "tisztázandó" felmerült** → új gap a forrás-doksi végén ÉS DOC-GAP-AUDIT 5. szekcióban. Ne hagyd csak a sessionben — különben elveszik.

### Speciális szabály — incidensek

Ha session során **production-érintő bug derült ki**:
1. **Hozz létre `internal-docs/INCIDENT-YYYY-MM-DD-<rövid-cím>.md`-t** —
   formátuma: Symptom, Root Cause, Fix, Post-incident actions.
2. **Linkelj rá** a `{{ARCH_DOC}}` "Sebezhetőségek" / "Ismert problémák"
   szakaszából.
3. **CHANGELOG bejegyzés** a fix sessionjével.

### Mit NE frissíts

- **`README.md`** ne változzon trivialis kódváltozásokra — ez a marketing
  / quick-orientation doksi. Csak nagyobb struktúra-vagy-stack-változáskor.
- **Verziókat** ne emelj a doksikban kódváltozás nélkül.

### CLAUDE.md self-update (KÖTELEZŐ minden session után)

A `CLAUDE.md` is "élő doksi" — minden session vége előtt ellenőrizd:

1. **Új modul / mappa / konvenció** került be? → frissíteni a "Projekt
   áttekintés" vagy "Aktuális állapot" szakaszokat.
2. **Új doksi-fájl** került az `internal-docs/`-be? → frissíteni a
   "Témakör → doksi leképzés" táblázatot.
3. **Új env változó kategória** (pl. új provider integráció)? → frissíteni
   a TL;DR + a Doksi szinkron táblázatot.
4. **Sprint mérföldkő** (új sprint indul / lezárul)? → frissíteni az
   "Aktuális állapot" szakaszt, és a CLAUDE.md fejlécében az "Utolsó
   frissítés" dátumot.
5. **DOC-GAP-AUDIT.md 4. szekció Q-bejegyzés** lefuttat egy javítást,
   ami érinti a CLAUDE.md-t? → CLAUDE.md is frissül.
6. **Új gap / sprint-pickable task** felfedezve? → topic-doksi végén
   ÉS DOC-GAP-AUDIT 5. szekcióban, ID-konvencióval (`G-NN-NN`).
7. **Új konvenció vagy work-style szabály** felmerül? → "Working style"
   bővítve.

A CLAUDE.md fejléc-dátumát ("Utolsó frissítés: ...") **csak** akkor mozdítsd
előre, ha tényleges változtatás történt — **ne** triviális csere miatt.

### End-of-session checklist (1 perces)

```
[ ] Új doksi készült? → "Témakör → doksi leképzés" táblázat frissítve.
[ ] Új env változó? → TL;DR + Doksi szinkron táblázat frissítve.
[ ] Sprint mérföldkő? → "Aktuális állapot" + fej-dátum frissítve.
[ ] DB séma változás? → DB doksi frissítve?
[ ] Új API route? → API doksi frissítve?
[ ] Új user-facing feature? → CHANGELOG bejegyzés bekerült?
[ ] Billing flow változás? → érintett provider-doksi frissítve?
[ ] Új gap / sprint-pickable task felfedezve? → topic-doksi végén ÉS DOC-GAP-AUDIT 5. szekcióban (mindkettő, sync-ben)?
[ ] Topic-doksi végén levő gap-szakasz módosult? → DOC-GAP-AUDIT 5. szekció is reflektálja?
[ ] DOC-GAP-AUDIT-ben Q-bejegyzés javítva? → CLAUDE.md is frissült (ha érinti).
[ ] CLAUDE.md self-update szabály szerinti változás? → CLAUDE.md frissítve?
```

**Ha bármelyikre IGEN, és a doksi NEM frissült — a session nem zárható le.**

---

## Sprint munkafolyamat

1. **Olvasd a `DOC-GAP-AUDIT.md` 5. szekciót** (negyedévente kötelező;
   sprint tervezéskor ajánlott). Ez a sprint-pickable backlog forrása.
2. **Pick 3–8 itemet** az 5. szekcióból + Q-bejegyzéseket a 4.-ből,
   amik egy 1–2 hetes sprintbe férnek.
3. **Új sprintet** vegyél fel a {{SPRINT_LOG_FILE}}-be (vagy
   `sprints/YYYY-WNN-<theme>.md` fájlba), pickelt item-ekkel mint
   story-k.
4. **Implementáld a feature-t.**
5. **Doksi frissítés** a fenti táblázat szerint, **még a session vége
   előtt**. Ez magába foglalja:
   - A feature által érintett `internal-docs/<topic>.md` tartalmi
     frissítését.
   - Új gap-ek / következő-iteráció-feladatok hozzáadását a doksi
     végén levő `## Open gaps & sprint-pickable tasks` szakaszhoz
     ÉS DOC-GAP-AUDIT 5. szekcióhoz (sync-ben).
   - Befejezett item áthúzása / státusz-változtatása DOC-GAP-AUDIT-ban.
6. **CHANGELOG bejegyzés**.
7. **Sprint vége: záró bejegyzés** a sprint-fájlban — eredmények,
   tanulságok, következő sprint javaslat. Lezárt itemek statusz-
   frissítése DOC-GAP-AUDIT-ban (5. szekció: ✅ vagy áthelyezés
   "Done" szakaszba).

---

## Aktuális állapot ({{YYYY-MM}})

**Befejezett milestone-ok:**
- {{COMPLETED_MILESTONE_1}}
- {{COMPLETED_MILESTONE_2}}

**Folyamatban:**
- {{IN_PROGRESS_ITEM}}

**Tervezett (nem implementált):**
- {{PLANNED_ITEM}}

---

## Kritikus tisztázandó kérdések

A `DOC-GAP-AUDIT.md` 4. szekció Quality audit Q1–Q… hibalistájából a
P0 prioritású tisztázások:

1. **{{Q1_TITLE}}** — {{Q1_DESCRIPTION}}
2. **{{Q2_TITLE}}** — {{Q2_DESCRIPTION}}

**Ha bármelyikbe belefutsz session során, hozz létre egy ADR-t**
(`internal-docs/adr/ADR-NNN-…md`), és linkelj rá a vonatkozó doksiból.

---

## Hivatkozások

- **`saas-playbook` repo** — [github.com/monooleate/saas-playbook](https://github.com/monooleate/saas-playbook).
  A 24 topicos váz forrása. A topic README-k a kanonikus tartalom-vázlat.
- **`internal-docs/DOC-GAP-AUDIT.md`** — **a single source of truth** a
  doksi-állapotra és a fejlesztési sorrendre.
- **Diátaxis** — [diataxis.fr](https://diataxis.fr) — a doksi-mappa
  reorganizáció keretrendszere.
- **{{PROJECT_SPECIFIC_LINKS}}** — pl. nyilvános docs URL, API spec, design
  system reference.
