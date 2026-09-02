# CLAUDE.md

## Kunden-Deliverables

Deliverables für Kunden werden mit `share <kunde> <datei>` veröffentlicht, nie manuell.
Kein Kopieren in Kunden-Repos von Hand, kein manuelles Editieren von `index.html`
oder `README.md` in einem Showroom-Repo — das erledigt das Script.

```
share [--to <typ>] [--force] <kunde> <datei-oder-ordner>
```

Script: `bin/share`. Einmalig verlinken:

```
ln -s "$PWD/bin/share" ~/.local/bin/share
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
