# R Guide

This guide is adapted from the excellent [R guide by Sean Higgins](https://github.com/skhiggins/R_guide). 

It provides instructions for using R on research projects. Its purpose is to to make code consistent, easier to read, and transparent within the CAUSALab @ KI.

## Folder structure

You will have a personal folder within `W:/C6_Berglund`.

### Project folder

Create a project folder for each project (with a sensible name) within your personal folder. This will contain;
* An .Rproj file for the project. (This can be created in RStudio, with File > New Project.)
  * Note that if you always open the Project within RStudio before working (see "Project" in the upper right-hand corner of RStudio) then the `here` package will work for relative filepaths
  * Use the same name as the project folder
* scripts folder - code goes in this folder
* output folder - .txt files of tables and .png files of figures go in this folder

### Data

All data (both raw and processed) are stored on the VDI within the data folder (`W:/C6_Berglund/data`). 
* The `rawdata` folder contains data as it was structured when received
* The `procdata` folder contains project specific data that has been manipulated from the raw data
  * The name of each subfolder in `procdata` must match the name of a project folder in you personal folder

## Using R packages on the VDI

As the VDI is not connected to the internet for security purposes, R cannot directly access the CRAN to install packages. IT has, therefore, set up a system that allows us to access the CRAN as normal, with a few additional steps. 
* In your personal folder, create a folder called `r-library` and an R script called `r-profile.R`.
* In the `r-profile.R` script include the following two lines of code:

  ```r
  options(repos = "HTTP://nexus.ki.se/repository/cran.r-project.org")
  .libPaths("W:/C6_Berglund/YOUR_PERSONAL_FOLDER/r-library")
  ```

* Every time you start a new R session, run the `r-profile.R` script.
* You can then access the CRAN and install packages as normal using the `install.packages()` function. All installed packages will be stored in your `r-library` folder, and R will load packages from this folder when you use the `library()` function. 


## Script structure

### Separating scripts
Because we often work with large data sets and efficiency is important, I advocate separating the following four actions into different scripts:  

1. Data preparation (cleaning and wrangling)
2. Functions
3. Analysis (regressions etc.)
4. Production of figures and tables
    
The analysis and figure/table scripts should not change the data sets at all (no pivoting from wide to long or adding new variables); all changes to the data should be made in the data cleaning scripts. The figure/table scripts should not run the regressions or perform other analysis; that should be done in the analysis scripts. This way, if you need to add a robustness check, you don't necessarily have to rerun all the data cleaning code (unless the robustness check requires defining a new variable). If you need to make a formatting change to a figure, you don't have to rerun all the analysis code (which can take a while to run on large data sets).

### Naming scripts
* Number scripts in the order in which they should be run: 00, 01, 02 etc.
* The first script should always be a 00_master.R script (described below).
* All other scripts should have the same name as the output they create.
* Data preperation scripts should contain "cr_".
* Scripts that write functions should contain "func_".
* Analysis scripts should contain "an_".
* Figure and table scripts should contain fig_/tab_

#### Examples
* Data preperation ("cr_")
	* 01_cr_eligible.R is the first script to be run.
	* It wrangles data and creates a data frame of individuals eligible for the study.
	* The name of the data frame the script creates is cr_eligible.
* Functions ("func_")
	* 02_func_ipw.R is the second script to be run.
	* It creates inverse probability weights.
	* The name of the function the script creates is func_ipw.
* Analysis ("an_")
	* 03_an_ittanalysis is the third script to be run.
	* It includes all code to run an intention-to-treat analysis.
	* The name of the output (data frame, array, etc.) the script creates is an_ittanalysis.
* Figures and tables ("fig_/tab_")
	*  04_tab_ittanalysis_1y is the fourth script to be run.
	*  It includes all code to create a table for the 1 yr results from the intention-to-treat analysis. 
	*  The name of the .txt file (stored in the output folder) the script creates is tab_ittanalysis_1y 

### 00_master.R script 
Keep a "master" script, 00_master.R that lists each script in the order they should be run to go from raw data to final results. Under the name of each script should be a brief description of the purpose of the script, as well all the input data sets and output data sets that it uses. Ideally, a user could run `00_master.R` to run the entire analysis from raw data to final results.
* Also include objects that can be set to 0 or 1 to only run some of the scripts from the 00_master.R script (see the example below).

Below is a brief example of a 00_master.R script. 
  
  ```r
  # Run script for example project
  
  # PACKAGES ------------------------------------------------------------------
  library(here)

  # PRELIMINARIES -------------------------------------------------------------
  # Control which scripts run
  run_01_cr_eligible <- 1
  run_02_func_ipw     <- 1
  run_03_an_ittanalysis   <- 1
  run_04_tab_ittanalysis_1y    <- 1
  
  # RUN SCRIPTS ---------------------------------------------------------------
  
  # Read data and create eligible patients
  if (run_01_cr_eligible) source(here("scripts", "01_cr_eligible.R"), encoding = "UTF-8")
  # INPUTS
  #  here("data", "example.csv") # raw data from XYZ source
  # OUTPUTS
  #  cr_eligible data frame
  
  # Create a function that estimates IP weights
  if (run_02_func_ipw) source(here("scripts", "02_func_ipw.R"), encoding = "UTF-8")
  # INPUTS
  #  none
  # OUTPUTS 
  #  func_ipw function is created
  
  # Run ITT analysis
  if (run_03_an_ittanalysis) source(here("scripts", "03_an_ittanalysis.R"), encoding = "UTF-8")
  # INPUTS 
  #  cr_eligible dataframe & func_ipw
  # OUTPUTS
  #  an_ittanalysis dataframe
  
  # Create table for 1 yr ITT results
  if (run_04_tab_ittanalysis_1y) source(here("scripts", "04_tab_ittanalysis_1y.R"), encoding = "UTF-8")
  # INPUTS
  #  an_ittanalysis dataframe
  # OUTPUTS
  #  here("output", "tab_ittanalysis_1y.txt") 
  ```

## Style

For coding style practices, follow the [tidyverse style guide](https://style.tidyverse.org/).

* While you should read the style guide and do your best to follow it, once you save the script you can use `styler::style_file()` to fix its formatting and ensure it adheres to the [tidyverse style guide](https://style.tidyverse.org/).
  * Note: `styler::style_file()` overwrites files (if styling results in a change of the code to be formatted). The documentation strongly suggests to only style files that are under version control or to create a backup copy.
  
## Packages

* Use `tidyverse` and/or `data.table` for wrangling data. 
  * For big data (millions of observations), the efficiency advantages of `data.table` become important. 
  * The efficiency advantages of `data.table` can be important even with smaller data sets for tasks like `rbind`ing, reshaping etc.
* Use `stringr` for manipulating strings.
* Use `lubridate` for working with dates.
* Use `conflicted` to explicitly resolve namespace conflicts.
  * `conflicted::conflict_scout()` displays namespace conflicts
  * `conflicted::conflict_prefer()` declares the package to use in namespace conflicts, and the `conflict_prefer()` calls should be a block of code towards the top of the script, underneath the block of `library()` calls.
* Never use `setwd()` or absolute file paths. Instead, use relative file paths with the `here` package.
  * To avoid conflicts with the deprecated `lubridate::here()`, if using both packages in a script, specify `conflict_prefer("here", "here")`.
* Use `assertthat::assert_that()` frequently to add programmatic sanity checks in the code.
* Use pipes like `%>%` from `magrittr`. See [here](https://r4ds.had.co.nz/pipes.html) for more on using pipes. Other useful pipes are the compound assignment pipe `%<>%`, and the `%$%` exposition pipe.
* Use `tabulator` for some common data wrangling tasks. 
  * `tabulator::tab()` efficiently tabulates based on a categorical variable, sorts from most common to least common, and displays the proportion of observations with each value, as well as the cumulative proportion.
  * `tabulator::tabcount()` counts the unique number of categories of a categorical variable or formed by a combination of categorical variables.
  * `tabulator::quantiles()` produces quantiles of a variable. It is a wrapper for base R `quantile()` but is easier to use, especially within `data.table`s or `tibble`s.
* Use `modelsummary` for formatting tables. 
* `Hmisc::describe()` and `skimr::skim()` can be useful to print a "codebook" of the data, i.e. some summary stats about each variable in a data set. Since they do not provide identical information, it might be best to run both.
  * This can be used in conjunction with `sink()` to print the codebook to a text file. F
  * or example:
  ```r 
  library(tidyverse)
  library(Hmisc)
  library(skimr)
  library(here)
  
  # Write codebook to text file
  sink(here("results", "mtcars_codebook.txt"))
  mtcars %>% describe() %>% print() # print() needed if running script from command line
  mtcars %>% skim() %>% print()
  sink() # close the sink
  ```

## Reproducibility 

Use `renv` to manage the packages used in an RStudio project, avoiding conflicts related to package versioning.
* `renv::init()` will develop a "local library" of the packages employed in a project. It will create the following files and folders in the project directory: `renv.lock`, `.Rprofile`, and `renv/`. Binaries of the project's packages will be stored in the `renv/library/` subfolder.
* When working on the project, use `renv::snapshot()` to update your `renv`-related files. Make sure these are updated when pushing project changes to GitHub, sharing files with others, or preparing the replication package.
* When deploying the project on a different machine, make sure that the `renv.lock`, `.Rprofile`, and `renv/` files/folders are present. The `renv/library/` subfolder, containing system-specific package binaries, should be excluded. Once these requirements are met, you can launch the project and run `renv::restore()` to restore all packages in the new machine, in their appropriate versions.

## Version control

### GitHub
Github is an important tool to maintain version control and for reproducibility purposes. There are many tutorials online, like Grant Mcdermott's slides [here](https://raw.githack.com/uo-ec607/lectures/master/02-git/02-Git.html#9), and I will share some tips from these notes. I will provide instructions for only the most basic commands here. 

We need to first create a git repository or clone an existing one.  

* To clone an existing github repository, use `git pull repolink` where repolink is the link copied from the repository on Github. 
* To initialize a new repo, use `git init` in the project directory

Once you you have initialized a git repository and you make some local changes, you will want to update the Github repository, so that other collaborators can have access to updated files. To do so, we will use the following process:   

* Check the status: `git status`. I like to use this frequently, in order to see file you've changed and those you still need to add or commit.
* Add files for staging: `git add <filename>`. Use this to add local changes to the staging area. 
* Commit: `git commit`. This command saves your changes to the local repository. It will prompt you to add a commit message, which can be more concisley written as `git commit -m "Helpful message"`
* Push changes to github: assuming there is a Github repository created, use `git push origin master` to push your saved local changes to the Github repo. 

However, there are often times when we encounter merge conflicts. A merge conflict is an event that occurs when Git is unable to automatically resolve differences in code between two commits. For example, if a collaborator edits a piece of code, stages, commits and pushes a change, and you try to push changes for the same piece of code, you will encounter a merge conflict. We need to figure out how fix these conflicts.  

* I like to start with `git status` which shows the conflicted files.
* If you open up the conflicted files with any text editor, you will see a couple of things. 
  * `<<<<<<< HEAD` shows the start of the merge conflict.
  * `=======` shows the break point used for comparison.
  * `>>>>>>> <long string>` shows the end of merge conflict.
* You can now manually edit the code and delete whatever lines of code you don't want and the special chanracters that Git added in the file. After that you can stage, commit and push your files without conflict. 

## Misc.

Some additional tips:

* Error handling: use `purrr::possibly()` and `purrr::safely()` rather than base R `tryCatch()`
* Progress bars: for intensive `purrr::map*()` tasks you can easily add progress bars with `dplyr::progress_estimated()` ([instructions](https://adisarid.github.io/post/2019-01-24-purrrying-progress-bars/))
* If you need to log some printed output, a quick way is `sink()`. 
