# R_GA_GSOC
For the R GSOC project [Github action for atime](https://github.com/rstats-gsoc/gsoc2023/wiki/Github%20action%20for%20atime)

The testing repo using this Github Action is [here](https://github.com/SiyaoIsHiding/quantmod_gsoc).
+ [Milestones](#milestones)
  + [Easy \& Medium Task](#easy--medium-task)
  + [Hard Task](#hard-task)
  + [Current Progress](#current-progress)
  + [To-Do List](#to-do-list)
+ [Questions To Ask](#questions-to-ask)
  + [1. Where should I install the binsegRcpp?](#1-where-should-i-install-the-binsegrcpp)
  + [2. Packaged with `r-lib/actions/setup-r` and `r-lib/actions/setup-r-dependencies` in it or not?](#2-packaged-with-r-libactionssetup-r-and-r-libactionssetup-r-dependencies-in-it-or-not)
  + [3. Using `actions/check-out` or `git2r`?](#3-using-actionscheck-out-or-git2r)
  + [4. How to test?](#4-how-to-test)
  + [5. Sorry that I failed to find `atime_pkg()`...](#5-sorry-that-i-failed-to-find-atime_pkg)

## Milestones
### Easy & Medium Task
I modified the R Github Action [check-r-package](https://github.com/r-lib/actions/tree/v2/check-r-package) so that it will log out "No testthat output available" if there is no testthat output.

The original code snippet:
```bash
find check -name 'testthat.Rout*' -exec cat '{}' \; || true
```

My modified one:
```bash
TESTTHAT_OUTPUT=$(find check -name 'testthat.Rout*' -exec cat '{}' \; )
if [ -z "$TESTTHAT_OUTPUT" ]; then
    echo  "No testthat output available."
else
    echo $TESTTHAT_OUTPUT
fi
```

The [commit of this repo](https://github.com/SiyaoIsHiding/R_GA_GSOC/tree/af91fff737e65a228cbf91758fcf3a4718fc39b8) in the medium task. The [commit of the testing repo](https://github.com/SiyaoIsHiding/quantmod_gsoc/tree/d3d46658693df688965ec160430ae1e65dc5f167) using this action in the medium task.

The screenshot with the `No testthat output available` at the end.
![Medium Task Screenshot](./images/Medium.png)
### Hard Task
![Hard Task Screenshot](./images/Hard.png)
Using the example code in the atime package documentation to test the performance of `binsegRcpp::binseg_normal`

### Current Progress
Hardcoded the `atime_versions()` in to GitHub Action shell script and ensured the repository using it can run it without error.
The hardcoded script looks like this:
```yaml
    - name: Check
      id: rcmdcheck    
      env:
        _R_CHECK_CRAN_INCOMING_: false
      run: |
        ## --------------------------------------------------------------------
        options(crayon.enabled = TRUE)
        cat("LOGNAME=", Sys.info()[["user"]], "\n", sep = "", file = Sys.getenv("GITHUB_ENV"), append = TRUE)
        if (Sys.getenv("_R_CHECK_FORCE_SUGGESTS_", "") == "") Sys.setenv("_R_CHECK_FORCE_SUGGESTS_" = "false")
        cat("check-dir-path=", file.path(getwd(), (${{ inputs.check-dir }})), "\n", file = Sys.getenv("GITHUB_OUTPUT"), sep = "", append = TRUE)
        check_results <- rcmdcheck::rcmdcheck(args = (${{ inputs.args }}), build_args = (${{ inputs.build_args }}), error_on = (${{ inputs.error-on }}), check_dir = (${{ inputs.check-dir }}))
      
        tdir <- tempfile()
        dir.create(tdir)
        git2r::clone("https://github.com/tdhock/binsegRcpp", tdir)
        atime.list <- atime::atime_versions(
          pkg.path=tdir,
          N=2^seq(2, 20),
          setup={
            max.segs <- as.integer(N/2)
            data.vec <- 1:N
          },
          expr=binsegRcpp::binseg_normal(data.vec, max.segs),
          cv="908b77c411bc7f4fcbcf53759245e738ae724c3e",
          "another name"="dcd0808f52b0b9858352106cc7852e36d7f5b15d",
          "another"="5942af606641428315b0e63c7da331c4cd44c091")
        refs.best <- atime::references_best(atime.list)
        plot(refs.best)
        atime::atime_versions_remove("binsegRcpp")
        
      shell: Rscript {0}
      working-directory: ${{ inputs.working-directory }}
```

### To-Do List
1. Migrate from `atime_versions` to `atime_pkg` if I finally find it.
2. Access the source code and the recent commit IDs through environment variables.
3. Upload the plots by `actions/upload-artifact` and verify they are correct.
4. Leave messages in PR.
5. Implement the functionality of comparing the performance between commits.

## Questions To Ask

### 1. Where should I install the binsegRcpp?

During my local testing, I found that I had to install `binsegRcpp` beforehand by running `install.packages("binsegRcpp")`, otherwise the script cannot compile, raising `Error in loadNamespace(x) : there is no package called ‘binsegRcpp’`. Therefore, in my current hardcoded Github Action, I have to install `binsegRcpp` during the `setup-r-dependencies` beforehand as well. However, I am not sure if this is the proper approach. May you provide some guidance on this?

### 2. Packaged with `r-lib/actions/setup-r` and `r-lib/actions/setup-r-dependencies` in it or not?

If the setup actions are not packaged in it, the user has to specify `atime` in their `r-lib/actions/setup-r-dependencies`. If packaged, there will be less flexibility for the user. Which one you think may be better?

### 3. Using `actions/check-out` or `git2r`?

Both of them seam feasible. Using `actions/check-out` will enable private repositories to use the Github Action but will require more configuration tasks. Which one do you suggest we use?

### 4. How to test?

I have noticed that the plots can be significantly different even on the same commits. Therefore, I am unsure about how to test my code thoroughly. One approach I am considering is to intentionally slow down the performance by adding code to sleep for a while and then comparing the changes in performance. However, this is not foolproof. Could you provide some suggestions on the testing oracle?

### 5. Sorry that I failed to find `atime_pkg()`...

May I ask where I can find it?

