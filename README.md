# How to specify survey settings

## Stas Kolenikov, Brady West, Peter Lugtig

In this repository, we document our understanding of, and recommendations for, 
appropriate best practices in specifying the complex sampling design settings 
in statistical software that enables design-based analyses of survey data.
We discuss features of the complex survey data such as stratification, clustering, 
unequal probabilities of selection, and calibration, and outline their impact on estimation procedures.
We demonstrate how statistical software treats them, and how the survey data providers 
can make data users' lifes easier by clearly documenting accurate and efficient ways to make sure 
that their software properly accounts for the complex sampling design features.

### About authors

Stas Kolenikov is Principal Scientist at [Abt Associates](http://www.abtassociates.com); 
@skolenik on GitHub; [@StatStas](https://twitter.com/StatStas) on Twitter.
His interest in this project is from the triple perspective of the data provider
at the Division of Data Science, Surveys, and Enabling Technologies of Abt Associates;
of an occasional continuing education instructor teaching courses on survey design and weighting, and estimation
with complex survey data; and of the developer of 
[statistical software for survey weight calibration](https://econpapers.repec.org/software/bocbocode/s458430.htm).

Brady West is Associate Research Professor
in the [Michigan Program in Survey Methodology](http://www.isr.umich.edu/gradprogram/) 
at the [Institute for Social Research](http://www.isr.umich.edu/),  University of Michigan-Ann Arbor.
[@bradytwest](https://twitter.com/bradytwest) on Twitter.
Website: http://www-personal.umich.edu/~bwest/

Peter Lugtig is 
Associate Professor at the Department of Methodology and Statistics
in the School of Social and Behavioural Sciences, University of Utrecht.
[@PeterLugtig](https://twitter.com/PeterLugtig) on Twitter.
Website: http://www.peterlugtig.com/

## Survey sampling features

Big four: strata, clusters, unequal probabilities, weight adjustments.

### Stratification

Stratification = breaking up the population/frames into mutually exclusive groups before sampling.

*  Geographic regions for in-person samples
*  Diagnostic groups for patient list samples
*  Industry and/or employment size and/or geographical regions for establishment samples

Why?

*  Oversample subpopulations of interest if they can be identified on the frame(s)
*  Oversample areas of higher concentration of the target rare population (c.f. http://www.asasrms.org/Proceedings/y2006/Files/JSM2006-000557.pdf)
*  Ensure specific accuracy targets in subpopulations of interest
*  Utilize different sampling designs/frames in different strata
*  Balance things around/avoid weird outlying samples/spread the sample across the whole population
*  Optimize costs vs. precision via [Neyman-Chuprow](https://support.sas.com/documentation/cdl/en/statug/63962/HTML/default/viewer.htm#statug_surveyselect_a0000000208.htm) or more complicated allocations

### Cluster samples

Cluster, or multistage sampling design = sampling groups of units rather than the ultimate observation units.

*  Geographic units (e.g., census tracts) in f2f samples
*  Entities in natural hierarchies (e.g. healt care providers within practices hospitals
                                    e.g. students within classes within schools)

Why?

*  Complete lists of all units are not available, but survey statistician
        can obtain lists of administrative units
        for which residence or other relevant eligibility status of observation units
        can be easily identified
*  Reduce interviewer travel time/cost in f2f surveys
*  Interest in multilevel modeling of hierarchical structures

Terminology: PSU = primary sampling unit = cluster

### Unequal probabilities of selection

Why?

*  Directly oversample (smaller) subpopulations of interest (e.g., ethnic/racial minorities) that would not have sufficient   
        sample sizes in an equal probability of selection method (epsem) sample
*  Indirectly oversample, by selecting areas with higher concentration of the target rare population
*  Result of multiple stage/cluster sampling
   - Most f2f samples are designed with probability proportional to size (PPS) sampling at the first few stages,
            fixed sample size at last stage $\Rightarrow$ approximately EPSEM. 
            In many cases, the sample size at last stage of selection (e.g. the size of a household) is unknown in advance.
   - If measures of sizes are not accurate, or nonresponse depends on cluster (size), no longer EPSEM
*  Unintended result of multiple frame sampling
    - dual phone users, i.e., those who have both landline and cell phone service, are more likely to be selected.

### Weight adjustments

Why? Corrections for...

*  eligibility
*  frame noncoverage
*  frame overlap in multiple frame surveys
*  statistical efficiency
*  nonresponse (unavoidable in real world)

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
            
As a result of all considerations above, most serious surveys that are using f2f or address-based sampling are using a complex 
survey design in their fieldwork. Data resulting from the survey cannot be naively analyzed, but survey weights have to be used. 
Survey statisticians routinely compute weights for users. These weights often take the form of a design weight that corrects for 
eligibility, frame overlap, and unequal seelction probabilities in sampling. A separate nonresponse weight corrects for 
nonresponse, and sometimes for noncoverage errors in the frame used. In some surveys additional weights are provided for the 
purpose of doing cross-national comparisons (multi-country surveys) or longitudinal analysis (cohort or panel studies).
For more information on how modern surveys are efficiently designed, and weights are computed, we refer the reader to Kalton, 
Flores-Cervantes (2003), Lohr (2010), Bethlehem (2011), Valliant, Dever, Kreuter (2013), Valliant and Dever (2017) or Kolenikov 
(2016). 
The weights included in the dataset shouwl be accompanied with documentation on how the weights were computed and should be used 
in practice by applied researchers. We have often encountered the documentation of survey weights to be inadequate. Sometimes, 
details on how the weights were designed are missing. More often, the decsription of the weights is sparse or very technical. 
This then leads to users not using weights at all, or using them incorrectly. West, Sakshaug and Aurelien (2016) have shown for 
example that analytic errors are prevalent in 145 analyses of the survey 'Scientists and Engineers Statistical Data System' 
(SESTAT). This paper seeks to provide a rubric for how survey weights should be documented. We will define a rubric consisting 
of [X] elements, and then use this rubric to discuss the survey documentation of [X] popular surveys. This paper is accompanied 
by a website, where applied researchers can paste example code from SAS, STATA and R and generate corresponding code in other 
software packages to facilitate the correct use of weights in future. https://statstas.shinyapps.io/svysettings/

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
  * Taylor Series Linearization (TSL) is generally the default variance estimation procedure in these settings
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

In Stata, survey settings can be specified once, and be used later with the `svy:` estimation prefix.
The settings can be saved with the data set. This is a recommended best practice for data providers.

```
use thisSurvey, clear
svyset 
* if empty, specify svyset on your own
svyset thisPSU [pw=thisWeight], strata(thisStrat)
* estimate the total
svy :  total y
* tabulate
svy : tab sex race, col se
* subpopulation estimation: subpop option
svy , subpop( if inrange(age,18,30) ) : total y
```

The RDD unequal weights design:

```
use thisSurvey, clear
svyset 
* if empty, specify svyset on your own
svyset thisPSU [pw=thisWeight]
* estimation commands as before
* estimate the total
svy :  total y
* tabulate
svy : tab sex race, col se
* subpopulation estimation: subpop option
svy , subpop( if inrange(age,18,30) ) : total y
```

The replicate weight design:

```
use thisSurvey, clear
svyset 
* if empty, specify svyset on your own
svyset [pw=thisWeight], vce(bootstrap) bsrw( bsWeight* ) mse
* estimate the total
svy :  total y
* tabulate
svy : tab sex race, col se
* subpopulation estimation: subpop option
svy , subpop( if inrange(age,18,30) ) : total y
```

The estimation commands themselves are identical to those for the cluster+strata designs.

The `mse` option of the `svyset` command requests the MSE version of the estimator 
where the original estimate is subtracted, <!--
$$
v[\hat\theta] = \frac1R \sum_{r=1}^R (\hat\theta^{(r)}-\hat\theta)^2
$$
-->
vs. variance version where the mean of the pseudo-values is substracted 
when the squared differences are formed.
<!--
$$
\tilde v[\hat\theta] = \frac1R \sum_{r=1}^R (\hat\theta^{(r)}-\bar{\hat\theta})^2, 
\quad \bar{\hat\theta} = \frac1R \sum_{r=1}^$ \hat\theta^{(r)}
$$
-->

Starting with Stata 15.1, calibrated weights [are supported](https://www.stata.com/bookstore/survey-weights/).

### SAS

In SAS, survey settings need to be declared in every `SURVEY` procedure.

[Tabulations and cross-tabulations](https://support.sas.com/documentation/cdl/en/statug/63962/HTML/default/viewer.htm#surveyfreq_toc.htm):

```
PROC SURVEYFREQ data=thisSurveyLib.thisSurvey;
   WEIGHTS thisWeight;
   CLUSTER thisPSU;
   STRATA thisStrat;
   TABLES sex*race;
RUN;
```

Subpopulation analysis:

```
DATA thisSurveyLib.thisSurvey;
   SET thisSurveyLib.thisSurvey;
   age_18to30 = (age>=18) & (age<=30);
RUN;
PROC SURVEYFREQ data=thisSurveyLib.thisSurvey;
   WEIGHTS thisWeight;
   CLUSTER thisPSU;
   STRATA thisStrat;
   TABLES sex*race;
   DOMAIN age_18to30;
RUN;
```

### See also

https://github.com/skolenik/ICHPS2018-svy

https://statstas.shinyapps.io/svysettings/


## Documentation on appropriate design-based analysis techniques for complex sample survey data: rubrics

1. **Can a survey statistician figure out from the documentation how to set the data up for correct estimation?**
This would be a person with training on par with or exceeding the level of the Lohr (1999) or Kish (1965) textbooks, and applied 
experience on par with or exceeding the Lumley (2010) or Heeringa, West and Berglund (2017) books.

2. **Can an applied researcher figure out from the documentation how to set the data up for correct estimation?**
This would be a person who has some background / training in applied statistical analysis, but has only cursory knowledge of 
survey methodology, based on at most several hours of classroom instruction in their "methods" class or a short course at a 
conference.

3. **Is everything described succinctly in one place, or scattered throughout the document?** 
It is of course easier on the user when all the relevant information is easily available in a single section. However, some 
reports put information about weights in one place, e.g. where sampling was described, while information about other complex 
sampling features (e.g., cluster/strata/variance estimation) only appears some twenty pages away.

4. **Are examples of specific syntax to specify survey settings provided?** 
Has the data producer provided worked and clearly-annotated examples of analyses of the complex sample survey data produced by a 
given survey using the syntax for existing procedures in one or more common statistical software packages? And as a bonus, have 
examples been provided in multiple languages (e.g., SAS, R, and Stata)? 

5. **Are there examples given for how to answer substantive research questions?**
In all languages, there are specific ways to run commands that are survey-design-aware. In other words, only specifying the 
design may not be sufficient in ensuring that estimation is done correctly. For instance, are examples provided for both 
descriptive and analytic (i.e., regression-driven) research questions?

6. (Bonus) **Is an executive summary description of the sample design available?**
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

7. (Bonus) **What kinds of references are provided?**
It is often helpful to the end users if the description of the sampling design features is
accompanied by the references to (a) methodological literature describing them in general
(e.g., texbooks such as Korn & Graubard, Kish, Lohr, Heeringa-West-Berglund, Lumley, etc.),
and (b) technical publications specific to the study in question, such as 
the JSM or AAPOR proceedings, technical reports on the provider website, or publications
in technical literature describing the study, if appropriate. E.g., the description
of clustered sampling designs used in the U.S. Census Bureau large scale surveys
such as the American Community Survey or Current Population Survey could refer 
to the general description of stratified clustered surveys,
to the user Handbooks, and to the technical papers on variance estimation ([Ash 2011](http://www.citeulike.org/user/ctacmo
/article/13018645)).

(Secondary bonus) Are the references pointing out to sources other than the authors? (Chances are if there's a JSM Proceedings 
paper by the same group of authors, it won't be any clearer, frankly.)

The seven rubrics defined here will be used below to "score" several existing examples of documentation for public-use survey 
data files based on these criteria. For example, if the documentation for a public-use data file successfully satisfies / meets 
the first five rubrics above, the documentation will be scored 5/5. These scores are designed to be **illustrative**, in terms 
of rating existing examples of documentation for public-use data files on how effectively they convey complex sampling features 
and how they should be employed in analysis to users. The scores are designed to motivate data producers to improve the clarity 
of their documentation for a variety of data users hoping to analyze large (and usually publically-funded) survey data sets.

## Evaluating documentation in practice

In this section, we will evaluate a convenience sample of the documentation for several public use survey data files (PUFs). We 
will apply the above rubrics to see how the documentation compares in terms of effectively describing appropriate analysis 
techniques to data users.

### Dealing with existing documentation

1. Search documentation for the software footprint as keywords: `svyset` per Stata, 
`PROC SURVEY` per SAS, `svydesign` per R `library(survey)`.

2. If that fails, search for "sampling weight", "final weight", "analysis weight", "survey weight" or "design weight". 
You can search for "weight" per se but you should expect that in health studies, this is likely to produce 
many false positives.

3. See if there is any description of the sampling strata and clusters near the text where weights are mentioned.

4. Search for "*PSU*" and "*cluster*" and "*strata*" and "*stratification*" to find
the variables that needed to be specified in survey settings.

5. Search for "*variance estimation*", the generic technical term to deal with complexities of survey estimation.

6. Search for "*replicate weights*", "*BRR*", "*jackknife*" and "*bootstrap*", the keywords for the popular
replicate variance estimation methods.


### The National Survey of Family Growth (NSFG), 2013--2015

⭐⭐⭐⭐⭐

**Funding**: 

* Eunice Kennedy Shriver National Institute of Child Health and Human Development
* Office of Population Affairs
* NCHS, CDC
* Division of HIV/AIDS Prevention, CDC
* Division of Sexually Transmitted Disease Prevention, CDC
* Division of Reproductive Health, CDC
* Division of Birth Defects and Developmental Disabilities, CDC
* Division of Cancer Prevention and Control, CDC
* Children’s Bureau, Administration for Children and Families (ACF)
* Office of Planning, Research and Evaluation, ACF

**Data collection**: The University of Michigan Survey Research Center (http://src.isr.umich.edu)

**Host**: The National Center for Health Statistics (http://www.cdc.gov/nchs/)

**URL**: http://www.cdc.gov/nchs/nsfg

**Rubrics**: 

1. **Can a survey statistician easily figure out how to declare the complex sampling features to survey analysis software for design-based analysis?** Yes. Electronic documents like 
[Example 1: Variance Estimates for Percentages](https://www.cdc.gov/nchs/data/nsfg/NSFG_2013_2015_VarEst_Ex1.pdf) 
linked from the [documentation page](https://www.cdc.gov/nchs/nsfg/nsfg_2013_2015_puf.htm) 
under *Variance estimation* subtitle make it very easy for survey statisticians and applied researchers alike 
to correctly declare complex sampling features to survey analysis software for design-based analyses.

2. **Can an applied researcher easily figure out how to declare the complex sampling features to survey analysis software for design-based analysis?** Yes. See above.

**3. Is everything that the data user needs to know about the complex sampling contained in one place?** 
Yes, although very little (if anything) is said about the actual complex sample design. 
Instead this information appears in separate electronic files, such as 
[Sample Design Documentation](https://www.cdc.gov/nchs/data/nsfg/NSFG_2013-2015_Sample_Design_Documentation.pdf). 
This is out of necessity, however, given the complexity of the NSFG sample design, 
and all of the information that a user needs to compute weighted point estimates and estimate variance 
accounting for the complex sampling can be found in examples like the one indicated above.

**4. Are examples of specific syntax for performing correct design-based analyses provided?**
Yes. Three examples are clearly documented 
([tabulations for categorical variables](https://www.cdc.gov/nchs/data/nsfg/NSFG_2013_2015_VarEst_Ex1.pdf); 
[means for continuous variables](https://www.cdc.gov/nchs/data/nsfg/NSFG_2013_2015_VarEst_Ex2.pdf);
[analysis with domains/subpopulations](https://www.cdc.gov/nchs/data/nsfg/NSFG_2013_2015_VarEst_Ex3.pdf)) 
and linked on the main documentation page, and both syntax and output are included in each case. 
Bonus: syntax and output are provided for both SAS and Stata.

**5. Are examples of analyses need for addressing specific substantive questions provided?**
Yes; see previous item.

**6. (Bonus) Is an executive summary of the sample design provided?**
Yes; such an executive summary is given in the first section of 
[the main sample document](https://www.cdc.gov/nchs/data/nsfg/NSFG_2013-2015_Sample_Design_Documentation.pdf)

**7. (Bonus) What kinds of references are provided?**
There are several references to the most important sample design literature included in Section 11 of the document linked above.

**Score**: 7/5

The NSFG provides an excellent example of the type of documentation 
that needs to be provided to data users to minimize the risk of analytic error 
due to a failure to account for complex sampling features.


### The Population Assessment of Tobacco and Health

**Funding**: The Population Assessment of Tobacco and Health (PATH) Study is a collaboration 
between the National Institute on Drug Abuse (NIDA), National Institutes of Health (NIH), 
and the Center for Tobacco Products (CTP), Food and Drug Administration (FDA). 

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: 

**Rubrics**: how well the documentation matches the desired criteria

**Score**:



### Understanding Society

:star: :star: :star: :star:

**Funding**: Economic & Social Research Council (ESRC).

**Data collection**: 
- The Institute for Social and Economic Research (ISER), University of Essex
- NatCen Social Research (wave 1-5) - Great Britian
- Central Survey Unit of NISRA (wave 1-5)- Northern Ireland
- Kantar Public UK (wave 6 onwards)

**Host**: The UK data archive: https://discover.ukdataservice.ac.uk/series/?sn=2000053

**URL**: https://discover.ukdataservice.ac.uk/series/?sn=2000053
 - ps://www.understandingsociety.ac.uk/sites/default/files/downloads/documentation/mainstage/user-guides/mainstage-waves-1-7-user-guide.pdf

**Rubrics**:

1. **Can a survey statistician easily figure out how to declare the complex sampling features 
to survey analysis software for design-based analysis?**
yes. See the link above. The stratification is well decribed, both for Understanding Society, and it's predecessor, 
The British Household Panel Study. The sample design is complex, as this study is longitudinal. 
The study inclused refreshment samples to increase sample sizes, include minorities, and new regions into the study. 
There are clear stratification variables available. As the study is a household study, no clustering is used.


2. **Can an applied researcher easily figure out how to declare the complex sampling features to survey analysis software for 
design-based analysis? **
Yes. The design of the study is highly complex however, and the applied researcher needs to have 
a very clear definition of what the target population of their survey exactly is. 
Guidance is provided on what survey weights to use depending on the choice of the target population, 
but the description is technical.

3. **Is everything that the data user needs to know about the complex sampling contained in one place?**
Yes, section 3.9 of the user documentation includes all information. 

4. **Are examples of specific syntax for performing correct design-based analyses provided?**
Yes. Example `svyset` syntax for Stata is given.

5. **Are examples of analyses need for addressing specific substantive questions provided?**
Yes and no. The description is technical, and practical examples of the kind of questions 
researchers would want to answer using this data may help users to select the right set of weights.

6. **(Bonus) Is an executive summary of the sample design provided?**
No. The sample design is also too complex for this.

7. **(Bonus) What kinds of references are provided?**
The documentation includes many references to additional papers and technical reports 
written on the design and analysis of *Understanding Society* Data.


**Score**: 6/7

### European Social Survey ###

:star: :star: :star: :star:

**Funding**: European Commission, Horizon 2020. 
Rounds 1-7 of ESS have been founded by national science foundations and/or European national governments

**Data collection**: coordinated by City University, London, UK. 
Data collection in separate European Countries coordinated within every country. 

**Host**: European Social Survey, formerly at Norwegian data Archive

**URL**: www.europeansocialsurvey.org
http://www.europeansocialsurvey.org/docs/methodology/ESS_weighting_data_1.pdf

**Rubrics**:

1. **Can a survey statistician easily figure out how to declare the complex sampling features 
to survey analysis software for design-based analysis?**
Yes. The European Social Survey is a repeated cross-sectional study conducted in about 30 different countries in Europe.
Sampling is conducted within every country, using either listing methods or registers (of individuals or addresses). 
Three weights (design, poststratification and population equivalence weights) are included in the main datafile. 
This allows for Horvitz-Thompson estimation, but not the specification of a complex survey design.
However, an Integrated Sample data file does include information on stratificiation or cluster variables, 
as well as selection probabilities for every respondent. 
On top of this, a multilevel file adds regional indicators to the main datafile, allowing for multilevel-analysis

2. **Can an applied researcher easily figure out how to declare the complex sampling features 
to survey analysis software for design-based analysis?** 
Yes, three weights are provided: a design weight, a poststratification weight and a population equivalence weight. 
Guidance is included on how to comibine the three weights, and when to use what weight in some examples of analyses.

3. **Is everything that the data user needs to know about the complex sampling contained in one place?**
Documentation is scattered across many different documents and files on the ESS website. 
However, most users in practice would use one round of ESS. 
In that case, the country report files contain details on how fieldwork (including sampling) was conducted.
One good aspect of the use of weights is that the users are explicitly warned that data need to be weighted 
when data are downloaded from the ESS website.

4. **Are examples of specific syntax for performing correct design-based analyses provided?**
No.

5. **Are examples of analyses need for addressing specific substantive questions provided?**
There are a few examples, but not including analysis syntax.

6. **(Bonus) Is an executive summary of the sample design provided?**
There is an executive summary that describes the basic sampling methodology. 
There is no easily accessible executive summary that explains how and why sampling differs over the countries.

7. **(Bonus) What kinds of references are provided?**
There are references to standard textbooks on complex survey design, and references to other documents 
on the ESS website, with more detailed documentation

Score: 2/7


### American Time Use Survey

**Funding**:

**Data collection**:

**Host**: ICPSR

**URL**: https://www.icpsr.umich.edu/icpsrweb/ICPSR/studies/36824/datadocumentation#

**Rubrics**:

**Score**:





### India Human Development Survey

**Funding**: 

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: https://www.icpsr.umich.edu/icpsrweb/content/DSDR/idhs-data-guide.html

**Rubrics**: how well the documentation matches the desired criteria

**Score**:




### A Portrait of Jewish Americans

⭐⭐⭐⭐☆

**Funding**: The Pew Research Center’s 2013 survey of U.S. Jews was conducted by the 
center’s Religion & Public Life Project with generous funding from 
The Pew Charitable Trusts and the Neubauer Family Foundation.

**Data collection**: Abt SRBI under contract to Pew Research Center 

**Host**: Pew Research Center http://www.pewresearch.org/

**URL**: http://www.pewforum.org/dataset/a-portrait-of-jewish-americans/

**Rubrics**: how well the documentation matches the desired criteria

1. **Can a survey statistician figure out from the documentation how to set the data up for correct estimation?**
Yes; survey documentation explains the differences between the household and the person-level weights,
and stresses that the boostrap weights should be used for variance estimation.

2. **Can an applied researcher figure out from the documentation how to set the data up for correct estimation?**
Yes; Stata syntax is provided early in the document, or can be found by search in the PDF file.

3. **Is everything described succinctly in one place, or scattered throughout the document?** 
Yes; all of the relevant information is contained in the **Key Elements of the Data** section in about 2 pages.

4. **Are examples of specific syntax to specify survey settings provided?** 
Yes; item 6 of **Key Elements of the Data** section identifies the variables and provides Stata syntax 
for individual level and household level anlayses. 
(Search for any of `Stata`, `SAS`, `weight`, `svyset` would lead the researcher to this information.)
A warninig is given that SPSS Statistics Base package cannot correctly compute standard errors.

5. **Are there examples given for how to answer substantive research questions?**
No examples are given.

6. (Bonus) **Is an executive summary description of the sample design available?**
Sample design is described in painstaking detail in about 9 pages. No short summary of the design is available
from the technical documentation, although such a summary can be found in the substantive report
(http://www.pewforum.org/2013/10/01/jewish-american-beliefs-attitudes-culture-survey/).

7. (Bonus) **What kinds of references are provided?**
No additional references are given.

**Score**:
4+/5

A Portrait of Jewish Americans is a very well described survey that most researchers will be able to 
analyze correctly by following the instructions of the data provider. 
Slight limitations of the documentation is that examples of the settings are only 
given for one package, Stata, and no examples of substantive analyses, e.g. those leading to the primary
tables in the substantive report, are provided.


### Survey name

**Funding**: the ultimate client of the study

**Data collection**: the organization who collected and documented the data

**Host**: the organization that hosts the data

**URL**: 

**Rubrics**: how well the documentation matches the desired criteria

**Score**:
