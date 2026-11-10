# Unser Somora-Fork: Wartung, Updates und Releases

Dieses Dokument ist die Arbeitsanweisung für Menschen und Coding-Agenten in **packetloss99/somora_agent**. Das Repository ist öffentlich; niemals Zugangsdaten, Produktionskonfiguration oder Gespräche committen. Das Original bleibt `thenaxon/somora_agent`.

## Grundregel

**Aktueller stabiler Original-Release + kleinster eigener Patch.** Keine parallele Produktentwicklung, keine ungeprüften Dependency-Updates und keine Installation eines beliebigen Branch-Stands. Änderungen an unserem Hauptbranch ausschließlich über PRs in unserem Fork. Upstream-PRs sind ein separater Auftrag.

## Branches und Herkunft

| Branch/Datei | Zweck |
| --- | --- |
| `upstream-stable` | Exakt ein unveränderter offizieller Release-Commit; keine eigenen Commits. |
| `main` | Diese Release-Basis plus unsere Anpassungen und Fork-Wartungsdateien. Installationsquelle nach Freigabe. |
| `feat/*`, `fix/*`, `chore/*` | Kleine Änderungen → PR nach `main`. |
| `update/*` | Nächster stabiler Upstream-Release plus unsere erhaltenen Änderungen → Update-PR. |
| `fork-upstream.json` | Repository, Original-Tag und vollständiger Commit der verwendeten Basis. Bei jedem Upstream-Update im PR aktualisieren. |

Initiale Basis: **v2026.1106.3**, Commit `1feaad08c8077230c7af1b96050611592652579f`. Ein neuerer Git-Tag ist nicht automatisch ein veröffentlichter stabiler Release. GitHub Releases und npm prüfen.

Unsere fachliche Abweichung ist die optionale Herdr-TUI-Anbindung: Status, Resume-Befehl, Exit-Freigabe und explizite Agent-/Session-Startargumente. Fork-Wartung bleibt eine eigene Änderung. Wird die Integration später upstream übernommen, unseren entsprechenden Patch entfernen, nicht doppelt pflegen.

`origin` zeigt auf `https://github.com/packetloss99/somora_agent.git`, `upstream` auf `https://github.com/thenaxon/somora_agent.git`.

## Einen Original-Release übernehmen

1. Neuesten **veröffentlichten stabilen** Release bestimmen, Release Notes lesen und Tag/Commit verifizieren.
2. Sauberen Checkout verwenden:

   ```bash
   git fetch origin
   git fetch upstream --tags
   git switch -c update/<release> origin/main
   git merge --no-ff <original-release-tag>
   ```

3. Konflikte bewusst lösen. Unsere Integration erhalten, aber upstream inzwischen vorhandene Lösungen nicht duplizieren. `fork-upstream.json` auf den neuen Original-Tag/Commit setzen. Mitgeänderte Workflows und deren Repository-Guards prüfen.
4. Branch pushen und PR **nach packetloss99/somora_agent:main** öffnen. Alten/neuen Release, eigene Patch-Auswirkungen, Konflikte, Prüfungen und Risiken nennen. Keine Dependency-Aufräumarbeiten beimischen.
5. Fork-CI abwarten und einen Live-Check auf einer separat freigegebenen Test-/Snapshot-Instanz durchführen. Keine laufenden Produktionsanfragen ungefragt unterbrechen.
6. PR nach Freigabe mergen. **Erst danach** den Basisbranch auf genau den neuen Original-Commit vorziehen:

   ```bash
   git push origin <original-release-tag>^{commit}:refs/heads/upstream-stable
   ```

   Das ist die einzige direkte Branch-Aktualisierung im normalen Ablauf: `upstream-stable` ist ein Release-Zeiger, kein Entwicklungsbranch. Niemals force-pushen; bei nicht vorziehbarer Historie Ursache klären.

7. Eigenen Release-Kandidaten aus `main` bauen und ausdrücklich zur Installation freigeben.

## CI: prüfen und bauen, nicht installieren

`.github/workflows/fork-ci.yml` läuft auf PRs und Pushes nach `main`; ein manueller Lauf ist nur auf `main` zulässig. Berechtigungen: `contents: read`, kein Deployment-/Publish-Token.

