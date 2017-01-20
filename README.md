# Welcome to Linked Data-Fu!

## Introduction

Linked Data-Fu is a declarative data processing system that can be used in data
integration and system interoperation scenarios.  Linked Data-Fu is based on a parallel
dataflow architecture that is capable of processing streams of triples, and evaluating
queries (a subset of SPARQL) and programs based on "if-then" rules.  The system can
access input data that is either available locally in files or accessible via HTTP as
Linked Data.

## TL;DR

Use:

    $ bin/ldfu.sh -p examples/dbpedia/get-timbl-awards.n3 rulesets/rdf.n3 rulesets/rdfs.n3 -o get-timbl-awards.nq

to evaluate program 'examples/dbpedia/get-timbl-awards.n3' consisting of an atomic HTTP
GET request and a request rule specifying which links to follow, while taking into account
the RDF and RDFS rulesets.  The results of program evaluation are serialised in N-Quads format
to the file 'get-timbl-awards.nq'.

## Parsing Files

Use:

    $ bin/ldfu.sh -i examples/lubm-1.nt > output

to parse the N-Triples file 'examples/lubm-1.nt' while streaming the resulting
triples to standard output (stdout).  '> output' redirects stdout to a file 'output'.

Supported formats for input streams are:

* RDF/XML (.rdf)
* N-Triples (.nt)
* N-Quads (.nq)
* Turtle (.ttl)
* JSON-LD (.jsonld)
* RDF/A (.html, .xhtml)

You can use URIs of RDF files.  If you specify multiple files via the '-i' option,
the files are multiplexed into a single stream.

If there is no query specified (see next paragraph on how to specify a query), the
output consists of all triples in N-Quads format.

## Evaluating Queries

Based on the input streams of triples, Linked Data-Fu can evaluate multiple queries
and either output streams of triples (via SPARQL CONSTRUCT queries) or streams of
solutions (via SPARQL SELECT queries).

On the stream you can register queries specified in SPARQL syntax.  Given the complexities
around combining queries and rules (see next paragraph), Linked Data-Fu just supports queries
with a single basic graph pattern in the WHERE clause.

CONSTRUCT queries return a stream of triples.
SELECT queries return a stream of tuples (SPARQL solutions).

Use:

    $ bin/ldfu.sh -i examples/lubm-1.nt -q examples/lubm-q1.rq lubm-q.tsv

to evaluate the query 'examples/lubm-q1.rq' and print the results in TSV format to
lubm-q.tsv.

Supported formats for SPARQL triple output streams (CONSTRUCT queries) are:

* RDF/XML (.rdf)
* N-Triples (.nt)
* N-Quads (.nq)

Supported formats for SPARQL solution output streams (SELECT queries) are:

* XML (.srx)
* JSON (.srj)
* HTML (.html)
* TSV (.tsv)

## Evaluating Programs

A program specifies an initial state consisting of input streams.  The program may
also specify the following state using rules:

* Derivation rules: if a given pattern holds on the data, then assume that another
  pattern holds.
* Request rules: if a given pattern holds on the data, then perform a HTTP request.

Programs itself are written in an extension of RDF, in a syntax called
Notation3 (N3).

### Evaluating a Derivation Rule Program

You can add specialised rulesets consisting of derivation rules to query evaluation.

Use:

    $ bin/ldfu.sh -p rulesets/rdfs.n3 -i examples/lubm-1.nt -q examples/lubm-q1.rq -

to evaluate the SPARQL SELECT query 'examples/lubm-q1.rq' over the data 'examples/lubm-1.nt'
and output the results to stdout, but take a subset of the RDFS semantics into account.

### Evaluating a Request Rule Program

Linked Data-Fu also supports the retrieval of Linked Data during runtime.  The next
program contains an atomic HTTP request, and request rules that specify which sources
should be dereferenced.

Use:

    $ bin/ldfu.sh -p examples/dbpedia/get-timbl-awards.n3 -o get-timbl-awards.nq

