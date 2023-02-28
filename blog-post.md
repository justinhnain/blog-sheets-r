---
title: "Pivoting Google Sheets spreadsheets with R"
categories:
  - Blog
tags:
  - R
  - Google Sheets
  - Tidyverse
---
## Summarising our data

Often, we must summarise our spreadsheets with the help of [pivot
tables](https://support.google.com/docs/answer/1272900?hl=en&co=GENIE.Platform%3DDesktop).
Pivot tables are designed to analyze and summarize data, but they rely
on data being structured in a specific way to do so effectively. Pivot
tables require data to be in a **long-form format**, where each
observation is a separate row, and each variable is a separate column.
However, data is often entered in **wide-form format** for ease of data
entry.

If the data is in wide-form format rather than long-form format, we will
encounter issues like this when we create pivot tables:

<div style="text-align: center;">

<video width="640" height="480" controls>
<source src="assets/images/pivoting-google/untidy.mp4" type="video/mp4"> Your browser does not support
the video tag. </video>

</div>

We’re unable to easily break down the yearly average market share for
each vendor, because each vendor is spread widely over many columns.
Additionally, our analysis is inflexible because it’s extremely time
consuming and error-prone to switch to a different summary, like the
yearly max market share for each vendor

To proceed to summarise our data, we need our data to be in long-form,
where those vendors belong to a single column.

## Wide and long data, and some reason for why they exist

Wide-form data is convenient for data entry, which is what we often do
in spreadsheets. However, when the time comes to summarise the data, we
may find ourselves at an impasse.

Long-form data provides a structured and organized format that is ideal
for data analysis and manipulation. In addition to being essential for
pivot tables, long-form data is easier to work with and less prone to
errors than wide-form data. With long-form data, each observation is a
separate row, and each variable is a column, making it easier to compute
with because many environments expect long form data.

Wide-form is useful for a few things, such as cross tables and data
entry, but it’s not suitable for pivot tables. By understanding the
importance of long-form data and using R’s tidyverse to manipulate and
transform the data, users can create more effective pivot tables and
make better-informed decisions.

The long-form format is essential for creating pivot tables in Google
Sheets.

![[Tidy](https://www.jstatsoft.org/article/view/v059i10) long-form data
vs wide-form data.](assets/images/pivoting-google/tidy.png)

## Why we’ll pivot from wide to long with R rather than Sheets

As the data matures, it’s normal to evolve from wide-form data (in the
data entry phase), to long-form data (in the data summarisation phase).
So then, it is disconcerting that Google Sheets is woefully unfit for
the task of pivoting from wide to long.

Google Sheets offers some basic features for data transformation, but it
can be inadequate for transforming data from a wide-form to a long-form
format. One of the main reasons is that Google Sheets lacks automated
tools for data cleaning, transformation, and manipulation.

Converting data from a wide-form to a long-form format can be a
time-consuming and error-prone process when done manually with Sheets.
For example, it often involves copying and pasting data, manually
creating new columns and rows, and deleting unwanted data. These manual
processes can be cumbersome, especially when dealing with large and
complex datasets.

In contrast, R’s [tidyverse](https://tidyverse.org) provides a suite of
powerful tools for data transformation and manipulation. The
[tidyr](https://tidyr.tidyverse.org/) package, in particular, offers
functions for converting data between wide-form and long-form formats,
making it much easier to restructure data for analysis. The package also
offers tools for cleaning, filtering, and transforming data, allowing
users to automate many of the manual processes that can be
time-consuming and error-prone in Google Sheets.

## The googlesheets4 library

The [googlesheets4](https://googlesheets4.tidyverse.org/) library is an
R package in the tidyverse that provides a simple and powerful interface
for interacting with Google Sheets directly from R. With googlesheets4,
users can easily read, write, and manipulate data in Google Sheets
without leaving the R environment.

One of the key features of googlesheets4 is its support for the latest
Google Sheets API. With this support, users can easily import data from
Google Sheets into R for further analysis, using tools from tidyr and
[dplyr](https://dplyr.tidyverse.org/) to manipulate the data as needed.

Once the data has been imported into R, users can use the full range of
R’s data manipulation tools to clean the data. What we’re interested in
here is pivoting from wide-form to long-form, which allows users to
summarize data by different variables and attributes. With
googlesheets4, users can easily pivot data in R and then export the
results back to Google Sheets.

## Install and load packages.

To install googlesheets4 and other tidyverse packages, input
`install.packages("tidyverse")` at your R session’s command prompt.

To load googlesheets4, and the R package’s we’ll use in this session,
input this similarly:

``` r
library(googlesheets4)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.4.0           ✔ purrr   1.0.1      
    ## ✔ tibble  3.1.8           ✔ dplyr   1.0.99.9000
    ## ✔ tidyr   1.3.0           ✔ stringr 1.5.0      
    ## ✔ readr   2.1.3           ✔ forcats 0.5.2      
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

## The workflow

### Import from Sheets to R

Here is the spreadsheet we’ll be pivoting from wide to long:
[<link>](https://docs.google.com/spreadsheets/d/1QG_9gkWtciEmHggb6BjLg-6hF9FyekzKN9cLjyteh00/edit#gid=1469251738&fvid=2032560885)

With the `read_sheet` function, users can easily specify the name of the
Google Sheet they want to import data from and the range of cells they
want to import. This makes it easy to work with large and complex
datasets in R, without the need to manually download and import the data
into R. Additionally, the `read_sheet()` function supports various data
types and automatically detects and converts data types as needed.

In your analyses, you’ll often need to tweak your inputs a bit. In this
case, we instruct the program to read from the data located in the first
worksheet in the workbook, in the table ranging over B3 to Z123.

``` r
URL_untidy <- "https://docs.google.com/spreadsheets/d/1QG_9gkWtciEmHggb6BjLg-6hF9FyekzKN9cLjyteh00/edit#gid=1469251738&fvid=2032560885"

df <- read_sheet(URL_untidy, sheet = 1, range = "B3:Z123")
```

    ## ✔ Reading from "Untidy Excel table".

    ## ✔ Range ''Top 20 Vendors'!B3:Z123'.

``` r
df
```

    ## # A tibble: 120 × 25
    ##    Date     Year Month Alcatel Apple  Asus   BBK Blackberry Gener…¹ Google   HTC
    ##    <chr>   <dbl> <chr>   <dbl> <dbl> <dbl> <dbl>      <dbl>   <dbl>  <dbl> <dbl>
    ##  1 2010-07  2010 Jul         0  30.1     0     0       19.0       0  0.21   0.07
    ##  2 2010-08  2010 Aug         0  29.8     0     0       19.7       0  0.19   0.07
    ##  3 2010-09  2010 Sep         0  26.7     0     0       21.0       0  0.17   0.06
    ##  4 2010-10  2010 Oct         0  26.5     0     0       21.5       0  0.16   0.05
    ##  5 2010-11  2010 Nov         0  25.7     0     0       22.6       0  0.150  0.04
    ##  6 2010-12  2010 Dec         0  28.0     0     0       21.5       0  0.15   0.04
    ##  7 2011-01  2011 Jan         0  30.2     0     0       18.2       0  0.14   0.09
    ##  8 2011-02  2011 Feb         0  29.8     0     0       17.6       0  0.13   0.15
    ##  9 2011-03  2011 Mar         0  29.8     0     0       17.2       0  0.13   0.17
    ## 10 2011-04  2011 Apr         0  28.6     0     0       16.6       0  0.11   0.18
    ## # … with 110 more rows, 14 more variables: Huawei <dbl>, Lenovo <dbl>,
    ## #   LG <dbl>, Micromax <dbl>, Mobicel <dbl>, Motorola <dbl>, Nokia <dbl>,
    ## #   Oppo <dbl>, Samsung <dbl>, Sony <dbl>, Xiaomi <dbl>, ZTE <dbl>,
    ## #   Other <dbl>, Unknown <dbl>, and abbreviated variable name ¹​`General Mobile`

The imported dataframe is in wide-form format.

### Pivot the data in R to long-form

To pivot the data imported with googlesheets4, users can use the tidyr
package, which offers a range of functions for manipulating data in
long-form and wide-form formats. For example, users can use the
[`pivot_longer()`](https://tidyr.tidyverse.org/reference/pivot_longer.html)
function to convert wide-form data to long-form data, making it easier
to work with in R. Users can also use the
[`pivot_wider()`](https://tidyr.tidyverse.org/reference/pivot_wider.html)
function to pivot the data back to wide-form.

``` r
df <- df |> 
  pivot_longer(
    cols      = 4:ncol(df),
    names_to  = "Vendor",
    values_to = "Market Share"
  )

df
```

    ## # A tibble: 2,640 × 5
    ##    Date     Year Month Vendor         `Market Share`
    ##    <chr>   <dbl> <chr> <chr>                   <dbl>
    ##  1 2010-07  2010 Jul   Alcatel                  0   
    ##  2 2010-07  2010 Jul   Apple                   30.1 
    ##  3 2010-07  2010 Jul   Asus                     0   
    ##  4 2010-07  2010 Jul   BBK                      0   
    ##  5 2010-07  2010 Jul   Blackberry              19.0 
    ##  6 2010-07  2010 Jul   General Mobile           0   
    ##  7 2010-07  2010 Jul   Google                   0.21
    ##  8 2010-07  2010 Jul   HTC                      0.07
    ##  9 2010-07  2010 Jul   Huawei                   0   
    ## 10 2010-07  2010 Jul   Lenovo                   0   
    ## # … with 2,630 more rows

With only a few lines of code, the data has automatically been made
long. This data is now suitable for a pivot table.

### Export from R to Sheets

Once the data has been transformed in R, users can use the write_sheet()
function in googlesheets4 to export the data back to Google Sheets. This
allows them to share the results with other team members or stakeholders
and collaborate on further analysis or reporting.

We begin by creating a destination spreadsheet.

``` r
gs4_create("tidy_spreadsheet")
```

    ## ✔ Creating new Sheet: "tidy_spreadsheet".

After locating the spreadsheet’s address, we record the address and then
export to that address.

``` r
URL_tidy <- "https://docs.google.com/spreadsheets/d/11dzJ6Mff6eFhl5h6QO0NK2PYDs8zmutvSh1qX-PFe7U/edit#gid=0"
write_sheet(data = df, ss = URL_tidy, sheet = 1)
```

    ## ✔ Writing to "tidy_spreadsheet".

    ## ✔ Writing to sheet 'Sheet1'.

### Outcome and summary

The data’s completely tidy, and we can now easily summarise it to
compare average market share with pivot tables

<div style="text-align: center;">

<video width="640" height="480" controls>
<source src="assets/images/pivoting-google/tidy_sheet.mp4" type="video/mp4"> Your browser does not
support the video tag. </video>

</div>

Overall, the googlesheets4 library provides an accessible roue for
transforming wide-form data into long-form data in Google Sheets. Its
support for the latest Google Sheets API and seamless integration with
R’s powertools make it an ideal choice for the multitude of people who
need to work with Google Sheets data.

### References & Links

- Tidy: <https://www.jstatsoft.org/article/view/v059i10> - link to an
  article comparing long-form data vs wide-form data.
- Tidyverse: <https://tidyverse.org> - link to the tidyverse website,
  which provides a suite of powerful tools for data transformation and
  manipulation.
- tidyr: <https://tidyr.tidyverse.org/> - link to the tidyr package,
  which offers functions for converting data between wide-form and
  long-form formats.
- dplyr: <https://dplyr.tidyverse.org/> - link to the dplyr package,
  which provides tools for cleaning, filtering, and transforming data.
- googlesheets4: <https://googlesheets4.tidyverse.org/> - link to the
  googlesheets4 package, which provides a simple and powerful interface
  for interacting with Google Sheets directly from R.
- Google Sheet example:
  <https://docs.google.com/spreadsheets/d/1QG_9gkWtciEmHggb6BjLg-6hF9FyekzKN9cLjyteh00/edit#gid=1469251738&fvid=2032560885> -
  link to a Google Sheet that will be used in the tutorial for pivoting
  from wide to long in R.
- pivot_longer():
  <https://tidyr.tidyverse.org/reference/pivot_longer.html> - link to
  the documentation for the pivot_longer() function in the tidyr
  package.
- pivot_wider():
  <https://tidyr.tidyverse.org/reference/pivot_wider.html> - link to the
  documentation for the pivot_wider() function in the tidyr package.
