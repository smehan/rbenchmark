`rbenchmark` is a simple routine for benchmarking code written in [R](http://www.r-project.org/), a free software environment for statistical computing and graphics.

**Table of contents**



# Summary #
`rbenchmark` is inspired by the Perl module [Benchmark](http://search.cpan.org/~tty/kurila-1.14_0/lib/Benchmark.pm), and is intended to facilitate benchmarking of arbitrary R code.

The library consists of just one function, `benchmark`, which is a simple wrapper around `system.time`.

Given a specification of the benchmarking process (counts of replications, evaluation environment) and an arbitrary number of expressions, `benchmark` evaluates each of the expressions in the specified environment, replicating the evaluation as many times as specified, and returning the results conveniently wrapped into a data frame.

# Installation #
`rbenchmark` can be installed directly from [CRAN](http://cran.r-project.org/) within an R session:

```
# installing rbenchmark directly from CRAN within an R session
install.packages(rbenchmark)
```

It can also be installed as an R package complete with documentation by [downloading](http://cran.r-project.org/web/packages/rbenchmark/index.html) the `tar.gz` package file (see the downloads section) and executing, at the command shell, the command

```
# installing rbenchmark from a downloaded archive within a shell session
R CMD INSTALL rbenchmark_xx.tar.gz
```

where `xx` must be replaced with the appropriate version number, as in the downloaded archive.

`rbenchmark` can also be loaded by sourcing, in an active R session, the `benchmark.r` source file directly from Google Code:

```
# loading rbenchmark directly from googlecode, does not permanently install the package
source('http://rbenchmark.googlecode.com/svn/trunk/benchmark.r')
```

Consult the appropriate R documentation for information on installation of packages, if needed.


# Specifications #

## Signature ##

`benchmark` has the following signature:

```
benchmark(..., columns, order, replications, environment, relative)
```

## Parameters ##

`benchmark` has the following parameters:

  * `...` captures any number of **unevaluated expressions** passed to `benchmark` as named or unnamed arguments.

  * `columns` is a **character** or **integer vector** specifying which columns should be included in the returned data frame.

  * `order` is a **character** or **integer vector** specifying which columns should be used to sort the data frame.  Any of the columns that can be specified for `columns` (see above) can be used, even if it is not included in `columns` and will not appear in the output data frame.

  * `replications` is a **numeric vector** specifying how many times an expression should be evaluated when the runtime is measured.  If `replications` consists of more than one value, each expression will be benchmarked multiple times, once for each value in `replications`.

  * `environment` is the **environment** in which the expressions will be evaluated.

  * `relative' is the name or index of the timing column used to calculate relative timings, with the lowest value in the specified column taken as the reference.

The parameters `columns`, `order`, `replications`, `environment`, and `relative` are **optional** and have the following **default values**:

  * `columns = c('test', 'replications', 'user.self', 'sys.self', 'elapsed', 'user.child', 'sys.child', 'relative')`
> By default, the returned data frame will contain all columns generated internally in `benchmark`.  These named columns will contain the following data:
    * `test`: a character string naming each individual benchmark.  If the corresponding expression was passed to `benchmark` in a _named_ argument, the name will be used; otherwise, the expression itself converted to a character string will be used.
    * `replications`: a numeric vector specifying the number of replications used within each individual benchmark.
    * `user.self`, `sys.self`, `elapsed`, `user.child`, and `sys.child` are columns containing values reported by `system.time`;  see [Sec. 7.1 Operating system access](http://cran.r-project.org/doc/manuals/R-lang.html#Operating-system-access) in [The R language definition](http://cran.r-project.org/doc/manuals/R-lang.html), or type `?system.time` in an R session.
    * `relative': a column containing benchmark values relative to the shortest benchmark value.  The benchmark values used in this computation are taken from the column specified with the \code{relative} argument.

  * `order = 'test'`
> By default, the data frame is sorted by the column `test` (the labels of the expressions or the expressions themselves; see above).

  * `replications = 100`
> By default, each expression will be benchmarked once, and will be evaluated 100 times within the benchmark.

  * `environment = parent.frame()`
> By default, all expressions will be evaluated in the environment in which the call to `benchmark` is made.

  * `relative = 'elapsed'`
> By default, relative timings are given based on values from the column 'elapsed'.


## Value ##

The value returned from a call to `benchmark` is a **data frame** with rows corresponding to individual benchmarks, and columns as specified above.

An individual benchmark corresponds to a unique combination (see below) of an expression from `...` and a replication count from `replications`;  if there are _n_ expressions in `...` and _m_ replication counts in `replication`, the returned data frame will consist of _n`*`m_ rows, each corresponding to an individual, independent (see below) benchmark.

If either `...` or `replications` contain duplicates, the returned data frame will contain _multiple_ benchmarks for the involved expression-replication combinations.
Note that such multiple benchmarks for a particular expression-replication pair will, in general, have different timing results, since they will be evaluated independently (_unless_ the expressions perform side effects that can influence each other's performance).


# Examples #

To see how `rbenchmark` works, you can copy-paste the examples, or `source` a demo file that will do this for you:

```
# loading benchmark examples directly from googlecode
source('http://rbenchmark.googlecode.com/svn/trunk/demo.r')
```

If you have installed `rbenchmark` as a package, you can run the demos by executing, in an R session, the commands

```
library(rbenchmark)
example(rbenchmark)
example(benchmark)
```

## Example 1 ##
Benchmarking the allocation of one 10^6-element numeric vector, by default replicated 100 times.  All parameters have default values.

```
benchmark(1:10^6)
```

Possible output:

```
    test replications user.self sys.self elapsed user.child sys.child
1 1:10^6          100       0.1     0.28   0.383          0         0
```

## Example 2 ##

The following functions will be used in Examples 2-4.

```
random.array = function(rows, cols, dist=rnorm) 
                  array(dist(rows*cols), c(rows, cols))
random.replicate = function(rows, cols, dist=rnorm)
                      replicate(cols, dist(rows))
```

Benchmarking an expression multiple times with the same replication count, output with selected columns only.

```
benchmark(replications=rep(100, 3),
          random.array(100, 100),
          random.array(100, 100),
          columns=c('test', 'elapsed', 'replications'))
```

Possible output:

```
                    test elapsed replications
1 random.array(100, 100)   0.130          100
2 random.array(100, 100)   0.126          100
3 random.array(100, 100)   0.126          100
4 random.array(100, 100)   0.126          100
5 random.array(100, 100)   0.126          100
6 random.array(100, 100)   0.126          100
```

## Example 3 ##

Benchmarking two named expressions with three different replication counts, output sorted by test name and replication count, with additional column added after the benchmark.

```
within(benchmark(rep=random.replicate(100, 100),
                 arr=random.array(100, 100),
                 replications=10^(1:3),
                 columns=c('test', 'replications', 'elapsed'),
                 order=c('test', 'replications')),
       { average = elapsed/replications })
```

Possible output:

```
  test replications elapsed  average
2  arr           10   0.013 0.001300
4  arr          100   0.126 0.001260
6  arr         1000   1.259 0.001259
1  rep           10   0.017 0.001700
3  rep          100   0.170 0.001700
5  rep         1000   1.703 0.001703
```

## Example 4 ##

Benchmarking a list of arbitrary predefined expressions.

```
tests = list(rep=expression(random.replicate(100, 100)), 
             arr=expression(random.array(100, 100)))
do.call(benchmark,
        c(tests, list(replications=100,
                      columns=c('test', 'elapsed', 'replications'),
                      order='elapsed')))
```

> Possible output:

```
  test elapsed replications
2  arr   0.128          100
1  rep   0.169          100
```



# Notes #

Not all expressions, if passed as unnamed arguments, will be cast to character strings as you might expect:

```
benchmark({x = 5; 1:x^x})
```

will output (modulo actual timings):

```
  test replications user.self sys.self elapsed user.child sys.child
1    {          100         0        0   0.002          0         0
```

`benchmark` performs no smart argument-parameter matching.
Any named argument whose name is not exactly '`replications`', '`environment`', '`columns`', '`order`', or '`relative`' will be treated as an expression to be benchmarked:

```
benchmark(1:10^5, repl=1000)
```

will output (modulo actual timings):

```
    test replications user.self sys.self elapsed user.child sys.child
1 1:10^5          100     0.032    0.012   0.047          0         0
2   repl          100     0.000    0.000   0.000          0         0
```


# Author #

Wacek Kusnierczyk, [waku@idi.ntnu.no](mailto:waku@idi.ntnu.no)

# Contributors #

Dirk Eddelbuettel, [edd@debian.org](mailto:edd@debian.org)

Berend Hasselman, [bhh@xs4all.nl](mailto:bhh@xs4all.nl)