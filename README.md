<!-- README.md is generated from README.Rmd. Please edit that file -->



# dbcooper

<!-- badges: start -->
[![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![Travis build status](https://travis-ci.org/dgrtwo/dbcooper.svg?branch=master)](https://travis-ci.org/dgrtwo/dbcooper)
<!-- badges: end -->

The dbcooper package turns a database connection into a collection of functions, handling logic for keeping track of connections and letting you take advantage of autocompletion when exploring a database.

It's especially helpful to use when authoring database-specific R packages, for instance in an internal company package or one wrapping a public data source.

The package's name is a reference to the bandit [D.B. Cooper](https://en.wikipedia.org/wiki/D._B._Cooper).

## Installation

You can install the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("dgrtwo/dbcooper")
```

## Example

### Initializing the functions

The dbcooper package asks you to create the connection first. As an example, we'll use the Lahman baseball database packaged with dbplyr.


```r
library(dplyr)

lahman_src <- dbplyr::lahman_sqlite()
lahman_src
#> src:  sqlite 3.30.1 [/var/folders/wp/6jpw10dj1b13vw5n9bvf1dvc0000gn/T//Rtmpsbz05N/lahman.sqlite]
#> tbls: AllstarFull, Appearances, AwardsManagers, AwardsPlayers, AwardsShareManagers, AwardsSharePlayers, Batting, BattingPost,
#>   CollegePlaying, Fielding, FieldingOF, FieldingPost, HallOfFame, LahmanData, Managers, ManagersHalf, Master, Parks, People, Pitching,
#>   PitchingPost, Salaries, Schools, SeriesPost, sqlite_stat1, sqlite_stat4, Teams, TeamsFranchises, TeamsHalf
```

You set up dbcooper with the `dbc_init` function, passing it a prefix `lahman` that will apply to all the functions it creates.


```r
dbc_init(lahman_src, "lahman")
```

`dbc_init` then creates user-friendly accessor functions in your global environment. (You could also pass it an environment in which the functions will be created).

### Using database functions

`dbc_init` adds several functions when it initializes a database source. In this case, each will start with the `lahman_` prefix.

* `_list`: Get a list of tables
* `_tbl`: Access a table that can be worked with in dbplyr
* `_query`: Perform of a SQL query and work with the result
* `_execute`: Execute a query (such as a `CREATE` or `DROP`)
* `_src`: Retrieve a `dbi_src` for the database

For instance, we could start by finding the names of the tables in the Lahman database.


```r
lahman_list()
#>  [1] "AllstarFull"         "Appearances"         "AwardsManagers"      "AwardsPlayers"       "AwardsShareManagers"
#>  [6] "AwardsSharePlayers"  "Batting"             "BattingPost"         "CollegePlaying"      "Fielding"           
#> [11] "FieldingOF"          "FieldingPost"        "HallOfFame"          "LahmanData"          "Managers"           
#> [16] "ManagersHalf"        "Master"              "Parks"               "People"              "Pitching"           
#> [21] "PitchingPost"        "Salaries"            "Schools"             "SeriesPost"          "Teams"              
#> [26] "TeamsFranchises"     "TeamsHalf"           "sqlite_stat1"        "sqlite_stat4"
```

We can access one of these tables with `lahman_tbl()`, then put it through any kind of dplyr operation.


```r
lahman_tbl("Batting")
#> # Source:   table<Batting> [?? x 22]
#> # Database: sqlite 3.30.1 [/var/folders/wp/6jpw10dj1b13vw5n9bvf1dvc0000gn/T//Rtmpsbz05N/lahman.sqlite]
#>    playerID yearID stint teamID lgID      G    AB     R     H   X2B   X3B    HR   RBI    SB    CS    BB    SO   IBB   HBP    SH    SF
#>    <chr>     <int> <int> <chr>  <chr> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int>
#>  1 abercda…   1871     1 TRO    NA        1     4     0     0     0     0     0     0     0     0     0     0    NA    NA    NA    NA
#>  2 addybo01   1871     1 RC1    NA       25   118    30    32     6     0     0    13     8     1     4     0    NA    NA    NA    NA
#>  3 allisar…   1871     1 CL1    NA       29   137    28    40     4     5     0    19     3     1     2     5    NA    NA    NA    NA
#>  4 allisdo…   1871     1 WS3    NA       27   133    28    44    10     2     2    27     1     1     0     2    NA    NA    NA    NA
#>  5 ansonca…   1871     1 RC1    NA       25   120    29    39    11     3     0    16     6     2     2     1    NA    NA    NA    NA
#>  6 armstbo…   1871     1 FW1    NA       12    49     9    11     2     1     0     5     0     1     0     1    NA    NA    NA    NA
#>  7 barkeal…   1871     1 RC1    NA        1     4     0     1     0     0     0     2     0     0     1     0    NA    NA    NA    NA
#>  8 barnero…   1871     1 BS1    NA       31   157    66    63    10     9     0    34    11     6    13     1    NA    NA    NA    NA
#>  9 barrebi…   1871     1 FW1    NA        1     5     1     1     1     0     0     1     0     0     0     0    NA    NA    NA    NA
#> 10 barrofr…   1871     1 BS1    NA       18    86    13    13     2     1     0    11     1     0     0     0    NA    NA    NA    NA
#> # … with more rows, and 1 more variable: GIDP <int>

lahman_tbl("Batting") %>%
  count(teamID, sort = TRUE)
#> # Source:     lazy query [?? x 2]
#> # Database:   sqlite 3.30.1 [/var/folders/wp/6jpw10dj1b13vw5n9bvf1dvc0000gn/T//Rtmpsbz05N/lahman.sqlite]
#> # Ordered by: desc(n)
#>    teamID     n
#>    <chr>  <int>
#>  1 CHN     4961
#>  2 PHI     4869
#>  3 PIT     4817
#>  4 SLN     4766
#>  5 CIN     4641
#>  6 CLE     4590
#>  7 BOS     4421
#>  8 CHA     4381
#>  9 NYA     4374
#> 10 DET     4315
#> # … with more rows
```

If we'd rather write SQL in some case than write, we could also run `lahman_query()` (which can also take a filename).


```r
lahman_query("SELECT
                playerID,
                sum(AB) as AB
              FROM Batting
              GROUP BY playerID")
#> # Source:   SQL [?? x 2]
#> # Database: sqlite 3.30.1 [/var/folders/wp/6jpw10dj1b13vw5n9bvf1dvc0000gn/T//Rtmpsbz05N/lahman.sqlite]
#>    playerID     AB
#>    <chr>     <int>
#>  1 aardsda01     4
#>  2 aaronha01 12364
#>  3 aaronto01   944
#>  4 aasedo01      5
#>  5 abadan01     21
#>  6 abadfe01      9
#>  7 abadijo01    49
#>  8 abbated01  3044
#>  9 abbeybe01   225
#> 10 abbeych01  1756
#> # … with more rows
```

Finally, `lahman_execute()` is for commands like `CREATE` and `DROP` that don't return a table, but that .


```r
lahman_execute("CREATE TABLE Players AS
                  SELECT playerID, SUM(AB) AS AB
                  FROM Batting
                  GROUP BY playerID")
#> [1] 0

lahman_execute("DROP TABLE Players")
#> [1] 0
```

### Autocompleted tables

Besides the `_list`, `_tbl`, `_query`, and `_execute` functions, the package also creates auto


```r
# Same result as lahman_tbl("Batting")
lahman_Batting()
#> # Source:   table<Batting> [?? x 22]
#> # Database: sqlite 3.30.1 [/var/folders/wp/6jpw10dj1b13vw5n9bvf1dvc0000gn/T//Rtmpsbz05N/lahman.sqlite]
#>    playerID yearID stint teamID lgID      G    AB     R     H   X2B   X3B    HR   RBI    SB    CS    BB    SO   IBB   HBP    SH    SF
#>    <chr>     <int> <int> <chr>  <chr> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int>
#>  1 abercda…   1871     1 TRO    NA        1     4     0     0     0     0     0     0     0     0     0     0    NA    NA    NA    NA
#>  2 addybo01   1871     1 RC1    NA       25   118    30    32     6     0     0    13     8     1     4     0    NA    NA    NA    NA
#>  3 allisar…   1871     1 CL1    NA       29   137    28    40     4     5     0    19     3     1     2     5    NA    NA    NA    NA
#>  4 allisdo…   1871     1 WS3    NA       27   133    28    44    10     2     2    27     1     1     0     2    NA    NA    NA    NA
#>  5 ansonca…   1871     1 RC1    NA       25   120    29    39    11     3     0    16     6     2     2     1    NA    NA    NA    NA
#>  6 armstbo…   1871     1 FW1    NA       12    49     9    11     2     1     0     5     0     1     0     1    NA    NA    NA    NA
#>  7 barkeal…   1871     1 RC1    NA        1     4     0     1     0     0     0     2     0     0     1     0    NA    NA    NA    NA
#>  8 barnero…   1871     1 BS1    NA       31   157    66    63    10     9     0    34    11     6    13     1    NA    NA    NA    NA
#>  9 barrebi…   1871     1 FW1    NA        1     5     1     1     1     0     0     1     0     0     0     0    NA    NA    NA    NA
#> 10 barrofr…   1871     1 BS1    NA       18    86    13    13     2     1     0    11     1     0     0     0    NA    NA    NA    NA
#> # … with more rows, and 1 more variable: GIDP <int>

# Same result as lahman_tbl("Master") %>% count()
lahman_Master() %>%
  count()
#> # Source:   lazy query [?? x 1]
#> # Database: sqlite 3.30.1 [/var/folders/wp/6jpw10dj1b13vw5n9bvf1dvc0000gn/T//Rtmpsbz05N/lahman.sqlite]
#>       n
#>   <int>
#> 1 19617
```

These are useful because they let you use autocomplete to complete table names as you're exploring a data source.

## Code of Conduct

Please note that the 'dbcooper' project is released with a
[Contributor Code of Conduct](CODE_OF_CONDUCT.md).
By contributing to this project, you agree to abide by its terms.

