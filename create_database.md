This journal records how PostgreSQL handles the command `CREATE DATABASE...` .

The function `ProcessUtility` is the main entry point for executing utility SQL commands in PostgreSQL. For queries user input at the PostgreSQL command line, the call chain is as follows:

1. User inputs `CREATE DATABASE test;`  in psql.
2. `src/backend/tcop/postgres.c->exec_simple_query(const char *query_string)` accepts SQL string as parameter;

At the beginning of `exec_simple_query(const char *query_string)` , these variables are declared:

```c
CommandDest dest = whereToSendOutput;
MemoryContext oldcontext;
List *parsetree_list;
ListCell *parsetree_item;
bool save_log_statement_stats = log_statement_stats;
bool was_logged = false;
bool use_implicit_block;
char msec_str[32];
```

After variables declaration it saves `query_string` to global variable `debug_query_stging` . And invokes

```c
pgstat_report_activity(STATE_RUNNING, query_string);
TRACE_POSTGRESQL_QUERY_START(query_string);
```

Let's look into this function someday.

Next, it starts up a transaction command.

```c
start_xact_command();
```

And zap any pre-existing unnamed statement.

```c
drop_unnamed_stmt();
```

Switch to appropriate context for constructing `parsetrees`.

```c
oldcontext = MemoryContextSwitchTo(MessageContext);
```

The preceding steps consitute the preparatory phase; the following step is of pratical significance.

```c
parsetree_list = pg_parse_query(query_string);
```

Assume, everything is good, the next switches back to transaction context to enter the loop.

```c
MemoryContextSwitchTo(oldcontext);
```



The function

```c
List * raw_parser(const char *str, RawParseMode mode)
```

how does it work?

Firstly, it declares three variables:

```c
core_yyscan_t yyscanner;
base_yy_extra_type yyextra;
int yyresult;
```

Step 2, it invokes

```c
yyscanner = scanner_init(str, &yyextra.core_yy_extra, &ScanKeywords, ScanKeywordTokens);
```

Step 3, it initialize the bison parser

``` c
parser_init(&yyextra);
```

Setp 4, it parses

```c
yyresult = base_yyparse(yyscanner);
```

Step 5, it clean up

```c
scanner_finish(yyscanner);
```

Step 6, it returns the tree of parsed string

```c
return yyextra.parsetree;
```

