
```r
library(knitr)
knitr::opts_chunk$set(echo = FALSE, message = FALSE, warning = FALSE, cache = FALSE,fig.cap = "", dpi = 400)
```

# Introduction

Retail churn is different than most other forms of churn since every transaction could be that customer's last, or one of a long sequence of transactions.  Normally, churn is a classification problem, but I don't think that classification is appropriate for non-contractual cases. By means of example, suppose you run a retail hardware store.  A competitor opens closer to your most loyal customers and thus offers them the benefit of saving time.  Your most loyal customers may churn without showing any signs.  A typical classification algorithm would missclassify these customers, and cost your business in the long run.

When a customer churns from retail, their between transaction times are large.  Perhaps so large that it may prompt retailers to think "Wow, I haven't seen customer X in a long time". You could even say the time between transactions is *anomalously large*.  Thus, churn modelling in retail is not a classification problem, it is an anomaly detection problem.  In order to determine when your customers are churning or likely to churn, you need to know when they are displaying anomalously large between transaction times.

We first need an idea of what "anomalously" means.  I want to be able to make claims like "9 times out of 10, Customer X will make his next transaction within Y days".  If Customer X does not make another transaction within Y days, we know that there is only a 1 in 10 chance of this happening, and that this behaviour is anomalous.

To do this, we will need each customer's between transaction time distribution. This may be difficult to estimate, especially if the distribution is multimodal or irregular.  To avoid this difficulty, I'll take a non-parametric approach and use the Empirical Cumulative Distribution Function to approximate the quantiles of each customer's between transaction time distribution.  Once I have the ECDF, I can approximate the 90th percentile, and obtain estimates of the nature I've described above.

To do demonstrate my methodology, I'll use retail data obtained from the [UCI Machine Learning Respository](http://archive.ics.uci.edu/ml/datasets/online+retail).  

Let's get started.


# Data Munging

The first thing we'll have to do is slurp in the data.  Once we do that, we'll find that the rows of the data contain information about products, such as: how many were bought (`Quantity`), the price per unit (`Price`), who bought the product (`Customer ID`), when the product was bought (`InvoiceDate`), and which transaction the product was bought under (`InvoiceNo`).

What I really need to know is who bought a product and when.  To do this, I can group by the `Invoice No`, `Customer ID`, and `Invoice Date`.  This will tell me when a customer made a distinct purchase.  We'll have to filter out the returns from the data set.  A return is made when `Quantity<0`, so that is easy enough using `filter`.  From there, we can determine the time between transactions for each customer.



 
 
Let's visualize the distributions for each customer (shown below).  Some distributions look as if they are exponential (which would be really nice because then we could model purchase incidence as a Poisson random variable).  Others are more irregular.  Modelling all these distributions as coming from one separately parameterized distribution would be very difficult.  Our non-parametric method is way easier, as we will see.

![](figure/unnamed-chunk-3-1.png)

# Computation of the ECDF
 
I've written a little function to compute the ECDF for each customer.  Then, I can plot each ECDF and draw a line at 0.9.  The time where the ECDF crosses the line is the the approximate 90th percentile.  So if the ECDF crosses our line at 23 days, that means 9 times out of 10 that customer will make another transaction within 23 days.

Better yet, we can compute the approximate 90th percentile and display it in a dataframe.


![](figure/unnamed-chunk-4-1.png)


```
## # A tibble: 10 x 2
##    CustomerID percentile.90
##        <fctr>         <dbl>
##  1      14911      4.940833
##  2      17841      5.073542
##  3      14606      6.291250
##  4      12748      6.973056
##  5      15311      7.011111
##  6      12971      8.925833
##  7      14527      9.018056
##  8      16422     11.015903
##  9      15039     11.433333
## 10      13089     12.022639
```

That's it!  We now know the point when each customer will begin to act "anomalously".  


# Discussion

Churn is very different for retailers, which means taking a different approach to modelling churn.  When a customer has churned, their time between transactions is anomalously large, so we should have an idea of what "anomalously" means for each customer.  Using the ECDF, we have estimated the 90th percentile of each customers between transaction time distribution in a non-parametric way. Now, we can examine the last time a customer has transacted, and if the time between then and now is near the 90th percentile (or any percentile you deem appropriate) then we can call them "at risk for churn" and take appropriate action to prevent them from churning.  Best of all, with more data our approach will become better and better since the ECDF converges in distribution to the CDF.

In this approach, I've not accounted for seasonality in transactions.  In implementation, the window in which we compute the ECDF would have to be appropriately short.  Parametric methods exist for modelling between transaction times, which would give better insight into the phenomena as a whole.  These methods are currently under investigation by the author.
