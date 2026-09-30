# Modyfikacje lokalne — Jakub Cyrklaf

To **nie jest oryginalna wersja** dodatku. Oryginał: [TKasperczyk/thunderbird-mcp](https://github.com/TKasperczyk/thunderbird-mcp)
(autor: Tomasz Kasperczyk, licencja MIT — patrz [`LICENSE`](LICENSE), zachowana bez zmian).
Ta kopia zawiera zmiany wprowadzone przez Jakuba Cyrklafa.

- **Wersja:** `0.8.0.1` = upstream `0.8.0` + modyfikacje poniżej (czwarta cyfra = numer lokalnej modyfikacji).
- **Nazwa w Thunderbirdzie:** „Thunderbird MCP (mod. Jakub Cyrklaf)”.
- **ID dodatku bez zmian** (`thunderbird-mcp@tkasperczyk.dev`) — dzięki temu instalacja zastępuje poprzednią wersję, a ustawienia (token, dostęp do kont) zostają.
- **Gałąź:** lokalnie `cyrklaf-0.8` (na bazie `origin/main` = 0.8.0), na GitHubie [`qbac/thunderbird-mcp` → `cyrklaf-0.8`](https://github.com/qbac/thunderbird-mcp/tree/cyrklaf-0.8).
  Poprzednia linia 0.7.5.x zostaje jako kopia zapasowa: lokalnie `feat/reply-save-draft`, na GitHubie `cyrklaf-mod`.
- **Już u autora w 0.8.0** (lokalne łatki niepotrzebne): [#186](https://github.com/TKasperczyk/thunderbird-mcp/pull/186) (`getMessage` + `bodyFormat`), [#174](https://github.com/TKasperczyk/thunderbird-mcp/pull/174) (`ccList` w `getRecentMessages`), [#209](https://github.com/TKasperczyk/thunderbird-mcp/pull/209) (test uprawnień katalogu tymczasowego na Windows).
- **Zgłoszone do autora, jeszcze nieprzyjęte:**
  - `saveAsDraft` → PR [#208](https://github.com/TKasperczyk/thunderbird-mcp/pull/208) (zamyka zgłoszenie #207; gałąź `feat/reply-save-as-draft` w forku).
  - Podpis przy `saveDraft`/`sendMail` → cudzy PR [#168](https://github.com/TKasperczyk/thunderbird-mcp/pull/168) (autor: mwatola-glitch), wciągnięty tutaj z jedną zmianą (bez separatora „-- ”).
  - Podmiana szkicu (`replaceMessageId`) → cudzy PR [#194](https://github.com/TKasperczyk/thunderbird-mcp/pull/194), wciągnięty bez zmian + lokalna poprawka IMAP (niżej).
  - `inlineImages` → PR [#230](https://github.com/TKasperczyk/thunderbird-mcp/pull/230) (gałąź `feat/inline-images` w forku).
  - Gdy wszystkie zostaną przyjęte i wydane, można wrócić do oryginalnej wtyczki z auto-aktualizacją (zainstalować XPI autora; zmienią się nazwa i wersja, ID zostaje to samo).

## Lista zmian (stan w 0.8.0.1)

| Zmiana | Pochodzenie | Plik |
|---|---|---|
| `replyToMessage` — parametr `saveAsDraft`: odpowiedź budowana w natywnym oknie Thunderbirda (cytat, podpis tożsamości, nagłówki wątku), zapisywana do Wersji roboczych, okno zamykane bez wysyłania. Nie łączy się ze `skipReview`. | własne, 0.7.5.1 (PR #208) | `extension/mcp_server/api.js` |
| Usunięty `update_url` z manifestu — auto-aktualizacja z serwera autora nadpisałaby lokalne zmiany. **Wrócił w 0.8.0 — sprawdzać przy każdej aktualizacji.** | własne, 0.7.5.1 | `extension/manifest.json` |
| Oznaczenie wersji zmodyfikowanej: numer, nazwa, opis i autor w manifeście, ten plik. | własne | `package.json`, `extension/manifest.json` |
| `saveDraft` i `sendMail` ze `skipReview` doklejają podpis tożsamości (plik podpisu albo tekst podpisu HTML), **bez separatora „-- ”**. `replyToMessage`/`forwardMessage` ze `skipReview` nadal bez podpisu; w trybie okna podpis wstawia sam Thunderbird. | PR #168 + zmiana lokalna, 0.7.5.2 | `extension/mcp_server/api.js` |
| `saveDraft` z `replaceMessageId` + `replaceFolderPath` podmienia istniejący szkic zamiast dokładać kopię. | PR #194, 0.7.5.3 | `extension/mcp_server/api.js`, `test/save-draft-replace.test.cjs` |
| `replyToMessage` — parametr `subject` (nadpisuje „Re: …”, nagłówki wątku zostają; działa z oknem, `saveAsDraft` i `skipReview`). Znacznik `<!--signature-->` w `body` dla `saveDraft`/`sendMail` ustawia miejsce podpisu; bez znacznika podpis idzie na koniec. | własne, 0.7.5.3 | `extension/mcp_server/api.js` |
| Poprawka do #194: na IMAP (TB ESR, nazwa.pl) `msgToReplace` zapisywał nowy szkic, ale stary zostawał w Wersjach roboczych. Po udanym zapisie stary szkic jest jawnie przenoszony do Kosza (ścieżka `deleteMessages`); ewentualny błąd w polu `oldDraftRemoveError`. | własne, 0.7.5.4 | `extension/mcp_server/api.js` |
| Parametr `inlineImages` w `saveDraft`, `sendMail` i `replyToMessage`: obrazy osadzane **w treści** jako części `multipart/related` z `Content-ID` i `Content-Disposition: inline` (nie jako załączniki), referencjonowane w body przez `<img src="cid:…">`. Element: `{cid?, name?, contentType?, path \| base64}` — zalecane `path` (plik czyta Thunderbird, dane obrazu nie przechodzą przez wywołanie narzędzia). Wymaga `isHtml: true`; każdy obraz musi mieć referencję `cid:` w body, inaczej całe wywołanie jest odrzucane. Ścieżki bez okna (`saveDraft`, `skipReview`): instancja JS `MessageSend` (TB 128+) z podstawionymi częściami zamiast skanu edytora. Ścieżki z oknem (okno kompozycji, `replyToMessage` + `saveAsDraft`): `cid:` podmieniane na `data:` URL, które edytor Thunderbirda sam zamienia na części `cid:`. `forwardMessage` — bez zmian. | własne, 0.7.5.5 (PR #230) | `extension/mcp_server/api.js`, `test/outbound-inline-images.test.cjs`, `test/privacy-messages.test.cjs` |

### Historia i weryfikacja

- **2026-09-30 — 0.7.5.5:** `inlineImages` potwierdzone na żywo (TB 153, IMAP) dla `saveDraft` i `replyToMessage` + `saveAsDraft`: `getMessage` pokazuje `attachments: []` i `<img src="imap://…&part=1.2&filename=…">`, tak jak obraz wklejony ręcznie; podpis i cytat na miejscu. Pierwszy test ścieżki z oknem padł na `ReferenceError: btoa is not defined` (w 0.7.5 Experiment nie ma globalnego `btoa`) — w 0.7.5.5 ręczny enkoder. Niesprawdzone na żywo: okno kompozycji `sendMail` i `skipReview` (wysyłka).
- **2026-09-30 — 0.8.0.1:** przeniesienie wszystkich lokalnych zmian na upstream 0.8.0. 0.8.0 importuje `DOMParser`/`atob`/`btoa`/`TextDecoder` do Experimentu (w 0.7.5 m.in. `bodyFormat: "markdown"` cicho nie działał), więc `inlineImages` używa już zwykłego `btoa`. Dopasowania: poprawka Windows (#209) i `bodyFormat`/`ccList` wzięte z upstreamu; w `saveDraft` podpis budowany dopiero po weryfikacji załączników (0.8.0 odrzuca niebezpieczne załączniki przed jakąkolwiek pracą nad treścią). Testy: wszystkie przechodzą poza błędami środowiskowymi Windows w `mcp-bridge` (występują też na czystym 0.8.0). **Potwierdzone na żywo 2026-09-30** (TB 153, IMAP, tylko szkice): `saveDraft` + `inlineImages` (obraz `part=1.2`, `attachments: []`, podpis na końcu); znacznik `<!--signature-->` (podpis w miejscu znacznika); podmiana szkicu (jeden szkic w Drafts, stary w Koszu, bez `oldDraftRemoveError`); `replyToMessage` + `saveAsDraft` + `subject` + `inlineImages` (własny temat, cytat z wątkiem, podpis, obraz); `getMessage(bodyFormat: "markdown")` na prawdziwym mailu HTML zwraca poprawny Markdown.

## Aktualizacja do nowszej wersji autora

1. `git fetch origin`, nowa gałąź od `origin/main`, przeniesienie lokalnych commitów (`git cherry-pick`) — albo `git rebase origin/main` na gałęzi `cyrklaf-0.8`.
2. Ustaw wersję `<nowa wersja upstream>.1` w `package.json` i `extension/manifest.json`.
3. Sprawdź, czy `update_url` nie wrócił do `extension/manifest.json` (wrócił w 0.8.0).
4. Sprawdź tabelę wyżej: co autor przyjął, tę łatkę usuń.
5. `node scripts/build-xpi.cjs` (nie `scripts/build.sh` — w Git Bash na Windows psuje ścieżki).
6. Zainstaluj `dist/thunderbird-mcp.xpi` w Thunderbirdzie (Dodatki → ⚙ → Zainstaluj dodatek z pliku), zrestartuj Thunderbirda i Claude Code.
7. Dopisz zmianę do historii wyżej.

Nie instaluj gotowego XPI z GitHuba autora — straci lokalne zmiany.
