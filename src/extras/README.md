---
title: extras
---

# Other Packages

While we think you can get really far in R with just **data.table** and 
**fixest**, of course these two packages don't cover everything.

This page covers a small list of packages you may find especially useful when 
getting started with R. We won't try to cover everything under the sun here. 
Just a few places to get going. For the rest, well, that's what StackOverflow
or your favourite search engine is for.

All of the below packages have far more applications than is shown here. We'll 
just provide one or two examples of how each can be used. Finally, don't forget 
to install them with `install.packages('PKGNAME')` and load them with 
`library(PKGNAME)`. The former command you only have to run once per package (or 
as often as you want to update it); the latter whenever you want to use a 
package in a new R session.

## ggplot2

_Beautiful and customizable plots_

[**ggplot2**](https://ggplot2.tidyverse.org/) is widely considered one of the 
preeminent plotting libraries available in any language. It provides an 
intuitive syntax that applies in the same way across many, many different kinds 
of visualizations, and with a deep level of customization. Plus, endless 
additional plugins to do what you want, including easy interactivity, animation, 
maps, etc. We thought about giving **ggplot2** its own dedicated page like 
**data.table** and **fixest**. But instead we'll point you to the 
[Figures](https://lost-stats.github.io/Presentation/Figures/Figures.html) 
section of the _Library of Statistical Techniques_, which already shows how to 
do many different graphing tasks in both Stata and **ggplot2**. For a more 
in-depth overview you can always consult the excellent 
[package documentation](https://ggplot2.tidyverse.org/), or Kieran Healy's 
wonderful [_Data Visualization_](https://socviz.co/) book.

#### Basic scatterplot(s)

<div class="code--container">
<div>

```stata
twoway scatter yvar xvar

twoway (scatter yvar xvar if group == 1, mc(blue)) \\\
        (scatter yvar xvar if group == 2, mc(red))
```
</div>
<div>

```r
ggplot(dat, aes(x = xvar, y = yvar)) + geom_point()

ggplot(dat, aes(x = xvar, y = yvar, color = group)) + 
  geom_point()
```
</div>
</div>


## tidyverse

_A family of data science tools_

The [**tidyverse**](https://www.tidyverse.org/) provides an extremely popular
framework for data science tasks in R. This meta-package is actually a
collection of smaller packages that are all designed to work together, based on
a shared philosophy and syntax. We've already covered **ggplot2** above, but
there are plenty more. These include **dplyr** and **tidyr**, which offer an
alternative syntax and approach to data wrangling tasks. While we personally
recommend **data.table**, these **tidyverse** packages have many ardent fans
too. You may find that you prefer their modular design and verbal syntax. But
don't feel bound either way: it's totally fine to combine them. Some other
**tidyverse** packages worth knowing about include **purrr**, which contains a suite
of functions for automating and looping your work, **lubridate** which makes
working with date-based data easy, and **stringr** which offers functions with
straightforward syntax for working with string variables.

#### Data wrangling with dplyr

_Note: **dplyr** doesn't modify data in place. So you'll need to (re)assign if you want to keep your changes. E.g. `dat = dat %>% group_by...`_

<div class="code--container">
<div>

```stata
* Subset by rows and then columns
keep if var1=="value"
keep var1 var2 var3
* Create a new variable by group
bysort group1: egen mean_var1 = mean(var1)
* Collapse by group
collapse (mean) arr_delay, by(carrier)
```
</div>
<div>

```r
# Subset by rows and then columns
dat %>%   # `%>%` is the tidyverse "pipe" operator
   filter(var1=="value") %>%
   select(var1, var2, var3)
# Create a new variable by group
dat %>%
  group_by(group1) %>%
  mutate(mean_var1 = mean(var1))
# Collapse by group
dat %>%
  group_by(group1) %>%
  summarise(mean_var1 = mean(var1))
```
</div>
</div>

#### Manipulating dates with lubridate

<div class="code--container">
<div>

```stata
* Shift a date forward one month (not 30 days, one month)
* ???
```
</div>
<div>

```r
# Shift a date forward one month (not 30 days, one month)
shifted_date = date + months(1)
```
</div>
</div>

#### Iterating with purrr 

<div class="code--container">
<div>

```stata
* Read in many files and append them together
local filelist: dir "Data/" files "*.dta"
local firsttime = 1
foreach f in filelist {
    use `f', clear
    if `firsttime' == 0 {
        append using compiled_data.dta
    }
    save compiled_data.dta, replace
}
```
</div>
<div>

```r
# Read in many files and append them together
# (this combines purrr with the data.table function fread)
filelist = list.files('Data/', pattern = '.csv')
dat = filelist %>%
    map_df(fread)
```
</div>
</div>

#### String operations with stringr 

<div class="code--container">
<div>

```stata
subinstr(string, "remove", "replace", .)
substr(string, start, length)
regex(string, "regex")
```
</div>
<div>

```r
str_replace_all(string, "remove", "replace")
str_sub(string, start, end)
str_detect(string, "regex")
# Note all the stringr functions accept regex input
```
</div>
</div>

  

## collapse

_Extra convenience functions and super fast aggregations_

Sure, we've gone on and on about how fast **data.table** is compared to just
about everything else. But there is another R package that can boast even faster
computation times for certain grouped calculations and transformations, and
that's 
[collapse](https://sebkrantz.github.io/collapse/index.html). 
The **collapse** package doesn't try to do everything that **data.table** does. 
But the two 
[play very well together](https://sebkrantz.github.io/collapse/articles/collapse_and_data.table.html) 
and the former offers some convenience functions like `descr` and `collap`,
which essentially mimic the equivalent functions in Stata and might be
particularly appealing to readers of this guide. (P.S. If you'd like to load
**data.table** and **collapse** at the same time, plus some other 
high-performance packages, check out the 
[**fastverse**](https://sebkrantz.github.io/fastverse/index.html).)



#### Quick Summaries

<div class='code--container'>
<div>

```stata
summarize
describe
```
</div>
<div>

```r
qsu(dat)
descr(dat)
```
</div>
</div>

#### Multiple grouped aggregations

<div class='code--container'>
<div>

```stata
collapse (mean) var1, by(group1)
collapse (min) min_var1=var1 min_var2=var2 (max) max_var1=var1 max_var2=var2, by(group1 group2)
```
</div>
<div>

```r
collap(dat, var1 ~ group1, fmean) # 'fmean' => fast mean
collap(dat, var1 + var2 ~ group1 + group2, FUN = list(fmin, fmax))
```
</div>
</div>

                     
## sandwich

_More standard error adjustments_

**fixest** package comes with plenty of shortcuts for accessing standard error
adjustments like HC1 heteroskedasticity-robust standard errors, Newey-West,
Driscoll-Kraay, clustered standard errors, etc. But of course there are still
more than that. A host of additional options are covered by the
[**sandwich**](https://sandwich.r-forge.r-project.org/) package, which comes
with a long list of functions like `vcovBS()` for bootstrapped standard errors,
or `vcovHC()` for HC1-5. **sandwich** supports nearly every model class in R, so
it shouldn't surprise that these can slot right into `fixest` estimates, too. 
You shouldn't be using those `, robust` errors for smaller samples anyway... but 
you [knew that](http://datacolada.org/99), right?

#### Linear Model Adjustments

<div class='code--container'>
<div>

```stata
* ", robust" uses hc1 which isn't great for small samples
regress Y X Z, vce(hc3)
```
</div>
<div>

```r
# sandwich's vcovHC uses HC3 by default
feols(Y ~ X + Z, dat, vcov = sandwich::vcovHC) 

# Aside: Remember that you can also adjust the SEs 
# for existing models on the fly 
m = feols(Y ~ X + Z, dat) 
summary(m, vcov = sandwich::vcovHC)
```
</div>
</div>


## modelsummary

_Summary tables, regression tables, and more_

The **fixest** package already has the `etable()` function for generating
regression tables. However, it is only really intended to work with models from
the same package. So we also recommend checking out the fantastic
[**modelsummary**](https://vincentarelbundock.github.io/modelsummary) package.
It works with all sorts of model objects, including those not from **fixest**,
is incredibly customizable, and outputs to a bunch of different formats (PDF,
HTML, DOCX, etc.) Similarly, **modelsummary** has a wealth of options for
producing publication-ready summary tables. Oh, and it produces coefficient
plots too. Check out the [package
website](https://vincentarelbundock.github.io/modelsummary/) for more.


#### Summary tables

<div class='code--container'>
<div>

```stata
* Summary stats table 
estpost summarize 
esttab, cells("count mean sd min max") nomtitle nonumber 

* Balance table 
by treat_var: eststo: estpost summarize 
esttab, cells("mean sd") label nodepvar
```
</div>
<div>

```r
# Summary stats table 
datasummary_skim(dat) 


# Balance table 
datasummary_balance(~treat_var, dat)
```
</div>
</div>


#### Regression tables

**Aside:** Here we'll use the base R `lm()` (linear model) function, rather than
`feols()`, to emphasize that **modelsummary** works with many different model 
classes.

<div class='code--container'>
<div>

```stata
reg Y X Z 
eststo est1 
esttab est1b

reg Y X Z, vce(hc3) 
eststo est1b 
esttab est1b 

esttab est1 est1b

reg Y X Z A, vce(hc3)
eststo est2
esttab est1 est1b est2
```
</div>
<div>

```r
est1 = lm(Y ~ X + Z, dat) 
msummary(est1) # msummary() = alias for modelsummary()

# Like fixest::etable(), SEs for existing models can
# be adjusted on-the-fly 
msummary(est1, vcov='hc3')

# Multiple SEs for the same model
msummary(est1, vcov=list('iid', 'hc3')) 

est3 = lm(Y ~ X + Z + A, dat) 
msummary(list(est1, est1, est3),
         vcov = list('iid', 'hc3', 'hc3'))
```
</div>
</div>


## lme4

_Random effects and mixed models_

**fixest** can do a lot, but it can't do everything. This site isn't even going
to attempt to go into how to translate every single model into R. But we'll
quick highlight random-effects and mixed models. The
[**lme4**](https://cran.r-project.org/web/packages/lme4/index.html) and its
`lmer()` function covers not just random-intercept models but also hierarchical
models where slope coefficients follow random distributions. (**Aside:** If you
prefer Bayesian models for this kind of thing, check out 
[**brms**](https://paul-buerkner.github.io/brms/).)

           
#### Random effects and mixed models

<div class='code--container'>
<div>

```stata
xtset group time
xtreg Y X, re
mixed lifeexp || countryn: gdppercap
```
</div>
<div>

```r
# No need for an xtset equivalent
m = lmer(Y~(1|group) + X, data = dat)
nm = lmer(Y~(1+x|group) + X, data = dat)
```
</div>
</div>



## marginaleffects

_Marginal effects, constrasts, etc._

 
The Stata `margins` command is great. To replicate it in R, we highly recommend
the [**marginaleffects**](https://vincentarelbundock.github.io/marginaleffects/)
package. Individual marginal effects or average marginal effects for nonlinear
models, or models with interactions or transformations, etc. It's also very
fast.


#### Basic logit marginal effects

<div class='code--container'>
<div>

```stata
* A logit:
logit Y X Z
margins, dydx(*)
```
</div>
<div>

```r
# This example incorporates the fixest function feglm()
m = feglm(Y ~ X + Z, family = binomial, data = mtcars)
summary(marginaleffects(m))
```
</div>
</div>



## multcomp / nlWaldTest

_Joint coefficient tests_

Stata provides a number of inbuilt commands for (potentially complex)
postestimation coefficient tests. We've already seen the `testparm` command
equivalent with `fixest::wald()`. But what about combinations of coefficients _a
la_ Stata's `lincom` and `nlcom` commands? The
[**multcomp**](http://multcomp.r-forge.r-project.org/) package handles a variety
of linear tests and combinations, while
[**nlWaldTest**](https://cran.r-project.org/web/packages/nlWaldTest/index.html)
has you covered for nonlinear combinations.


#### Test other null hypotheses and coefficient combinations

<div class='code--container'>
<div>

```stata
regress y x z 

* One-sided test 
test _b[x]=0 
local sign_wgt = sign(_b[x]) 
display "H0: coef <= 0  p-value = " ttail(r(df_r),`sign_wgt'*sqrt(r(F))) 

* Test linear combination of coefficients 
lincom x + z 


* Test nonlinear combination of coefficients 
nlcom _b[x]/_b[z]
```
</div>
<div>

```r
m = feols(y ~ x + z, dat)

# One-sided test 
m2 = multcomp::ghlt(m, '<=0')
summary(m2) 


# Test linear combination of coefficients 
m3 = multcomp::glht(m, 'x + z = 0') 
summary(m3) # or confint(m3) 

# Test nonlinear combination of coefficients 
nlWaldtest::nlWaldtest(m, 'b[2]/b[3]') # or nlWaldtest::nlConfint()
```
</div>
</div>


## sf

_Geospatial operations_

R has outstanding support for geospatial computation and mapping. There are a
variety of packages to choose from here, depending on what you want (e.g. vector
vs raster data, interactive maps, high-dimensional data cubes, etc.) But the
workhorse geospatial tool for most R users is the incredibly versatile
[**sf**](https://r-spatial.github.io/sf/) package. We'll only provide a simple
mapping example below. The **sf** [website](https://r-spatial.github.io/sf/) has
several in-depth tutorials, and we also recommend the [_Geocomputation with
R_](https://geocompr.robinlovelace.net/) book by Robin Lovelace, Jakub Nowosad,
and Jannes Muenchow.

#### Simple Map

<div class='code--container'>
<div>

```stata
* Mapping in Stata requires the spmap and shp2dta 
* commands, and also that you convert your (say) 
* shapefile to .dta format first. We won't go through 
* all that here, but see: 
* https://www.stata.com/support/faqs/graphics/spmap-and-maps/
```
</div>
<div>

```r
# This example uses the North Carolina shapefile that is
# bundled with the sf package. 
nc = st_read(system.file("shape/nc.shp", package = "sf")) 
plot(nc[, 'BIR74'])
# Or, if you have ggplot2 loaded: 
ggplot(nc, aes(fill=BIR74)) + geom_sf()
```
</div>
</div>





                     

