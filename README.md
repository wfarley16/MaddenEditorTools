# MaddenEditorTools

An admin interface for editing the Madden NFL 11 (PlayStation 3) franchise/roster database.

## USR-DATA

`USR-DATA` is a PlayStation 3 save file from **Madden NFL 11**, containing the game's franchise/roster database in a proprietary binary format.

### File Overview

| Property | Value |
|----------|-------|
| Size | ~577 KB |
| Format | Madden DB binary format |
| Platform | PlayStation 3 |
| Game | Madden NFL 11 |
| Endianness | Big-endian (PowerPC/PS3) |
| Total readable strings | ~8,570 |

---

## Binary Format

### File Layout

```
[0x00]  Magic header         "DB" (0x44 0x42)
[0x02]  File metadata        ~100 bytes — version, total size, section IDs
[0x68]  Table of Contents    16 bytes × N tables
[????]  pedd (field defs)    8 bytes × 128+ field descriptors
[????]  Attribute code map   16 bytes × ~150 entries
[????]  Record data          variable-length player/team records
```

All multi-byte integers are **big-endian**.

---

### Table of Contents

Each TOC entry is **16 bytes**:

```
Bytes  0–3:   Table ID (4-char ASCII, e.g. "DIGP")
Bytes  4–5:   Padding (0x0000)
Bytes  6–7:   Record count (u16 big-endian)
Bytes  8–9:   Unknown
Bytes 10–11:  Field count (u16 big-endian)
Bytes 12–13:  Padding
Bytes 14–15:  Field descriptor offset (u16)
```

---

### `pedd` — Field Definition Table

The schema-within-the-schema. Must be parsed first before any record can be decoded.
Each descriptor is **8 bytes**:

```
Bytes 0–1:  Field offset within record (u16)
Bytes 2–3:  Field type/encoding:
              0x0080 = integer
              0x0084 = ?
              0x0088 = ?
              0x008c = ?
              0x00d4 = bitfield variant A
              0x00d8 = bitfield variant B
              0x00e4 = ?
Bytes 4–7:  Metadata — bit-shift for packing (0x20, 0x40, 0x60, 0x80, 0xa0)
```

Multiple attributes are bit-packed into single storage words. The shift values in bytes 4–7 control how to extract each sub-field.

#### Field Type Comparison (PC Madden reference)

