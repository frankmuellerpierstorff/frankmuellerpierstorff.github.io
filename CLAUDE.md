# CLAUDE.md

## Kunden-Deliverables

Deliverables für Kunden werden mit `share <kunde> <datei>` veröffentlicht, nie manuell.
Kein Kopieren in Kunden-Repos von Hand, kein manuelles Editieren von `index.html`
oder `README.md` in einem Showroom-Repo — das erledigt das Script.

```
share [--to <typ>] [--note <text>] [--status draft|review|final]
      [--expires 30d|2w|3m|1y|YYYY-MM-DD] [--force] <kunde> <datei-oder-ordner>
unshare <kunde> <artefakt|slug>
share --protect [--password <pw>] <kunde>
share --unprotect <kunde>
```

Script: `bin/share`, `unshare` ist ein Symlink darauf. Einmalig verlinken:

```
ln -s "$PWD/bin/share" ~/.local/bin/share
ln -s "$PWD/bin/share" ~/.local/bin/unshare
```

### Was passiert

1. Kunden-Repo unter `~/showrooms/<kunde>/` (Repo `directional-showroom-<kunde>`).
   Fehlt der Klon, wird geklont — existiert das Repo nicht, bricht `share` ab.
2. Jedes Deliverable wird ein **Artefakt-Ordner**: `<typ>/<datum>-<slug>/`.
   Verzeichnisse (Websites, Prototypen) werden 1:1 samt Unterordnern kopiert;
   Einzeldateien bekommen ein generiertes `index.html` als Viewer + Download.
3. Root-`index.html` wird komplett neu generiert: alle Artefakte, neueste oben,
   statisch, kein Framework.
4. `README.md` → Abschnitt `## Aktuell`, neuer Eintrag oben.
5. `git add` / `commit "share: <dateiname>"` / `push`.
6. Ausgabe: Repo-URL + Vercel-URL (aus `.showroom-config`).

### Versionen

Derselbe Dateiname legt eine neue Version daneben: `…-pitch-deck/`, dann
`…-pitch-deck-v2/`. Alte Versionen bleiben erreichbar und stehen im Index
ausgegraut. `latest/<slug>/` leitet immer auf die neueste — **diesen Link an
Kunden schicken**, dann veraltet keine verschickte URL.

### Passwortschutz

```
share --protect acme                      # generiert ein Passwort und zeigt es einmal
share --protect --password "geheim" acme  # eigenes Passwort setzen
share --unprotect acme                    # Schutz entfernen
```

`--protect` legt Salt und PBKDF2-Hash (120k Runden, SHA-256) in
`.showroom-auth.json` ab — **das Passwort selbst wird nirgends gespeichert**.
Ein generiertes Passwort erscheint genau einmal in der Ausgabe, danach ist es
weg; direkt in den Passwortmanager.

`share` generiert dazu `middleware.js` im Showroom-Repo neu (Vercel Edge
Middleware, `@vercel/edge` in der `package.json`, kein Build-Script). Die
Middleware liegt vor allen Dateien, also greift der Schutz auch auf der
`*.vercel.app`-URL. Der Kunde tippt das Passwort einmal auf einer schlichten
Loginseite, danach traegt ein signiertes HttpOnly-Cookie 30 Tage.

Der Cookie-Signierschluessel wird aus dem gespeicherten Hash abgeleitet. Wer
zusaetzlich haerten will, setzt in Vercel die Env-Var `SHOWROOM_SECRET` — dann
reicht ein Repo-Leak allein nicht mehr, um Cookies zu faelschen.

Grenze: ein Passwort gilt fuer alle Empfaenger. Wer es weitergibt, gibt Zugang
weiter, und die Logs zeigen nicht, wer drin war.

### Ablauf und Rueckzug

`--expires` schreibt ein Ablaufdatum in `.share-meta`. Abgelaufene Artefakte
fallen beim naechsten Lauf aus Index und `latest/`, und die Middleware liefert
ab dem Stichtag 410 aus — auch fuer Dateien tief im Artefakt-Ordner und auch
ohne Passwortschutz.
`unshare <kunde> <artefakt|slug>` loescht ein Artefakt wirklich und
regeneriert Index, `latest/` und README.

Kein Deploy im Script — Vercel hängt am Repo und deployt beim Push.

### Routing

| Endung | Ziel |
| --- | --- |
| `pdf` `pptx` `ppt` `key` | `decks/` |
| `html` `htm` | `html/` |
| `mp4` `mov` `webm` `m4v` | `videos/` |
| `png` `jpg` `jpeg` `gif` `svg` `webp` | `images/` |
| `docx` `doc` `md` `txt` `rtf` `xlsx` `csv` | `docs/` |
| alles andere + Verzeichnisse | `prototypes/` |

Pro Kunde überschreibbar in `.showroom-config` im Showroom-Repo, Einzelfall mit `--to`:

```
VERCEL_URL=https://<kunde>-showroom.vercel.app
ROUTE_zip=downloads
INDEX_DIRS=downloads
```

`INDEX_DIRS` nimmt zusätzliche Ordner in den Root-Index auf.

### Env

`SHOWROOM_ROOT` (Default `~/showrooms`), `SHOWROOM_OWNER` (Default
`frankmuellerpierstorff`), `SHOWROOM_REPO_PREFIX`, `SHOWROOM_REMOTE_BASE`.
