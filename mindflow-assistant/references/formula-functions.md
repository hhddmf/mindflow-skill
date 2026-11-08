# Formula (fx) functions

The functions offered by the **fx** menu of any field: 243 functions in 12 categories,
the same categories the menu shows. Names are exact, so search this file by name;
descriptions are short summaries, and the Studio shows the full help when you hover a
function. All are available by default except where marked **may be switched off**. The
list changes with releases: if the menu differs, trust the menu.

How they work (see [canvas-and-steps.md](canvas-and-steps.md#fx-functions)): functions
chain into a pipeline with a live preview; new functions start with **Wrap** and
**Quotes** on, so turn both off for a plain value; a recursive JSON Path (`$..x`) returns
a list, so add `$[0]`.

## The usual suspects

Most transformations need only a handful: Find / Replace, JSON Path, Regular
expression, Get Time, Extract URLs, From UNIX Timestamp, Array push, Array split, Array
get item, Sum, Escape string, JSONata, Array concat, Slice, Extract IP addresses, From
Base64, Unescape string, Array unique.

**Found in existing flows but not in the menu list below:** Modify date with duration
(one unit per call: minutes, hours, days, weeks or months), To UNIX Timestamp, To JSON,
Round, Change case, Concatenate arrays, To Base64 V2, Is empty. Flows that already use
them keep working; check the menu before using one in a new flow. A few names exist in
two casings (`JSON Path` / `JSON path`): pick the one the menu shows.

## Settings of the common functions

Each function takes the field's value as input, plus the settings below (names as
stored by Mindflow; the panel shows the same or very close labels). Settings in
brackets are optional; units, operators and formats are picked from lists in the
panel.

| Function | Settings |
|---|---|
| From UNIX Timestamp | unit (pick the one matching the input: a 10-digit epoch is in seconds, 13 digits in milliseconds), outputFormat |
| To UNIX Timestamp | unit, [showDatetime] |
| Modify date with duration | operator (add / subtract), **one** unit field among years / months / weeks / days / hours / minutes, [utcOutput] |
| Translate DateTime Format | [inputFormat], outputFormat, sourceTimezone, targetTimezone, [locale]. outputFormat uses Java-style letters: `EEEE` = day name (`Sunday`), `HH:mm` = 24-hour time; moment-style `dddd` is printed as is. Time zones by name (`Europe/Paris`), daylight saving included |
| Parse DateTime | Input format string (or Built in formats), Input timezone |
| Get Time | [Granularity]; with no input it gives the current time |
| JSON Path | path, wrap, quotes, [evaluate], [flatten] |
| JSONata | query |
| Find / Replace | Find, Replace, Global match, Case insensitive, Multiline matching, Dot matches all |
| Regular expression | Regex, Output format, Case insensitive, Unwrap, … |
| Array split / Array join | separator |
| Array get item | index, [quotes] |
| Array push | valueToPush |
| Array concat | array2 |
| Slice / Array slice | start, [end] |
| Concatenate | toConcat |
| Sum / Subtract | Delimiter (the input is a delimited list of numbers) |
| Divide | divisor, or Delimiter for a list |
| Extract IP addresses / URLs / domains | Display total, Sort, Unique, Unwrap, (IPv4 / IPv6, Remove local IPv4 addresses) |
| UUID V4 | none (no input either) |

**Dates, end to end:** epoch → *From UNIX Timestamp* (unit seconds) gives a UTC
date-time; add days with *Modify date with duration* (operator add, days 3) on that
date. If *Modify date with duration* isn't in the menu, add the seconds before
converting (3 days = 259,200 s), e.g. in a Run script, then use *From UNIX Timestamp*.
Check the preview: a year of 1970 means the unit is wrong.

## Custom (12) — Mindflow's own

| Function | Does |
|---|---|
| Assert | Fails the step if the input isn't true, else passes it on |
| Compare | Compares operands with a logical operator (used by Condition branches) |
| Concatenate | Joins two strings |
| CSV to JSON | Converts CSV to JSON |
| Extract Email Addresses | Pulls email addresses out of text |
| Levenshtein distance | Edit distance between two strings |
| MSG to JSON | Converts an Outlook .msg file to JSON |
| Pad text to equal length | Pads text to a given length with a character |
| Random Number Generator | Random number in a range |
| UUID V4 | Generates a UUID v4 |
| XML to JSON | Converts an XML document to JSON (namespace prefixes kept as keys) |
| XPath | Reads from XML with XPath; always returns a string |

## Array (12)

| Function | Does |
|---|---|
| Array concat | Joins two arrays |
| Array flat | Flattens nested arrays |
| Array get item | Item at an index |
| Array join | Joins items into a string with a separator |
| Array length | Number of items |
| Array pop | Removes the last item, returns the array |
| Array push | Appends a value |
| Array range | Array of numbers in a range |
| Array slice | Part of an array (end excluded) |
| Array sort | Sorts |
| Array split | Splits a string into an array on a separator |
| Array unique | Removes duplicates |

## Data format (59)

| Function | Does |
|---|---|
| JSON Path | Selects data in JSON with a JSONPath expression |
| JSONata | Queries and reshapes JSON with JSONata |
| To Base64, From Base64 | Base64 encode / decode (several alphabets) |
| To Base32, From Base32, To Base45, From Base45, To Base58, From Base58, To Base62, From Base62, To Base85, From Base85 | Other base encodings |
| To Base, From Base | Number to / from another numeric base |
| To Hex, From Hex, To Hex Content, From Hex Content, To Hexdump, From Hexdump | Hexadecimal forms |
| To Binary, From Binary, To Octal, From Octal, To Decimal, From Decimal, To Charcode, From Charcode, To BCD, From BCD | Other byte representations |
| To HTML Entity, From HTML Entity | HTML entities |
| To Punycode, From Punycode | Internationalised domain names |
| To Quoted Printable, From Quoted Printable | QP email encoding |
| To MessagePack, From MessagePack | JSON ⇄ MessagePack |
| URL Encode, URL Decode | Percent-encoding |
| Encode text, Decode text | Text ⇄ a chosen character encoding |
| To Upper case, To Lower case | Change case |
| Escape Unicode Characters, Unescape Unicode Characters | `\uXXXX` notation |
| Normalise Unicode, Remove Diacritics, Unicode Text Format | Unicode clean-up and styling |
| Convert to NATO alphabet | Spells text phonetically |
| Strip HTML tags | Removes HTML tags |
| Hex to PEM, PEM to Hex | Certificate formats |
| Parse TLV | Type-Length-Value to JSON |
| Show Base64 offsets | Locates a string inside Base64 data |
| Swap endianness | Byte order |
| Text Encoding Brute Force | Tries every text encoding |

## Utils (35)

| Function | Does |
|---|---|
| Find / Replace | Replaces text (plain or regex) |
| Split, Filter, Sort, Unique, Reverse | Line/delimiter list operations on a string |
| Head, Tail | First / last n lines |
| Trim, Remove whitespace, Remove null bytes | Clean-up |
| Escape string, Unescape string | Escapes special characters (quotes, newlines) |
| Count occurrences | How often a string appears |
| Pad lines, Add line numbers, Remove line numbers | Line formatting |
| Take bytes, Drop bytes | Byte slices |
| To Table | Renders delimited data as a table |
| Fuzzy Match, Hamming Distance, Offset checker | String similarity and comparison |
| Get All Casings, Expand alphabet range | String generation |
| To Case Insensitive Regex, From Case Insensitive Regex | Regex case conversion |
| Convert area, Convert distance, Convert mass, Convert speed, Convert data units, Convert co-ordinate format | Unit conversions |
| Parse ObjectID timestamp | Time from a MongoDB ObjectID |
| Parse UNIX file permissions | Explains `rwx` / octal permissions |

## Extractors (8)

| Function | Does |
|---|---|
| Regular expression | Your own regex over the input |
| Slice | Characters between two indices |
| Extract IP addresses, Extract URLs, Extract domains, Extract MAC addresses, Extract file paths | Indicators from text |
| Extract dates | Dates in yyyy-mm-dd, dd/mm/yyyy, mm/dd/yyyy |

## Date & Time (5)

| Function | Does |
|---|---|
| Get Time | Current time since the UNIX epoch |
| From UNIX Timestamp | Epoch to a date-time string (UTC) |
| Parse DateTime | Reads a date in a given format |
| Translate DateTime Format | Rewrites an ISO 8601 date in another format |
| Windows Filetime to UNIX Timestamp | Windows time to epoch |

## Arithmetic (25)

| Function | Does |
|---|---|
| Sum, Subtract, Multiply, Divide | On a list of numbers |
| Mean, Median, Standard Deviation | Statistics on a list |
| Set Union, Set Intersection, Set Difference, Symmetric Difference, Cartesian Product, Power Set | Set operations |
| ADD, SUB, AND, OR, XOR, NOT, XOR Brute Force | Byte-wise operations with a key |
| Bit shift left, Bit shift right, Rotate left, Rotate right | Bit operations |
| ROT13 | Letter rotation cipher |

## Hashing (40)

| Function | Does |
|---|---|
| MD2, MD4, MD5, MD6, SHA0, SHA1, SHA2, SHA3, Shake, Keccak, BLAKE2b, BLAKE2s, RIPEMD, Whirlpool, Streebog, GOST hash, SM3, HAS-160, Snefru | Hash functions |
| HMAC | Keyed hash |
| Bcrypt, Bcrypt compare, Bcrypt parse, Scrypt | Password hashing |
| CTPH, SSDEEP, Compare CTPH hashes, Compare SSDEEP hashes | Fuzzy hashing and similarity |
| CRC-8 Checksum, CRC-16 Checksum, CRC-32 Checksum, Adler-32 Checksum, Fletcher-8 Checksum, Fletcher-16 Checksum, Fletcher-32 Checksum, Fletcher-64 Checksum, Luhn Checksum, TCP/IP Checksum | Checksums |
| Analyse hash | Guesses a hash's algorithm |
| Generate all hashes | Every hash at once |

## Cryptography (13)

| Function | Does |
|---|---|
| JWT Decode | Reads a JWT without checking its signature |
| JWT Sign, JWT Verify | Signs / verifies a JWT with a secret or key |
| Generate TOTP | Time-based one-time password |
| Generate UUID | RFC 4122 v4 UUID |
| Parse X.509 certificate, Parse ASN.1 hex string | Certificate and ASN.1 parsing |
| Hex to Object Identifier, Object Identifier to Hex | OID conversion |
| SM4 Encrypt, SM4 Decrypt | SM4 block cipher |
| Atbash Cipher, Substitute | Substitution ciphers |

## Networking (23)

| Function | Does |
|---|---|
| IP Address In CIDR Range Check | True if an IP is in a subnet |
| Parse IP range, Group IP addresses, Change IP format, Parse IPv6 address | IP handling |
| Defang IP Addresses, Defang URL | Makes indicators unclickable |
| Parse URI, Parse User Agent | URL and user-agent parsing |
| Strip HTTP headers, Dechunk HTTP response | Raw HTTP clean-up |
| Format MAC addresses | MAC formats |
| JA3 Fingerprint, JA3S Fingerprint, HASSH Client Fingerprint, HASSH Server Fingerprint | TLS / SSH fingerprints |
| Parse SSH Host Key, Parse IPv4 header, Parse UDP | Protocol parsing |
| Encode NetBIOS Name, Decode NetBIOS Name, VarInt Encode, VarInt Decode | Encodings |

## Others (10)

| Function | Does |
|---|---|
| Render Markdown | Markdown to HTML (raw HTML disabled) |
| YARA Rules | Runs YARA rules on the input |
| Generate HOTP | Counter-based one-time password |
| Haversine distance | Distance between two GPS points |
| Chi Square, Index of Coincidence, Frequency distribution | Data statistics |
| Disassemble x86 | Machine code to assembly |
| Generate Lorem Ipsum | Placeholder text |
| XKCD Random Number | Always 4 (a joke) |

## Create temporary credential (1)

| Function | Does |
|---|---|
| HashiCorp Based Credential | Creates a temporary credential from Vault as a variable to inject — **may be switched off** |