The PC Madden DB editor ([bep713/madden-db-editor](https://github.com/bep713/madden-db-editor)) uses a simpler flat type enum for its version of the format:

| Value | Type | Notes |
|-------|------|-------|
| 0 | STRING | Fixed-length, null-padded |
| 1 | BINARY | Raw bytes |
| 2 | SINT | Signed integer, bit-packed |
| 3 | UINT | Unsigned integer, bit-packed |
| 4 | FLOAT | Float, bit-packed |

The PS3 format's `0x00d4` / `0x00d8` bitfield variants likely correspond to SINT/UINT variants in this scheme. Exact mapping is TBD.

#### Bit-Packing Algorithm

For numeric fields, both the PC and PS3 formats use the same bit-packing approach (confirmed from bep713 source):

```
1. Slice the relevant bytes from the record: record[floor(offset/8) .. ceil((bits+offset)/8)]
2. Convert each byte to an 8-bit array (MSB first)
3. Concatenate all bits into one flat bit array
4. Read N bits starting at (offset % 8) within that array
5. Reconstruct integer value (MSB first)
```

On write, the inverse: set individual bits in the flat array, then pack back into bytes.

---

## Table Reference

> **Note:** Table names ending in reversed ASCII are a known Madden DB convention (e.g. `MAET` = TEAM, `YALP` = PLAY). Attribute codes use a suffix convention: `P` = player attribute, `T` = team attribute, `SB` = stat block, `LP` = likely "level/progress", `RT` = rating.
>
> Entries marked *(guess)* are inferred from abbreviation patterns or context and have not been verified by decoding actual records.

---

### Core / Index Tables

| Table | Full Name (guess) | Description |
|-------|-------------------|-------------|
| `DIGP` | DIG Player | Primary player index / ID table |
| `DIGT` | DIG Team | Primary team index / ID table |
| `DIGL` | DIG League | League-level index |
| `DIGS` | DIG Season | Season-level index |
| `DIGC` | DIG Coach | Coach index |
| `DIGD` | DIG Draft | Draft-related index |
| `DIRF` | DIG Roster File | Roster file pointer/index |
| `DIYC` | DIG Year/Cycle | Year or cycle counter |
| `DIOT` | DIG Options/Type | Global options or type flags |
| `NSID` | N/A — Side ID? | Unknown; possibly player side/handedness index |
| `NCSI` | N/A — CSI? | Unknown |
| `DLFA` | DL Free Agent? | Free agent pool index *(guess)* |

---

### Player Attribute Tables (`*P` suffix)

| Table | Full Name (guess) | Description |
|-------|-------------------|-------------|
| `SPEP` | Speed Player | Speed rating |
| `SPOP` | Speed Option Player? | Alternate speed / burst *(guess)* |
| `SMFP` | Special Move / Finesse Player | Agility or special move attribute |
| `SOPP` | Soph/Position Player | Player position data |
| `MATP` | Master Attributes Player | Master player record / overall |
| `BILP` | Block IL Player | Interior lineman blocking |
| `DBLP` | DB Lateral Player? | Defensive back lateral movement *(guess)* |
| `DPSP` | Deep Pass Speed Player? | Deep route speed / pass attributes *(guess)* |
| `EGAP` | Evasion / Gap Player? | Evasion or gap-finding ability *(guess)* |
| `EHGP` | Evasion High Player? | High tackle evasion *(guess)* |
| `EHUP` | Evasion High Up Player? | *(guess)* |
| `EPLP` | Evasion Pass / Lateral Player? | *(guess)* |
| `ERBP` | Evasion Run Block Player? | *(guess)* |
| `ESRP` | Evasion Short Route Player? | *(guess)* |
| `EYEP` | Eye / Awareness Player | Awareness or vision rating *(guess)* |
| `FBPP` | Fullback Power Player | Fullback power blocking *(guess)* |
| `FBRP` | Fullback Route Player? | *(guess)* |
| `GHSP` | Grab High Short Player? | *(guess)* |
| `GSBP` | Game Speed Base Player? | *(guess)* |
| `HGTP` | Height Player | Player height *(guess)* |
| `HKSP` | Hook Short Player? | Short route hook *(guess)* |
| `HPCP` | Hip / Center Player? | *(guess)* |
| `HSLP` | High Speed Lateral Player? | *(guess)* |
| `HSRP` | High Speed Route Player? | Short route speed *(guess)* |
| `HTCP` | Hit / Tackle Center Player? | *(guess)* |
| `HTLP` | Hit / Tackle Left Player? | *(guess)* |
| `HTRP` | Hit / Tackle Right Player? | *(guess)* |
| `ICLP` | Inside Cut / Lateral Player? | *(guess)* |
| `IGAP` | Inside Gap Player? | *(guess)* |
| `IKSP` | Inside Kick Speed Player? | *(guess)* |
| `IPDP` | Impact / Power Drive Player? | Power running or blocking *(guess)* |
| `ITPP` | Impact Tackle Power Player? | *(guess)* |
| `JNIP` | Injury Player | Injury rating |
| `KATP` | Kick Accuracy / Total Player | Kicking accuracy *(guess)* |
| `KBPP` | Kick Block Power Player? | *(guess)* |
| `LYCP` | Loyalty / Cycle Player? | *(guess)* |
| `MCSLP` | Multi-Cut Short Lateral Player? | *(guess)* |
| `MJLP` | Mid Jump / Lateral Player? | *(guess)* |
| `MLHP` | Mid-Level High Player? | *(guess)* |
| `MSLP` | Mid Speed Lateral Player? | *(guess)* |
| `MTSP` | Mid Tackle Speed Player? | *(guess)* |
| `NAHP` | N/A High Player? | *(guess)* |
| `NCIP` | N/A / Coverage Inside Player? | *(guess)* |
| `NEJP` | N/A / Edge Jump Player? | *(guess)* |
| `NETP` | N/A / Edge Tackle Player? | *(guess)* |
| `NKLP` | N/A / Kick Left Player? | *(guess)* |
| `NKRP` | N/A / Kick Right Player? | *(guess)* |
| `NOCP` | N/A / Option / Coverage Player? | *(guess)* |
| `NSHP` | N/A / Short High Player? | *(guess)* |
| `NTSP` | N/A / Total Speed Player? | *(guess)* |
| `OBSP` | Overall Block Speed Player? | *(guess)* |
| `OCVP` | Outside Coverage Player | Zone/man coverage *(guess)* |
| `OGEP` | Offensive Gauge / Edge Player? | *(guess)* |
| `OHFP` | Outside High Finesse Player? | *(guess)* |
| `OMLP` | Offensive Mid Lateral Player? | *(guess)* |
| `OPLP` | Outside Pass Lateral Player? | *(guess)* |
| `ORDP` | Outside Route / Drive Player? | *(guess)* |
| `PHTP` | Physical / Hit Player? | *(guess)* |
| `PMIP` | Pass Mid Inside Player? | *(guess)* |
| `PMJP` | Pass Mid Jump Player? | *(guess)* |
| `PRVOP` | Previous Overall Player? | Prior year overall rating *(guess)* |
| `PRYP` | Prior Year Player? | *(guess)* |
| `PXSP` | Pass Experience Speed Player? | *(guess)* |
| `RACP` | Race / Acceleration Player? | Acceleration *(guess)* |
| `RATP` | Rating Total Player? | Overall rating *(guess)* |
| `REJP` | Rejection / Redirect Player? | Block shedding or pass deflection *(guess)* |
| `ROMP` | Route / Move Player? | Route running *(guess)* |
| `SBQP` | Speed Base Quick Player? | *(guess)* |
| `SBRP` | Speed Base Route Player? | *(guess)* |
| `SIVP` | Situational / Vision Player? | *(guess)* |
| `VCBP` | Vision Coverage Base Player? | *(guess)* |
| `WHLP` | Weight / High Lateral Player? | Player weight *(guess)* |
| `WRWAP` | Wide Receiver Awareness Player? | WR-specific awareness *(guess)* |
| `YHLP` | Year High Lateral Player? | *(guess)* |
| `YTJP` | Year Total Jump Player? | *(guess)* |
| `YTSP` | Year Total Speed Player? | *(guess)* |
| `UPLP` | Upgrade Level Player? | Player development stage *(guess)* |
| `ULEP` | Upgrade Level / Experience Player? | *(guess)* |
| `TWYP` | Two-Way Player? | *(guess)* |
| `TRKP` | Truck / Trucking Player | Trucking / run power *(guess)* |
| `THLP` | Throwing Left Player? | *(guess)* |
| `TGHP` | Tight / High Player? | *(guess)* |
| `TGWP` | Tight Gap Wide Player? | *(guess)* |
| `TMCP` | Team / Coverage Player? | *(guess)* |
| `ACENP` | Acceleration / Endurance Player? | *(guess)* |
| `AHLP` | Arm High / Lateral Player? | *(guess)* |
| `AHRP` | Arm High Route Player? | *(guess)* |
| `AHTP` | Arm High Tackle Player? | *(guess)* |
| `ALECP` | Acceleration / Level / Experience Player? | *(guess)* |
| `ALPP` | Agility / Lateral Power Player? | *(guess)* |
| `ANLP` | Angle / Lateral Player? | *(guess)* |
| `ASCP` | Awareness / Speed / Coverage Player? | *(guess)* |
| `ASLP` | Awareness / Speed / Lateral Player? | *(guess)* |
| `ASTP` | Awareness / Speed / Tackle Player? | *(guess)* |
| `ATSP` | Agility / Total Speed Player? | *(guess)* |
| `BRTLP` | Break Tackle Lateral Player? | Break tackle attribute *(guess)* |
| `CCMLP` | Coverage / Cut / Mid / Lateral Player? | *(guess)* |
| `HLELP` | High Lateral / Edge / Lateral Player? | *(guess)* |
| `IRTSP` | Inside Route / Total Speed Player? | *(guess)* |
| `JCPMP` | Jump / Cut / Power / Mid Player? | *(guess)* |
| `TCZLP` | Tackle / Cut / Zone / Lateral Player? | *(guess)* |

---

### Player Attribute Tables — Longer Codes

| Table | Full Name (guess) | Description |
|-------|-------------------|-------------|
| `ASUMB` | Awareness / Sum / Base? | *(guess)* |
| `HBPAT` | Hit / Block / Power / Attribute? | *(guess)* |
| `PBPLT` | Pass Block Power / Lateral? | Pass blocking *(guess)* |
| `QASLT` | Quick / Awareness / Speed / Lateral? | *(guess)* |
| `MLELT` | Mid / Lateral / Edge / Lateral? | *(guess)* |
| `WLERT` | Weight / Level / Edge / Route? | *(guess)* |
| `DSIVT` | Defense / Situational / Vision? | *(guess)* |
| `ESQVT` | Evasion / Squat / Vision? | *(guess)* |
| `FTSPT` | Feet / Speed? | *(guess)* |
| `LTSRT` | Lateral / Total Speed / Route? | *(guess)* |
| `SVAET` | Sack / Velocity / Awareness / Endurance? | *(guess)* |
| `YVORT` | Year / Volume / Overall / Rating? | Historical overall *(guess)* |
| `ZASMT` | Zone / Awareness / Sum? | Zone coverage awareness *(guess)* |

---

### Team Attribute Tables (`*T` suffix)

| Table | Full Name (guess) | Description |
|-------|-------------------|-------------|
| `MAET` | Master Team (TEAM reversed) | Team identity record |
| `DIGT` | DIG Team | Team index |
| `ANDT` | Announce / Total Team? | Team announcement data *(guess)* |
| `ANLT` | Announce / Left Team? | *(guess)* |
| `ANST` | Announce / Short Team? | Short team name (abbreviation) *(guess)* |
| `BPOT` | Bye / Points / Overall Team? | Bye week or point total *(guess)* |
| `BQRT` | Base Quality / Rating Team? | Team overall rating *(guess)* |
| `BRRT` | Base / Record / Rating Team? | Win/loss record *(guess)* |
| `BSTT` | Base / Stats Team? | Base team stats *(guess)* |
| `CNMT` | City / Name Team? | City name *(guess)* |
| `CSHT` | City / Short Team? | City abbreviation *(guess)* |
| `DIOT` | DIG Options Team | Team option flags *(guess)* |
| `DIST` | Distance / Stats Team? | *(guess)* |
| `DROT` | Draft / Roster Team? | Draft/roster data *(guess)* |
| `EDRT` | Edit / Draft Team? | Editable draft data *(guess)* |
| `EHCT` | Edit / High / Center Team? | *(guess)* |
| `FORT` | Formation / Roster Team | Team formation/depth chart *(guess)* |
| `GCBT` | General / Cap / Budget Team? | Salary cap data *(guess)* |
| `GLAT` | Geo / Location / Attributes Team? | Geographic/stadium data *(guess)* |
| `GPAT` | General / Points / Attributes Team? | *(guess)* |
| `GPLT` | General / Plays / Team? | *(guess)* |
| `GPTT` | General / Points / Total Team? | *(guess)* |
| `GSAT` | General / Stats / Attributes Team? | *(guess)* |
| `GSLT` | General / Stats / Left Team? | *(guess)* |
| `GSTT` | General / Stats / Total Team? | *(guess)* |
| `LDRT` | League / Draft Team? | *(guess)* |
| `LFAT` | Left / Formation / Attributes Team? | *(guess)* |
| `LFMT` | Left / Formation Team? | *(guess)* |
| `LORT` | Left / Overall / Rating Team? | *(guess)* |
| `LSPT` | Left / Speed Team? | *(guess)* |
| `NADT` | N/A / Depth Team? | *(guess)* |
| `NFPT` | N/A / Formation / Play Team? | *(guess)* |
| `ODCT` | Overall / Depth / Chart Team | Depth chart entry *(guess)* |
| `OHST` | Overall / High / Stats Team? | *(guess)* |
| `OHUT` | Overall / High / Unit Team? | *(guess)* |
| `OLFT` | Overall / Left / Formation Team? | *(guess)* |
| `OUAT` | Overall / Unit / Attributes Team? | *(guess)* |
| `PRCT` | Progress / Count Team? | *(guess)* |
| `RCBT` | Record / Cap / Budget Team? | Salary cap remaining *(guess)* |
| `RPAT` | Rating / Points / Attributes Team? | *(guess)* |
| `RPLT` | Rating / Points / Left Team? | *(guess)* |
| `RPTT` | Rating / Points / Total Team? | *(guess)* |
| `RSAT` | Rating / Stats / Attributes Team? | *(guess)* |
| `RSLT` | Rating / Stats / Left Team? | *(guess)* |
| `RSTT` | Rating / Stats / Total Team? | *(guess)* |
| `XTCT` | Extra / Count Team? | *(guess)* |

---

### Stat Block Tables (`*SB` suffix)

These likely represent blocks of seasonal or career stats.

| Table | Full Name (guess) | Description |
|-------|-------------------|-------------|
| `AASB` | Attack A Stat Block? | *(guess)* |
| `ABSB` | Attack B Stat Block? | *(guess)* |
| `ACSB` | Attack C Stat Block? | *(guess)* |
| `AFSB` | Attack F Stat Block? | *(guess)* |
| `AGSB` | Attack G Stat Block? | *(guess)* |
| `APSB` | Attack P Stat Block? | *(guess)* |
| `ASSB` | Attack S Stat Block? | *(guess)* |
| `ATSB` | Attack T Stat Block? | *(guess)* |
| `AWSB` | Awareness Stat Block? | *(guess)* |
| `TASB` | Team A Stat Block | Team stats block A *(guess)* |
| `TBSB` | Team B Stat Block | Team stats block B *(guess)* |
| `TCSB` | Team C Stat Block | *(guess)* |
| `TFSB` | Team F Stat Block | *(guess)* |
| `TGSB` | Team G Stat Block | *(guess)* |
| `TPSB` | Team P Stat Block | *(guess)* |
| `TSSB` | Team S Stat Block | *(guess)* |
| `TTSB` | Team T Stat Block | *(guess)* |
| `TWSB` | Team W Stat Block | *(guess)* |

---

### Roster Variant Tables (`0*SP`, `1*SP`, etc.)

These appear to be numbered sub-tables, possibly representing different roster slots, positions, or depth chart tiers.

| Table | Description |
|-------|-------------|
| `0ASP` | Roster slot A, tier 0 *(guess)* |
| `0BSP` | Roster slot B, tier 0 *(guess)* |
| `1ASP` | Roster slot A, tier 1 *(guess)* |
| `1BSP` | Roster slot B, tier 1 *(guess)* |
| `5ASP` / `5BSP` | Roster slot tier 5 *(guess)* |
| `6ASP` / `6BSP` | Roster slot tier 6 *(guess)* |
| `0UBHR` | Unknown base header/root *(guess)* |
| `1DIS` | Distance table 1 *(guess)* |
| `1VRT` | Variant/Version table 1 *(guess)* |

---

### Special / Miscellaneous Tables

| Table | Full Name (guess) | Description |
|-------|-------------------|-------------|
| `YALP` | Player (PLAY reversed) | Player master list / cross-reference |
| `MAET` | Team (TEAM reversed) | Team master list |
| `YJNI` | Injury (INJY reversed) | Injury data |
| `THCD` | Table/Header Chunk Descriptor | File structure metadata |
| `pedd` | Edit DB | Field definition table — the schema descriptor |
| `SOPP` | Position Options | Player position assignments |
| `NCSI` | N/A — CSI? | Unknown |
| `NSID` | N/A — Side ID? | Possibly handedness / player side |

---

### Team Roster Data

Each of the 35 rosters is identified by a `teamdb_` prefixed string:

| Identifier | Team |
|------------|------|
| `teamdb_Bears` … `teamdb_Vikings` | All 32 NFL franchises |
| `teamdb_NFLGreats` | NFL legends / greats roster |
| `teamdb_FreeAgents` | Free agent pool |
| `teamdb_HallOfFame` | Hall of Fame unlockable players |

---

## Parsing Order

To decode any record, parse in this order:

```
1. Read file header (magic + metadata)
2. Parse Table of Contents → get table IDs, record counts, field offsets
3. Parse pedd → build field definition map (offset → type + bit-shift)
4. Parse attribute code table → map codes (e.g. "SPEP") to pedd field indices
5. Decode player/team records using the pedd map
```

---

## CRC Checksums

Based on the PC Madden reference implementation, the format likely uses **CRC-32 big-endian** checksums (polynomial `0x04C11DB7`, same as used in Linux kernel CRC32 BE). The PC format stores:

- A **file-level header CRC** covering the first 20 bytes of the file header
- A **per-table header CRC** (`headercrc`) covering the table header minus its first and last 4 bytes
- A **per-table data CRC** (`priorcrc`) — a chained CRC where each table's value covers the *previous* table's data block
- An **EOF CRC** covering the last table's data block

The final CRC value written is `~crc32_be(0, buffer, len, start)` (bitwise NOT of the result).

> **Note:** Whether the PS3 format uses identical CRC layout and polynomial is unconfirmed. This is a strong candidate to investigate when implementing the writer.

---

## Prior Art

- [bep713/madden-db-editor](https://github.com/bep713/madden-db-editor) — Electron/Vue desktop editor for PC Madden. Different format constants but same binary DB family. Source for the bit-packing algorithm and CRC approach documented above.

---

## Goals

- Parse and display the binary DB format
- Browse and search players across all 35 rosters
- Edit player attributes and ratings
- Save changes back to `USR-DATA` binary format for use on PS3
