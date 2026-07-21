# 🎉 JGA Punkte-Tracker („Mission Ludi")

Eine kleine Web-App, um bei einem Junggesellenabschied die Punkte des Junggesellen zu tracken. Alle stellen ihm Spiele und Challenges – er sammelt Punkte gegen eine Zielpunktzahl.

**Live:** https://krisswi.github.io/ludi/

## Was die App kann

- **Ein Junggeselle, eine Zielpunktzahl** mit prominentem Fortschrittsbalken (`247 / 500`)
- **Spiele anlegen** – im Voraus planbar, inklusive Runden, Punkte trägt man später ein
- **Runden mit Titel**, aus-/einklappbar (Akkordeon – immer nur ein Spiel offen)
- **Pro Runde: Offen / ✅ Punkte / 😖 Strafe**
  - Mögliche Punkte pro Spiel als Vorschlag, pro Runde übersteuerbar
  - **Joker** für doppelte Punkte (×2)
- **Strafpunkte statt Minuspunkte:** eine Strafe schiebt das Ziel nach oben, statt Punkte abzuziehen
- **Spiele sortierbar** (▲▼) und nachträglich bearbeitbar (✏️)
- **Verlauf** aller Einträge, „letzten Eintrag rückgängig", Rückfrage vor jedem Löschen
- **Live-Sync** auf allen Geräten (Supabase Realtime) und **Konfetti** bei Zielerreichung

## Technik

- **Frontend:** eine einzige statische Datei (`index.html`), gehostet über GitHub Pages
- **Backend/Datenbank:** [Supabase](https://supabase.com) (Postgres)
- **Zugang:** offen, ohne Login – wer den Link hat, kann eintragen und ändern

> ⚠️ **Hinweis:** Der Zugang ist bewusst offen gehalten (praktisch für einen Freundeskreis). Teile den Link nur mit euren Leuten. Der öffentliche Supabase-Schlüssel im Code ist dafür vorgesehen, im Frontend zu stehen.

## Einrichtung der Datenbank (Supabase)

Falls du das Projekt neu aufsetzt: dieses SQL im Supabase **SQL Editor** ausführen.

```sql
-- Einstellungen (eine Zeile)
create table settings (
  id int primary key default 1,
  bachelor_name text,
  target_score int not null default 500,
  constraint single_row check (id = 1)
);
insert into settings (id, bachelor_name, target_score) values (1, 'Junggeselle', 500);

-- Spiele
create table games (
  id bigint generated always as identity primary key,
  title text not null,
  description text,
  has_rounds boolean not null default false,
  round_points_mode text not null default 'individuell',
  fixed_points int,
  suggested_points int,      -- Vorschlag für mögliche Punkte pro Runde
  sort_order int,            -- Reihenfolge der Spiele
  planned_date date,
  created_at timestamptz not null default now()
);

-- Einträge / Runden
create table entries (
  id bigint generated always as identity primary key,
  game_id bigint references games(id) on delete cascade,
  round_no int,
  title text,                                  -- Titel der Runde
  result_type text not null default 'offen',   -- 'offen' | 'punkte' | 'strafe'
  points int not null default 0,
  suggested_points int,                        -- mögliche Punkte dieser Runde
  multiplier int not null default 1,           -- Joker: 2 = doppelt
  target_delta int not null default 0,         -- Strafe: erhöht das Ziel
  note text,
  created_at timestamptz not null default now()
);

-- Offener Zugang: jeder darf lesen und schreiben
alter table settings enable row level security;
alter table games    enable row level security;
alter table entries  enable row level security;
create policy "offen" on settings for all using (true) with check (true);
create policy "offen" on games    for all using (true) with check (true);
create policy "offen" on entries  for all using (true) with check (true);

-- Live-Sync aktivieren
alter publication supabase_realtime add table settings, games, entries;
```

Danach in `index.html` oben im `<script>` die eigene Supabase-URL und den `publishable`-Key eintragen:

```js
const SUPABASE_URL = 'https://DEINPROJEKT.supabase.co';
const SUPABASE_KEY = 'sb_publishable_...';
```

## Veröffentlichen (GitHub Pages)

1. Repository anlegen (Public)
2. `index.html` hochladen
3. **Settings → Pages → Source: „Deploy from a branch" → Branch `main` / `root` → Save**
4. Nach ~1 Minute ist die Seite unter `https://<benutzer>.github.io/<repo>/` erreichbar

## Änderungen einspielen

App anpassen = einfach die neue `index.html` im Repo hochladen (überschreiben). Die Seite aktualisiert sich automatisch.
