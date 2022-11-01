# POC: renv demo repo

> **Warning**
> This is part of a repository that is prone to  Changes and overhauls will happen without notice, but feel free to reach out with any questions/corrections/or help needs at lisamaeanders@gmail.com

**Access the slides at: <https://colorado.rstudio.com/rsc/reproduceable_workflows/>**

**Why use renv?**

There is an excellent video by David Aja discussing why he started using renv at the 2022 RStudio Conference [here](https://www.rstudio.com/conference/2022/talks/you-should-use-renv/).  

Ever had your code mysteriously stop working or start producing different results after upgrading packages, and had to spend hours debugging to find which package was the culprit? Ever tried to collaborate on code just to get stuck on trying to decipher various package dependencies? 

[renv](https://rstudio.github.io/renv/articles/renv.html) helps you track and control package changes - making it easy to revert back if you need to. It works with your current methods of installing packages (`install.packages()`), and was designed to work with most data science workflows. 

Who shouldn't use renv? 

 - Package developers 
 - ? 
 
# Terms 

 - R Project - a special kind of directory of files and supporting functionality. 
 - Package - a collection of functions beyond base R that developers can install and use.
 - Library - a directory containing installed packages. 

# Workflow 

New project -> updates -> reverting -> advanced 

# New project 

Initialize your project with: 

```
library(renv)
renv::init()
```

Look at the renv.lock file and see the information that has been captured about the packages supporting your project. 

# Making updates

Try installing a new package and then look at the renv.lock file. What did you expect to happen? What do you see? 

Now try running `renv::snapshot()`. What do you see now when you look at the renv.lock file? 

The renv lock file is updated by you when you run the command to snapshot. This means you can update packages, or install new packages, without changing your lock file. 

# How to revert 

Practice updating the packages your project relies on, each time saving to git. You can see the history of your changes with: 

`renv::history()`

If you want to revert back to an earlier snapshot you can do that with: 

```
## Revert to the most recent snapshot 
renv::revert()

## Alternatively we can revert to a specific snapshot by specifying the commit: 
db <- renv::history()

# choose an older commit
commit <- db$commit[5]

# revert to that version of the lockfile
renv::revert(commit = commit)
```

Note: Currently, only Git repositories are supported by renv::history() and renv::revert().

# Advanced renv 

There are a couple other commands that can be used depending on your workflow that are useful to know about. 

Say that your organization has a managed package host, for example Package Manager, and is using the [Shared Baseline](strategy). Meaning that all developers in the organization are pointed to a snapshot of available packages frozen to a particular date when the managing team had intentionally tested and made them available. On some cadence, let's say quarterly, the managing team goes through, performs testing again, and provides a new updated snapshot that is available for developers to switch to. There are a lot of advantages in switching with new features, resolved bugs, etc. 

In order for developers to know that a new release has happened (that the local package dates are out of date in reference to the repo being pointed at) they can run: 

    view(old.packages())

The same process would be followed with then updating the packages, testing as needed, and then creating a new snapshot so the renv lock file is updated to the latest. This provides the added security that in case it does not pass testing the renv lock file will still point at the supported package versions and not the ones that caused the breakage. 

One of the more recent releases of the renv package has included [having separate profiles for the project](https://rstudio.github.io/renv/articles/profiles.html). Meaning that a developer could have a production profile, a validation profile, and testing profile that can be easily switched between while working through their workflow towards deployment into production. 

The renv.lock file can be manually changed to update the packages that are included with:

    renv::modify()
    
Remove packages that are no longer needed with: 

`renv::clean()`

Update everything to the latest for each package (according to the repository you are pointed at) with: 

`renv::update()`

# Repositories 

Check your current repo with: `options('repos')`

For example, you might see: `https://packagemanager.rstudio.com/all/latest` or `https://cran.rstudio.com/`. 

Change your repo with: `options(repos = c(REPO_NAME = "https://colorado.rstudio.com/rspm/cran/__linux__/focal/2022-06-29"))` or `options(repos = c(REPO_NAME = "https://packagemanager.rstudio.com/all/latest"))`

# Changing topic: Environment Variables

When working with pulling data from secure databases or other sources a developer might find themselves in a situation of needing to provide very sensitive information, such as a password or a token, in order to access the data that is needed or to successfully deploy a project. Handling those secrets in way that doesn't expose them in the code directly is critical and where using [environmental variable's for securing sensitive variables](https://db.rstudio.com/best-practices/deployment/) is strongly recommended. 

Additionally there may be parameters that are often needed that can be accessed as a variable more easily rather than having to type in every time. 

For both of these cases knowing how environment variables can be leveraged can be very rewarding and it is surprising how little effort it can to take to set up. 

## Working with the .Renviron file

When R starts it loads a bunch of variables, settings, and configs for the user. This is really powerful and some of the magic for how it can work so apparently seamlessly. 

However for power users we can leverage these behind the scenes config files so that we can include such things as variables in our project without including it in our code. The .Renviron file is the one most commonly interacted with for adding sensitive variables to a project in order to protect them from being exposed in the code. 

With increased use of these behind the scenes config files and the growing direction of arranging code into projects there was the development of giving, on startup, having multiple options for each config file that can be loaded depending on what the user specifies. Broadly speaking there are two levels of config files: 
 - User 
 - Project 
 
On startup, since R is trying to make things as seamless as possible for the user, it will use some logic to figure out which config to use. It will assume that if a project level config exists it should load that one (and not any others). If that project level config doesn't exist, then it will default to the user level config. For more details on the different config files and the nuances see [Managing R with .Rprofile, .Renviron, Rprofile.site, Renviron.site, rsession.conf, and repos.conf](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf). 

Just to re-iterate the key takeaway: When in doubt note that the **project level file is given preference over user level config files**. Only if the project level config file doesn't exist will the user level file be sourced/pulled in. 

There is a really excellent [overview of R's startup process here](https://rviews.rstudio.com/2017/04/19/r-for-enterprise-understanding-r-s-startup/) but in short it can be thought of this way: 

#### Example with a user level .Renviron config file

usethis has a function for creating and editing the .Renviron file

    library(usethis)
    usethis::edit_r_environ()

Add the variables to that file in the format `variable_name = "variable_value"` and save it. Restart the session so the new environment variables will be loaded with ctrl shift f10 or through the RStudio IDE

Saved variables can be accessed with:

    variable_name <- Sys.getenv("variable_name")

When working in a more complex environment structure where separate project, site, and user environments are being support [this support article has useful information](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf) with a [deeper dive into R's startup here](https://rviews.rstudio.com/2017/04/19/r-for-enterprise-understanding-r-s-startup/).

#### Example with a project level .Renviron config file

Storing secrets securely can be done using the [edit_r_environ function from the usethis package](https://usethis.r-lib.org/reference/edit.html). For more overview see [this overview](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf). 

Example: 

```
library(usethis)
usethis::edit_r_environ(scope = "project")
```

Accessing those stored parameters later can be done using `Sys.getenv("DB_NAME")`. 

Be sure to add the project level .Renviron file to your .gitignore so you aren't exposing secrets when code is being saved to your git repository. Similarly this can be done with the `edit_git_ignore(scope = c("user", "project"))` function. For more best practices see [securing credentials](https://db.rstudio.com/best-practices/managing-credentials). 

 - While typically explicitly listing the file name is the desired addition, wildcards can be added to exclude a type of file. For example: `*.html`. 

After updating these files the project should be closed and re-opened for any additions to be pulled in. One way to do this is through session -> restart R (ctrl-shift-f10). 

#### Example with project level github secrets for environment variables

Another approach, particularly useful when automating testing and deployments using github actions, is to include the environment variables as secrets. Once this has been added through the git UI for the project they can then be referenced in the relevant deployment .yaml file with something like `CONNECT_ENV_SET_ZD_USER: ${{ secrets.ZD_USER }}`. In the R scripts they will be referenced as usual with something like `Sys.getenv("DB_NAME")`.

#### References for adding environment variables through the Connect UI 

Starting with version 1.6, RStudio Connect allows [Environment Variables](https://docs.rstudio.com/connect/admin/security-and-auditing/#application-environment-variables). The variables are encrypted on-disk, and in-memory.

This can be done at the project level with [securing deployment](https://db.rstudio.com/best-practices/deployment/) through the [Connect UI](https://support.rstudio.com/hc/en-us/articles/228272368-Managing-your-content-in-RStudio-Connect).

