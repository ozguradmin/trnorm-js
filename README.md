# trnorm-js

**Turkish text normalization for speech, in JavaScript.** Turns written Turkish into the way it is spoken (for
text-to-speech), gives a canonical form for scoring speech recognition (WER/CER), and turns spoken numbers back into
digits. A port of the Python [trnorm](https://github.com/ozguradmin/ahenk-turkish-tts/tree/main/src/trnorm) with
**the same output for the same input**. No dependencies; runs in browsers and Node 18+.

```sh
npm install @ozgurguler/trnorm
```

```js
import { normalize, normalize_for_eval, inverse_normalize, number_to_words } from '@ozgurguler/trnorm';

normalize("Saat 09:30'da 3. kata gelin, ücret 1.250,50 TL.");
// "Saat dokuz otuzda üçüncü kata gelin, ücret bin iki yüz elli lira elli kuruş."

normalize_for_eval("ABD'de 5 TL mi?");                            // "a be dede beş lira mı"
inverse_normalize('saat on dört otuzda yüzde on beşlik indirim'); // "saat 14.30'da %15'lik indirim"
number_to_words(1923);                                            // "bin dokuz yüz yirmi üç"
```

## What it reads

Numbers, dates, times, money, percentages, units, ordinals, abbreviations, acronyms, phone numbers, IBANs, licence
plates, URLs and e-mail addresses, foreign names. Turkish suffixes are re-agreed with the spoken form, which is the
hard part a TTS model gets wrong:

| Written | Spoken (`normalize`) |
|---|---|
| `15.00'de` | on beşte |
| `%15'lik` | yüzde on beşlik |
| `4'üncü`, `3. kat` | dördüncü, üçüncü kat |
| `1.250,50 TL` | bin iki yüz elli lira elli kuruş |
| `5 TL mi?` | beş lira mı? |
| `14.10.2026` | on dört Ekim iki bin yirmi altı |
| `5 km/sa` | saatte beş kilometre |
| `SGK'ye` | se ge kaya |
| `Dr. Yılmaz` | doktor Yılmaz |
| `iPhone 15` | ayfon on beş |
| `0555 555 55 55` | sıfır beş yüz elli beş, beş yüz elli beş, elli beş, elli beş |
| `34 ABC 123` | otuz dört a be ce yüz yirmi üç |

The rules and the reasons behind each choice (in Turkish):
[docs/TRNORM.md](https://github.com/ozguradmin/ahenk-turkish-tts/blob/main/docs/TRNORM.md).

## API

| Function | What it does |
|---|---|
| `normalize(text)` | Written → spoken form, for TTS input. Case, punctuation and apostrophes on proper nouns are kept (models use them for prosody). |
| `normalize_for_eval(text)` | Canonical form for WER/CER: spoken form, Turkish lowercase, no punctuation, "buçuk" = "virgül beş". Apply it to both the reference and the hypothesis. |
| `inverse_normalize(text)` | Spoken → written with digits, for ASR output. Small numbers standing alone stay words ("üç kişi"), as Turkish style guides recommend. |
| `number_to_words(n)`, `ordinal_to_words(n)` | `4` → "dört", "dördüncü". `n` is a bigint or an integer; any other number is read as the Python float it is. |
| `tr_lower(text)`, `tr_upper(text)` | Turkish-aware case (I ↔ ı, İ ↔ i). |

TypeScript types are included. Where the Python raises, this throws (for example a `ValueError` for a number of more
than 4300 digits).

## Is it really the same as the Python?

That is the point of it. The port reproduces Python's `str`, `re` and `int` behaviour where JavaScript differs
(`src/py.js`): indexing by code point, Python's Unicode `\w \d \s \b`, `$` also matching before a final newline,
unbounded integers (BigInt), Unicode 15.0 case tables (`src/case_tables.js`), characters newer than Python 3.12's
Unicode (`src/unassigned.js`), and lone surrogates.

- `npm test` runs 7,566 texts and 2,505 numbers from `test/golden.json` through every function. `tools/golden.py`
  writes that file from the Python trnorm in [ahenk-turkish-tts](https://github.com/ozguradmin/ahenk-turkish-tts);
  after a change on the Python side:

  ```sh
  git clone --depth 1 https://github.com/ozguradmin/ahenk-turkish-tts ../ahenk-turkish-tts
  npm run golden   # needs Python 3.12
  npm test
  ```
- A wider parity harness lives in
  [ahenk-cep-pdfreader](https://github.com/ozguradmin/ahenk-cep-pdfreader/tree/main/tools/parity):
  test suites, real documents and generated hostile input (huge numbers, invisible characters, combining marks,
  emoji, mixed scripts, lone surrogates, Unicode 16 characters). Its last run matched Python 3.12 on all 742,716
  results. The tests pass in V8 (Node, Chrome) and JavaScriptCore (Safari). This library is the in-browser text
  front-end of that site, [Ahenk Okur](https://ozguradmin.github.io/ahenk-cep-pdfreader/), which reads PDFs aloud
  with the Ahenk Cep voice.

**Safari 26.0–26.5:** their RegExp engine reads a character in U+F800–U+FBFF (or a surrogate) followed by one in
U+FC00–U+FFFF (such as `ﬁ５`) as a single character. For a text with such a pair, the functions throw an error with
`name === 'EngineError'` rather than give a wrong answer; send that text to a server or to the Python instead.

Requirements: a JavaScript engine with ES2022 and Unicode property escapes (Node 18+, current browsers, Safari 16.4+).

## Türkçe

Yazılı Türkçeyi okunuş formuna çeviren [trnorm](https://github.com/ozguradmin/ahenk-turkish-tts/tree/main/src/trnorm)'un
JavaScript sürümü: tarayıcıda ve Node'da, bağımlılıksız, Python'la **aynı girdiye aynı çıktıyı** verir. Sayı, tarih,
saat, para, yüzde, birim, kısaltma, telefon, IBAN, plaka, URL okunur; ekler okunuşa uyar (`15.00'de` → "on beşte",
`SGK'ye` → "se ge kaya").

- `normalize`: yazılı metin → okunuş (TTS girdisi); büyük/küçük harf ve noktalama korunur.
- `normalize_for_eval`: WER/CER ölçümü için kanonik biçim; referansa ve çıktıya aynen uygulanır.
- `inverse_normalize`: okunuş → rakamlı yazı (ASR çıktısı için).
- `number_to_words`, `ordinal_to_words`, `tr_lower`, `tr_upper`.

`npm test`, Python trnorm'un 7.566 metin ve 2.505 sayıdaki çıktısıyla karşılaştırır; daha geniş eşitlik testinde
742.716 sonucun hepsi Python 3.12'yle aynı çıktı. Kurallar ve tercihler:
[docs/TRNORM.md](https://github.com/ozguradmin/ahenk-turkish-tts/blob/main/docs/TRNORM.md).

## License

Apache-2.0. The Python trnorm, its rules and the Ahenk speech models:
[ozguradmin/ahenk-turkish-tts](https://github.com/ozguradmin/ahenk-turkish-tts).
