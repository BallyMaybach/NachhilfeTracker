# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

- **Projekt:** NachhilfeTracker
- **Was es tut:** PWA zur Verwaltung von Nachhilfestunden, Einnahmen und Rechnungsgenerierung
- **Stand:** Vollständig funktionsfähig — Auth, Sessions, Schüler, Statistiken, Text-Rechnungen, PDF-Rechnungen, Einstellungen
- **Deployment:** GitHub Pages — `git push` auf `master` genügt. URL: https://ballymaybach.github.io/NachhilfeTracker/
- **Design-Richtung:** Dark Mode, Standard-Palette aus globalem CLAUDE.md

## Architektur

Single-file PWA: alles in `index.html` (CSS im `<style>`-Block, JS im `<script>`-Block am Ende). Kein Build-System, keine Dependencies außer Supabase JS via CDN.

## Backend: Supabase

- **Projekt-ID:** `yyxnhwkdwcgpqajeknfe` (Region: eu-central-1, kostenloser Free Tier)
- **URL:** `https://yyxnhwkdwcgpqajeknfe.supabase.co`
- **Client:** `window.supabase.createClient(URL, ANON_KEY)` — Anon Key ist im `<script>`-Block hardcoded (sicher, da RLS alle Tabellen schützt)

**Tabellen:**
- `students` — `{ id text PK, user_id uuid, name text, rate numeric, adresse text }`
- `sessions` — `{ id text PK, user_id uuid, student_id text FK→students, date text, duration numeric, rate numeric, paid bool, abgerechnet bool, anfahrt bool, created_at timestamptz }`
- `settings` — `{ user_id uuid PK, iban text }`

Alle Tabellen haben Row Level Security: `auth.uid() = user_id`. Schreibzugriff nur für den eingeloggten User.

**DB-Operationen** (alle async, Fehler via `dbRun()` geloggt):
- `dbAddStudent`, `dbUpdateStudent`, `dbDeleteStudent`
- `dbAddSession`, `dbUpdateSession`, `dbDeleteSession`, `dbMarkAbgerechnet`
- `dbSaveSettings`

`dbRun(promise)` ist ein Wrapper der `{ error }` prüft und bei Fehler `console.error('[DB Error]', ...)` aufruft.

## Auth

Login-Screen (`#loginScreen`) erscheint beim Start wenn keine Session vorhanden. `init()` prüft per `sb.auth.getSession()`. `onAuthStateChange` reagiert auf `SIGNED_IN` / `SIGNED_OUT`. Logout-Button in den Einstellungen.

Einziger User: `balthasar.beyer@gmail.com` — E-Mail-Bestätigung in Supabase manuell via SQL bestätigt worden.

## Datenfluss

State: `students[]`, `sessions[]`, `settings`, `currentUser`. Nach jeder Mutation: DB-Operation aufrufen (async, fire-and-don't-wait für UI-Updates) + `render()`.

`render()` ruft immer alle fünf Teil-Renderer auf: `renderStats`, `renderFilter`, `renderStatusFilter`, `renderStudentDetail`, `renderSessions`.

**Session-Zustände:**
- Offen: `paid: false, abgerechnet: false`
- Bezahlt: `paid: true`
- Abgerechnet: `abgerechnet: true`

`sessionAmount(s)` = `(duration / 60) * rate + (anfahrt ? 5 : 0)`

**Filter-State:**
- `activeFilter`: `'all'` oder Student-ID
- `statusFilter`: `'all'` | `'open'` | `'paid'`

**Sortierung:** Offen → Abgerechnet → Bezahlt; innerhalb jeder Gruppe nach Datum absteigend.

## Rechnungen

- **Text-Rechnung** (`buildInvoiceText` + `openInvoiceModal`): Kopierbarer Plaintext. Markiert Sessions als `abgerechnet: true` via `dbMarkAbgerechnet`.
- **PDF-Rechnung** (`generiereRechnung`): Öffnet `window.open('', '_blank')` mit vollständigem HTML-Dokument, ruft `window.print()` auf. Rechnungsnummer-Counter bleibt in localStorage (`nh_rechnungsNummer_counter`) — einzige verbleibende localStorage-Nutzung.

**Modals:** `showModal(id)` / `hideModal(id)` via CSS-Klasse `.visible`. Overlay-Click schließt alle. Modals: `sessionModal`, `studentModal`, `settingsModal`, `invoiceModal`.

## PWA / Service Worker

- `manifest.json` + `sw.js` — Cache-Name `nachhilfe-v1`
- SW cached: `./`, `./index.html`, `./icon.png`, `./manifest.json`
- **Wichtig:** Bei Asset-Änderungen Cache-Name in `sw.js` erhöhen (z.B. `nachhilfe-v2`), sonst bekommen Nutzer die alte Version aus dem Cache.
- `safe-area-inset-*` CSS-Variablen für iPhone-Notch/Home-Indicator

## AIOS

Ballys AI Operating System liegt unter:
`C:\Users\Bally\OneDrive\Desktop\BUSINESS\Bally Ordner\AI-OS-Bally`

Dort liegen Skills, Connections, Context und alle Automatisierungen.
