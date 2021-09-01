Up and running with googlesheets4
================

## What’s this?

Given the new(ish), major releases of
[**{googledrive}**](https://googledrive.tidyverse.org/) and
[**{googlesheets4}**](https://googlesheets4.tidyverse.org/) (see Jenny
Bryan’s release posts for [googledrive
2.0.0](https://www.tidyverse.org/blog/2021/07/googledrive-2-0-0/) and
[googlesheets4
1.0.0](https://www.tidyverse.org/blog/2021/07/googlesheets4-1-0-0/)), we
thought it might be helpful to see how the two packages can be used
together in a “real-world” scenario.

## What am I trying to do?

I’m taking downloaded Excel data on the modern pentathlon, and uploading
it to Google Drive as a Google Sheet. From there I want to read some of
the data into R, clean it up, and modify the original spreadsheet in
Google Drive by adding the tidied version of the data as a new
sheet/tab.

## Authenticating and uploading

``` r
library(tidyverse)
library(googledrive)
library(googlesheets4)
```

First thing’s first, I’ll to upload the `.xls` file from [Union
Internationale de Pentathlon Moderne (UIPM) 2021 Pentathlon World
Championships](https://www.uipmworld.org/event/uipm-2021-pentathlon-world-championships)
to Google Drive.

To authenticate with my Google Drive account (and specify which one I
want to use), I’ll use
[`googledrive::drive_auth()`](https://googledrive.tidyverse.org/reference/drive_auth.html),
which allows you to either interactively select a pre-authorized account
in R, or takes you to the browser to generate obtain a new token for
your account. I’ll also (unnecessarily) specify
[`googlesheets4::gs4_auth()`](https://googlesheets4.tidyverse.org/reference/gs4_auth.html),
just so you can see what that function looks like as well.

To learn more about authenticating your account in an R Markdown
document, see the [Non-interactive
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
determined from the file extension. I’m using the `overwrite` argument
here to avoid clogging my Google Drive with this same data over and
over, since I’ve knit this document several times.

``` r
drive_upload(
  media = "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls",
  overwrite = TRUE
)
#> Local file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#> Uploaded into Drive file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#>   <id: 1sZr2uqxmV1i2pKI_X8Zu53R9NyU5ERmo>
#> With MIME type:
#> • 'application/vnd.ms-excel'
```

However, I can also specify a type (e.g. `type = "spreadsheet"`), which
will convert the file to its Google equivalent (in this case, a Google
Sheet) if the source file is a compatible type. Note that I’ll also get
a message telling me what file has been trashed by the `overwrite`
argument. In this case, it should match the id for the upload I did in
the previous chunk.

``` r
drive_upload(
  media = "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls",
  type = "spreadsheet",
  overwrite = TRUE
)
#> File trashed:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#>   <id: 1sZr2uqxmV1i2pKI_X8Zu53R9NyU5ERmo>
#> Local file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships.xls'
#> Uploaded into Drive file:
#> • 'Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships'
#>   <id: 12kgw4ZJf9bNUxahYaRl3MT4Cu9NfcNyBZ-wci_XQcmE>
#> With MIME type:
#> • 'application/vnd.google-apps.spreadsheet'
```

## Finding and getting the data into R

Now, let’s say I want to read my sheet into R to do some data wrangling
(it’s about as untidy as a dataset can get). I’ll use
[`googledrive::drive_find()`](https://googledrive.tidyverse.org/reference/drive_find.html)
to find the sheet I want by name or keyword.

``` r
drive_find("Pentathlon")
#> # A dribble: 3 × 3
#>   name                                   id                      drive_resource 
#>   <chr>                                  <drv_id>                <list>         
#> 1 Competition_Results_Exports_UIPM_2021… 12kgw4ZJf9bNUxahYaRl3M… <named list [3…
#> 2 Competition_Results_Exports_UIPM_2021… 1W5Y46CAMjpvnqOtPIzA48… <named list [3…
#> 3 UIPM_Competition_Results_Exports_UIPM… 1qfDtXN2FO7O412pBxdXlJ… <named list [3…
```

Since I know which one I want, I can retrieve the sheet by `id` with
[`googledrive::drive_get()`](https://googledrive.tidyverse.org/reference/drive_get.html).

``` r
pentathlon_ss <- drive_get(id = "1W5Y46CAMjpvnqOtPIzA48hmCJElb6vKE1-Vo8v2jj_4")
```

To actually read the file into R as a data frame, we’ll need
googlesheets4. We can use
[`googlesheets4::gs4_get()`](https://googlesheets4.tidyverse.org/reference/gs4_get.html)
to see all of the sheets available within our spreadsheet.

``` r
gs4_get(pentathlon_ss)
#> Spreadsheet name: Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships
#>               ID: 1W5Y46CAMjpvnqOtPIzA48hmCJElb6vKE1-Vo8v2jj_4
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
`gs4_get()` into
[`googlesheets4::read_sheet()`](https://googlesheets4.tidyverse.org/reference/range_read.html)
to read that specific sheet into R as a data frame. In this case, the
sheets are all names (thanks to to UIPM’s meticulous record keeping), so
I’ll specify the `sheet` argument as a string. However, you can also
specify the `sheet` by position (e.g. 6, in the case of the sheet I want
here). If `sheet` is not specified, `read_sheet()` will default to the
first visible sheet.

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

## Adding and writing to the spreadsheet

As you can probably see from this `glimpse()` at the data above, it is
most definitely *not* tidy. If you want to see how I went about cleaning
it up and/or learn more about the sport of modern pentathlon, you can
check out the episode of [Ellis
Hughes](https://twitter.com/ellis_hughes) and [Patrick
Ward’s](https://twitter.com/OSPpatrick) [TidyX
Screencast](https://www.youtube.com/channel/UCP8l94xtoemCH_GxByvTuFQ),
[Episode 69 \| Modern Pentathlons with Mara
Averick](https://www.youtube.com/watch?v=1356s1-as4o), and find all of
the code in [this GitHub
repo](https://github.com/batpigandme/modern-pentathlon).

For the purposes of this post, suffice it to say that cleaning was done,
resulting in a tidy data frame called `women_finals_tidy`.

``` r
glimpse(women_finals_tidy)
#> Rows: 36
#> Columns: 22
#> $ rank            <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,…
#> $ name            <chr> "PROKOPENKO Anastasiya", "CLOUVEL Elodie", "GULYAS Mic…
#> $ uipm_id         <chr> "W039969", "W039469", "W042365", "W003945", "W040837",…
#> $ dob             <date> 1985-09-20, 1989-01-14, 2000-10-24, 1990-04-03, 1999-…
#> $ nation          <chr> "BLR", "FRA", "HUN", "GER", "ITA", "GER", "MEX", "UKR"…
#> $ fencing_pts     <dbl> 246, 220, 233, 214, 214, 227, 214, 227, 220, 212, 167,…
#> $ fencing_pos     <dbl> 1, 8, 3, 14, 13, 5, 15, 7, 9, 17, 34, 20, 10, 4, 16, 1…
#> $ fencing_wins    <dbl> 24, 20, 22, 19, 19, 21, 19, 21, 20, 18, 11, 18, 20, 22…
#> $ fencing_losses  <dbl> 11, 15, 13, 16, 16, 14, 16, 14, 15, 17, 24, 17, 15, 13…
#> $ swim_pts        <dbl> 251, 292, 286, 270, 285, 267, 270, 267, 271, 269, 268,…
#> $ swim_pos        <dbl> 35, 1, 4, 15, 5, 22, 16, 21, 14, 18, 20, 2, 13, 23, 25…
#> $ swim_time       <chr> "02:29.79", "02:09.48", "02:12.27", "02:20.27", "02:12…
#> $ riding_pts      <dbl> 275, 286, 285, 293, 286, 293, 297, 298, 300, 255, 299,…
#> $ riding_pos      <dbl> 30, 17, 21, 9, 18, 13, 5, 4, 1, 35, 2, 25, 11, 24, 22,…
#> $ riding_score    <dbl> 71, 65, 68, 61, 67, 64, 70, 69, 65, 85, 68, 77, 67, 77…
#> $ laser_run_pts   <dbl> 581, 543, 535, 553, 539, 537, 542, 520, 515, 567, 564,…
#> $ lr_pos          <dbl> 1, 6, 10, 4, 8, 9, 7, 14, 16, 2, 3, 17, 22, 21, 15, 20…
#> $ lr_time         <chr> "11:59.80", "12:37.10", "12:45.60", "12:27.90", "12:41…
#> $ lr_secs         <Duration> 719.8s (~12 minutes), 757.1s (~12.62 minutes), 76…
#> $ mp_points       <dbl> 1353, 1341, 1339, 1330, 1324, 1324, 1323, 1312, 1306, …
#> $ time_difference <Duration> 0s, 12s, 14s, 23s, 29s, 29s, 30s, 41s, 47s, 50s, …
#> $ finish_time     <Duration> 719.8s (~12 minutes), 769.1s (~12.82 minutes), 77…
```

To add this back to our original spreadsheet, we’ll use two functions:
[`googlesheets4::sheet_add()`](https://googlesheets4.tidyverse.org/reference/sheet_add.html)
to add a new named sheet to our existing spreadsheet, and
[`googlesheets4::sheet_write()`](https://googlesheets4.tidyverse.org/reference/sheet_write.html)
to write our data into that sheet.

Without a `sheet` argument, `sheet_add()` would create a new sheet in
the specified file as “SheetN”, but I’ll specify the argument to call it
“Tidy Women Finals”. Since I want the sheet to be put at the end of the
workbook (which is the default behaviour), I am not going to specify an
argument for `.before` or `.after`.

``` r
sheet_add(pentathlon_ss, sheet = "Tidy Women Finals")
#> ✓ Adding 1 sheet to
#>   "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships":
#> • 'Tidy Women Finals'
```

Now we’ll actually write the data frame from R into that tab with
`write_sheet()`.

``` r
write_sheet(women_finals_tidy, 
            ss = pentathlon_ss, 
            sheet = "Tidy Women Finals")
#> ✓ Writing to
#>   "Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships".
#> ✓ Writing to sheet 'Tidy Women Finals'.
```

To see if that worked, we can use `gs4_get()` again to show us the tabs
in our workbook:

``` r
gs4_get(pentathlon_ss)
#> Spreadsheet name: Competition_Results_Exports_UIPM_2021_Pentathlon_World_Championships
#>               ID: 1W5Y46CAMjpvnqOtPIzA48hmCJElb6vKE1-Vo8v2jj_4
#>           Locale: en_US
#>        Time zone: America/Los_Angeles
#>      # of sheets: 15
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
#>              Tidy Women Finals: 37 x 22
```

If you’re a very astute observer, you may be wondering why the
dimensions of the “Tidy Women Finals” do not look exactly the same as
our `women_finals_tidy` data frame. This is because `sheet_write()`
styles the target sheet as a table, which means that the column names
become the first/header row, conveniently frozen for your scrolling
pleasure.

![Tidy Women Finals tab of UIPM World Championships Google Sheets
workbook with first row showing column
names](https://i.imgur.com/0X0Ame2.png)

## Fin

Success! We’ve added our tidy data frame to the original data we
uploaded to Google Drive.

To learn more about working with these packages, I highly recommend
checking out the bevvy of helpful [googlesheets4
articles](https://googlesheets4.tidyverse.org/articles/index.html)
written by Jenny Bryan.
