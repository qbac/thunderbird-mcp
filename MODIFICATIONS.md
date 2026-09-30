# Modyfikacje lokalne — Jakub Cyrklaf

To **nie jest oryginalna wersja** dodatku. Oryginał: [TKasperczyk/thunderbird-mcp](https://github.com/TKasperczyk/thunderbird-mcp)
(autor: Tomasz Kasperczyk, licencja MIT — patrz [`LICENSE`](LICENSE), zachowana bez zmian).
Ta kopia zawiera zmiany wprowadzone przez Jakuba Cyrklafa.

- **Wersja:** `0.7.5.5` = upstream `0.7.5` + modyfikacje poniżej (czwarta cyfra = numer lokalnej modyfikacji).
- **Nazwa w Thunderbirdzie:** „Thunderbird MCP (mod. Jakub Cyrklaf)”.
- **ID dodatku bez zmian** (`thunderbird-mcp@tkasperczyk.dev`) — dzięki temu instalacja zastępuje poprzednią wersję, a ustawienia (token, dostęp do kont) zostają.
- **Gałąź:** lokalnie `feat/reply-save-draft` (na bazie `origin/main`), na GitHubie [`qbac/thunderbird-mcp` → `cyrklaf-mod`](https://github.com/qbac/thunderbird-mcp/tree/cyrklaf-mod).
- **Zgłoszone do autora:**
  - `saveAsDraft` → PR [#208](https://github.com/TKasperczyk/thunderbird-mcp/pull/208) (zamyka zgłoszenie #207; gałąź `feat/reply-save-as-draft` w forku).
  - Poprawka Windows → już zgłoszona przez kogoś innego jako PR [#206](https://github.com/TKasperczyk/thunderbird-mcp/pull/206) (nie dublujemy).
  - Podpis przy `saveDraft`/`sendMail` → cudzy PR [#168](https://github.com/TKasperczyk/thunderbird-mcp/pull/168) (autor: mwatola-glitch), wciągnięty tutaj z jedną zmianą (bez separatora „-- ”).
  - Cudze PR-y wciągnięte bez zmian: [#186](https://github.com/TKasperczyk/thunderbird-mcp/pull/186) (`getMessage` + `bodyFormat`), [#194](https://github.com/TKasperczyk/thunderbird-mcp/pull/194) (`saveDraft` podmienia szkic), [#174](https://github.com/TKasperczyk/thunderbird-mcp/pull/174) (`ccList` w `getRecentMessages`).
  - `inlineImages` → PR [#230](https://github.com/TKasperczyk/thunderbird-mcp/pull/230) (gałąź `feat/inline-images` w forku, na bazie upstream 0.8.0 — bez `saveAsDraft`/`subject`/`msgToReplace`, których upstream nie ma).
  - Gdy wszystkie zostaną przyjęte i wydane, można wrócić do oryginalnej wtyczki z auto-aktualizacją (zainstalować XPI autora; zmienią się nazwa i wersja, ID zostaje to samo).

## Lista zmian

| Data | Zmiana | Plik |
|---|---|---|
| 2026-09-11 | `replyToMessage` — nowy parametr `saveAsDraft`: odpowiedź budowana w natywnym oknie Thunderbirda (cytat, podpis tożsamości, nagłówki wątku), zapisywana do Wersji roboczych, okno zamykane bez wysyłania. Nie łączy się ze `skipReview`. | `extension/mcp_server/api.js` |
| 2026-09-11 | Pominięty test uprawnień katalogu tymczasowego na Windows — `nsIFile.permissions` jest tam syntetyczne i zawsze dawało fałszywy alarm, blokując zapis `connection.json` (łatka przeniesiona z wcześniejszego lokalnego buildu 0.7.4). | `extension/mcp_server/api.js` |
| 2026-09-11 | Usunięty `update_url` z manifestu — auto-aktualizacja z serwera autora nadpisałaby lokalne zmiany. | `extension/manifest.json` |
| 2026-09-11 | Oznaczenie wersji zmodyfikowanej: numer `0.7.5.1`, nazwa, opis i autor w manifeście, ten plik. | `package.json`, `extension/manifest.json` |
| 2026-09-11 | **0.7.5.2** — `saveDraft` i `sendMail` ze `skipReview` doklejają podpis tożsamości (plik podpisu przy „dołącz podpis z pliku” albo tekst podpisu HTML) na końcu treści. Kod z cudzego PR [#168](https://github.com/TKasperczyk/thunderbird-mcp/pull/168) (commit z zachowanym autorem), plus lokalna zmiana: **bez separatora „-- ”** przed podpisem. `replyToMessage`/`forwardMessage` ze `skipReview` nadal bez podpisu (poza zakresem #168); w trybie okna podpis wstawia sam Thunderbird. | `extension/mcp_server/api.js` |
| 2026-09-11 | **0.7.5.3** — wciągnięte cudze PR-y (commity z zachowanymi autorami): [#186](https://github.com/TKasperczyk/thunderbird-mcp/pull/186) `getMessage` z `bodyFormat: "html"`/`"markdown"` zwraca część HTML zamiast po cichu tekstu; [#194](https://github.com/TKasperczyk/thunderbird-mcp/pull/194) `saveDraft` z `replaceMessageId` + `replaceFolderPath` podmienia istniejący szkic zamiast dokładać kopię; [#174](https://github.com/TKasperczyk/thunderbird-mcp/pull/174) `getRecentMessages` zwraca `ccList`. | `extension/mcp_server/api.js`, `test/` |
| 2026-09-11 | **0.7.5.3** — własne: `replyToMessage` ma parametr `subject` (nadpisuje „Re: …”, nagłówki wątku zostają; działa z oknem, `saveAsDraft` i `skipReview`). Znacznik `<!--signature-->` w `body` dla `saveDraft`/`sendMail` ustawia miejsce podpisu (np. między wstępem a przekazaną treścią); bez znacznika podpis idzie na koniec. | `extension/mcp_server/api.js` |
| 2026-09-12 | **0.7.5.4** — poprawka do #194: na IMAP (TB ESR, nazwa.pl) mechanizm `msgToReplace` zapisywał nowy szkic, ale stary zostawał w Wersjach roboczych (mimo komunikatu „Draft replaced”). Po udanym zapisie stary szkic jest teraz jawnie przenoszony do Kosza (ta sama ścieżka co `deleteMessages`); ewentualny błąd w polu `oldDraftRemoveError`. | `extension/mcp_server/api.js` |
| 2026-09-30 | **0.7.5.5** — nowy parametr `inlineImages` w `saveDraft`, `sendMail` i `replyToMessage`: obrazy osadzane **w treści** jako części `multipart/related` z `Content-ID` i `Content-Disposition: inline` (nie jako załączniki), referencjonowane w body przez `<img src="cid:…">`. Element: `{cid?, name?, contentType?, path \| base64}` — zalecane `path` (plik czyta Thunderbird, dane obrazu nie przechodzą przez wywołanie narzędzia). Wymaga `isHtml: true`; każdy obraz musi mieć referencję `cid:` w body, inaczej całe wywołanie jest odrzucane (bez cichego gubienia obrazków). Ścieżki bez okna (`saveDraft`, `skipReview`): instancja JS `MessageSend` (TB 128+) z podstawionymi częściami zamiast skanu edytora — `MimeMessage` buduje `multipart/related` tak jak dla obrazka wklejonego ręcznie. Ścieżki z oknem (okno kompozycji, `replyToMessage` + `saveAsDraft`): `cid:` podmieniane na `data:` URL, które edytor Thunderbirda sam zamienia na części `cid:` przy zapisie/wysłaniu. `forwardMessage` — bez zmian. Testy: `test/outbound-inline-images.test.cjs`. **Potwierdzone na żywo 2026-09-30** (TB 153, IMAP): `saveDraft` z `path` → `getMessage` pokazuje `attachments: []` i `<img src="imap://…&part=1.2&filename=…">`, czyli tę samą strukturę co obraz wklejony ręcznie w oknie kompozycji (sprawdzona ścieżka bez okna). Pierwszy test ścieżki z oknem (`replyToMessage` + `saveAsDraft`) padł na `ReferenceError: btoa is not defined` — w zakresie eksperymentu TB nie ma globalnego `btoa`; zamienione na ręczny enkoder `encodeBytesToBase64` (z testem). Po poprawce **potwierdzone na żywo** także `replyToMessage` + `saveAsDraft`: „Reply saved as draft”, `attachments: []`, obraz jako `part=1.2` w miejscu `cid:`, podpis HTML i cytat z wątkiem na miejscu. Niesprawdzone na żywo: okno kompozycji `sendMail` i `skipReview` (wysyłka). Brak podobnego PR/issue upstream (#31/#30 dotyczą tylko odczytu) — kandydat do zgłoszenia. | `extension/mcp_server/api.js`, `test/` |

## Aktualizacja do nowszej wersji autora

1. `git fetch origin && git rebase origin/main` (na gałęzi `feat/reply-save-draft`).
2. Ustaw wersję `<nowa wersja upstream>.1` w `package.json` (manifest dostaje ją automatycznie przy buildzie).
3. Sprawdź, czy `update_url` nie wrócił do `extension/manifest.json`.
4. `node scripts/build-xpi.cjs` (nie `scripts/build.sh` — w Git Bash na Windows psuje ścieżki).
5. Zainstaluj `dist/thunderbird-mcp.xpi` w Thunderbirdzie (Dodatki → ⚙ → Zainstaluj dodatek z pliku), zrestartuj Thunderbirda i Claude Code.
6. Dopisz zmianę do tabeli wyżej.

Nie instaluj gotowego XPI z GitHuba autora — straci lokalne zmiany.
