Add more interactivity to interactive charts
================

[![CRAN Status Badge](http://www.r-pkg.org/badges/version/manipulateWidget)](http://cran.r-project.org/package=manipulateWidget) [![CRAN Downloads Badge](https://cranlogs.r-pkg.org/badges/manipulateWidget)](http://cran.r-project.org/package=manipulateWidget) [![Travis-CI Build Status](https://travis-ci.org/rte-antares-rpackage/manipulateWidget.svg?branch=master)](https://travis-ci.org/rte-antares-rpackage/manipulateWidget) [![Appveyor Build Status](https://ci.appveyor.com/api/projects/status/6y3tdofl0nk7oc4g/branch/master?svg=true)](https://ci.appveyor.com/project/rte-antares-rpackage/manipulatewidget/branch/master)
[![codecov](https://codecov.io/gh/rte-antares-rpackage/manipulateWidget/branch/master/graph/badge.svg)](https://codecov.io/gh/rte-antares-rpackage/manipulateWidget)


`manipulateWidget` lets you create in just a few lines of R code a nice user interface to modify the data or the graphical parameters of one or multiple interactive charts. It is useful to quickly explore visually some data or for package developers to generate user interfaces easy to maintain.

![Combining widgets and some html content](vignettes/fancy-example.gif)

This R package is largely inspired by the `manipulate` package from Rstudio. It provides the function `manipulateWidget` that can be used to create in a very easy way a graphical interface that let the user modify the data or the parameters of an interactive chart. Technically, the function generates a Shiny gadget, but the user does not even have to know what is Shiny.

Why should you use it?
----------------------

All functionalities of this package can be replicated with other packages like [shiny](https://shiny.rstudio.com/), [flexdashboard](http://rmarkdown.rstudio.com/flexdashboard/), [crosstalk](http://rstudio.github.io/crosstalk/) and others. So why another package?

`manipulateWidget` has three advantages:

-   It is easy and fast to use. Only a few lines of `R` are necessary to create a user interface.
-   Code can be included in any R script. No need to create a dedicated .R or .Rmd file.
-   It works with all htmlwidgets. In contrast, `crosstalk` only supports a few of them.

`manipulateWidget` can be especially powerful for users who are exploring some data set and want to quickly build a graphical tool to see what is in their data. `manipulateWidget` has also some advanced features that can be used with almost no additional code and that could seduce some package developers: grouping inputs, conditional inputs and comparison mode.

Installation
------------

The package can be installed from CRAN:

``` r
install.packages("manipulateWidget")
```

You can also install the latest development version from github:

``` r
devtools::install_github("rte-antares-rpackage/manipulateWidget", ref="develop")
```

Getting started
---------------

The hard part for the user is to write a code that generates an interactive chart. Once this is done, he only has to describe what parameter of the code should be modified by what input control. For instance, consider the following code that identifies clusters in the iris data set and uses package `plotly` to generate an interactive scatter plot.

``` r
library(plotly)
data(iris)

plotClusters <- function(xvar, yvar, nclusters) {
  clusters <- kmeans(iris[, 1:4], centers = nclusters)
  clusters <- paste("Cluster", clusters$cluster)
  
  plot_ly(x = ~iris[[xvar]], y = ~iris[[yvar]], color = ~clusters,
          type = "scatter", mode = "markers") %>% 
    layout(xaxis = list(title=xvar), yaxis = list(title=yvar))
}

plotClusters("Sepal.Width", "Sepal.Length", 3)
```

<img src="README_files/figure-markdown_github/kmeans-1.png" width="600" height="400" />

Once this code has been written, it is very easy to produce a UI that lets the user change the values of the three parameters of the function `plotClusters`:

``` r
varNames <- names(iris)[1:4]

manipulateWidget(
  plotClusters(xvar, yvar, nclusters),
  xvar = mwSelect(varNames),
  yvar = mwSelect(varNames, value = "Sepal.Width"),
  nclusters = mwSlider(1, 10, value = 3)
)
```

![An example of output of manipulateWidget](vignettes/example-kmeans.gif)

The package also provides the `combineWidgets` function to easily combine multiple interactive charts in a single view. Of course both functions can be used together. Here is the code to generate the UI in the first animation.

``` r
myPlotFun <- function(distribution, range, title) {
  randomFun <- switch(distribution, gaussian = rnorm, uniform = runif)
  myData <- data.frame(
    year = seq(range[1], range[2]),
    value = randomFun(n = diff(range) + 1)
  )
  combineWidgets(
    ncol = 2, colsize = c(2, 1),
    dygraph(myData, main = title),
    combineWidgets(
      plot_ly(x = myData$value, type = "histogram"),
      paste(
        "The graph on the left represents a random time series generated using a <b>",
        distribution, "</b>distribution function.<br/>",
        "The chart above represents the empirical distribution of the generated values."
      )
    )
  )
  
}

manipulateWidget(
  myPlotFun(distribution, range, title),
  distribution = mwSelect(choices = c("gaussian", "uniform")),
  range = mwSlider(2000, 2100, value = c(2000, 2100), label = "period"),
  title = mwText()
)
```

For more information take a look at the [package vignette](https://cran.r-project.org/web/packages/manipulateWidget/vignettes/manipulateWidgets.html).