- Node 24.22.2, Installation mit expliziter npm-12-Scriptfreigabe.
- Dependencies in Root, `web` und `web-mobile` vor den Tests installieren; `ripgrep` für die Dateitests bereitstellen.
- `npm run typecheck`, `npm test`, beide Webclients direkt bauen, anschließend `npm pack --ignore-scripts`. Der redundante `prepack`-Aufruf würde erneut verschachtelte `npm ci` starten und kann wegen npm 12s vererbter Scriptfreigabe scheitern; keine Tests oder eigentlichen Builds werden ausgelassen.
- Linux-x86_64-Paket, `BUILD-INFO.txt`, `fork-upstream.json` und `SHA256SUMS` als Actions-Artefakt, 14 Tage Aufbewahrung.
- Automatische Läufe: Version `<basis>-somora.ci.<commit>`; **keine freigegebenen Releases**.
- Manueller Lauf mit `fork_revision=N`: Version `<basis>-somora.N`, beispielsweise `2026.1106.3-somora.1`. Paketversion und Lockfile werden nur im Build-Workspace entsprechend gesetzt.

Der geerbte Discord-Workflow ist auf `thenaxon/somora_agent` beschränkt. Neue Upstream-Workflows vor jedem Merge prüfen. Dependabot darf Vorschläge machen, aber nichts automatisch mergen. CI-Erfolg ersetzt weder Browser- noch Live-Integrationstests.

## Eigenen Release veröffentlichen

**Bewusster manueller Schritt; der Workflow veröffentlicht und installiert nichts.**

1. Gewünschten `main`-Commit festhalten und neue, noch nie veröffentlichte Fork-Revision wählen.
2. `gh workflow run fork-ci.yml --repo packetloss99/somora_agent --ref main -f fork_revision=1` ausführen; tatsächliche neue Revision einsetzen. Run und dessen Commit prüfen, erfolgreichen Abschluss abwarten.
3. Artefakt dieses Runs herunterladen, beispielsweise mit `gh run download <run-id> --repo packetloss99/somora_agent --dir <leeres-verzeichnis>`. Im entpackten Artefaktordner `sha256sum -c SHA256SUMS` ausführen. Version und Commit in `BUILD-INFO.txt` prüfen.
4. Release-Tag **auf diesen gebauten Commit** setzen, nicht auf einen inzwischen weitergewanderten Branch:

   ```bash
   git tag -a v2026.1106.3-somora.1 <gebauter-commit> -m 'Somora fork release'
   git push origin refs/tags/v2026.1106.3-somora.1
   gh release create v2026.1106.3-somora.1 <artefaktordner>/* \
     --repo packetloss99/somora_agent --verify-tag \
     --title 'Somora 2026.1106.3-somora.1' --notes-file <release-notizen>
   ```

   Werte sind Beispiele. Keine Original-Tags umbenennen/überschreiben. Release Notes: Original-Basis, eigene Änderungen, getestete Plattform, genaue Prüfungen, bekannte Einschränkungen und Rückweg. Nie automatisch auf npm veröffentlichen.

Releases bewahren Artefakte dauerhaft auf; Actions-Artefakte laufen ab. Keine bestehenden Release-Assets austauschen, sondern eine neue Fork-Revision veröffentlichen.

## Installation und Rückweg

Nur nach gesonderter Freigabe auf dem Zielhost:

- Exaktes Release-Paket und Prüfsumme herunterladen/verifizieren. Bestehende Installation und benötigte Daten sichern; ein VM-Snapshot-Rollback verliert spätere Daten.
- Paket am vereinbarten Pfad installieren; Produktionsdaten unter `~/.somora` weder ersetzen noch ins Repo kopieren. Dienstneustart mit laufenden Anfragen abstimmen.
- SQLite/Health, Web/Mobile, TUI-Session, Herdr-Status, `agent prompt --wait`, Rückfragen und Exit prüfen. Automatischen Session-Restore separat testen, nicht allein aus einem gespeicherten Startbefehl ableiten.
- Bei Fehlern vorheriges geprüftes Paket wieder installieren und Dienst neu starten; Daten nur bewusst zurücksetzen.

**Kein reguläres `somora update` für diese Installation:** Das folgt dem offiziellen npm-Paket und kann unseren Patch überschreiben. Der Updateweg ist Original-Release → Update-PR → geprüfter Fork-Release → freigegebene Installation. Auto-Update-Jobs auf dem Zielhost dürfen nicht auf das Original zeigen.

Bekannte offene Punkte: alte Rückfrage nach `/reset`, vollständiger Restore-Test, Verlust der selbst gemeldeten Statusanmeldung bei Herdr-Live-Handoff. Die letzten beiden betreffen auch unseren separaten privaten Herdr-Fork; beide Repos unabhängig versionieren und die konkret getestete Kombination in den Release Notes festhalten.

## GitHub-Einstellungen

`main` soll PRs und den Check **somora-fork-check-build** verlangen; keine automatische Zusammenführung. Kein allgemeines `actions: write` oder `contents: write` für CI. Falls GitHub den Schutz wegen Tarif/Berechtigungen ablehnt, gilt dieser Ablauf organisatorisch weiter; fehlenden technischen Schutz ausdrücklich nennen, nicht als aktiviert behaupten.
