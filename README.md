# Rpackage
Tutorial on creating a custom R package. I have put this together mostly for my own benefit so I don't have to relearn how to create a custom R package.

# 0. Environment setup (windows)

* Ensure R is installed

* Ensure environment variable "path" contains path to R executables, something like `C:\Program Files\R\R-4.4.2\bin\`.

## 0.1 R libraries needed

1. devtools
2. usethis
3. testthat

# 1. Creating the package

## 1.0 Package fundamentals

Start up command prompt.

Navigate to the path (e.g. `C:\src`) in which we want to develop the package.

Start up R using `R.exe` in windows command prompt.

Create the package fundamentals using:

<pre><code>> library(devtools)
> devtools::create('newPackage')
</code></pre>

This creates the following (or similar) under the development directory:

<pre><code>----newPackage
|   DESCRIPTION
|   NAMESPACE
|
+---R
|
</code></pre>

This is a working package and is/should be buildable and installable (see section 1.2)

### 1.0.1 New package contents

The directory `R` will contain R source code.

The `NAMESPACE` file contains a warning that this is auto-generated (by roxygen2) and not to edit it by hand. OK.

The `DESCRIPTION` file contains package metadata, and we will typically change this as required.

## 1.1 Customising the package fundamentals

At the moment, the only thing we can change is `DESCRIPTION`, so let's go in and change some things.

Currently the file looks like

<pre><code>Package: newPackage
Title: What the Package Does (One Line, Title Case)
Version: 0.0.0.9000
Authors@R:
    person("First", "Last", , "first.last@example.com", role = c("aut", "cre"))
Description: What the package does (one paragraph).
License: `use_mit_license()`, `use_gpl3_license()` or friends to pick a
    license
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.3.2
</code></pre>

### 1.1.1 LICENSE

Let's start with the license, we can specify a file here, so let's download Apache 2.0 https://www.apache.org/licenses/LICENSE-2.0.txt and save this in the directory. Update the License to read `file LICENSE-2.0.txt`. Alternatively, use `usethis` to automatically generate your license, e.g. `usethis::use_apache_license()`, see section 4. below.

### 1.1.2 Version number

We can change the version number, let's call it `0.1`.

### 1.1.3 Package title and description

Change `Title` and `Description` accordingly.

## 1.2 Building and installing the package

At this stage (actually just after running `devtools::create`) we have a working package. Let's go ahead and build it.

Navigating to the parent directory (i.e. the directory containing `newPackage`), run the following commands on the command line interface:

`R CMD build newPackage`

This builds the package and creates the file `newPackage_0.1.tar.gz`

`R CMD INSTALL newPackage_0.1.tar.gz`

This installs the package into R.

Starting up R, the command `library(newPackage)` works! Although unfortunately it currently does nothing!

# 2 Creating first R functions

Let's create a function.

<pre><code>square <- function(x) x^2
</code></pre>

As mentioned above, this goes in the `R` directory as a file, which we shall call `square.R`.

## 2.1 Exporting a function to the namespace

As is, this function can't be used because it's not visible in the namespace. The package roxygen2 provides functionality for exporting (and documenting) functions. I think we don't need to use this if we just start editing `NAMESPACE` although, as mentioned above, this is not recommended.

Before the function declaration (i.e. in the file `square.R`, we put the following line:

<code>&#35;' @export</code>

This tells roxygen2 to export this function into the namespace. Starting up R, we call `setwd('newPackage')` and `devtools::document()`. This then makes the function discoverable.

After `library(newPackage)`, `newPackage::square(5)` actually does something!

### 2.1.2 Private functions

Note that the roxygen `@export` command was required. We could omit this for private functions to have a few things working behind the scenes. An example is having `square.R` look like this:

<pre><code>power <- function(x, a) x**a

&#35;' @export
square <- function(x) power(x, 2)
</code></pre>

Here `square` is available to the user but `power` is not.


## 2.2 Documenting functions

Using roxygen2 we can easily document functions as well. Include the following above the `square` declaration:

<pre><code>&#35;' Square
&#35;'
&#35;' Squares a number
&#35;'
&#35;' @param x The number to be squared
&#35;'
&#35;' @examples
&#35;' square(5) == 25
&#35;'
&#35;' @export
</code></pre>

After calling `devtools::document()` and rebuilding the package, this information comes up when calling `?square` or `help(square)` in R.

## 2.3 Package documentation

The roxygen specs page (see 2. in references) is outdated. Including the file `newPackage_package.R` in the `R` directory:

<pre><code>&#35;' @keywords internal
&#35;' @aliases newPackage-package NULL
"_PACKAGE"
</code></pre>
creates a help page for the package and populates it with information in the `DESCRIPTION` file (see section 1.1).

# 3. Setting up testing

Testing can be done with the `testthat` package. Running the following:

<pre><code>setwd('newPackage')
library(usethis)
usethis::use_testthat()
</code></pre>

sets up a few things in the package directory:

<pre><code>----newPackage

...

\---tests
    |   testthat.R
    |
    \---testthat
</code></pre>
We put testing scripts in the `testthat` directory; the file `testthat.R` is auto-generated and once again we don't need to/shouldn't change it.

We can use `usethis::use_test('asdf')` to create new test files or just do it ourselves:

Create the following file and save it as `testthat/test-pass.R`

<pre><code>test_that("basic pass test", {
  expect_true(TRUE)
})
</code></pre>

This should always pass.

## 3.1 Running unit tests

Within R we can use the command `devtools::test()` (while working directory is correct) to run the unit tests.

We can include additional tests in the `test-pass.R` file such as

<pre><code>test_that("addition works", {
  expect_equal(5+7,12)
})
</code></pre>

and this results in two succesful tests.

## 3.2 Data for unit tests

Best practice seems to be creating a 'inst/testdata' folder:

https://www.repeato.app/optimal-placement-of-test-data-for-automated-testing-with-testthat/

# 4. Package checks

While not exactly unit testing, R provides checks for package consistency with the command line command `R CMD check`. Some of the things `R CMD check` comes up with are not particularly useful (but most are). But, in particular, I appreciate the checks for object scoping and global variables, which often get missed (by me) when developing functions and tools in R.

The following can cause problems when setting up a new R package:

1. license (1 warning)
<pre><code>Non-standard license specification:
  file LICENSE-2.0.txt
</code></pre>
This is probably OK and we can revisit this if anything gets to the production stage. However, an alternative approach is deleting license information from the DESCRIPTION file and using `usethis::use_apache_license()` within R, see https://stackoverflow.com/a/73992145. This creates the license file in the directory and instructs build to ignore it.

2. building the pdf manual (1 warning and 1 error)
<pre><code>O checking PDF version of manual ... WARNING
LaTeX errors when creating PDF version.
This typically indicates Rd problems.
O checking PDF version of manual without index ... ERROR
</code></pre>
As far as I can tell the latex/pdf conversion functionality is missing here. I find the pdf manuals pretty annoying and prefer the html and/or in-R help functionality. As such we can turn this off by building without compliation of the pdf manuals:
<pre><code>R CMD build newPackage --no-manual
R CMD check --no-manual newPackage_0.1.tar.gz
</code></pre>
3. aliases (1 note)
The alias `pkgName-package` is automatically added to the package 'Rd' file. This can be turned off by having the command `@aliases newPackage-package NULL` rather than `@aliases newPackage-package` in the package description file (see section 2.3). Also:
https://github.com/r-lib/roxygen2/issues/1289

With these three issues either ignored or fixed, `R CMD check` works properly.


## 4.1 Incorporating unit testing in package checks

Unit testing using testthat is automatically run unit tests while checking the package, although detailed test results are not evident if the tests pass. This can be seen by including a new file `test_fail.R` in the `tests/testthat` path such as

<pre><code>test_that("basic fail test", {
  expect_true(FALSE)
})
</code></pre>

# 5. Summary

# 5.1 Final package architecture
<pre><code>----newPackage
|   .Rbuildignore
|   DESCRIPTION
|   LICENSE.md
|   NAMESPACE
|
+---man
|       newPackage-package.Rd
|       square.Rd
|
+---R
|       newPackage_package.R
|       square.R
|
+---inst
|   |
|   +--- testdata
|
\---tests
    |   testthat.R
    |
    \---testthat
            test-pass.R
</code></pre>

## 5.2 Batch file for building

Let's create a batch file to build the package:

Create the file `build.bat` in the `newPackage` path containing:

```
cd ..\
R CMD build newPackage --no-manual  
R CMD check --no-manual newPackage_0.1.tar.gz
R CMD INSTALL newPackage_0.1.tar.gz
```

Then `build.bat` from the command line will build, test and install the package as described above.

This could be improved by including a variable for the package version.

# 6. References

1. [R package tutorial](https://tinyheero.github.io/jekyll/update/2015/07/26/making-your-first-R-package.html)

2. [roxygen help](https://stuff.mit.edu/afs/athena/software/r/current/RStudio/resources/roxygen_help.html)

3. [Testing](https://r-pkgs.org/testing-basics.html)
