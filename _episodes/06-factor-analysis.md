---
# Please do not edit this file directly; it is auto generated.
# Instead, please edit 06-factor-analysis.md in _episodes_rmd/
title: "Factor analysis"
author: "GS Robertson"
teaching: 25
exercises: 10
questions:
- What is factor analysis and when can it be used?
- What are communality and uniqueness in factor analysis?
- How to decide on the number of factors to use?
- How to interpret the output of factor analysis?
objectives:
- Perform a factor analysis on high-dimensional data.
- Select an appropriate number of factors.
- Interpret the output of factor analysis.
keypoints:
- Factor analysis is a method used for reducing dimensionality in a dataset by reducing variation contained in multiple variables into a smaller number of uncorrelated factors.
- PCA can be used to identify the number of factors to initially use in factor analysis.
- The `factanal` function in R can be used to fit a factor analysis where the number of factors is specified by the user.
- Factor analysis can take into account expert knowledge when deciding on the number of factors to use, but a disadvantage is that the output requires careful interpretation.
math: yes
---



# Introduction

Biologists often encounter high-dimensional datasets from which they wish to
extract features which are expected to be represented within the data (e.g.
clinicians may suspect several diseases given a list of clinical signs and
symptoms in patients with respiratory distress).

Another method that may be used to reduce dimensionality in high-dimensional
datasets is a statistical method called **factor analysis (FA)**. Factor
analysis is used to identify *latent features* in a dataset from among a set of
original variables. Like PCA, this method can be used to reduce a large set of
variables in the original dataset to a smaller set of variables that represent
most of the variance in the original variables. FA does this in a similar way
to PCA, by creating a linear combination of factors that represent similarity
between variables.

Unlike PCA, researchers carrying out FA can specify the number of latent
variables (or factors) they expect to find before carrying out the analysis.
Researchers may use exploratory data analysis methods (including PCA) to provide
an initial estimate of how many factors adequately explain variation in the
dataset.

Examples of datasets and research questions for which FA may be appropriate
include: assessing two factors 'writing ability' and 'mathematical ability'
using students' examination scores including sections with multiple questions
on writing and mathematics. FA could be used to give each student a score for
'writing ability' and 'mathematical ability' based on responses to a larger
selection of examination questions (Figure 1). FA may be more appropriate than
PCA in this example as researchers know in advance of analysis how many factors
(or features) the data have and this knowledge can be included in analysis.
Another example of how FA can be used is creating new factors based on a
researcher's knowledge of how genes group into clusters.


<img src="../fig/table_for_fa.png" title="plot of chunk table" alt="plot of chunk table" style="display: block; margin: auto;" />


FA is designed to identify certain unobservable factors from observable
variables included in the original dataset. This is slightly different from
PCA which does not directly do this; instead PCA creates as many features as
there are variables in the dataset which represent different combinations of
variables.


# Introducing an example dataset from prostate cancer patients 

The prostate dataset represents data from 97 men who have prostate cancer.
The data come from a study which examined the correlation between the level
of prostate specific antigen and a number of clinical measures in men who were
about to receive a radical prostatectomy. The data have 97 rows and 9 columns.

Columns are:


- `lcavol`: log(cancer volume)
- `lweight`: log(prostate weight)
- `age`: age (years)
- `lbph`: log(benign prostatic hyperplasia amount)
- `svi`: seminal vesicle invasion
- `lcp`: log (capsular penetration); amount of spread of cancer in outer walls
  of prostate
