`svyset_manifesto`

# How to specify survey settings

## Stas Kolenikov, Brady West, Peter Lugtig

In this repository, we document our understanding of, and recommendations for, appropriate best practices
in specifying the complex sampling design settings in statistical software. We will briefly talk about these features,
their impact on estimation procedures, how statistical software treates them, and how the survey data providers
can make data users' life easier by clearly documenting the technically accurate and efficient ways to tell the software
that the data should be analyzed according to the complex sampling design features.

## Survey sampling features

Big four: strata, clusters, unequal probabilities, weight adjustments.

### Stratification

Stratification = breaking up the population/frames into mutually exclusive groups before sampling.

*  Geographic regions in f2f samples
*  Diagnostic groups in patient list samples

Why?

*  Oversample subpopulations of interest if they can be identified on the frame(s)
*  Oversample areas of higher concentration of the target rare population
*  Ensure specific accuracy targets in subpopulations of interest
*  Utilize different sampling designs/frames in different strata
*  Balance things around/avoid weird outlying samples/spread the sample across the whole population

### Cluster samples

Cluster, or multistage sampling design = sampling groups of units rather than the ultimate observation units.

*  Geographic units (e.g., census tracts) in f2f samples
*  Entities in natural hierarchies (e.g., hospitals/practices and providers within a practice)

Why?

*  Complete lists of all units are not available, but survey statistician
        can obtain lists of administrative units
        for which residence or health service of observation units
        can be easily identified
*  Reduce interviewer travel time/cost in f2f surveys
*  Interest in multilevel modeling of hierarchical structures

Terminology: PSU = primary sampling unit = cluster
### Unequal probabilities of selection

Why?

*  Oversample (smaller) subpopulations of interest (e.g., ethnic/racial minorities) that would not have sufficient sample
   sizes in an equal probability of selection method (epsem) sample
*  Oversample areas of higher concentration of the target rare population
*  Result of multiple stage/cluster sampling
   - Most f2f samples are design with probability proportional to size (PPS) sampling at the first few stages,
            fixed sample size at last stage $\Rightarrow$ approximately EPSEM
   -  If measures of sizes are not accurate, or differential nonresponse is encountered, no longer EPSEM
*  Unintended result of multiple frame sampling
    - dual phone users, i.e., those who have both landline and cell phone service, are more likely to be selected

### Weight adjustments

Why? Corrections for...

*  eligibility
*  nonresponse (unavoidable in real world)
*  frame noncoverage
*  frame overlap in multiple frame surveys
*  statistical efficiency

Kalton, Flores-Cervantes (2003), Valliant, Dever, Kreuter (2013), Valliant and Dever (2017), Kolenikov (2016), etc.

### Sampling is about doing the best job for the money

In the end of the day, all of the sampling features are there for 1+ of the following reasons:

*  Save money
            - use cluster samples to save on travel costs
            - use stratified samples to realize statistical efficiency gains
*  Cannot get the full population listing
            - ... so have to use area samples to gradually zoom down to individuals
            - ... so have to use infrastructure created for a different purpose (telecom or postal) to contact people
            - ... so have to sample a larger, general population, and screen out the eligible rare/hard to reach population
*  Overcome the real world data collection difficulties
            -  nonresponse weight adjustments

## Survey settings in statistical software

The most common public use data file specification of an areal probability sampling design is that
of a two-stage stratified clustered sample. It is nearly always an approximation to the true design,
as most typically the design would include more stages, and some additional modifications of the 
sampling design variables would be undertaken: true sampling strata or units would be combined
or split, units would be swapped with one another, etc., typically in order to mask the true geographical
locations of respondents, as geography is one of the strongest factors putting individuals
at risk of identification and disclosure.

In the examples below, we provide the following semi-standardized examples:

- a "public use" stratified two-stage design:
  * the data file in the package native format is `PUMS_svy`, with an appropriate extension
  * strata are `thisStrat`
  * clusters are `thisPSU`
  * weights are `thisWeight`
- a "dual frame RDD" design, approximated by an unequal probability design:
  * the data file in the package native format is `RDD_svy`, with an appropriate extension
  * weights are `thisWeight`
- a design with the bootstrap replicate weights:
  * the data file in the package native format is `BSTRAP_svy`, with an appropriate extension
  * the main weights are `thisWeight`
  * the replicate weights are `bsWeight1`, `bsWeight2`, ..., `bsWeight100`

