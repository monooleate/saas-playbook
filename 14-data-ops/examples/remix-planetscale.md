# 14 — Example: Remix + PlanetScale

> **Status:** Outline.

PlanetScale uses Vitess; migrations are *deploy requests* on a branch.
Drizzle Kit generates the SQL; you push to a dev branch, create a
deploy request, review, then deploy. Vitess's online-DDL handles big
table changes.
