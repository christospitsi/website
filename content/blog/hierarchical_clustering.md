---
title: "Unsupervised learning - Hierarchical clustering"
date: 2018-01-15T14:23:58Z
tags: ["R", "Statistical Learning"]
draft: true
---
### Hierarchical Clustering

`USArrests` data set contains statistics,
in arrests per 100,000 residents for assault,
murder,
and rape in each of the 50 US states in 1973.

We perform hierarchical clustering on the data set,
with complete linkage and Euclidean distance and
we cut the dendrogram at a height that results in three distinct clusters.
We can see below the clusters created by the model.

``` r
hc.complete <- hclust(dist(USArrests), method = "complete")
a <- cutree(hc.complete, k = 3)

writeLines("Cluster 1")
(c1 <- names(a[a == 1]))
```
    ## Cluster 1
    ##  [1] "Alabama"        "Alaska"         "Arizona"        "California"    
    ##  [5] "Delaware"       "Florida"        "Illinois"       "Louisiana"     
    ##  [9] "Maryland"       "Michigan"       "Mississippi"    "Nevada"        
    ## [13] "New Mexico"     "New York"       "North Carolina" "South Carolina"

``` r
writeLines("\n")
writeLines("Cluster 2")
(c2 <- names(a[a == 2]))
```

    ## Cluster 2
    ##  [1] "Arkansas"      "Colorado"      "Georgia"       "Massachusetts"
    ##  [5] "Missouri"      "New Jersey"    "Oklahoma"      "Oregon"       
    ##  [9] "Rhode Island"  "Tennessee"     "Texas"         "Virginia"     
    ## [13] "Washington"    "Wyoming"

``` r
writeLines("\n")
writeLines("Cluster 3")
(c3 <- names(a[a == 3]))
```
    ## Cluster 3
    ##  [1] "Connecticut"   "Hawaii"        "Idaho"         "Indiana"      
    ##  [5] "Iowa"          "Kansas"        "Kentucky"      "Maine"        
    ##  [9] "Minnesota"     "Montana"       "Nebraska"      "New Hampshire"
    ## [13] "North Dakota"  "Ohio"          "Pennsylvania"  "South Dakota"
    ## [17] "Utah"          "Vermont"       "West Virginia" "Wisconsin"

### Scaled data

We now scale the variable to have standard deviation equal to 1 and
re-apply hierarchical clustering with complete linkage and Euclidean distance.

``` r
USArrest.sc <- scale(USArrests)
hc.complete.sc <- hclust(dist(USArrest.sc), method = "complete")
b <- cutree(hc.complete.sc, k = 3)
```

    ## Cluster 1
    ## [1] "Alabama"        "Alaska"         "Georgia"        "Louisiana"     
    ## [5] "Mississippi"    "North Carolina" "South Carolina" "Tennessee"

    ## Cluster 2
    ##  [1] "Arizona"    "California" "Colorado"   "Florida"    "Illinois"  
    ##  [6] "Maryland"   "Michigan"   "Nevada"     "New Mexico" "New York"  
    ## [11] "Texas"

    ## Cluster 3
    ##  [1] "Arkansas"      "Connecticut"   "Delaware"      "Hawaii"       
    ##  [5] "Idaho"         "Indiana"       "Iowa"          "Kansas"       
    ##  [9] "Kentucky"      "Maine"         "Massachusetts" "Minnesota"    
    ## [13] "Missouri"      "Montana"       "Nebraska"      "New Hampshire"
    ## [17] "New Jersey"    "North Dakota"  "Ohio"          "Oklahoma"     
    ## [21] "Oregon"        "Pennsylvania"  "Rhode Island"  "South Dakota"
    ## [25] "Utah"          "Vermont"       "Virginia"      "Washington"   
    ## [29] "West Virginia" "Wisconsin"     "Wyoming"


We can see above the new clusters created using the scaled data.
The results indicate that by using variable scaling,
the generated clusters are significantly different;
Clusters 1 and 2 are now much smaller and
many states have been moved to a different cluster.

As we can see below,
the `Assault` variable has very high mean value compared to the rest and
as a result it has a much stronger effect on the dissimilarities calculation.

By scaling the variables to have standard deviation equal to one before the dissimilarities are computed,
we effectively give to all variables equal importance during the hierarchical clustering.

``` r
apply(USArrests, 2, mean)
```

    ##   Murder  Assault UrbanPop     Rape
    ##    7.788  170.760   65.540   21.232

*This analysis is based on Exercise 10.7.9 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
