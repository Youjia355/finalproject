# Project proposal

BST260 Teaching Team, Fall 2026

## Research question

How does having a restrictive handgun waiting-period law in effect
relate to the state firearm mortality rate?

The project uses two datasets already in this package. `gunleg` supplies
the exposure, and `cdcfiremort` supplies the outcome. One vignette will
document and construct the waiting-period exposure. A second vignette
will describe firearm mortality and estimate the association.

## Unit of analysis and population

The unit of analysis is the state-year. The population is the 50 states
and the District of Columbia in the years for which CDC firearm
mortality rates are present in `cdcfiremort`: 2005 and 2014 through
2024. That file has 51 locations in each of those 12 years, so the
analysis set has 612 state-years. Rates for 2006 through 2013 are not in
the file and will not be imputed.

`cdcfiremort$Location` stores the full jurisdiction name. `gunleg`
stores the postal abbreviation in `State.Postal.Abbreviation`. The two
files will be linked with the abbreviation crosswalk in
[`datasets::state.abb`](https://rdrr.io/r/datasets/state.html) and
[`datasets::state.name`](https://rdrr.io/r/datasets/state.html), plus
District of Columbia as `DC`.

## Exposure

The exposure is binary. A state-year is exposed when a restrictive
waiting-period law covering handguns is in effect on July 1 of that
year. Handguns are specified because the RAND class mostly records
handgun waiting periods, while the mortality rate is not split by
firearm type. A law that becomes effective after July 1 is counted in
the next mortality year, so a partial year is not coded as a full year
of exposure.

The source is the RAND state firearm-law records in `gunleg`, restricted
to `Law.Class == "waiting period"`. This class has 111 records across 44
jurisdictions, with effective year, month, and day present on every
record. Within each state, handgun records (`Handguns.or.Long.Guns`
equal to `handgun` or `handgun and long gun`) will be ordered by
effective date and read as a sequence:

- `Restrictive` and `Implement` turns the waiting period on.
- `Restrictive` and `Modify` leaves it on and can change the recorded
  length.
- `Permissive` and `Repeal` turns it off.

Five records do not follow that pattern: a permissive implement and a
permissive modify in Connecticut, both for long guns; a permissive
modify for long guns in Illinois; a permissive modify for handguns in
Rhode Island in 1959; and a restrictive repeal for handguns in Iowa in
2021. Long-gun-only rows are outside the primary exposure. The two
atypical handgun rows will be coded from the statute text in `Content`,
and the vignette will show the text and the resulting on or off
decision. They will not be dropped quietly.

Delaware, Minnesota, Nevada, Ohio, South Carolina, Utah, and Virginia
have no waiting-period record. Those states will be coded as not exposed
in every analysis year. In this database, absence of a record means no
waiting-period statute was catalogued, so it is an unexposed code rather
than a missing value. The supersession date is blank on 65 of 111
waiting-period rows. The coding above uses the sequence of changes, so a
blank supersession date will not be imputed and will not, by itself,
mark a law as missing.

A sensitivity analysis will repeat the construction with long-gun
waiting periods included, so the exposure is on when either a handgun or
a long-gun waiting period is in effect on July 1.

## Outcome

The outcome is `Death.Rate` in `cdcfiremort`, a continuous firearm
mortality rate for each state-year. The rate is present for all 612
rows. `Deaths.` is the corresponding count and will be summarized, but
it is not the outcome, because states differ in population size. The
rate combines firearm deaths and is not limited to homicide.

## Confounders

These adjustment variables are specified from the structure of the
question, before estimating the association.

The primary model will include state indicators and year indicators.
State indicators absorb stable differences across states, including
long-run differences in gun ownership, urbanicity, and economic
conditions. Year indicators absorb shocks shared by all states in a
given year, including the mortality increase around 2020.

Age composition, poverty, firearm ownership, and other firearm laws
passed in the same year are not in these two files. They remain
unmeasured confounders. As one pre-specified sensitivity analysis, a
restrictive permit-to-purchase law will be coded from `gunleg` with the
same July 1 rule and added as a covariate, because that policy is often
adopted together with a waiting period. No other law class will be added
after seeing the waiting-period estimate.

## Planned summaries and report

The gun-law vignette will give the RAND provenance, the count of
waiting-period records, and a table of which states are exposed in each
analysis year. The mortality vignette will give the CDC provenance, the
distribution of `Death.Rate` overall and by exposure, and the
state-and-year-adjusted association, with a confidence interval and
standard errors clustered by state. The written conclusion will be
limited to this state-year comparison. It will state the gap from 2006
to 2013, the difference between a handgun law and an all-firearm death
rate, and the confounders that these files do not measure.
