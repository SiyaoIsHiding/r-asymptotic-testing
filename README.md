# GitHub Action r-asymptotic-testing
Perform asymptotic testing on your R package upon every push/pull requests!

This GitHub Action will run the tests in `pkg/inst/atime/tests.R` by `atime::atime_pkg`, on different versions of the package (HEAD vs base, or HEAD vs CRAN), apply t test on the test results, upload plots, and leave messages in the PR if significant performance regression or improvements are detected.