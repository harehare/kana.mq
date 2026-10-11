<h1 align="center">kana.mq</h1>

Fullwidth/halfwidth conversion as an [mq](https://github.com/harehare/mq) module.

`unicode_normalize(s, "nfkc")` already converts fullwidth to halfwidth, but it cannot go the other way and it cannot be limited to some character classes. It also expands symbols such as `㈱` into `(株)`. `kana.mq` converts in both directions, only the classes you pick, and leaves everything else alone.

## Installation

Copy `kana.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp kana.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

HTTP imports are disabled by default; pass `--allow-http-import` to import directly from GitHub:

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/kana.mq" | kana::to_fullwidth(., "all")' input.txt
```

## Usage

```sh
mq -L /path/to/modules -I raw 'import "kana" | kana::to_fullwidth(., "all")' input.txt
```

## API

### `to_fullwidth(s, kinds)`

Converts halfwidth characters in `s` to fullwidth.

### `to_halfwidth(s, kinds)`

Converts fullwidth characters in `s` to halfwidth.

### `kinds`

A single string:

| kinds | Converts |
|---|---|
| `"digit"` | `0-9` |
| `"alpha"` | `A-Z`, `a-z` |
| `"ascii"` | digits, letters, symbols (`U+0021-007E`), space, and `¢£¬¯¦¥₩` |
| `"kana"` | halfwidth katakana |
| `"all"` | `"ascii"` and `"kana"` |

Any other value raises an error. Characters outside the selected class, including hiragana and kanji, are returned unchanged. An empty string is returned as is.

## Examples

```sh
mq -L . -I null 'import "kana" | kana::to_fullwidth("ABC123 ｶﾞｷﾞ", "all")'
# => "ＡＢＣ１２３　ガギ"

mq -L . -I null 'import "kana" | kana::to_halfwidth("ＡＢＣ１２３　ガギ", "all")'
# => "ABC123 ｶﾞｷﾞ"

# Only digits
mq -L . -I null 'import "kana" | kana::to_fullwidth("a1 ｱ", "digit")'
# => "a１ ｱ"

# Only kana
mq -L . -I null 'import "kana" | kana::to_halfwidth("ＡＢＣ　ガ", "kana")'
# => "ＡＢＣ　ｶﾞ"

# No NFKC expansion
mq -L . -I null 'import "kana" | kana::to_halfwidth("㈱ＡＢＣ", "all")'
# => "㈱ABC"
```

## Conversion details

### ASCII

- `U+0021-007E` and `U+FF01-FF5E` map one to one (offset `0xFEE0`). `~` maps to `～` (`U+FF5E`), `\` to `＼`.
- `U+0020` and `U+3000` (ideographic space) map to each other.
- `¢£¬¯¦¥₩` map to `￠￡￢￣￤￥￦` (`U+FFE0-FFE6`).

### Kana

- Halfwidth katakana `U+FF61-FF9F` map to the corresponding fullwidth characters, including punctuation (`｡｢｣､･` and `。「」、・`) and the long vowel mark (`ｰ` and `ー`).
- `to_fullwidth` composes a voiced mark `ﾞ` or semi-voiced mark `ﾟ` into the preceding character: `ｶﾞ` to `ガ`, `ﾊﾟ` to `パ`, `ｳﾞ` to `ヴ`.
- `to_halfwidth` decomposes them: `ガ` to `ｶﾞ`.
- `ヷ` and `ヺ` are converted too, as `ﾜﾞ` and `ｦﾞ`. This matches NFKC.
- A mark with nothing to combine with (`ｱﾞ`, or one at the start of the string) becomes the standalone fullwidth marks `゛` (`U+309B`) and `゜` (`U+309C`), and the reverse. No character is dropped.
- Fullwidth kana with no halfwidth form are unchanged: `ヰ ヱ ヮ ヵ ヶ ヽ ヾ`.
- Halfwidth symbol forms `U+FFE8-FFEE` and fullwidth parentheses `⦅⦆` are not converted.

### Combining marks

Input that writes a voiced mark with a combining character (`U+3099`, `U+309A`) must be normalized first:

```sh
mq -L . -I null 'import "kana" | unicode_normalize("ガ", "nfc") | kana::to_halfwidth(., "kana")'
# => "ｶﾞ"
```

### Markdown

The functions only transform strings and know nothing about Markdown structure. Converting fullwidth syntax characters such as `＊` or `＃` to halfwidth can change how the result is parsed when it is read back as Markdown:

```sh
mq -L . -I null 'import "kana" | kana::to_halfwidth("＊強調＊", "ascii")'
# => "*強調*"
```

## Tests

```sh
mq-test kana_test.mq
```

## Compatibility

Tested with mq 0.9.2. The module has no dependency on other modules. The tests use the standard `test` module and the `unicode_normalize` builtin.

## License

MIT
