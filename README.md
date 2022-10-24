# POC: renv demo repo

This is a work in progress. Please come back later :) 

### Variables are being saved to the user level .Renviron config file

For this project the following variables are being saved in the .Renviron file (credentials are stripped for security), as well as added to Connect as environment variables: 

 - CONNECT_API_KEY=**REDACTED**
 - CONNECT_SERVER=**REDACTED**
 - OWNER_GUID=**REDACTED**

[`usethis`](https://usethis.r-lib.org/) has a function for creating and editing the .Renviron file: 

```r
library(usethis)
usethis::edit_r_environ()
```

Add the variables to that file in the format `variable_name = "variable_value"` and save it. Restart the session so the new environment variables will be loaded with `ctrl shift f10` or through the RStudio IDE through the **Session** dropdown and selecting **Restart R**. 

Saved variables are accessed with:

```r
variable_name <- Sys.getenv("variable_name")
```

<details>
  <summary>Relevant reading:</summary>

When working in a more complex environment structure where separate project, site, and user environments are being used [this support article has useful information](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf) with a [deeper dive into R's startup here](https://rviews.rstudio.com/2017/04/19/r-for-enterprise-understanding-r-s-startup/).

</details>

### Package management is using renv

This project is using [renv](https://rstudio.github.io/renv/articles/collaborating.html). In order to set up this example and have a working example run: 

```
library(renv)
renv::restore()
```





