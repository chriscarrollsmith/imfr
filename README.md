imfr
====================================

[![CRAN Version](http://www.r-pkg.org/badges/version/imfr)](https://cran.r-project.org/package=imfr)
[![Build Status](https://travis-ci.org/christophergandrud/imfr.svg?branch=master)](https://travis-ci.org/christophergandrud/imfr)
![CRAN Monthly Downloads](http://cranlogs.r-pkg.org/badges/last-month/imfr)
![CRAN Total Downloads](http://cranlogs.r-pkg.org/badges/grand-total/imfr)

R package for interacting with the [International Monetary Funds's](http://data.imf.org/) [RESTful JSON API](http://datahelp.imf.org/knowledgebase/articles/667681-using-json-restful-web-service).

# How to download IMF data

You can use the `imf_data` function to download the data the IMF makes available via its API. To do this you will need at least the following information:

- `database_id`: the ID of the specific database you wish to download the data series from. You can find the list of IDs and their description using the `imf_ids` function.

- `indicator`: the IMF indicators of the variables you want to download. One way to find these is to:

    1. Use the `database_id` for the database you want to access with the `imf_codelist` function to find the code list of the database.

    2. Then using the indicator code (usually `CL_INDICATOR_database_id`) in `imf_codes`, you can find the data series indicator codes in that database.

    *Tip*: if you have a number of country identifiers that are not in ISO2C format, you can use the helpful [countrycode]( https://cran.r-project.org/package=countrycode) package to convert them.

- `country`: one or more [ISO two letter country codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) for the countries you would like to download the data for. If `country = 'all'` then all available countries will be downloaded.

- `start` and `end`: the start and end years for which you would like to download the data.

- `freq`: the frequency of the series you want to download. Often series are available annually, quarterly, and monthly.

## Examples

### Simple Country-Time-Variable

Imagine that we want to download Effective Exchange Rate (CPI base) for China and the UK for 2013:


```r
library(imfr)

real_ex <- imf_data(database_id = 'IFS', indicator = 'EREER_IX',
                    country = c('CN', 'GB'), freq = 'A',
                    start = 2013, end = current_year())
```




```r
real_ex
```

```
##   iso2c year EREER_IX
## 1    CN 2013 115.2938
## 2    GB 2013 105.7680
```

### More complex data formats

While many quantities of interest from the IMF database are in simple 
country-time-variable format, many are not. For example, Direction of Trade 
Statistics include country-year-variable and a "counterpart area". By default,
`imf_data` would only return the first, but not the last. 

Because of the many possible data structures available from the imf, `imf_data`
allows you to return the entire API call as a list. From this list you can then
extract the requested data. To do this use the `return_raw = TRUE` argument, e.g.:


```r
data_list <- imf_data(database_id = "DOT", indicator = "TXG_FOB_USD", 
                      country = "US", return_raw = TRUE)
```



Then extract the data series (it is typically contained in `CompactData$DataSet$Series`):


```r
data_df <- data_list$CompactData$DataSet$Series

names(data_df)
```

```
## [1] "@FREQ"             "@REF_AREA"         "@INDICATOR"       
## [4] "@COUNTERPART_AREA" "@UNIT_MULT"        "@TIME_FORMAT"     
## [7] "Obs"
```

You can then subset and clean up `data_df` to suit your purposes.