to evaluate program 'examples/dbpedia/get-timbl-awards.n3' consisting of an atomic HTTP
GET request and a request rule specifying which links to follow.  The results of program
evaluation are serialised in N-Quads format to the file 'get-timbl-awards.nq'.

Use:

    $ bin/ldfu.sh -p examples/dbpedia/get-timbl-awards.n3 rulesets/rdfs-plus.n3 -o get-timbl-awards.nq

to evaluate the program under RDFS Plus semantics.  Now, many more HTTP requests
are carried out, as program evaluation takes into account owl:sameAs and other
constructs.

You may also specify SPARQL queries via the -q option:

    -q examples/dbpedia/timbl-awards.rq timbl-awards.srx

evaluates the SPARQL SELECT query 'examples/dbpedia/timbl-awards.rq' and outputs the
results to the file 'timbl-awards.srx' in XML SPARQL result set format.  The input
data is either specified via the '-i' option or via HTTP requests as part of a program.

## Builtin Functions

There is support for various built-in filters:
log:equalTo, log:notEqualTo, math:equalTo, math:greaterThan, math:lessThan,
math:notEqualTo, math:notGreaterThan, math:notLessThan, string:contains,
string:containsIgnoringCase, string:containsRoughly, string:endsWith,
string:equalIgnoringCase, string:greaterThan, string:lessThan,
string:notContainsRoughly, string:notEqualIgnoringCase, string:notGreaterThan,
string:notLessThan, string:startsWith.

and functions:

log:dtlit, log:stringToUri, log:uriToString, math:difference,
math:exponentiation, math:product, math:quotient, math:sqrt, math:sum,
string:concatenation, string:replace.

See http://www.w3.org/2000/10/swap/doc/CwmBuiltins for a description of
the various functions.

See examples/euclidean-distance.n3 for a program that uses builtins for
calculation.

## Configuration

### Memory Issues

Use:

    $ export JAVA_OPTS="-Xmx8G -Xms4G"
    $ bin/ldfu.sh

Or:

    $ JAVA_OPTS="-Xmx8G -Xms4G" bin/ldfu.sh

to reserve 8G of heap memory (start with 4G heap memory reserved) to the Java Virtual
Machine.  Linked Data-Fu relies on main memory a lot: the more memory you give to
the JVM, the better.

### Parsing Performance

Especially when running queries over large local files, the parsing step is a bottleneck.
The N-Triples parser is fastest, followed by the Turtle parser.  The RDF/XML parser is
the slowest.  So, use N-Triples as RDF format for files if possible.

Also, parsing is one thread per file.  If you split up files, and feed them into multiple
arguments for '-i', parsing can be done in parallel, resulting in a speedup.

### Proxy

To specify a HTTP proxy, use the http\_proxy environment variable.  The scripts
in bin/ should grok http\_proxy.  Assuming there is a local proxy available, use:

    $ export http_proxy="http://localhost:3128/"

### Logging

The shell scripts in bin/ take conf/logging.properties into account.  Next to the
general logging, there is a logger "edu.kit.aifb.datafu.stderr".  The logger
"edu.kit.aifb.datafu.stderr" receives high-level status log entries.

### Configuration

For fine-grained configuration, check out the config files in conf/ that can
be used in conjunction with the '-c' command line option.

You can also use the properties on the command line.
The following sets the maximum routes per host to 16.

    $ JAVA_OPTS=-Dldfu.http.maxroutesperhost=16 bin/ldfu.sh

With the '-v' option set, ldfu prints the current settings for all properties.

For also allowing parallel access, you also need to specify a different queue
implementation TBD.

## Mailing List

For discussion of Linked Data-Fu use the mailing list at <a href="https://groups.google.com/forum/#!forum/ldfu-discuss">Google Groups</a>.
Subscribe via email to ldfu-discuss+subscribe@googlegroups.com.