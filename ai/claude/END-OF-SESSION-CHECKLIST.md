# End-of-session checklist — Claude Code

> 1 perces checklist amit minden Claude Code session vége előtt át kell futni.
> A `CLAUDE.md`-ben dokumentált "Doksi szinkron" + "CLAUDE.md self-update"
> szabályok kondenzált formája.

## Használat

**Mód 1 — emberi review:** session vége előtt nyisd meg ezt a fájlt és
fusd át a checklistet. Bármi IGEN → fejezd be a doksit, mielőtt lezárod.

**Mód 2 — Claude prompt:** illeszd be a session zárása előtt mint promptot,
és kérd hogy Claude maga futtassa át az ellenőrzést és frissítsen ami kell.

```
Session lezárása előtt fusd át az ai/claude/END-OF-SESSION-CHECKLIST.md
listáját. Minden IGEN-re tedd meg a megfelelő doksi-frissítést, és
sorold fel a kimenetben mit frissítettél (vagy "minden NEM, nincs
frissítés szükséges").
```

---

## Checklist

### Doksi szinkron — kódváltozás → doksi-frissítés

```
[ ] Volt-e DB séma változás (új tábla, oszlop, RLS, index, constraint)?
    → DB doksi (architecture.md DB szakasz / DATABASE-SCHEMA.md) frissítve?

[ ] Volt-e új migration fájl?
    → MIGRATIONS doksi + DB doksi frissítve?

[ ] Volt-e új API route / endpoint?
    → API doksi (architecture.md API szakasz / OpenAPI / similar) frissítve?

[ ] Volt-e új page / route a frontend-ben?
    → Routing doksi frissítve (ha új path-konvenció)?

[ ] Volt-e új env változó (process.env.<NAME> vagy import.meta.env.<NAME>)?
    → ENV-VARIABLES doksi + SETUP doksi (4. szakasz általában) frissítve?

[ ] Volt-e új cron job / scheduled task?
    → CRON-JOBS doksi frissítve?

[ ] Volt-e billing / subscription flow változás?
    → érintett provider-doksi (Stripe / Lemon / Paddle / Barion / etc.)
      frissítve?

[ ] Volt-e auth / permission flow változás?
    → AUTH / PERMISSIONS doksi frissítve?

[ ] Volt-e új email küldés / sablon?
    → EMAIL doksi frissítve?

[ ] Volt-e tier / addon / limit változás (config + ár)?
    → TIER / ADDON / PRICING doksi frissítve?

[ ] Volt-e új user-facing feature vagy jelentős fix?
    → CHANGELOG bejegyzés bekerült?

[ ] Volt-e production-érintő bug-felderítés?
    → INCIDENT-YYYY-MM-DD-...md létrehozva (Symptom / Root Cause / Fix /
      Post-incident actions)?

[ ] Új feature érinti az i18n string-eket?
    → minden támogatott nyelv frissítve (vagy TODO marker)?
```

### CLAUDE.md self-update

```
[ ] Új modul / mappa / kódolási konvenció került be?
    → CLAUDE.md "Projekt áttekintés" / "Aktuális állapot" / "Working style"
      szakasz frissítve?

[ ] Új doksi-fájl került az internal-docs/-be?
    → CLAUDE.md "Témakör → doksi leképzés" táblázat frissítve?

[ ] Új env változó kategória (pl. új provider integráció)?
    → CLAUDE.md TL;DR + Doksi szinkron táblázat frissítve?

[ ] Sprint mérföldkő (új sprint indul / lezárul)?
    → CLAUDE.md "Aktuális állapot" + fej-dátum ("Utolsó frissítés:")
      frissítve?

[ ] DOC-GAP-AUDIT.md 4. szekció Q-bejegyzés javítva session során?
    → Q-bejegyzés a táblázatban kihúzva + CLAUDE.md is frissült (ha érinti)?

[ ] Új konvenció / work-style szabály felmerült?
    → CLAUDE.md "Working style" vagy új szakasz hozzáadva?
```

### DOC-GAP-AUDIT státusz

```
[ ] Új doksi készült session során?
    → DOC-GAP-AUDIT 3. szekció táblázat státusz frissítve (❌→🟡 vagy 🟡→✅)?

[ ] Quality fix lefuttatva (4. szekció Q[N] javítás)?
    → Q-bejegyzés kihúzva a 4. szekció priority táblázatából?
```

---

## Ha bármelyikre IGEN, és a doksi NEM frissült

**A session nem zárható le.** Fejezd be a doksit, majd zárd a sessiont.

A doksi **NEM** a kód duplikációja, hanem **a kód mögötti döntések,
kapcsolatok, és belső logika** rögzítése. Egy ALTER TABLE doksi-frissítés
nélkül = a következő session vakon dolgozik vele.

Ezt a checklist-et a `CLAUDE.md` Doksi szinkron + CLAUDE.md self-update
szabályok kondenzálják — a részletes leírást ott találod.
