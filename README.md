`svyset_manifesto`

# How to specify survey settings

## Stas Kolenikov, Brady West, Peter Lugtig

In this repository, we wanted to document our understanding of, and recommendations for, appropriate best practices
in specifying the complex sampling design settings in statistical software. We will briefly talk about these features,
their impact on estimation procedures, how statistical software treates them, and how the survey data providers
can make data users' life easier by clearly documenting the technically accurate and efficient ways to tell the software
that the data should be analyzed according to the complex sampling design features.

## Survey sampling features

Big four: strata, clusters, unequal probabilities, weight adjustments.

## Survey settings in statistical software

The most common public use data file specification of an areal probability sampling design is that
of a two-stage stratified clustered sample. It is nearly always an approximation to the true design,
as most typically the design would include more stages, and some additional modifications of the 
sampling design variables would be undertaken: true sampling strata or units would be combined
or split, units would be swapped with one another, etc., typically in order to mask the true geographical
locations of respondents, as geography is one of the strongest factors putting individuals
at risk of identification and disclosure.

### R

```
svydesign(id =~ thisPSU, strat =~ thisStrat, weight =~thisWeight, data =~ thisSurvey)
```

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

4. **Is specific syntax given?** (bonus) ... in all common statistical languages? 

5. (Bonus) **Is an executive summary description of the study design available?**
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

6. (Bonus) **What kind of references are provided?**
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