In addition, three analyses are discussed:
- estimation of the total of a continuous variable `y`;
- cross-tabulation of two categorical variables `sex` and `race`;
- analysis in subpopulation/domain defined by age restriction, `age` between 18 and 30.

### R

Implementation of complex survey estimation in `library(survey)` separates the steps of declaring the sampling design
and running estimation.

(In terms of reading the input data, we assume that the user follows the best practices of workflow management
and uses `library(here)` to identify the root of the project; 
see [Bryan (2017)](https://www.tidyverse.org/articles/2017/12/workflow-vs-script/)).

The "public use" stratified two-stage design:

```
# prerequisites
library(survey)
library(here)
# read the data
thisSurvey <- readData(here("data/PUMS_svy.Rdata"))
# specify the design
thisDesign <- svydesign(id =~ thisPSU, strat =~ thisStrat, weights =~thisWeight, data =~ thisSurvey)
# estimate the total
(total_y <- svytotal(~y, design = thisDesign) )
# tabulate
(tab1_sex_race <- svymean( ~interaction(sex,race,drop=TRUE), design = thisDesign ) )
(tab2_sex_race <- svytable( ~sex+race, design = thisDesign) )
(tab3_sex_race <- svyby(~sex, by = ~race, design = thisDesign, FUN = svymean)
# subpopulation estimation: redeclare the design
young_adults <- subset( design = thisDesign, ( (age>=18) & (age<=30) ) )
(total_y_young <- svytotal(~y, design = young_adults )
```

In the above, the line `( object <- function_call(input1, ... ) )` simultaneously creates and assigns 
the object, and prints it. Lumley (2010) notes that by default, all functions give missing values (`NA`)
when they encounter item missing data. To discard the missing data from analysis, `na.rm=TRUE` should
be specified as an option to the `svy...(...,na.rm=TRUE)` functions, with the effect of treating 
the non-missing data data as a subpopulation.

The RDD unequal weights design:

```
# prerequisites
library(survey)
library(here)
# read the data
thisSurvey <- readData(here("data/BSTRAP_svy.Rdata"))
# specify the design
thisDesign <- svrepdesign(id =~ 1, weights =~thisWeight, data =~ thisSurvey)
# estimation can use the same syntax as above
```

The replicate weight design:

```
# prerequisites
library(survey)
library(here)
# read the data
thisSurvey <- readData(here("data/RDD_svy.Rdata"))
# specify the design
thisDesign <- svydesign(weights =~thisWeight, data =~ thisSurvey, 
                        repweights =~ "bsWeight[0-9]+", type="bootstrap",
                        combined.weights = TRUE)
# estimation can use the same syntax as above
```

In the above syntax, `"bsWeight[0-9]+"` is a [*regular expression*](https://regexr.com/) which, in this case, builds
a filter for variable names as follows:
1. must start with the text `bsWeight` exactly;
2. this prefix must be followed by a digit `[0-9]`
3. this digit must happen at least once, and may happen an unlimited number of times (`+` modifier).

For more examples, see Thomas Lumley's documentation of the `library(survey)` package:
- [Lumley (2010)](https://www.amazon.com/Complex-Surveys-Guide-Analysis-Using/dp/0470284307) book;
- http://r-survey.r-forge.r-project.org/survey/, home of the `library(survey)` package.

### Stata

```
use thisSurvey, clear
svyset 
* if empty, specify svyset on your own
svyset thisPSU [pw=thisWeight], strata(thisStrat)
```

### SAS

### See also

https://github.com/skolenik/ICHPS2018-svy

https://statstas.shinyapps.io/svysettings/

## Survey settings documentation: rubrics

1. **Can a survey statistician figure out from the documentation how to set the data up for correct estimation?**
This would be a person with training on par or exceeding the level of Lohr or Kish textbooks, and applied experience
on par or exceeding Lumley or Heeringa, West and Berglund books.

2. **Can an applied researcher figure out from documentation how to set the data up?**
This would be a person who has only cursory knowledge of survey methodology, based on at most several hours 
of classroom instruction in their "methods" class or a short course at a conference.

3. **Is everything described succinctly in one place, or scattered through the document?** 
It is of course easier on the user when all the relevant information is easily available in a single section.
However some reports put weights in one place, e.g. where sampling was described, 
while cluster/strata/variance estimation only appears some twenty pages away.

4. **Is specific syntax to specify survey settings given?** (bonus) ... in all three common statistical languages (R, SAS and Stata)? 

5. **Are there examples given for how to answer substantive research questions?**
In all languages, there are specific ways to run commands that are survey-design-aware. 
In other words, only specifying the design may not be sufficient in ensuring that estimation is done correctly.

6. (Bonus) **Is an executive summary description of the study design available?**
Many researchers would appreciate a two-three sentence paragraph to summarize the sampling design that
they could copy and paste into their papers, e.g.,

> {This survey} is a three-stage areal sampling design survey with census tracts, households, and individuals
as sampling units. The final analysis weights provided by {the organization who collected the data} account 
for unequal selection probabilities, nonresponse, and study eligibility, and are used in all analyses
reported in this paper. Standard errors are estimated using the complex survey bootstrap variance estimation procedures.

or

> {This survey} is a dual-frame RDD survey that collected data on both landline and mobile phones.
The final analysis weights provided by {the organization who collected the data} account 
for unequal selection probabilities, nonresponse, and study eligibility, and are used in all analyses
reported in this paper. Standard errors are estimated using Taylor series linearization,
the default analytical method available in most statistical packages.

7. (Bonus) **What kind of references are provided?**
It is often helpful to the end users if the description of the sampling design features is
accompanied by the references to (a) methodological literature describing them in general
(e.g., texbooks such as Korn & Graubard, Kish, Lohr, Heeringa-West-Berglund, Lumley, etc.),
and (b) technical publications specific to the study in question, such as 
the JSM or AAPOR proceedings, technical reports on the provider website, or publications
in technical literature describing the study, if appropriate. E.g., the description
of clustered sampling designs used in the U.S. Census Bureau large scale surveys
such as the American Community Survey or Current Population Survey could refer 
to the general description of stratified clustered surveys,
to the user Handbooks, 
and to the technical papers on variance estimation ([Ash 2011](http://www.citeulike.org/user/ctacmo/article/13018645)).


(Secondary bonus) Are the references pointing out to sources other than the authors? (Chances are if there's a JSM Proceedings paper by the same group of authors, it won't be any clearer, frankly.)

## Survey settings documentation: practice

In this section, we will provide a convenience sample of several public use file (PUF) data set. We will apply the above rubrics
to see how the PUF documentation scores against them.







### Dealing with existing documentation



1. Search documentation for the software footprint as keywords: `svyset` per Stata, `PROC SVY` per SAS, `svydesign` per R
`library(survey)`.

2. If that fails, search for "sampling weight", "final weight", "analysis weight" or "design weight". 
You can search for "weight" per se but you should expect that in health studies, this is likely to produce 
many false positives.

3. See if there is any description of strata and clusters near the text where weights are mentioned.

4. Search for "*PSU*" and "*cluster*" and "*strata*" and "*stratification*" to find
the variables that needed to be specified in survey settings.

5. Search for "*replicate weights*", "*BRR*", "*jackknife*" and "*bootstrap*", the keywords for the popular
replicate variance estimation methods.


### Survey name

**Funding**: the ultimate client of the study

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: 

**Rubrics**: how well the documentation matches the desired criteria

**Score**:












### The Population Assessment of Tobacco and Health

**Funding**: The Population Assessment of Tobacco and Health (PATH) Study is a collaboration 
between the National Institute on Drug Abuse (NIDA), National Institutes of Health (NIH), 
and the Center for Tobacco Products (CTP), Food and Drug Administration (FDA). 

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: 

**Rubrics**: how well the documentation matches the desired criteria

**Score**:









### American Time Use Survey

**Funding**:

**Data collection**:

**Host**: ICPSR

**URL**: https://www.icpsr.umich.edu/icpsrweb/ICPSR/studies/36824/datadocumentation#

**Rubrics**:

**Score**:





### India Human Development Survey

**Funding**: the ultimate client of the study

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: https://www.icpsr.umich.edu/icpsrweb/content/DSDR/idhs-data-guide.html

**Rubrics**: how well the documentation matches the desired criteria

**Score**:



### Survey name

**Funding**: the ultimate client of the study

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: 

**Rubrics**: how well the documentation matches the desired criteria

**Score**:
