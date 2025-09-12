The function `ProcessUtility` is the main entry point for executing utility SQL commands in PostgreSQL. For queries user input at the PostgreSQL command line, the call chain is as follows:

1. User inputs a command in psql (e.g., `CREATE TABLE ...`, `DROP TABLE ...`, etc.).



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

