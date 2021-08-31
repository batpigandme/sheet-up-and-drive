Up and running with googlesheets4
================

``` r
library(tidyverse)
library(googledrive)
library(googlesheets4)
```

First thing’s first, I need to upload the `.xls` file from [UIPM 2021
World Championship
Results](https://www.uipmworld.org/event/uipm-2021-pentathlon-world-championships)
to googledrive.

Here, I’ll use
[`drive_auth()`](https://googledrive.tidyverse.org/reference/drive_auth.html),
which allows you to either interactively select a pre-authorized account
in R, or takes you to the browser to generate obtain a new token for
your account. To learn more about authenticating your account in an R
Markdown document, see the [Non-interactive
auth](https://gargle.r-lib.org/articles/non-interactive-auth.html#sidebar-2-i-just-want-my-rmd-to-render)
article for [{gargle}](https://gargle.r-lib.org/).

``` r
drive_auth(email = "mara@rstudio.com")
gs4_auth(email = "mara@rstudio.com")
```

Now I can upload the Excel file to my Google Drive as a spreadsheet
using
[`googledrive::drive_upload()`](https://googledrive.tidyverse.org/reference/drive_upload.html).
If I don’t specify the `type` argument, a MIME type is automatically
determined from the file extension.

``` r
drive_upload(
  media = "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls"
)
#> Local file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#> Uploaded into Drive file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#>   <id: 1nY47mJWyxBjudUfjCJQoi6M5utWcAcHs>
#> With MIME type:
#> • 'application/vnd.ms-excel'
```

However, I can also specify a type (e.g. `type = "spreadsheet"`), which
will convert the file to its Google equivalent (in this case, a Google
Sheet) if the source file is a compatible type.

``` r
drive_upload(
  media = "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls",
  type = "spreadsheet"
)
#> Local file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#> Uploaded into Drive file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships'
#>   <id: 1v99zwZL_Qn8zazwQ9uBOjaI6brKtF8hTqvW714yuWbM>
#> With MIME type:
#> • 'application/vnd.google-apps.spreadsheet'
```

Now, let’s say I want to read my sheet into R to do some data wrangling
(it’s about as untidy as a dataset can get). I’ll use `drive_find()` to
find the sheet I want by name or keyword.

``` r
drive_find("Pentathlon")
#> # A dribble: 13 x 3
#>    name                                  id                      drive_resource 
#>    <chr>                                 <drv_id>                <list>         
#>  1 Competition_Results_Exports_UIPM_202… 1v99zwZL_Qn8zazwQ9uBOj… <named list [3…
#>  2 Competition_Results_Exports_UIPM_202… 1nY47mJWyxBjudUfjCJQoi… <named list [4…
#>  3 Competition_Results_Exports_UIPM_202… 1t6qD7vbZ_Zecf3grB_lar… <named list [3…
#>  4 Competition_Results_Exports_UIPM_202… 1kna136rerfq9-CcstX3Hp… <named list [4…
#>  5 Competition_Results_Exports_UIPM_202… 1WRBiAvq3u4s4JiQnTqeqD… <named list [3…
#>  6 Competition_Results_Exports_UIPM_202… 1lEXtLgz1AVh0WCiJOpT4r… <named list [4…
#>  7 Competition_Results_Exports_UIPM_202… 1bv4T90Xzh99YczWl_WlDC… <named list [3…
#>  8 Competition_Results_Exports_UIPM_202… 16ymDQTxInzGC_TyPHH1Ah… <named list [4…
#>  9 Competition_Results_Exports_UIPM_202… 15S_e4paHStmoz89V-NSEJ… <named list [3…
#> 10 Competition_Results_Exports_UIPM_202… 1volih0NsJh_g1Y19AwN84… <named list [4…
#> 11 Competition_Results_Exports_UIPM_202… 16aS_6_bkury6qK0mirANG… <named list [4…
#> 12 Competition_Results_Exports_UIPM_202… 1pKhkuWa0H7Lnq9X_gefsX… <named list [4…
#> 13 UIPM_Competition_Results_Exports_UIP… 1qfDtXN2FO7O412pBxdXlJ… <named list [3…
```

Since I know that I want the one with the name we used, and the one
that’s in the native Google Sheet format (also the most recent), I can
retrieve the sheet by `id`. (The `id` was also shown to us when I
uploaded the file.)

``` r
pentathlon_ss <- drive_get(id = "1bv4T90Xzh99YczWl_WlDCDj2kH3ZpgBWNAoAbXLRoVg")
```

To actually read the file into R as a data frame, we’ll need
googlesheets4. We can use `gs4_get()` to see all of the sheets available
within our spreadsheet.

``` r
gs4_get(pentathlon_ss)
#> Spreadsheet name: Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships
#>               ID: 1bv4T90Xzh99YczWl_WlDCDj2kH3ZpgBWNAoAbXLRoVg
#>           Locale: en_US
#>        Time zone: America/Los_Angeles
#>      # of sheets: 14
#> 
#>                   (Sheet name): (Nominal extent in rows x columns)
#>         Women Qualifications A: 1000 x 26
#>         Women Qualifications B: 1000 x 26
#>           Men Qualifications A: 1000 x 26
#>           Men Qualifications B: 1000 x 26
#>           Men Qualifications C: 1000 x 26
#>                   Women Finals: 1000 x 26
#>                     Men Finals: 1000 x 26
#>                      Men Relay: 1000 x 26
#>                    Women Relay: 1000 x 26
#>                    Mixed Relay: 1000 x 26
#> Women Final Individual Results: 1000 x 26
#>    Men Final Indvidual Results: 1000 x 26
#>           Men Team Competition: 1000 x 26
#>         Women Team Competition: 1000 x 26
```

Let’s say we want the “Women Finals” results. We can pipe the results of
`gs4_get()` into `googlesheets4::read_sheet()` to read that specific
sheet into R as a data frame.

``` r
w_finals <- gs4_get(pentathlon_ss) %>%
  read_sheet(sheet = "Women Finals")
#> ✓ Reading from
#>   "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships".
#> ✓ Range ''Women Finals''.
```

``` r
glimpse(w_finals)
#> Rows: 36
#> Columns: 9
#> $ Rank              <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 1…
#> $ Name              <chr> "PROKOPENKO Anastasiya\nW039969 1985-09-20", "CLOUVE…
#> $ Nation            <chr> "BLR", "FRA", "HUN", "GER", "ITA", "GER", "MEX", "UK…
#> $ Fencing           <chr> "246 (1)\n24 V - 11 D", "220 (8)\n20 V - 15 D", "233…
#> $ Swimming          <chr> "251 (35)\n02:29.79", "292 (1)\n02:09.48", "286 (4)\…
#> $ Riding            <chr> "275 (30)\n71.00", "286 (17)\n65.00", "285 (21)\n68.…
#> $ LaserRun          <chr> "581 (1)\n11:59.80", "543 (6)\n12:37.10", "535 (10)\…
#> $ `MP Points`       <dbl> 1353, 1341, 1339, 1330, 1324, 1324, 1323, 1312, 1306…
#> $ `Time Difference` <chr> NA, "12''", "14''", "23''", "29''", "29''", "30''", …
```
