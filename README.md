# plumber <a href='https://rstudio.github.io/plumber'><img src='man/figures/logo.png' align="right" height="138.5" /></a>

<!-- badges: start -->
[![R build status](https://github.com/rstudio/plumber/workflows/R-CMD-check/badge.svg)](https://github.com/rstudio/plumber/actions)
[![](https://www.r-pkg.org/badges/version/plumber)](https://www.r-pkg.org/pkg/plumber)
[![CRAN RStudio mirror downloads](https://cranlogs.r-pkg.org/badges/plumber?color=brightgreen)](https://www.r-pkg.org/pkg/plumber)
[![codecov](https://codecov.io/gh/trestletech/plumber/branch/master/graph/badge.svg)](https://codecov.io/gh/trestletech/plumber)
[![RStudio community](https://img.shields.io/badge/community-plumber-blue?style=social&logo=rstudio&logoColor=75AADB)](https://community.rstudio.com/tags/plumber)
<!-- badges: end -->

<!-- <img align="right" src="https://www.rplumber.io/components/images/plumber-broken.png" /> -->

The R Programming Language [@R-base] has become one of the most dominant programming languages for data analysis and visualization in recent years. At the same time, web services have become a common language for allowing various systems to interact with one another. The `plumber` R package [@plumber] allows users to expose existing R code as a service available to others on the Web. Plumber is best illustrated with an example:

```r
# plumber.R

#' Echo the parameter that was sent in
#' @param msg The message to echo back.
#' @get /echo
function(msg=""){
  list(msg = paste0("The message is: '", msg, "'"))
}

#' Plot out data from the iris dataset
#' @param spec If provided, filter the data to only this species (e.g. 'setosa')
#' @get /plot
#' @png
function(spec){
  myData <- iris
  title <- "All Species"

  # Filter if the species was specified
  if (!missing(spec)){
    title <- paste0("Only the '", spec, "' Species")
    myData <- subset(iris, Species == spec)
  }

  plot(myData$Sepal.Length, myData$Petal.Length,
       main=title, xlab="Sepal Length", ylab="Petal Length")
}
```

Even without knowing R, you can probably get a rough idea for what the above Plumber API will do. The first function above defines the `/echo` endpoint which simply echoes back the text that it was sent. The second function generates a plot based on Edgar Anderson's famous Iris Dataset; it includes a filter that allows the caller to subset the dataset to a particular species.

Plumber makes use of these comment "annotations" above your functions to define the web service. When you feed the above file into Plumber, you'll get a runnable web service that other systems can interact with over a network.

## Web APIs

The Hypertext Transfer Protocol (HTTP) is the dominant medium by which information is exchanged on the Internet. An Application Programming Interface (API) is a broad term that defines the rules that guide your interaction with some software. In the case of HTTP APIs, you have a defined set of endpoints that accept particular inputs. Plumber translates the annotations you place on your functions into an HTTP API that can be called from other machines on your network. If you execute your Plumber API on a public server, you can even make your API available to the public Internet.

HTTP APIs have become the predominant language by which software communicates. By creating an HTTP API, you'll empower your R code to be leveraged by other services -- whether they're housed inside your organization or hosted on the other side of the world. Here are just a few ideas of the doors that are opened to you when you wrap your R code in a Plumber API:

 - Software written in other languages in your organization can run your R code. Your company's Java application could now pull in a custom ggplot2 graph that you generate on-demand, or a Python client could query a predictive model defined in R.
 - You can have [some third-party](https://www.mailgun.com/) receive emails on your behalf and then notify your Plumber service when new messages arrive.
 - You could register a "[Slash Command](https://api.slack.com/slash-commands)" on Slack, enabling you to execute your R function in response to a command being entered in Slack.
 - You can write JavaScript code that queries your Plumber API from a visitor's web browser. Even further, you could use Plumber exclusively as the back-end of an interactive web application.


Plumber allows you to create a web API by merely decorating your existing R
source code with special comments. Take a look at an example.

```r
# plumber.R

#* Echo back the input
#* @param msg The message to echo
#* @get /echo
function(msg=""){
  list(msg = paste0("The message is: '", msg, "'"))
}

#* Plot a histogram
#* @png
#* @get /plot
function(){
  rand <- rnorm(100)
  hist(rand)
}

#* Return the sum of two numbers
#* @param a The first number to add
#* @param b The second number to add
#* @post /sum
function(a, b){
  as.numeric(a) + as.numeric(b)
}
```

These comments allow plumber to make your R functions available as API
endpoints. You can use either `#*` as the prefix or `#'`, but we recommend the
former since `#'` will collide with Roxygen.

```r
library(plumber)
r <- plumb("plumber.R")  # Where 'plumber.R' is the location of the file shown above
r$run(port=8000)
```

You can visit this URL using a browser or a terminal to run your R function and
get the results. For instance
[http://localhost:8000/plot](http://localhost:8000/plot) will show you a
histogram, and
[http://localhost:8000/echo?msg=hello](http://localhost:8000/echo?msg=hello)
will echo back the 'hello' message you provided.

Here we're using `curl` via a Mac/Linux terminal.

```
$ curl "http://localhost:8000/echo"
 {"msg":["The message is: ''"]}
$ curl "http://localhost:8000/echo?msg=hello"
 {"msg":["The message is: 'hello'"]}
```

As you might have guessed, the request's query string parameters are forwarded
to the R function as arguments (as character strings).

```
$ curl --data "a=4&b=3" "http://localhost:8000/sum"
 [7]
```

You can also send your data as JSON:

```
$ curl --data '{"a":4, "b":5}' http://localhost:8000/sum
 [9]
```

## Installation

You can install the latest stable version from CRAN using the following command:

```r
install.packages("plumber")
```

If you want to try out the latest development version, you can install it from
GitHub. The easiest way to do that is by using `devtools`.

```r
library(devtools)
install_github("rstudio/plumber")
library(plumber)
```

## Hosting

If you're just getting started with hosting cloud servers, the
[DigitalOcean](https://www.digitalocean.com) integration included in plumber
will be the best way to get started. You'll be able to get a server hosting your
custom API in just two R commands. Full documentation is available at
https://www.rplumber.io/docs/digitalocean/.

[RStudio Connect](https://www.rstudio.com/products/connect/) is a commercial
publishing platform that enables R developers to easily publish a variety of R
content types, including Plumber APIs. Additional documentation is available at
https://www.rplumber.io/docs/hosting.html#rstudio-connect.

A couple of other approaches to hosting plumber are also made available:

 - PM2 - https://www.rplumber.io/docs/hosting/
 - Docker - https://www.rplumber.io/docs/docker/

## Related Projects

- [OpenCPU](https://www.opencpu.org/) - A server designed for hosting R APIs
  with an eye towards scientific research.
- [jug](http://bart6114.github.io/jug/index.html) - *(development discontinued)*
  an R package similar to Plumber but uses a more programmatic approach to
  constructing the API.

## Provenance

plumber was originally released as the `rapier` package and has since been
renamed (7/13/2015).
