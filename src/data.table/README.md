---
title: data.table
---

# Data wrangling with data.table

[**data.table**](https://rdatatable.gitlab.io/data.table) (by Matt Dowle, Arun
Srinivasan _et. al._) is a package written in C to make high-performance data 
wrangling tasks a breeze. Despite being incredibly powerful, it is dependency 
free and has a rock-solid API. **data.table** code reliably works decades apart.

## Installation

Before continuing, make sure that you have installed **data.table**. You only 
have to do this once (or as often as you want to update the package).

```r
# Install from CRAN (recommended)
install.packages('data.table')

# Install the development version from GitHub (advanced)
# Requires Rtools and the remotes package
# remotes::install_github('Rdatatable/data.table')
```

Once **data.table** is installed, don't forget to load it whenever you want to 
use it. Unlike Stata, you have to re-load a package every time you start a new R 
session.

```r
# Load data.table into our current R session
library(data.table)
```

All of the examples in this section will use real-life 2014 New York air traffic 
data. You can use the following commands to import the dataset into both Stata 
and R.

<div class='code--container'>
<div>

```stata
import delimited using \\\
    "https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv", clear
```
</div>
<div>

```r
# library(data.table) ## Don't forget to load the library
dat = fread('https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv')
```
</div>
</div>

## Introduction

