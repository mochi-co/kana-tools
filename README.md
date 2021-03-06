# Japanese Kana Tools
### A Go/Golang package for Wapuro Romaji, Katakana, and Hiragana Detection and Conversion

![kana-tools](https://github.com/mochi-co/kana-tools/actions/workflows/build.yml/badge.svg) 
[![Coverage Status](https://coveralls.io/repos/github/mochi-co/kana-tools/badge.svg?branch=main)](https://coveralls.io/github/mochi-co/kana-tools?branch=main)
[![Go Report Card](https://goreportcard.com/badge/github.com/mochi-co/kana-tools)](https://goreportcard.com/report/github.com/mochi-co/kana-tools)
[![Go Reference](https://pkg.go.dev/badge/github.com/mochi-co/kana-tools.svg)](https://pkg.go.dev/github.com/mochi-co/kana-tools)

Kana Tools provides Romaji ←→ Kana transliteration based on [Wāpuro rōmaji (ワープロローマ字)](https://en.wikipedia.org/wiki/Wāpuro_rōmaji) romanization.

Where possible, the library uses a static rather than computational approach in order to perform conversions, relying on order-of-operations to ensure the correct output and provide a higher degree of wapuro conformity and maintainability.


### Usage
```go
import "github.com/mochi-co/kana-tools"
```

```go
// Convert Hiragana and Katakana to Romaji
// ToRomaji(s string, phonetic bool) string
kana.ToRomaji("ひらがな", false) // -> "hiragana"
kana.ToRomaji("カタカナ", false) // -> "katakana"
kana.ToRomaji("ひらがな and カタカナ", false) // -> "hiragana and katakana"
```

```go
// Convert Hiragana and Katakana to Cased Romaji
// ToRomajiCased(s string, phonetic bool) string
kana.ToRomajiCased("ひらがな", false) // -> "hiragana"
kana.ToRomajiCased("カタカナ", false) // -> "KATAKANA"
kana.ToRomajiCased("ひらがな and カタカナ", false) // -> "hiragana and KATAKANA"
```

```go
// The 2nd parameter, `phonetic bool` indicates if the the output is phonetic or wapuro literal.
// See the 'Phonetic vs Unphonetic Romaji' section below.
kana.ToRomaji("つづく", false) // -> "tsuduku"
kana.ToRomaji("つづく", true) // -> "tsuzuku" (phonetic)

kana.ToRomaji("まぢか", false) // -> "madika"
kana.ToRomaji("まぢか", true) // -> "majika" (phonetic)

kana.ToRomaji("マッチャ", false) // -> "maccha"
kana.ToRomaji("マッチャ", true) // -> "matcha" (phonetic)
```

```go
// Convert Romaji and Katakana to Hiragana
kana.ToHiragana("hiragana") // -> "ひらがな"
kana.ToHiragana("hiragana + カタカナ") // -> "ひらがな + かたかな"
```

```go
// Convert Romaji and Hiragana to Katakana
kana.ToKatakana("katakana") // -> "カタカナ"
kana.ToKatakana("katakana + ひらがな") // -> "カタカナ + ヒラガナ"
```

```go
// Convert Romaji to Hiragana and Katakana (case sensitive romaji)
kana.ToKana("hiragana + KATAKANA") // -> "ひらがな + カタカナ"
```

```go
// String IS Hiragana
kana.IsHiragana("たべる") // -> true
kana.IsHiragana("食べる") // -> false
```

```go
// String CONTAINS Hiragana
kana.ContainsHiragana("たべる") // -> true
kana.ContainsHiragana("食べる") // -> true
kana.ContainsHiragana("カタカナ") // -> false
```

```go
// String IS Katakana
kana.IsKatakana("バナナ") // -> true
kana.IsKatakana("バナナ茶") // -> false
```

```go
// String CONTAINS Katakana
kana.ContainsKatakana("バナナ") // -> true
kana.ContainsKatakana("バナナ茶") // -> true
kana.ContainsKatakana("ひらがな") // -> false
```

```go
// String IS Kanji
kana.IsKatakana("水") // -> true
kana.IsKatakana("also 茶") // -> false
```

```go
// String CONTAINS Kanji
kana.ContainsKanji("食べる") // -> true
kana.ContainsKanji("also 茶") // -> true
kana.ContainsKanji("ひらがな + カタカナ") // -> false
```

```go
// Extract Kanji from String
kana.ExtractKanji("また、平易な日本語で伝える週刊ニュースも放送します。日本語") 
// -> []string{"平", "易", "日", "本", "語", "伝", "週", "刊", "放", "送", "日", "本", "語"}
```


### Linguistic Considerations
A number of rule considerations and assumptions have been made while creating this library in order to conform to Wapuro romanization.

* __Long Vowels__ are indicated using using repeating characters instead of macrons/circumflexes: oo/おお instead of ō:
    * benkyou/べんきょう, not benkyō.
    * toukyou/とうきょう, not Tōkyō.
    * obaasan/おばあさん, not obāsan.
  * __Chōonpu (ー) are preferred__ for katakana and loan words, and will be preserved or converted to minus-dashes.
    * セーラー, not セエラア; becoming se-ra-
    * パーティー, not パアティィ; becoming pa-ti-
* __Particles__ are always converted literally:
    * は is ha, not wa.
    * を is wo, not o.
    * へ is he, not e, etc.
* __Moraic n's are used__ to disambiguate ん and な,に,ぬ,ね,の,にゃ,にゅ,にょ:
    * かんい is kan'i
    * しにょう is shin'you
    * ぜんにん is zennin
    * ぜんいん is zen'in
    * あんない is annai
* __Long Consonants__ marked with sokuons are doubled:
    * いっしょ is issho
    * ぱっぱ is pappa
    * ざっし is zasshi
    * まっちゃ is maccha (matcha when `phonetic = true`)
* __la, li, lu, le, lo__ are converted to _ra, ri, ru, re, ro_ before transliteratio.
* __じゃ, じゅ and じょ are ja, ju, and jo,__ however, _jya, jyu, and jyo_ are also valid for a one-way romaji→kana conversion.
* __Isolated small vowel kana__ are romanized with 'x' prefixes, _if they are not part of a larger composite:_ 
    * フォト becomes "foto", as the ォ is part of the larger composite フォ.
    * The unnatural spelling _パーティィ or ぱーてぃぃ_ becomes _pa-tixi,_ not pa-tii or pa-texixi (the correct spelling is パーティ pa-ti).
    * xa and XA are ぁ and ァ
    * xi and XI are ぃ and ィ
    * xu and XU are ぅ and ゥ
    * xe and XE are ぇ and ェ
    * xo and XO are ぉ and ォ
    * __Dangling _x_'s__ that remain after all other transliterations are converted into っ and ッ for hiragana and katakana respectively. The unnatural sequence "xx" will always become っっ or ッッ.
 
#### Phonetic vs Unphonetic Romaji
Both `ToRomaji` and `ToRomajiCased` take a boolean parameter to indicate if the romaji returned should be in a wapuro or 'phonetic' format, which more closely describes the pronounciation of the string. This can be useful if you don't need to preserve the character mappings for converting it back to the same kana and simply wish to show the pronunciation of a word. 

* When _Unphonetic `ToRomaji(string, false)`:_ 
    * Nihon-Shiki romanization is used to map input-ambiguous characters:
        * ぢ and づ are di and du
        * ヂ and ヅ are DI and DU
        * ぢゃ, ぢゅ, ぢょ are dya, dyu, dyo
        * ヂャ, ヂュ, ヂョ are DYA, DYU, DYO
    * cch double consonants are literal:
        * まっちゃ is maccha
        * こっち is kocchi
* When _Phonetic `ToRomaji(string, true)`:_
    * input-ambiguous characters are converted to their phonetic equivalents:
        * ぢ and づ are ji and zu
        * ヂ and ヅ are JI and ZU
        * ぢゃ, ぢゅ, ぢょ are ja, ju, jo
        * ヂャ, ヂュ, ヂョ are JA, JU, JO
    * cch double consonants use the Revised Hepburn intepretation (tch):
        * まっちゃ is matcha
        * こっち is kotchi

Review `tables.go` for romaji and kana character mapping references. 
 
### Contributions
Open an issue to report a bug, ask a question, or make a feature request!

### License
MIT License.
