- [ID3v2.2](#id3v22)
  - [frame](#frame)
- [ID3v2.3](#id3v23)
  - [extended header](#extended-header)
  - [frame](#frame-1)
- [ID3v2.4](#id3v24)
  - [extended header](#extended-header-1)
  - [frame](#frame-2)
  - [footer](#footer)

`ID3v2` タグはおもに `MP3` 音楽ファイルのメタ情報を保存するために使われます。
`MP3` ファイルの最初に保存され、`"ID3"` で始まります。

# type

- **`int`**: 整数 (Integer)、ビッグエンディアン
- **`syncsafe int`**: 同期安全整数 (SuncSafe Integer)、ビッグエンディアン
  各バイトの最上位ビットが必ず0になります。
- **`str`**: 文字列、`ISO-8859-1` (`latin-1`) エンコード
- **`encode str`**: エンコード指定文字列
  - `0x00` の場合は `ISO-8859-1` (`latin-1`) エンコード
  - `0x01` の場合は `ISO/IEC 10646-1:1993` (`ucs-2`) エンコード
- **`str null`**: ヌル終端文字列
- **`encode str null`**: エンコード指定ヌル終端文字列
  - `latin-1` エンコードの場合は `0x00`
  - `ucs-2` エンコードの場合は `0x00 0x00`
- **`lang`**: 文字列、ISO-639-2
- **`url`**: 文字列

Text = "XXXX"\
Integers = $ xx xx xx xx\
Synchsafe integers = %0xxxxxxx 0xxxxxxx 0xxxxxxx 0xxxxxxx\
($ xx is hexa number, %xxxxxxxx is binary number)

# Header

| size | name | type |
| ---: | --- | --- |
| 3 | tag header | str |
| 1 | major version | int |
| 1 | revision version | int |
| 1 | flags | flag |
| 4 | size | syncsafe int |

- `ID3v2` タグはMP3ファイルの最初に保存されます。
- `tag header` は必ず `"ID3"` (`0x49 0x44 0x33`)で、 `ID3v2` タグが存在することを表します。
- `major version` はタグのバージョンです。
- `revision version` はタグのリビジョンバージョンです。
- `flags` はタグのフラグです。
  `0xabcd0000`
  - フラグ `a` がセットされた場合、非同期を使用します。
  - フラグ `b` がセットされた場合
    - `ID3v2.2` タグでは圧縮タグを使用します。
    - `ID3v2.3` `ID3v2.4` タグでは拡張ヘッダーを使用します。
  - フラグ `c` がセットされた場合、を使用します。
  - フラグ `d` がセットされた場合、を使用します。
- `size` はヘッダー (10バイト) を除いたタグのサイズです。同期安全整数を使用します。



![id3v2 header svg](id3v2-header.svg)

- Version represents the version of the tag.
  Different versions are not compatible.
  - `0x02` = ID3v2.2
  - `0x03` = ID3v2.3
  - `0x04` = ID3v2.4
- Can be read by applications that support larger revision versions of tags. (backward compatibility)
- Size does not include common header and footer.

# Extended Header

Extended header added in v2.3.
It is present when the "Extended header" flag of the common header is set.

## ID3v2.3 Extended Header

![id3v2.3 extended header svg](id3v2-3-extended-header.svg)

## ID3v2.3 Extended Header

![id3v2.4 extended header svg](id3v2-4-extended-header.svg)

# Footer

## ID3v2.4 Footer

![id3v2.4 footer svg](id3v2-4-footer.svg)

# Frame

## ID3v2.2 frame header

| size | name | type |
| ---: | --- | --- |
| 3 | frame id | str |
| 3 | frame size | int |

- `frame id` は大文字 `A-Z` または数字 `0-9` の文字列です。
- `frame size` はヘッダー (6バイト) を除いたフレームのサイズです。
  フレームのサイズは1バイトより大きい必要があります。

# Unique file identifier フレーム

## ID3v2.2 `"UFI"`

| size | name | type |
| ---: | --- | --- |
| 3 | frame id | str |
| 3 | frame size | int |
|   | owner id | str null |
| < 64 | id | bytes |

- Unique file identifier フレームはデータベース内でオーディオファイルの情報の識別子を保存します。
- `frame id` は `"UFI"` です。
- `owner id` はデータベースの識別子です。
  長さが0の場合はこのフレームは無視されます。
  同じ `owner id` を持つ UFI フレームはファイルに一つだけです。
- `id` はデータベース内のオーディオファイルの識別子です。

# Text information フレーム

## ID3v2.2 `"T00" - "TZZ"` 

| size | name | type |
| ---: | --- | --- |
| 3 | frame id | str |
| 3 | frame size | int |
| 1 | encoding | byte |
|   | information | encode str null |

- Text information フレームはテキスト情報を保存します。
- `frame id` は `"T00" - "TZZ"` から `"TXX"` を除くいずれかです。
- `encoding` は文字列のエンコードです。
  - `0x00` の場合は `latin-1` です。
  - `0x01` の場合は `ucs-2` です。
- `information` はテキスト情報です。

### 詳細

- `"TT1"` - コンテンツグループ
- `"TT2"` - タイトル/曲名/コンテンツ
- `"TT3"` - サブタイトル/説明
- `"TP1"` - リードアーティスト/リードパフォーマー/ソリスト/パフォーミンググループ
  `"/"` の文字で区切られます。
- `"TP2"` - バンド/オーケストラ/伴奏
- `"TP3"` - 指揮者
- `"TP4"` - 解釈/リミックス
- `"TCM"` - 作曲者
  `"/"` の文字で区切られます。
- `"TXT"` - 作詞家/テキストライター
  `"/"` の文字で区切られます。
- `"TLA"` - 言語コード ISO-639-2
- `"TCO"` - コンテンツタイプ
  `"("` と `")"` で囲ってID3v1.1のジャンルやID3v2のコンテンツタイプを参照できます。
  `"("` で開始する場合は `"(("` で開始します。
  - `"(RX)"` Remix
  - `"(CR)"` Cover
- `"TAL"` - アルバム/映画/番組タイトル
- `"TPA"` - セットの部分
  `"TAL"` フレームのどの部分から来たのかを説明します。
  `"/" 総数` で拡張できます。
- `"TRK"` - トラック番号/セット内の位置
  `"/" 総数` で拡張できます。
- `"TRC"` - 国際標準レコーディングコード(ISRC)
- `"TYE"` - 年 YYYY
  常に四文字です。
- `"TDA"` - 日付 MMDD
  常に四文字です。
- `"TIM"` - 時刻 HHMM
  常に四文字です。
- `"TRD"` - 記録日
- `"TMT"` - メディアタイプ
- `"TT3"` - サブタイトル/説明
- `"TT3"` - サブタイトル/説明
- `"TT3"` - サブタイトル/説明

  TFT
   The 'File type' frame indicates which type of audio this tag defines.
   The following type and refinements are defined:
   
     MPG    MPEG Audio
       /1     MPEG 2 layer I
       /2     MPEG 2 layer II
       /3     MPEG 2 layer III
       /2.5   MPEG 2.5
       /AAC   Advanced audio compression
     
   but other types may be used, not for these types though. This is used
   in a similar way to the predefined types in the "TMT" frame, but
   without parenthesis. If this frame is not present audio type is
   assumed to be "MPG".

  TBP
   BPM is short for beats per minute, and is easily computed by
   dividing the number of beats in a musical piece with its length. To
   get a more accurate result, do the BPM calculation on the main-part
   only. To acquire best result measure the time between each beat and
   calculate individual BPM for each beat and use the median value as
   result. BPM is an integer and represented as a numerical string.

  TCR
   The 'Copyright message' frame, which must begin with a year and a
   space character (making five characters), is intended for the
   copyright holder of the original sound, not the audio file itself. The
   absence of this frame means only that the copyright information is
   unavailable or has been removed, and must not be interpreted to mean
   that the sound is public domain. Every time this field is displayed
   the field must be preceded with "Copyright " (C) " ", where (C) is one
   character showing a C in a circle.

  TPB
   The 'Publisher' frame simply contains the name of the label or
   publisher.

  TEN
   The 'Encoded by' frame contains the name of the person or
   organisation that encoded the audio file. This field may contain a
   copyright message, if the audio file also is copyrighted by the
   encoder.

  TSS
   The 'Software/hardware and settings used for encoding' frame
   includes the used audio encoder and its settings when the file was
   encoded. Hardware refers to hardware encoders, not the computer on
   which a program was run.

  TOF
   The 'Original filename' frame contains the preferred filename for the
   file, since some media doesn't allow the desired length of the
   filename. The filename is case sensitive and includes its suffix.

  TLE
   The 'Length' frame contains the length of the audiofile in
   milliseconds, represented as a numeric string.

  TSI
   The 'Size' frame contains the size of the audiofile in bytes
   excluding the tag, represented as a numeric string.

  TDY
   The 'Playlist delay' defines the numbers of milliseconds of silence
   between every song in a playlist. The player should use the "ETC"
   frame, if present, to skip initial silence and silence at the end of
   the audio to match the 'Playlist delay' time. The time is represented
   as a numeric string.

  TKE
   The 'Initial key' frame contains the musical key in which the sound
   starts. It is represented as a string with a maximum length of three
   characters. The ground keys are represented with "A","B","C","D","E",
   "F" and "G" and halfkeys represented with "b" and "#". Minor is
   represented as "m". Example "Cbm". Off key is represented with an "o"
   only.

  TOT
   The 'Original album/Movie/Show title' frame is intended for the title
   of the original recording(/source of sound), if for example the music
   in the file should be a cover of a previously released song.
   
  TOA
   The 'Original artist(s)/performer(s)' frame is intended for the
   performer(s) of the original recording, if for example the music in
   the file should be a cover of a previously released song. The
   performers are seperated with the "/" character.

  TOL
   The 'Original Lyricist(s)/text writer(s)' frame is intended for the
   text writer(s) of the original recording, if for example the music in
   the file should be a cover of a previously released song. The text
   writers are seperated with the "/" character.

  TOR
   The 'Original release year' frame is intended for the year when the
   original recording, if for example the music in the file should be a
   cover of a previously released song, was released. The field is
   formatted as in the "TDY" frame.

# ID3v2.2

https://id3.org/id3v2-00

| length | name    | value              | info                                      |
| -----: | :------ | :----------------- | :---------------------------------------- |
|        |         |                    | start of file                             |
|      3 | header  | "ID3"              |                                           |
|      2 | version | $ 02 00            |                                           |
|      1 | flags   | %ab000000          | a = unsynchronisation<br/>b = compression |
|      4 | size    | Synchsafe integers | exclude header (10 Bytes)                 |
|   size | frames  |                    |                                           |

## frame

| length | name             | value    | info                     |
| -----: | :--------------- | :------- | :----------------------- |
|      3 | frame identifier | Text     | A-Z and 0-9              |
|      3 | frame size       | Integers | exclude header (6 Bytes) |
|   size | frame data       |          |

# ID3v2.3

https://id3.org/d3v2.3.0

| length | name            | value              | info                                                                         |
| -----: | :-------------- | :----------------- | :--------------------------------------------------------------------------- |
|        |                 |                    | start of file                                                                |
|      3 | header          | "ID3"              |                                                                              |
|      2 | version         | $ 03 00            |                                                                              |
|      1 | flags           | %abc00000          | a = Unsynchronisation<br/>b = Extended header<br/>c = Experimental indicator |
|      4 | size            | Synchsafe integers | exclude header (10 Bytes)                                                    |
|     ex | extended header |                    | if flag b is 1                                                               |
|     fr | frames          |                    |                                                                              |

## extended header

| length | name                 | value              | info                   |
| -----: | :------------------- | :----------------- | :--------------------- |
|      4 | extended header size | Integers           | exclude size (4 Bytes) |
|      2 | flags                | %a0000000 00000000 | a = CRC data present   |
|      4 | padding size         | Integers           |                        |
|      4 | CRC                  | Byte datas         | if flag a is 1         |

## frame

| length | name        | value              | info                                                                                                                                          |
| -----: | :---------- | :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
|      4 | frame ID    | Text               | A-Z and 0-9                                                                                                                                   |
|      4 | frame size  | Integers           | exclude header (10 Bytes)                                                                                                                     |
|      2 | frame flags | %abc00000 ijk00000 | a = Tag alter preservation<br/>b = File alter preservation<br/>c = Read only<br/>i = Compression<br/>j = Encryption<br/>k = Grouping identity |
|   size | frame data  |                    |                                                                                                                                               |

# ID3v2.4

https://id3.org/id3v2.4.0-structure
https://id3.org/id3v2.4.0-frames

| length | name            | value              | info                                                                                                |
| -----: | :-------------- | :----------------- | :-------------------------------------------------------------------------------------------------- |
|        |                 |                    | start of file                                                                                       |
|      3 | header          | "ID3"              |                                                                                                     |
|      2 | version         | $ 04 00            |                                                                                                     |
|      1 | flags           | %abcd0000          | a = Unsynchronisation<br/>b = Extended header<br/>c = Experimental indicator<br/>d = Footer present |
|      4 | size            | Synchsafe integers | exclude header (10 Bytes)                                                                           |
|     ex | extended header |                    | if flag b is 1                                                                                      |
|     fr | frames          |                    |                                                                                                     |
|     10 | footer          |                    | if flag d is 1                                                                                      |

## extended header

| length | name                 | value              | info                                                                                                                                                                                      |
| -----: | :------------------- | :----------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      4 | extended header size | Synchsafe integers | whole size (>= 6 Bytes)                                                                                                                                                                   |
|      1 | flags size           | $ 01               |
|      1 | flags                | %0bcd0000          | b = Tag is an update<br/>c = CRC data present<br/>d = Tag restrictions                                                                                                                    |
|      1 | b data size          | $ 00               | if flag b is 1                                                                                                                                                                            |
|      1 | c data size          | $ 05               | if flag c is 1                                                                                                                                                                            |
|      5 | c data               | Byte datas         | if flag c is 1                                                                                                                                                                            |
|      1 | d data size          | $ 01               | if flag d is 1                                                                                                                                                                            |
|      1 | d data               | %ppqrrstt          | p = Tag size restrictions<br/>q = Text encoding restrictions<br/>r = Text fields size restrictions<br/>s = Image encoding restrictions<br/>t = Image size restrictions<br/>if flag d is 1 |

## frame

| length | name                   | value              | info                                                                                                                                                                                                  |
| -----: | :--------------------- | :----------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      4 | frame ID               | Text               | A-Z and 0-9                                                                                                                                                                                           |
|      4 | frame size             | Synchsafe integers | exclude header (10 Bytes)                                                                                                                                                                             |
|      2 | frame flags            | %0abc0000 0h00kmnp | a = Tag alter preservation<br/>b = File alter preservation<br/>c = Read only<br/>h = Grouping identity<br/>k = Compression<br/>m = Encryption<br/>n = Unsynchronisation<br/>p = Data length indicator |
|     ad | additional information |                    |
|     dt | frame data             |                    |

## footer

| length | name    | value              | info                                                                                                |
| -----: | :------ | :----------------- | :-------------------------------------------------------------------------------------------------- |
|      3 | header  | "3DI"              |
|      2 | version | $ 04 00            |
|      1 | flags   | %abcd0000          | a = Unsynchronisation<br/>b = Extended header<br/>c = Experimental indicator<br/>d = Footer present |
|      4 | size    | Synchsafe integers | exclude header (10 Bytes)                                                                           |

# 参照リンク

- [ID3v2.2 standard](https://id3.org/id3v2-00)
- [ID3v2.3 stamdard](https://id3.org/d3v2.3.0)
- [ID3v2.4 structure](https://id3.org/id3v2.4.0-structure)
- [ID3v2.4 frames](https://id3.org/id3v2.4.0-frames)
