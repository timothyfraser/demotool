# demo_tool
- **Description**: Repository for demoing how to make a usable tool in an R package
- **Author**: Tim Fraser, PhD

Want to make a small, compact R package? It's not as hard as you'd think! Here's a brief tutorial that will walk you through the process of building your first publicly available tool as an R package!

## Necessary Files
Your R package will need the following documents.

- `DESCRIPTION`: the package metadata file, including eg. the package name, description, maintainers, etc. Always work from a template, because it can be persnickety.
- `/R` code folder, with scripts whose names should match your functions, 1 function per script, usually.
- `NAMESPACE`: a list of all functions for exporting. You don't edit this yourself; `devtools::document()` does it for you.
- `/man`: manual folder, with manuals summarizing your functions. You don't make this yourself; `devtools::document()` does it for you. Automatically generated from your `roxygen` comments for your functions.
- `/data`: data folder, with `.rda` files for any objects your package relies on to run functions, eg. `data.frames`, `vectors`, etc. Name of `.rda` file should match name of object when loaded.
- `/z`: extra folder, for any other scripts or files you may need. I put scripts for building and testing the package here, as well as raw data.
- `.buildignore`: a list of all files and folders to ignore when building the package. Eg. `/z` should go in here.

### `DESCRIPTION` file template

For example, [my `DESCRIPTION` file](https://github.com/timothyfraser/demo_tool/blob/main/DESCRIPTION#L1C1-L18C10) looks like this (see below)! Update the `Package:` name, `Title:`, `Authors@R:`, `Maintainer:`, and `Description:` fields. 
```
Package: demo_tool
Type: Package
Title: Demo package for making a publicly available tool as an R package.
Version: 0.1.0
Authors@R: 
  person("Fraser", "Timothy", , "tmf77@cornell.edu", role = c("aut", "cre"),
  comment = c(ORCID = "0000-0002-4509-0244"))
  person("First Name", "Last Name", , "myemail@cornell.edu", role = c("aut"))
Maintainer: Timothy Fraser <tmf77@cornell.edu>
Description: One-line description for package goes here. Make sure this script ends with an extra blank line after `Imports:`
License: MIT + file LICENSE
Encoding: UTF-8
LazyData: true
Rxoygen: list(markdown = TRUE)
RoxygenNote: 7.2.3
Depends: 
    R (>= 3.5.0)
Imports: 

```

### `NAMESPACE` file template

Your `NAMESPACE` file is automatically generated when you `devtools::document()` the package. Don't edit it yourself. It tells the package what functions to export for use. Here's a short sample:
```
# Generated by roxygen2: do not edit by hand

export(get_prob)
```

### `function` template

Your `/R` folder contains scripts, one for each function. Each script will need a a special header, called `roxygen2` commenting. Instead of `#`, we write `#'`, followed by a tag like `@name`, which carries special meaning and helps the package auto generate its own documentation. I've written up a short function called `plus_one()`, as well as a long function called `get_prob()` that you can use as templates when building your functions.

Let's look at them!

#### Short `function` template: `plus_one()`

```r
#' @name plus_one
#' @title Plus One
#' @description Function to add 1 to a numeric input vector.
#' @author Tim Fraser, PhD
#' @params x (numeric) vector of 1 or more numeric values.
#' @note Adding `@export` below means this function will become accessible by package users, rather than being an internal-only function.
#' @export
plus_one = function(x){
  output = x + 1
  return(output)
} 
```

#### Long `function` template: `get_prob()`

Here's a template for a longer, more complex function. Check out my extra commenting, which will help you edit the code to suit your needs.

```r
#' @name get_prob
#' @title `get_prob()`
#' @description
#' A short description of the function!
#' Can be multiple lines
#' Notice how roxygen commenting always starts with #'
#' For example:
#' This function calculates system reliability over time t given a set of lambdas supplied.
#' @note
#' - `@name` should have no spaces and match the function name.
#' - `@title` is the official title and can be anything.
#' - `@description` is a description that can span multiple lines
#' - `@param` marks each input parameters for your function.
#' - `@export` exports your function to be available to a user of your package. (Eg. not an internal function)
#' - `@importFrom` bundles into your package 1 or more specific functions from another package, so that your package will always function.
#' @param t [integer] time passed. Can be a single integer or a vector of integers.
#' @param lambdas [vector] a vector of failure rates for components, named $\lambda$
#' @param type [character] a single value describing whether these probabilities should be combined using the rules of series or parallel systems.
#'
#' @note You can specify default inputs for an input parameter like with `type = "series"` below.
#' @examples 
#' 
#' # Get series system probability at each time t
#' get_prob(t = c(2,4,5,6), lambdas = c(0.001, 0.02), type = "series")
#' 
#' # Get parallel system probability at each time t
#' get_prob(t = c(2,4,5,6), lambdas = c(0.001, 0.02), type = "parallel")
#' 
#' @importFrom tidyr expand_grid
#' @importFrom dplyr `%>%` mutate summarize group_by
#' 
#' @export

get_prob = function(t = 100, lambdas, type = "series"){
  
  # Testing values #########################################
  #    I often store my testing values at the top of the function, 
  #    **commented out**, so I can run them then test the function easily.
  # t = c(100, 200, 300, 400)
  # lambdas = c(0.001, 0.002, 0.01)
  # type = "series"
  
  # Testing packages #######################################
  #    I often list my packages for testing here. 
  #    But be sure to add specific functions to @importFrom
  # library(dplyr)
  # library(tidyr)
  
  # Error handling #####################################
  # If type is not in series or parallel
  if(!type %in% c("series", "parallel")){
    # Stop and provide this error message.
    stop("type must equal 'series' or 'parallel'.")
  }
  
  
  # Get a grid of t and lambdas
  # Numbering your objects can help show progression when developing functions
  grid1 = tidyr::expand_grid(t = t, lambda = lambdas)
  
  grid2 = grid1 %>%
    # For each row,
    dplyr::mutate(
      # Calculate cumulative probability of failure by time t 
      # using exponential distribution, with rate from lambda column
      p_fail = pexp(t, rate = lambda),
      # Then get probability of survival/reliability by time t
      p_reliability = 1 - p_fail)
  
  # If type parameter equals 'series'
  if(type == "series"){
    # Get stats...
    stat = grid2 %>%
      # For each time t
      group_by(t) %>%
      # calculating the product of each component's reliability
      summarize(prob = prod(p_reliability) )
    
    # Or, instead, if type parameter equals 'parallel'
  }else if(type == "parallel"){
    # Get stats...
    stat = grid2 %>%
      # For each time t
      group_by(t) %>%
      # calculating 1 minus the product of each component's chance of failure
      summarize(prob = 1 - prod(p_fail))  
    # Or, if some other value was input...
  }
  
  # I like to always use return(). It's very clear.
  return(stat)
 
}
```

## `devtools`

Once you have your prerequisite files, you can build your R package!
To build an R package, you will need to install `devtools`, the package for R package development!

There are 3 main steps: `document()` your functions, `build()` your package, and then install it with `install.packages("packagename.tar.gz", type = "source").
