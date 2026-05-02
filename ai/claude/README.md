# `ai/claude/` — Claude Code projekt-onboarding kit

> Sablon-fájlok ahhoz, hogy egy új SaaS projekt **első napjától** be legyen
> állítva a Claude Code-dal való alapos, doksi-szinkronos munkára.

## Mi ez

Amikor egy új SaaS projekt készül a `saas-playbook` alapján, két fájlt érdemes
azonnal a projekt gyökerébe / `internal-docs/`-be másolni:

1. **`CLAUDE.template.md`** → másold a projekt gyökerébe `CLAUDE.md` néven.
   Ez a "projekt-alkotmány" — minden Claude Code session elején betöltődik.
2. **`DOC-GAP-AUDIT.template.md`** → másold az `internal-docs/DOC-GAP-AUDIT.md`
   néven. Ez a single source of truth a doksi-fedettségre, prioritásra,
   és a quality auditra.

A template-ek **placeholderekkel** vannak — ki kell tölteni a projekt-specifikus
részeket (név, stack, doksi-mappastruktúra, etc.).

## Miért külön mappa

Ha a `saas-playbook`-ot mint planning eszközt használod *több* SaaS-hoz, akkor
ezeket a sablonokat újra fel akarod használni — ez a kanonikus hely.

A `ai/claude/` mappa nevezéktan-konvenciója:
- `ai/` = AI-asszisztens-specifikus eszközök.
- `claude/` = Claude Code (Anthropic) számára.
- Jövőbeli kiterjesztés: `ai/cursor/`, `ai/aider/`, `ai/copilot/` ha szükségessé válik.

## Tartalom

| Fájl | Cél |
|------|-----|
| `CLAUDE.template.md` | Sablon `CLAUDE.md`-hez (projekt-alkotmány) |
| `DOC-GAP-AUDIT.template.md` | Sablon `internal-docs/DOC-GAP-AUDIT.md`-hez |
| `END-OF-SESSION-CHECKLIST.md` | A 1 perces session-vége checklist (akár Claude prompt is, akár emberi szem) |

## Használat új projekthez

```bash
# A saas-playbook gyökerében:
cp ai/claude/CLAUDE.template.md /path/to/new-project/CLAUDE.md
cp ai/claude/DOC-GAP-AUDIT.template.md /path/to/new-project/internal-docs/DOC-GAP-AUDIT.md

# Aztán nyisd meg mindkettőt és töltsd ki a {{PLACEHOLDER}}-eket:
# - {{PROJECT_NAME}}, {{PROJECT_DESCRIPTION}}, {{TECH_STACK}}, etc.
```

A két template **összekapcsolt**: a CLAUDE.md hivatkozik a DOC-GAP-AUDIT-ra mint
single-source-of-truth, és a DOC-GAP-AUDIT minden új doksi-feladatot / quality
fix-et összevet a CLAUDE.md doksi-szinkron szabályaival.

## Az alapelv

- **CLAUDE.md** = **how we work here** — projekt-konvenciók, kódolási stílus,
  doksi-szinkron szabályok.
- **DOC-GAP-AUDIT.md** = **what's documented and what isn't** — fedettség
  state-machine + quality audit + roadmap.
- A kettő együtt biztosítja, hogy Claude Code **minden session után**
  következetesen frissítse a kódhoz tartozó doksikat.

## Élő példák

A `saas-playbook` jelenlegi referencia-projektjei mindkét sablon teljesen
kitöltött verzióját élben tartják:

- **Grabit** (`bookolj/`) — `CLAUDE.md` + `internal-docs/DOC-GAP-AUDIT.md`
  (24 topic mapping + Quality audit Q1–Q12 + Master prompt).
- **CutOptim** (`opticut/`) — `CLAUDE.md` + `internal-docs/DOC-GAP-AUDIT.md`
  (24 topic mapping + Quality audit Q1–Q12 + Master prompt).

Ha új projektet indítasz a sablonokból, érdemes ezeket a kitöltött verziókat
is egy pillantással megnézni példaként.
