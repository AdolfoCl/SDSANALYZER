# SDSANALYZER

The same question DBANALYZER answers — which structure in your DMSII database is
about to stop it — as XML instead of a printed page, so you can keep the answers
and watch them move.

A data set that fills the space its declared population allowed for does not
slow down and it does not warn you. It fails, and the database is down until
somebody reorganises that structure. One report tells you where you stand today.
A series of them tells you how long you have.

## Why XML

A printed report is read once and thrown away. This one loads.

Into Excel, straight, for sorting by saturation and filtering out the structures
you do not care about. Or into MySQL or MariaDB, which is what the 2020 revision
was for:

```sql
LOAD XML INFILE 'SAMPLEDB.xml'
INTO TABLE ispecs
ROWS IDENTIFIED BY '<DATASET>';
```

Run it monthly into the same table and you have a history. Then the question
stops being *how full is CUSTOMER* and becomes *how many months until it is
full*, which is the one worth asking:

```sql
SELECT `STR-NAME`,
       MAX(`ACTIVE-RECORDS`) - MIN(`ACTIVE-RECORDS`) AS growth,
       MAX(`ACTIVE-RECORDS`) / MAX(`POPULATION`) * 100 AS saturation
FROM ispecs
GROUP BY `STR-NAME`
ORDER BY saturation DESC;
```

## The schema

One `<DATABASE>` element, one `<DATASET>` per standard data set:

```xml
<DATASET STR-NAME="CUSTOMER" NUM="5" SECTIONS="0" FAMILY="DBPACK01"
         SECTORS="24120" MB="4.14"
         AREAS-ALLOWED="340" AREAS-INUSE="24" PCTJ-INUSE="0.071"
         RECORDS-ALLOCATED="41880" RECORDS-DELETED="1204" PCTJ-DELETED="0.029"
         ACTIVE-RECORDS="40676" POPULATION="500000"/>
```

| Attribute | Reading it |
|---|---|
| `ACTIVE-RECORDS` / `POPULATION` | The ratio to watch. At 1.0 the structure is out of room and takes the database with it. Anything past 0.8 is a reorganisation to schedule rather than suffer. |
| `SECTORS`, `MB` | What it occupies on the pack. A sector is 180 bytes. |
| `AREAS-ALLOWED`, `AREAS-INUSE`, `PCTJ-INUSE` | Areas are preallocated extents: the ceiling, how many exist, and the ratio. |
| `RECORDS-ALLOCATED`, `RECORDS-DELETED`, `PCTJ-DELETED` | Slots created and slots freed. Deleted space stays with the structure until a reorganisation gives it back — dead weight you back up every night. |
| `SECTIONS` | Non-zero for sectioned structures. |

## It never opens the database

The input is the **description file**, not the database. The program walks the
structure list inside it, so it takes no locks, needs no window, and cannot
disturb anything that is running. On a production machine that is the whole
reason it is usable: run it at any hour, as often as you like.

## How to compile

From CANDE:

```
C SYMBOL/SDSANALYZER AS SDSANALYZER WITH DMALGOL
```

## How to run

From CANDE. With no file equation it reads `DESCRIPTION/<usercode>`:

```
RUN SDSANALYZER
```

To point it at another description file:

```
RUN SDSANALYZER;FILE DASDL(TITLE = <DESCRIPTION FILE TITLE>)
```

The report is written to `XML/STATUS/<database name>`.

## Sample output

`SYMBOL/SAMPLEDB.xml` is in this repository. The database is made up, with
figures consistent with each other, to show the shape:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<DATABASE DB-NAME="SAMPLEDB" UPDATE-LEVEL="7" REPORT-DATE="14/03/2025 09:22:41" >
<DATASET STR-NAME="GLOBALS" NUM="1" SECTIONS="0" FAMILY="DBPACK01" SECTORS="180" MB="0.03" AREAS-ALLOWED="2" AREAS-INUSE="1" PCTJ-INUSE="0.500" RECORDS-ALLOCATED="1" RECORDS-DELETED="0" PCTJ-DELETED="0.000" ACTIVE-RECORDS="1" POPULATION="1"/>
<DATASET STR-NAME="CUSTOMER" NUM="5" SECTIONS="0" FAMILY="DBPACK01" SECTORS="24120" MB="4.14" AREAS-ALLOWED="340" AREAS-INUSE="24" PCTJ-INUSE="0.071" RECORDS-ALLOCATED="41880" RECORDS-DELETED="1204" PCTJ-DELETED="0.029" ACTIVE-RECORDS="40676" POPULATION="500000"/>
...
</DATABASE>
```

`GLOBALS` sits at 1.0 by construction — the global data set holds exactly one
record and is declared for one. Every other structure at 1.0 is a problem.

## Related

[DBANALYZER](https://github.com/AdolfoCl/DBANALYZER) prints the same inventory
as a report, for when you want to read it rather than load it.

[dmsii-to-mariadb](https://github.com/AdolfoCl/dmsii-to-mariadb) is the other
half of knowing a DMSII database: these two tell you how much room is left, that
one tells you what shape it has, by compiling the DASDL into a relational schema.

## History

Written in 2007 and maintained since: percentages over 100 % in 2007, access by
full title in 2008, sectioned structures in 2009, and the XML made loadable in
2020.

## License

MIT — see [LICENSE](LICENSE).
