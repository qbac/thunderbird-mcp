# Modyfikacje lokalne — Jakub Cyrklaf

To **nie jest oryginalna wersja** dodatku. Oryginał: [TKasperczyk/thunderbird-mcp](https://github.com/TKasperczyk/thunderbird-mcp)
(autor: Tomasz Kasperczyk, licencja MIT — patrz [`LICENSE`](LICENSE), zachowana bez zmian).
Ta kopia zawiera zmiany wprowadzone przez Jakuba Cyrklafa.

- **Wersja:** `0.7.5.1` = upstream `0.7.5` + modyfikacje poniżej (czwarta cyfra = numer lokalnej modyfikacji).
- **Nazwa w Thunderbirdzie:** „Thunderbird MCP (mod. Jakub Cyrklaf)”.
- **ID dodatku bez zmian** (`thunderbird-mcp@tkasperczyk.dev`) — dzięki temu instalacja zastępuje poprzednią wersję, a ustawienia (token, dostęp do kont) zostają.
- **Gałąź:** lokalnie `feat/reply-save-draft` (na bazie `origin/main`), na GitHubie [`qbac/thunderbird-mcp` → `cyrklaf-mod`](https://github.com/qbac/thunderbird-mcp/tree/cyrklaf-mod).
- **Zgłoszone do autora:**
  - `saveAsDraft` → PR [#208](https://github.com/TKasperczyk/thunderbird-mcp/pull/208) (zamyka zgłoszenie #207; gałąź `feat/reply-save-as-draft` w forku).
  - Poprawka Windows → już zgłoszona przez kogoś innego jako PR [#206](https://github.com/TKasperczyk/thunderbird-mcp/pull/206) (nie dublujemy).
  - Gdy oba zostaną przyjęte i wydane, można wrócić do oryginalnej wtyczki z auto-aktualizacją (zainstalować XPI autora; zmienią się nazwa i wersja, ID zostaje to samo).

## Lista zmian

| Data | Zmiana | Plik |
|---|---|---|
| 2026-09-11 | `replyToMessage` — nowy parametr `saveAsDraft`: odpowiedź budowana w natywnym oknie Thunderbirda (cytat, podpis tożsamości, nagłówki wątku), zapisywana do Wersji roboczych, okno zamykane bez wysyłania. Nie łączy się ze `skipReview`. | `extension/mcp_server/api.js` |
| 2026-09-11 | Pominięty test uprawnień katalogu tymczasowego na Windows — `nsIFile.permissions` jest tam syntetyczne i zawsze dawało fałszywy alarm, blokując zapis `connection.json` (łatka przeniesiona z wcześniejszego lokalnego buildu 0.7.4). | `extension/mcp_server/api.js` |
| 2026-09-11 | Usunięty `update_url` z manifestu — auto-aktualizacja z serwera autora nadpisałaby lokalne zmiany. | `extension/manifest.json` |
| 2026-09-11 | Oznaczenie wersji zmodyfikowanej: numer `0.7.5.1`, nazwa, opis i autor w manifeście, ten plik. | `package.json`, `extension/manifest.json` |

## Aktualizacja do nowszej wersji autora

1. `git fetch origin && git rebase origin/main` (na gałęzi `feat/reply-save-draft`).
2. Ustaw wersję `<nowa wersja upstream>.1` w `package.json` (manifest dostaje ją automatycznie przy buildzie).
3. Sprawdź, czy `update_url` nie wrócił do `extension/manifest.json`.
4. `node scripts/build-xpi.cjs` (nie `scripts/build.sh` — w Git Bash na Windows psuje ścieżki).
5. Zainstaluj `dist/thunderbird-mcp.xpi` w Thunderbirdzie (Dodatki → ⚙ → Zainstaluj dodatek z pliku), zrestartuj Thunderbirda i Claude Code.
6. Dopisz zmianę do tabeli wyżej.

Nie instaluj gotowego XPI z GitHuba autora — straci lokalne zmiany.