The [**data.table**](https://rdatatable.gitlab.io/data.table) package centers
around _data.tables_, which are highly efficient data frames that can be
manipulated using the package's concise syntax. For example, say we have a
data.table called `dat` (you can call it whatever you want). Then we can
manipulate it by putting arguments into its square brackets, i.e. `dat[]`. The
three main components of a **data.table** operation are `i`, `j`, and `by`,
which go in the order **`dat[i, j, by]`**. Note you don't have to specify the
latter two if you're not currently using them.

- **`i`**, the first component, selects the rows of the data.table that you'll be working with, like how in Stata the `if` or `in` command options let you refer to certain rows.
- **`j`**, the second component, both selects and operates on the columns of the data.table, like how in Stata the `keep` or `drop` commands select specific columns of your data, or how `generate` or `replace` create or modify columns in your data.
- **`by`**, the third component, gives the variable(s) designating groups that you'll be doing your calculations within, like how in Stata you can precede a command with `bysort`.

**data.table** uses these simple components very flexibly. The upshot is that 
you can perform complicated operations in a single line of concise **data.table** 
code, which may have required multiple commands in other languages or libraries 
to accomplish. But even if you aren't doing anything fancy, **data.table** has 
you covered with a stable set of functions that can be deployed on virtually 
any data wrangling task.

Like Stata, **data.table** also provides some special shortcut symbols for 
common operations. For example, `_N` in Stata is equivalent to `.N` in 
**data.table**, while `.(x1, x2)` is short for `list(x1, x2)`. We'll see more 
examples in cheatsheat that follows. But we do want to quickly highlight one 
special symbol in particular: **`.SD`** refers to the (**S**)ubset of (**D**)ata you're 
working with. This can be used to do complex within-group calculations when you 
have `by` specified. But more generally it's a way to perform operations on lots 
of columns with one line of code. By default, `.SD` refers to all columns in the
dataset (excepting those in `by`). But you can specify the columns you want with 
the `.SDcols` argument. Again, we'll see a bunch of examples below.

Finally, **data.table** is extremely fast. It has long set the standard for 
in-memory data wrangling [benchmarks](https://h2oai.github.io/db-benchmark) 
across a variety of libraries and languages. You will likely see an order(s) of 
magnitude performance difference as you compare the code chunks below. As a 
bonus for Stata users, who are used to operations changing a single dataset in 
memory, many **data.table** operations can be done _in-place_. This means that 
you don't have to (re)assign the result to a new **data.table**. In-place 
modifications are also very efficient, since they will only affect the parts 
you're actually changing, without wasting memory and time on the parts that 
aren't being changed. Any time in the below cheat sheet you see a function with 
the word `set` in it, or the `:=` operator, that's an in-place operation.

                     
## Data I/O

Like Stata's `.dta` file format, R has its own native `.rds` storage format.
(See also the [**fst**](http://www.fstpackage.org/) package.) However,
we generally recommend that users avoid native—especially proprietary—data types
since they hamper interoperability and reproducibility. We'll hence concentrate
on common open-source file types below. We'll make an exception for `.dta` given
our target audience, but we still recommend avoiding it if possible.

           
### Read and write .csv

<div class='code--container'>
<div>

```stata
import delimited using "https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv", clear 
export delimited using "flightdata.csv", replace
```
</div>
<div>

```r
dat = fread('https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv')
fwrite(dat, 'flightdata.csv')
```
</div>
</div>
           
### Read and write .dta

<div class='code--container'>
<div>

```stata
* .dta is Stata's native (proprietary) filetype 
use "filename.dta", clear 


save "filename.dta", replace
```
</div>
<div>

```r
# These commands require the `haven` package 
dat = haven::read_dta('filename.dta') 
setDT(dat) # Or: dat = as.data.table(dat) 
 
haven::write_dta(dat, 'filename.dta')
```
</div>
</div>
                     
### Read and write .parquet

<div class='code--container'>
<div>

```stata
* Stata currently has limited support for parquet files 
* (and Linux/Unix only). 
* See: https://github.com/mcaceresb/stata-parquet
```
</div>
<div>

```r
# These commands require the `arrow` package 
pfiles = dir(pattern = ".parquet") 
rbindlist(lapply(pfiles, arrow::read_parquet)) 
rbindlist(lapply(pfiles, arrow::read_parquet, col_select=1:10))
```
</div>
</div>
           
                     
## Order

           
### Sort rows

<div class='code--container'>
<div>

```stata
sort air_time 
sort air_time dest 
gsort -air_time
```
</div>
<div>

```r
setorder(dat, air_time) 
setorder(dat, air_time, dest) 
setorder(dat, -air_time)
```
</div>
</div>
           
### Sort columns

<div class='code--container'>
<div>

```stata
order month day
```
</div>
<div>

```r
setcolorder(dat, c('month','day'))
```
</div>
</div>
           
### Rename columns

<div class='code--container'>
<div>

```stata
* rename (old) (new) 

rename arr_delay arrival_delay 
rename (carrier origin) (carrier_code origin_code) 
rename arr_* arrival_*
```
</div>
<div>

```r
# setnames(dat, old = ..., new = ...) 

setnames(dat, 'arr_delay', 'arrival_delay') 
setnames(dat, c('carrier','origin'), c('carrier_code','origin_code')) 
setnames(dat, gsub('arr_', 'arrival_', names(dat)))
```
</div>
</div>
                     
                     
## Subset

In newer versions of Stata, it's possible to keep multiple datasets in memory,
or "frames" as Stata calls them. However, this still requires extra steps that
would be unusual to users of other languages. It's also not the typical way that
most peope use Stata. In contrast, keeping multiple datasets in memory is
extremely common in R. Moreover, subsetting and collapsing operations don't
overwrite your original dataset. The upshot is that you don't need to wrap 
everything in `preserve/restore`. However, it also means that you'll need to 
(re)assign your subsetted/collapsed data if you want to use it again later. E.g.
`dat1 = dat[origin=='LGA']`.

           
### Subset rows

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the
original dataset in the examples that follow._

```stata
keep in 1/200 
keep if day > 5 & day < 10
keep if inrange(day,5,10)
keep if origin == "LGA"
keep if regex(origin,"LGA") 
keep if inlist(month,3,4,11,12) 
keep if inlist(origin,"JFK","LGA") 
drop if month == 1
```
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

```r
dat[1:200] 
dat[day > 5 & day < 10] 
dat[between(day,5,10)] # Or: dat[day %in% 5:10] 
dat[origin=='LGA']
dat[origin %like% 'LGA'] 
dat[month %in% c(3,4,11,12)] 
dat[origin %chin% c("JFK","LGA")] # %chin% is a faster %in% for (ch)aracter strings 
dat[month!=1]
```
</div>
</div>
           
### Subset columns

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the 
original dataset in the examples that follow._

```stata
keep month day carrier


```
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

```r
dat[, .(month, day, carrier)] 
dat[, list(month, day, carrier)]    # another option
dat[, c('month', 'day', 'carrier')] # and another
```
</div>
</div>

<div class='code--container'>
<div>

```stata
keep year-arr_delay
keep *_delay 
```
</div>
<div>

```r
dat[, year:arr_delay] 
dat[, .SD, .SDcols=patterns('*_delay')]
```
</div>
</div>


<div class='code--container'>
<div>

```stata
drop origin dest 


ds, has(type string) 
drop `r(varlist)' 

ds, has(type int) 
keep `r(varlist)'
```
</div>
<div>

```r
dat[, -c('origin', 'dest')]
dat[, c('origin', 'dest') := NULL] # same, but in-place 

# Matches the two lines on the left:
dat[, .SD, .SDcols=!is.character] 

# Matches the two lines on the left: 
dat[, .SD, .SDcols=is.integer]
```
</div>
</div>
           
          
### Subset rows and columns

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the
original dataset in the examples that follow._

```stata
keep if origin == "LGA"
keep month day carrier
```
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

```r
# Matches the two lines on the left:
dat[origin=="LGA", .(month, day, carrier)]
```
</div>
</div>
           
### Drop duplicates

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the
original dataset in the examples that follow._

```stata
duplicates drop
duplicates drop month day carrier, force
```
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

```r
unique(dat) 
unique(dat, by = c('month', 'day', 'carrier'))
```
</div>
</div>
           
### Drop missing

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the
original dataset in the examples that follow._

```stata
keep if !missing(dest)

* Requires: ssc inst missings
missings dropvars, force 
missings air_time dest, force 

```
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

```r
dat[!is.na(dest)]


na.omit(dat) 
na.omit(dat, cols = c('air_time', 'dest')) 
# dat[!is.na(air_time) & !is.na(dest)] # same
```
</div>
</div>
                     
                     
## Modify

**Aside:** You can force print a data.table's in-place modifications to screen by 
adding a trailing `[]`, e.g. `dat[, dist_sq := distance^2][]`.
    
### Create new variables

<div class='code--container'>
<div>

```stata
gen dist_sq = distance^2 
gen tot_delay = dep_delay + arr_delay 
gen first_letter = substr(origin, 1,1) 
gen flight_path = origin + '_' + dest 
```
</div>
<div>

```r
dat[, dist_sq := distance^2] 
dat[, tot_delay := dep_delay + arr_delay] 
dat[, first_letter := substr(origin,1,1)] 
dat[, flight_path := paste(origin, dest, sep='_')] 
```
</div>
</div>

Here are some **data.table** modifying operations that don't have direct Stata 
equivalents (although you could implement a loop).

```r
# Multiple variables can be created at once. These next few lines all do the 
# same thing. Just pick your favourite. 

dat[, c('dist_sq', 'dist_cu') := .(distance^2, distance^3)] 
dat[, ':=' (dist_sq=distance^2, dist_cu=distance^3)]        # "functional" equivalent 
dat[, let(dist_sq=distance^2, dist_cu=distance^3)]          # dev version only

# We can also chain back-to-back with "dat[...][...]" (this holds for any 
# data.table operation) 

dat[, dist_sq := distance^2][
    , dist_cu := distance*dist_sq]
```


### Create new variables within groups

**Aside:** In R, any missing (i.e. "NA") values will propagate during
aggregating functions. If you have `NA` values in your real-life dataset—we
don't in this example dataset—you probably want to add `na.rm=TRUE` to remove
these on the fly. E.g. `mean(var1, na.rm=TRUE)` or 
`lapply(.SD, mean, na.rm=TRUE)`.

<div class='code--container'>
<div>

```stata
bysort origin: egen mean_dep_delay = mean(dep_delay) 
bysort origin dest: egen mean_dep_delay2 = mean(dep_delay) 
```
</div>
<div>

```r
dat[, mean_dep_delay := mean(dep_delay), by=origin] 
dat[, mean_dep_delay2 := mean(dep_delay), by=.(origin,dest)] 
```
</div>
</div>

Some shortcut symbols.

<div class='code--container'>
<div>

```stata
bysort carrier: g rows_per_carrier = _N 
bysort carrier: g index_within_carrier = _n 
egen origin_index = group(origin)
```
</div>
<div>

```r
dat[, rows_per_carrier := .N, by = carrier] 
dat[, index_within_carrier := .I, by = carrier] 
dat[, origin_index := .GRP, by = origin]
```
</div>
</div>
  
Multiple grouped variables (manual demean example).

<div class='code--container'>
<div>

```stata
foreach x of varlist dep_delay arr_delay air_time {
    egen mean_`x'=mean(`x'), by(origin) 
    gen `x'_dm = `x` - mean_`x' 
    drop mean* 
}

```
</div>
<div>

```r
dmcols = c('dep_delay', 'arr_delay', 'air_time') 
dat[,
    paste0(dmcols,'_dm') := lapply(.SD, \(x) x-mean(x)),
    .SDcols = dmcols,
    by = origin] 

# Note: `\(x)` is a shorthand for `function(x)`, introduced 
# in R 4.1.1. You'll need to use the latter if you're still 
# on an older version of R.
```
</div>
</div>

Relative modification (i.e. refer to other rows)

<div class='code--container'>
<div>

```stata
* This will be easier to demonstrate with a collapsed 
* dataset: Grab the total monthly flights out of each 
* origin airport. (Don't forget preserve/restore!)
contract origin month, freq(N)
sort origin month

* Simple month-on-month growth
by origin: gen growth = N/N[_n-1]

* Relative growth
by origin: gen growth_since_jan = N/N[1]
* ?
```
</div>
<div>

```r
# This will be easier to demonstrate with a collapsed 
# dataset: Grab the total monthly flights out of each 
# origin airport.
dat2 = dat[, .N, by = .(origin, month)]
setorder(dat2, origin, month)

# Simple month-on-month growth
dat2[, growth := N/shift(N, 1), by = origin][]

# Relative growth
dat2[, growth_since_jan := N/first(N), by = origin][]
dat2[, growth_since_may := N/N[month==5], by = origin][]
```
</div>
</div>

### Work with dates

<div class='code--container'>
<div>

```stata
* Make a date variable
tostring year month day, replace
gen day_string = year + "/" + month + "/" + day
gen date = date(day_string, "YMD")
format date %td

* Pull out year (quarter, month, etc. work too)
gen the_year = year(date)

* Shift forward 7 days
replace date = date + 7
```
</div>
<div>

```r
# Make a date variable
dat[, date := as.IDate(paste(year, month, day, sep='-'))] 




# Pull out year (quarter, month, etc. work too)
dat[, the_year := year(date)]

# Shift forward 7 days
dat[, date := date + 7]
```
</div>
</div>
           
### Modify existing variables

**Aside:** We don't normally use a gen -> replace workflow in R, the way we do in 
Stata. See the [Using Booleans & control-flow](#using-booleans-control-flow) 
section below for a more idiomatic approach.

<div class='code--container'>
<div>

```stata
replace tot_delay = dep_delay + arr_delay 

* Conditional modification 
replace distance = distance + 1 if month==9
replace distance = 0 in 1 

* Modify multiple variables (same function) 
foreach x of varlist origin dest {
    replace `x' = `x' + " Airport"
}
```
</div>
<div>

```r
dat[, tot_delay := dep_delay + arr_delay] 

# Conditional modification 
dat[month==9, distance := distance + 1]
dat[1, distance := 0]

# Modify multiple variables (same function) 
cols = c('origin','dest')
dat[, (cols) := lapply(.SD, \(x) paste(x,'Airport')), 
    .SDcols = cols] 
```
</div>
</div>
           
### Using Booleans & control-flow

<div class='code--container'>
<div>

```stata
gen long_flight = air_time>500 & !missing(air_time) 

gen flight_length = "Long" if air_time>500 & !missing(air_time)
replace flight_length = "Short" if missing(flight_length) & !missing(air_time) 


gen flight_length2 = "Long" if !missing(air_time) 
replace flight_length2 = "Med" if air_time<=500  
replace flight_length2 = "Short" if air_time<=120
```
</div>
<div>

```r
dat[, long_flight := air_time>500] 

dat[, flight_length := fifelse(air_time>500, 'Long', 'Short')] 
# fifelse is like base-R ifelse, but (f)aster! 

# for nested ifelse, easier to use fcase 
dat[, flight_length2 := fcase(air_time<=120, 'Short', 
                              air_time<=500, 'Med', 
                              default = 'Long')]
```
</div>
</div>
           
### Row-wise calculations

<div class='code--container'>
<div>

```stata
* Pre-packaged row calculations: 
egen tot_delay = rowtotal(*_delay)
egen any_delay = rowfirst(*_delay)

* Custom row calculations:
* ?

```
</div>
<div>

```r
# Pre-packaged row calculations: 
dat[, tot_delay := rowSums(.SD), .SDcols=patterns('*_delay')]
dat[, any_delay := fcoalesce(.SD), .SDcols=patterns('*_delay')] 

# Custom row calculations: 
dat[, new_var := mapply(custom_func, var1, var2)] 
dat[, new_var := custom_func(var1, var2), by=.I] # dev version only
```
</div>
</div>
           
### Fill in Time Series/Panel Data

<div class='code--container'>
<div>

```stata
* Carry forward the last-known observation
sort id time
by id: replace x = x[_n-1] if missing(x)

* Carry back the next-known observation
gsort id -time
by id: replace x = x[_n-1] if missing(x)
```
</div>
<div>

```r
# Carry forward the last-known observation
setorder(dat, id, time)
dat[, x := nafill(x, type = 'locf'), by = id]

# Carry back the next-known observation
dat[, x := nafill(x, type = 'nocb'), by = id]

```
</div>
</div>
                     
                     
## Collapse

In newer versions of Stata, it's possible to keep multiple datasets in memory,
or "frames" as Stata calls them. However, this still requires extra steps that
would be unusual to users of other languages. It's also not the typical way that
most peope use Stata. In contrast, keeping multiple datasets in memory is
extremely common in R. Moreover, subsetting and collapsing operations don't
overwrite your original dataset. The upshot is that you don't need to wrap 
everything in `preserve/restore`. However, it also means that you'll need to 
(re)assign your subsetted/collapsed data if you want to use it again later. E.g.
`dat1 = dat[, mean(var1)]`. Finally, remember our earlier note about aggregating
functions on columns that have missing values: Use `na.rm=TRUE` to remove these
on the fly. E.g. `dat[, mean(var1, na.rm=TRUE)]`.

           
### Collapse with no grouping

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the
original dataset in the examples that follow._

```stata
collapse (mean) dep_delay 
```
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

```r
dat[, mean(dep_delay)] # returns a scalar
```
</div>
</div>

<div class='code--container'>
<div>

```stata
collapse (mean) mean_ddel = dep_delay 
```
</div>
<div>

```r
dat[, .(mean_ddel = mean(dep_delay))] # returns a data.table
```
</div>
</div>

<div class='code--container'>
<div>

```stata
collapse (mean) mean_ddel=dep_delay mean_adel=arr_delay 
```
</div>
<div>

```r
# These lines all do the same thing. Just pick your favourite.
dat[, .(mean_ddel=mean(dep_delay), mean_adel=mean(arr_delay))]
dat[, lapply(.SD, mean), .SDcols=c('arr_delay','dep_delay')]
dat[, lapply(.SD, mean), .SDcols=arr_delay:dep_delay]
```
</div>
</div>

<div class='code--container'>
<div>

```stata
collapse (mean) *delay 
```
</div>
<div>

```r
dat[, lapply(.SD, mean), .SDcols=patterns('delay')] 
```
</div>
</div>

<div class='code--container'>
<div>

```stata
ds, has(type long)
collapse (mean) `r(varlist)'
```
</div>
<div>

```r
 # Matches the two lines on the left
dat[, lapply(.SD, mean), .SDcols=is.numeric]
```
</div>
</div>

### Collapse by group

<div class='code--container'>
<div>

_Reminder: You'll need to use `preserve/restore` if you want to retain the
original dataset in the examples that follow._
</div>
<div>

_Reminder: You'll need to (re)assign the subsetted dataset if you want to use it
later, e.g. `dat1 = dat[...]`._

</div>
</div>

<div class='code--container'>
<div>

```stata
collapse (mean) arr_delay, by(carrier) 
* collapse (mean) V1 = arr_delay, by(carrier)
```
</div>
<div>

```r
dat[, .(arr_delay = mean(arr_delay)), by=carrier] 
# dat[, mean(arr_delay), by=carrier] 
```
</div>
</div>


<div class='code--container'>
<div>

```stata
collapse (mean) arr_delay, by(carrier month) 
```
</div>
<div>

```r
dat[, .(arr_delay = mean(arr_delay)), by=.(carrier, month)] 
```
</div>
</div>


<div class='code--container'>
<div>

```stata
collapse (min) min_d=distance (max) max_d=distance, by(origin) 
```
</div>
<div>

```r
dat[, .(min_d=min(distance), max_d=max(distance)), by=origin] 
```
</div>
</div>


<div class='code--container'>
<div>

```stata
collapse (mean) *_delay, by(origin) 
```
</div>
<div>

```r
dat[, lapply(.SD, mean), .SDcols=patterns('_delay'), by=origin] 
```
</div>
</div>


<div class='code--container'>
<div>

```stata
ds, has(type long)
collapse (mean) `r(varlist)', by(origin) 
```
</div>
<div>

```r
# matches the two lines on the left
dat[, lapply(.SD, mean), .SDcols=is.numeric, by=origin] 
```
</div>
</div>

<div class='code--container'>
<div>

```stata
collapse (mean) dep_delay arr_delay air_time distance, by(origin) 
```
</div>
<div>

```r
dat[, lapply(.SD, mean), .SDcols=c('dep_delay','arr_delay','air_time','distance'), by=origin] 
#dat[, lapply(.SD, mean), .SDcols = c(4,5,9,10), by=origin] # same 
```
</div>
</div>

<div class='code--container'>
<div>

```stata
egen unique_dest = tag(dest origin) 
collapse (sum) unique_dest, by(origin)
```
</div>
<div>

```r
# Matches the final two lines on the left: 
dat[, .(unique_dest = uniqueN(dest)), by = origin]
```
</div>
</div>

### Count rows

<div class='code--container'>
<div>

```stata
count
count if month==10

* Count rows by group:
tabulate origin
```
</div>
<div>

```r
dat[, .N] # Or: nrow(dat) 
dat[month==10, .N] # Or: nrow(dat[month==10]

# Count rows by group:
dat[, .N, by = origin]
```
</div>
</div>

### Advanced collapse (tips and tricks)

These next few examples are meant to highlight some specific **data.table**
collapse tricks. They don't really have good Stata equivalents (that we're aware
of).


#### Use keys for even faster grouped operations

The **data.table** website 
[describes](https://rdatatable.gitlab.io/data.table/articles/datatable-keys-fast-subset.html)
keys as "supercharged rownames". Essentially, _setting a key_ means ordering 
your data in a way that makes it very efficient to do subsetting or aggregating 
operations. **data.table** is already highly performant, but setting keys can 
give a valuable speed boost for big data tasks.

```r
## Set keys. You normally want whatever you're grouping by
setkey(dat, month, origin)

## Same collapse syntax as before, just faster
dat[, mean(dep_delay), by = .(month, origin)]

## Tip: Turn on automatic printing of keys. The dev version
## of data.table (v1.14.3) does this by default.
options(datatable.print.class = TRUE, datatable.print.keys = TRUE)
dat

## Turn keys back off
setkey(dat, NULL)
```

#### Grouped calculations and complex objects inside a data.table

**data.table** supports list columns, so you can have complex objects like
regression models inside a data.table. Among many other things, this means you
can nest simulations inside a data.table as easily as you would perform any
other (grouped) operation. Here we illustrate with a simple grouped regression,
i.e. a separate regression for each month of our dataset.

```r 
# Let's run a separate regression of arrival delay on 
# departure delay for each month _inside_ our data.table

# Just get the coefficients
dat[,
    .(beta = coef(lm(arr_delay ~ dep_delay, .SD))['dep_delay']),
    by = month]

# As above, but now keep the whole model for each month
# in a dedicated "mod" column
mods = dat[,
           .(mod = list(lm(arr_delay ~ dep_delay, .SD))),
           by = month] 

# Now you can do things like put all 10 models in a 
# regression table or coefficient plot. Here we use the
# modelsummary package to do that.
modelsummary::msummary(mods$mod) 
modelsummary::modelplot(mods$mod, coef_omit = 'Inter')
```
                     
#### Grouped aggregations when reshaping

You can do complicated (grouped) aggregations as part of a `data.table::dcast()`
(i.e. reshape wide) call. Here's an example where we summarise both the
departure and arrival delays—getting the minimum, mean, and maximum
values—by origin airport.

```r
dcast(dat, origin~., fun = list(min, mean, max),
      value.var = c('dep_delay', 'arr_delay'))
```


## Reshape

### Reshape prep (this dataset only)

_Note: We need to do a bit of prep to our air-traffic dataset to better
demonstrate the reshape examples in this section. You probably don't need to do
this for your own dataset._

<div class='code--container'>
<div>

```stata
* We'll generate row IDs to avoid the (reshape) ambiguity 
* of repeated entries per date 
gen id = _n 

* For the Stata reshape, it's also going to prove 
* convenient to rename the delay vars. 
rename (dep_delay arr_delay) (delay_dep delay_arr)
```
</div>
<div>

```r
# We'll generate row IDs to avoid the (reshape) ambiguity 
# of repeated entries per date 
dat[, id := .I] 
```
</div>
</div>
           
### Reshape long

<div class='code--container'>
<div>

```stata
reshape long delay_, i(id) j(delay_type) s
```
</div>
<div>

```r
ldat = melt(dat, measure=patterns('_delay'))

# Aside: you can also choose different names for your
# new reshaped columns if you'd like, e.g. 
melt(dat, measure=patterns('_delay'), variable='d_type')
```
</div>
</div>
           
### Reshape wide

<div class='code--container'>
<div>

```stata
* This starts with the reshaped-long data from above
reshape wide delay_, i(id) j(delay_type) s
```
</div>
<div>

```r
# This starts with the reshaped-long data from above
wdat = dcast(ldat, ... ~ variable)

# Aside 1: If you only want to keep the id & *_delay cols
dcast(ldat, id ~ variable)

# Aside 2: It's also possible to perform complex and 
# powerful data aggregating tasks as part of the dcast 
# (i.e. reshape wide) call.
dcast(dat, origin~., fun=list(min,mean,max),
      value.var=c('dep_delay','arr_delay'))
```
</div>
</div>
                     
                     
## Merge

### Import and prep secondary dataset

**Note:** Our secondary dataset for demonstrating the merges in this section
will be one on airport characteristics.

<div class='code--container'>
<div>

```stata
import delimited using "https://vincentarelbundock.github.io/Rdatasets/csv/nycflights13/airports.csv", clear
* Stata requires that merge ID variables have the same 
* name across datasets. 
rename faa dest 
save dat2.dta, replace
import delimited using "https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv", clear
```
</div>
<div>

```r
dat2 = fread("https://vincentarelbundock.github.io/Rdatasets/csv/nycflights13/airports.csv") 
# R _doesn't_ require that merge ID variables share the 
# same name across datasets. But we'll add this anyway.
dat2[, dest := faa]
```
</div>
</div>
           
### Inner merge

_Only keep the matched rows across both datasets._

<div class='code--container'>
<div>

```stata
merge m:1 dest using dat2.dta, keep(3) nogen
```
</div>
<div>

```r
mdat = merge(dat, dat2, by='dest') 
```
</div>
</div>
           
### Full merge

_Keep all rows of both datasets, regardless of whether matched._

<div class='code--container'>
<div>

```stata
merge m:1 dest using dat2.dta, nogen
```
</div>
<div>

```r
mdat = merge(dat, dat2, by='dest', all=TRUE)
```
</div>
</div>
           
### Left merge

_Keep all rows from the "main" dataset._

<div class='code--container'>
<div>

```stata
merge m:1 dest using dat2.dta, keep(1 3) nogen
```
</div>
<div>

```r
mdat = merge(dat, dat2, by='dest', all.x=TRUE)
```
</div>
</div>
           
### Right merge

_Keep all rows from the "secondary" dataset._

<div class='code--container'>
<div>

```stata
merge m:1 dest using dat2.dta, keep(2 3) nogen
```
</div>
<div>

```r
mdat = merge(dat, dat2, by='dest', all.y=TRUE)
```
</div>
</div>
           
### Anti merge

_Keep non-matched rows only._

<div class='code--container'>
<div>

```stata
merge m:1 dest using dat2.dta, keep(1 2) nogen
```
</div>
<div>

```r
mdat = dat[!dat2, on='dest']
```
</div>
</div>

### Appending data

<div class='code--container'>
<div>

```stata
* This just appends the flights data to itself
save data_to_append.dta, replace
append using data_to_append.dta
```
</div>
<div>

```r
# This just appends the flights data to itself
rbindlist(list(dat, dat)) # Or rbind(dat, dat)
# The fill = TRUE option is handy if the one data set has columns the other doesn't
```
</div>
</div>

### Advanced merges (tips and tricks)

These next few examples are meant to highlight some specific **data.table**
merge tricks. They don't really have good Stata equivalents (that we're aware
of).

#### Merge on different ID names 

<div class='code--container grid-cols-1'>
<div>

```r
mdat = merge(dat, dat2, by.x='dest', by.y='faa') 
```
</div>
</div>

#### Set keys for even faster merges and syntax shortcuts 

<div class='code--container grid-cols-1'>
<div>

```r
setkey(dat, dest); setkey(dat2, dest) 
mdat = merge(dat, dat2) ## note: don't need 'by' 
```
</div>
</div>

#### Non-equi joins

Non-equi joins are a bit hard to understand if you've never seen them before.
But they are incredibly powerful and solve a surprisingly common problem:
Merging datasets over a range (e.g. start to end dates), rather than exact
matches. Here's a simple example where we want to subset the 1st quarter flights
for American Airlines and the 2nd quarter flights for United Airlines:

<div class='code--container grid-cols-1'>
<div>

```r
# The things we want to match on. Note the different start and
# end months for AA and UA.
dat3 = data.table(carrier     = c('AA', 'UA'),
                  start_month = c(1, 4),
                  end_month   = c(3, 6)) 

# Rolling join that catches everything between the distinct
# start and end dates for each carrier.
dat[dat3, on = .(carrier,
                 month >= start_month,
                 month <= end_month)] 
```
</div>
</div>

#### Rolling Joins

Rolling joins are similar and allow you to match a set of dates forwards or
backwards. For example, our `dat`  dataset ends in October. Let's say we want to
carry the last known entries for American and United Airlines  forward to
(random) future dates.

<div class='code--container grid-cols-1'>
<div>

```r
# Make sure we have a date variable
dat[, date := as.IDate(paste(year, month, day, sep='-'))] 

# New DT with the (random) target dates
dat4 = data.table(carrier  = c('AA', 'UA'),
                  new_date = as.IDate(c('2014-11-01', '2014-11-15'))) 

# Join on these target dates, so they take the last known value 
dat[dat4, on = .(carrier, date=new_date), roll='nearest']
```
</div>
</div>
