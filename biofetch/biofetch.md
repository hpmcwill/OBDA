BioFetch - Light weight universal translator
============================================

This is a document for a standard Web interface to retrieve entries
from biological databases using unique identifiers.

The capabilities of the program/server are not covered. The programs
calling BioFetch CGI are assumed to get details (e.g what databases
are available, what formats are supported, how many entries can be
retrieved at one go) from a separate registry.

Justification
-------------

Biological databases are large and there are several common ways of
indexing (SRS, EMBL indexes, RDB, Berkeley DB) them for random
access. While various approaches to unify sequence access (CORBA,
SOAP, NOVELLA). All bio* projects implement parsing of sequence
database entry parsing in their original format. Likewise all relevant
programming languages have standard ways of accessing documents over
the Web. Taken together, these two facts make it easy to build a
portable system of retrieving any entries transparently from up to
date databases.

The main purpose of this document is to define the parameters to a
(CGI) program and its response. The internal details of the program
are dependent on implementation. Also, the indexing system used to
serve the entries, is not specified.

Program Behaviour
-----------------

In addition to GET method the BioFetch should be able to recieve its
parameters from POST method, too.

The BioFetch program should return entries only if they match the ID
string and other attributes completely. If any of the IDs submitted
does not find the match in the query database, an error reporting the
cause should be thrown and no entries returned.

Apart from content type definition, the returned string should not
contain any characters which are not part of the entry format.

Sequences in EMBL/GenBank/DDBJ public nucleotide sequence databases
share three identifiers: ID, accession and accession.version. If a
specific version is requested, only that should be returned.

The program is guaranteed to return only one, latest matching version
of the entry for each ID. This is important in cases where the
database and its update database are indexed separately (e.g. EMBL and
EMBLNEW).

The next section of the document deals with the URL and parameter
structure for the BioFetch program.

URL
---

This interface does not specify what happens when biofetch is called
in interactive context. The implementations can return the entries
decorated with HTML tags and hypertext links.

A URL for biofetch consists of four sections:

1. Protocol:        `http://`
2. Host and port:   `www.ebi.ac.uk:80`
3. Path to program: `/cgi-bin/dbfetch`
4. Query string:    `?style=raw;format=embl;db=embl;id=J00231`

Query String
------------

The query string options are separated from the base URL (protocol +
host + path) by a question mark (?) and from each other by a semicolon
';' (or by ampersand '&'). See CGI GET documents at
http://www.w3.org/CGI/). The order of options is not critical. It is
recommended to leave the ID to be the last item.

Input for options should be case insensitive.

#### Option: db

| Option | db                                      |
|--------|-----------------------------------------|
| Descr  | database name                           |
| Type   | required                                |
| Usage  | `db=embl` or `db=genbank` or `db=swall` |
| Arg    | string                                  |

#### Option: style

| Option | style
|--------|-----------------------------------|
| Descr  | +/- HTML tags                     |
| Type   | required                          |
| Usage  | `style=raw` or `style=html`       |
| Arg    | enum (raw or html)                |

In non-interactive context, always give `style=raw`. This uses
"Content-Type: text/plain". If other content types are needed (XML),
this part of the specifications can be extended to accommodate them.

#### Option: format

| Option | format                                   |
|--------|------------------------------------------|
| Descr  | format of the database entries returned  |
| Type   | optional                                 |
| Usage  | `format=embl` or `format=fasta`          |
| Arg    | enum                                     |

Format defaults to the distribution format of the database (`embl` for
EMBL database). If some other supported format is needed this option
is needed (E.g. formats for EMBL: `fasta`, `bsml`, `agave`).

#### Option: id

| Option | id                                |
|--------|-----------------------------------|
| Descr  | unique database identifier(s)     |
| Type   | required                          |
| Usage  | `db=J00231` or `id=J00231+BUM`    |
| Arg    | string                            |

The ID option should be able to process all UIDS in a database. It
should not be necessary to know if the UID is an ID, accession number
or accession.version.

The number of entry UIDs allowed is implementation specific. If the
limit is exceeded, the the program reports an error. The UIDs should
be separated by spaces (use '+' in a GET method string).

Error Messages
--------------

The following standardized one line messages should be printed out in
case of an error.

* `ERROR 1 Unknown database [$db].`
* `ERROR 2 Unknown style [$style].`
* `ERROR 3 Format [$format] not known for database [$db].`
* `ERROR 4 ID [$id] not found in database [$db].`
* `ERROR 5 Too many IDs [$count]. Max [$MAXIDS] allowed.`

Implementations
---------------

The first implementation of the BioFetch server is 'dbfetch', a perl
CGI program. It is distributed as part of the Bioperl package in the
directory 'scripts/DB':

https://github.com/bioperl/bioperl-live/blob/master/examples/db/dbfetch

The URL to dbfetch running at EMBL-EBI is:

http://www.ebi.ac.uk/Tools/dbfetch/dbfetch

The EMBL-EBI service is used in Bioperl by specific Bio::DB::BioFetch 
modules as well as more generic Bio::DB::EMBL, Bio::DB::Swissprot and
Bio::DB::RefSeq modules. 

An BioRuby biofetch server implementation can be found at:

https://github.com/bioruby/bioruby/blob/master/sample/biofetch.rb