- `gleason`: [Gleason score](https://en.wikipedia.org/wiki/Gleason_grading_system)
- `pgg45`: percentage Gleason scores 4 or 5
- `lpsa`: log(prostate specific antigen)


In this example we use the clinical variables to identify factors representing
various clinical variables from prostate cancer patients. Two principal
components have already been identified as explaining a large proportion
of variance in the data when these data were analysed in the PCA episode.
We may expect a similar number of factors to exist in the data.

Let's subset the data to just include the log-transformed clinical variables
for the purposes of this episode:


~~~
library(lasso2)
data(Prostate)
~~~
{: .language-r}


~~~
View(Prostate)
~~~
{: .language-r}


~~~
nrow(Prostate)
~~~
{: .language-r}



~~~
[1] 97
~~~
{: .output}



~~~
head(Prostate)
~~~
{: .language-r}



~~~
      lcavol  lweight age      lbph svi       lcp gleason pgg45       lpsa
1 -0.5798185 2.769459  50 -1.386294   0 -1.386294       6     0 -0.4307829
2 -0.9942523 3.319626  58 -1.386294   0 -1.386294       6     0 -0.1625189
3 -0.5108256 2.691243  74 -1.386294   0 -1.386294       7    20 -0.1625189
4 -1.2039728 3.282789  58 -1.386294   0 -1.386294       6     0 -0.1625189
5  0.7514161 3.432373  62 -1.386294   0 -1.386294       6     0  0.3715636
6 -1.0498221 3.228826  50 -1.386294   0 -1.386294       6     0  0.7654678
~~~
{: .output}



~~~
#select five log-transformed clinical variables for further analysis
pros2 <- Prostate[, c(1, 2, 4, 6, 9)]
head(pros2)
~~~
{: .language-r}



~~~
      lcavol  lweight      lbph       lcp       lpsa
1 -0.5798185 2.769459 -1.386294 -1.386294 -0.4307829
2 -0.9942523 3.319626 -1.386294 -1.386294 -0.1625189
3 -0.5108256 2.691243 -1.386294 -1.386294 -0.1625189
4 -1.2039728 3.282789 -1.386294 -1.386294 -0.1625189
5  0.7514161 3.432373 -1.386294 -1.386294  0.3715636
6 -1.0498221 3.228826 -1.386294 -1.386294  0.7654678
~~~
{: .output}

# Performing factor analysis

Factor analysis may be implemented in R using the `factanal()` function
from the stats package (which is a built in package in base R). This
function fits a factor analysis by maximising the log-likelihood using a
data matrix as input. The number of factors to be fitted in the analysis
is specified by the user using the `factors` argument. Options for
transforming the factors by rotating the data in different ways are
available via the `rotation` argument (default is 'none').


> ## Challenge 1 (3 mins)
> 
> Use the `factanal` function to identify the minimum number of factors
> necessary to explain most of the variation in the data
> 
> > ## Solution
> > 
> > 
> > ~~~
> > # Include one factor only
> > pros_fa <- factanal(pros2, factors = 1)
> > pros_fa
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > 
> > Call:
> > factanal(x = pros2, factors = 1)
> > 
> > Uniquenesses:
> >  lcavol lweight    lbph     lcp    lpsa 
> >   0.149   0.936   0.994   0.485   0.362 
> > 
> > Loadings:
> >         Factor1
> > lcavol  0.923  
> > lweight 0.253  
> > lbph           
> > lcp     0.718  
> > lpsa    0.799  
> > 
> >                Factor1
> > SS loadings      2.074
> > Proportion Var   0.415
> > 
> > Test of the hypothesis that 1 factor is sufficient.
> > The chi square statistic is 29.81 on 5 degrees of freedom.
> > The p-value is 1.61e-05 
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > # p-value <0.05 suggests that one factor is not sufficient 
> > # we reject the null hypothesis that one factor captures full
> > # dimensionality in the dataset
> > 
> > # Include two factors
> > pros_fa <- factanal(pros2, factors = 2)
> > pros_fa
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > 
> > Call:
> > factanal(x = pros2, factors = 2)
> > 
> > Uniquenesses:
> >  lcavol lweight    lbph     lcp    lpsa 
> >   0.121   0.422   0.656   0.478   0.317 
> > 
> > Loadings:
> >         Factor1 Factor2
> > lcavol   0.936         
> > lweight  0.165   0.742 
> > lbph             0.586 
> > lcp      0.722         
> > lpsa     0.768   0.307 
> > 
> >                Factor1 Factor2
> > SS loadings      2.015   0.992
> > Proportion Var   0.403   0.198
> > Cumulative Var   0.403   0.601
> > 
> > Test of the hypothesis that 2 factors are sufficient.
> > The chi square statistic is 0.02 on 1 degree of freedom.
> > The p-value is 0.878 
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > # p-value >0.05 suggests that two factors is sufficient 
> > # we cannot reject the null hypothesis that two factors captures
> > # full dimensionality in the dataset
> > 
> > #Include three factors
> > pros_fa <- factanal(pros2, factors = 3)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > Error in factanal(pros2, factors = 3): 3 factors are too many for 5 variables
> > ~~~
> > {: .error}
> > 
> > 
> > 
> > ~~~
> > # Error shows that fitting three factors are not appropriate
> > # for only 5 variables (number of variables too high)
> > ~~~
> > {: .language-r}
> {: .solution}
{: .challenge}


The output of `factanal` shows the loadings for each of the input variables
associated with each factor. The loadings are values between -1 and 1 which
represent the relative contribution each input variable makes to the factors.
Positive values show that these variables are positively related to the
factors, while negative values show a negative relationship between variables
and factors. Loading values are missing for some variables because R does not
print loadings less than 0.1. 

To select the most appropriate number of factors we repeat the factor
analysis using different values in the `factors` argument. If we have an
idea of how many factors there will be before analysis we can start with
that number. The final section of the analysis output shows the results of
a hypothesis test in which the null hypothesis is that the number of factors
used in the model is sufficient to capture most of the variation in the
dataset. If the p-value is less than 0.05, we reject the null hypothesis
and accept that the number of factors included is too small. If the p-value
is greater than 0.05, we accept the null hypothesis that the number of
factors used captures variation in the data.

Like PCA, the fewer factors that can explain most of the variation in the
dataset, the better. It is easier to explore and interpret results using a
smaller number of factors which represent underlying features in the data. 

# Communality and uniqueness

An estimate of the total amount of variation in the original data,
$\hat{\Sigma}$, is calculated by summing our estimate of total communality
and uniqueness of each variable.

- *Communality* represents the degree to which variables explain the same
variation in the data and is calculated for each variable by summing the
(squared) loadings. 
- *Uniqueness* is the opposite of communality and represents the amount of
variation in the data uniquely explained by one variable. Uniqueness is
calculated by subtracting the communality value from 1.


~~~
apply(pros_fa$loadings^2, 1, sum)  #communality
~~~
{: .language-r}



~~~
   lcavol   lweight      lbph       lcp      lpsa 
0.8793780 0.5782317 0.3438669 0.5223639 0.6832788 
~~~
{: .output}



~~~
1 - apply(pros_fa$loadings^2, 1, sum)  #uniqueness
~~~
{: .language-r}



~~~
   lcavol   lweight      lbph       lcp      lpsa 
0.1206220 0.4217683 0.6561331 0.4776361 0.3167212 
~~~
{: .output}

# Visualising the contribution of each variable to the factors

We can plot the loadings in a biplot and label points by variable names
showing the contribution of each variable to the factors.


~~~
#First, carry out factor analysis using two factors
pros_fa <- factanal(pros2, factors = 2)

#plot loadings for each factor
plot(
  pros_fa$loadings[, 1], 
  pros_fa$loadings[, 2],
  xlab = "Factor 1", 
  ylab = "Factor 2", 
  ylim = c(-1, 1),
  xlim = c(-1, 1),
  main = "Factor analysis of prostate data"
)
abline(h = 0, v = 0)

#add column names to each point
text(
  pros_fa$loadings[, 1] - 0.08, 
  pros_fa$loadings[, 2] + 0.08,
  colnames(pros2),
  col = "blue"
)
~~~
{: .language-r}

<img src="../fig/rmd-06-biplot-1.png" title="plot of chunk biplot" alt="plot of chunk biplot" width="432" style="display: block; margin: auto;" />


> ## Challenge 2 (3 mins)
> 
> Use the output from your factor analysis and the plots above to interpret
> the results of your analysis.
> 
> What variables are most important in explaining each factor? Do you think
> this makes sense biologically? Discuss in groups.
> 
> > ## Solution
> > 
> > This plot suggests that the variables lweight and lbph are associated with
> > high values on factor 2 (but lower values on factor 1) and the variables
> > lcavol, lcp and lpsa are associated with high values on factor 1
> > (but lower values on factor 2). There appear to be two 'clusters' of
> > variables which can be represented by the two factors.
> > 
> > The grouping of weight and enlargement (lweight and lbph) makes sense
> > biologically, as we would expect prostate enlargement to be associated
> > with greater weight. The groupings of lcavol, lcp, and lpsa also make
> > sense biologically, as larger cancer volume may be expected to be
> > associated with greater cancer spead and therefore higher PSA in the blood.
> {: .solution}
{: .challenge}

# Advantages and disadvantages of Factor Analysis (FA)

There are several advantages and disadvantages of using factor analysis as a
dimensionality reduction method.

Advantages:

* FA is a useful way of combining different groups of data into known
  representative factors, thus reducing dimensionality in a dataset.
* FA can take into account researchers' expert knowledge when choosing
  the number of factors to use, and can be used to identify latent or hidden
  variables which may not be apparent from using other analysis methods.
* It is easy to implement with many software tools available to carry out FA.

Disadvantages:

* Any analysis method is only as good as the data. FA cannot be used when
  factors are not clearly defined by the user. Justifying the choice of
  number of factors to use is difficult if little is known about the
  structure of the data before analysis is carried out.
* Sometime it can be difficult to interpret what factors mean after
  analysis has been completed. 
* FA is not a commonly used method in some areas of biology
  (including genomics), although this is changing.
* Like PCA, standard methods of carrying out FA assume that input variables
  are continuous, although extensions to FA allow ordinal and binary
  variables to be included (after transforming the input matrix).


# Further reading 

- Gundogdu et al. (2019) Comparison of performances of Principal Component Analysis (PCA) and Factor Analysis (FA) methods on the identification of cancerous and healthy colon tissues. International Journal of Mass Spectrometry 445:116204.
- Kustra et al. (2006) A factor analysis model for functional genomics. BMC Bioinformatics 7: doi:10.1186/1471-2105-7-21.
- Yong, A.G. & Pearce, S. (2013) A beginner's guide to factor analysis: focusing on exploratory factor analysis. Turotials in Quantitative Methods for Psychology 9(2):79-94.

{% include links.md %}
