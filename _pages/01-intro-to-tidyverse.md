---
layout: page
title: Intro to R and the Tidyverse
---

This workshop will help you get started with a few foundational steps for importing, cleaning and analyzing data using R, the desktop development environment RStudio and several common libraries for working with data.

In this repository, you'll find both the data we'll be working with and the full R script for our in-class exercises for your review.

## Getting started
After starting RStudio, click "File" > "New File" > "R Script" in the menu to start a new script. Then go ahead and "Save As..." to give it a name. You should get in the habit of saving your work often.

At the top of your script, write a quick comment that tells you something about what your new script does. Starting each line with a `#` character will ensure this line is not executed when you run your code.

```R
    #R script for the Duke Data Journalism Lab
    #workshop on Jan. 21
```

To save us some headaches down the road, we want to tell RStudio where we want to do our work by setting our working directory. It's also good practice to comment your code as you go for readability.

```R
    #set working directory to the data folder of the downloaded repository
    setwd("~/Desktop/data-journalism-with-r/data")
```

Execute the code by clicking "Run" or with **CMD + Enter**.

R has a lot of great, basic functionality built in. But an entire community of R developers has created a long list of packages that give R a wealth of additional tricks. One of the most popular is the [Tidyverse](https://www.tidyverse.org/), a collection of packages designed for data science. Let's install a few of those packages. **This step we'll only have to do once.**

```R
    #install the tidyverse package
    install.packages("tidyverse")
    install.packages("readxl")
    install.packages("knitr")
    install.packages("janitor")
```

Then load them from our library. **This step we'll have to do each time we start R or start a new workspace.**

```R
    #load our packages from our library into our workspace
    library(tidyverse)
    library(readxl)
    library(knitr)
    library(janitor)
```

## Downloading the data

The data we'll be working with today contains information about the recipients of loans through the federal Small Business Administration's [Paycheck Protection Program](https://data.sba.gov/dataset/ppp-foia).

Start by downloading the `public_150k_plus_220102.csv` file.

![Simple addition in the RStudio console]({{ site.baseurl }}/assets/screenshots/r001_ppp_foia.png)

You'll also want to download the PPP data dictionary at the bottom of the list.

Save the files to your working directory.

Load it in with a function from our Tidyverse package.

```R
    #load in our fresh data
    ppp_150k <- read_csv('public_150k_plus_220102.csv')
```

Take note that in your "Environment" window (by default in the top right), you should be able to see your ppp dataframe with 968,538 rows and 53 variables.

## Basic gut checks

Before we start working with our data in earnest, let's get to know our data a little bit. You can click on the dataset in your environment window to view it in a new window, much like you would in Excel or some other spreadsheet software.

We can see a number of columns that are pretty self explanatory. Some less so. We'll need to explore those as we go. Navigate back to your script window, and let's do a few things to make sure our head is on straight. For this section, we'll be using the "pipe," which looks like this `%>%` to chain together operations on our data.

If you want to save some time typing, you can use the keyboard shortcut <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>M</kbd> to add a pipe.

First, let's check to make sure we got it all through the load.

```R
#how many rows do we have?
ppp_150k %>% 
  nrow()
  ```

How many states' worth of data are we looking at? And let's check that with more than one variable.

```R
#are we only looking at NC?
ppp_150k %>% 
  count(BorrowerState)
  
ppp_150k %>% 
  count(ProjectState)
  ```

Check how much money we're talking about here?

```R
#what's our total loan value?
ppp_150k %>% 
  summarize(total = sum(InitialApprovalAmount))
  ```

Let's see how much are borrowers getting on average vs. the typical/middle value

```R
#what about the average?
ppp_150k %>% 
  summarize(avg = mean(InitialApprovalAmount))
  ```

```R
#what's the median?
ppp_150k %>% 
  summarize(total = median(InitialApprovalAmount))
```

We're going to be working with some large numbers here, so let's disable scientific notation so we can read the output a little better.

```R
#disable scientific notation
options(scipen=999)
```

Now let's look at how got the largest and smallest loans, and simplify our output a bit to make it a little more readable with a function from the `knitr` package.

```R
#who got the largest loan?
ppp_150k %>% 
  arrange(desc(InitialApprovalAmount)) %>%
  select(BorrowerName, BorrowerCity, InitialApprovalAmount) %>% 
  head() %>% 
  kable('simple')
  ```

Conversely, who got the smallest loan amount. That might be a little weird!

```R
#who got the smallest loan?
ppp_150k %>% 
  arrange(InitialApprovalAmount) %>%
  select(BorrowerName, BorrowerCity, InitialApprovalAmount) %>% 
  head() %>% 
  kable('simple')
  ```

Although we can see them in the big table view, it might be helpful to get a list of all of our fields in one place so we can see what's there – and what we might need to ask about during our reporting.

```R
#give me a rundown of the column names
ppp_150k %>% 
  names()
  ```

## Cleaning and grouping

Now that we have a good idea of what's here, let's see if we can answer some basic questions about the geographic distribution of these loans. Let's start with the county (NC only), which looks pretty clean

```R
#group by county
ppp_150k %>%
  filter(ProjectState == 'NC') %>% 
  count(ProjectCountyName, name = 'loan_count') %>% 
  arrange(desc(loan_count)) %>% 
  kable('simple')
  ```

But the city is a little different. Notice how often names are misspelled and inconsistent.

```R
#group by city
ppp_150k %>% 
filter(ProjectState == 'NC') %>% 
  count(ProjectCity, name = 'loan_count') %>% 
  arrange(desc(loan_count)) %>% 
  kable('simple')
  ```

One thing we can do to fix that is to create a "clean" column, where we generate a little code to fix our wonky data. Here, we are essentially overwriting our old data.

```R
ppp_150k_clean <- ppp_150k %>% 
  mutate(ProjectCity_clean = toupper(ProjectCity)) %>% 
  mutate(ProjectCity_clean = str_remove_all(ProjectCity_clean, '\\.')) %>% 
  relocate(ProjectCity_clean, .after = ProjectCity)
  ```
 
There are other ways to clean data that we'll get to in future lessons. But for now, we might make a decision that this is a bit too dirty to work with for the time being.

## Joining other data

Among the fields in our data is a column called `NAICSCode`, which is the code assigned by the North American Industry Classification System. It basically describes what type of business the company is in. The codes are standardized, meaning we can use them to find out how funds are distributed across industry types, but we'll need to join them up to do that.

First, [download our NAICS lookup table here]({{ site.baseurl }}/assets/data/naics_lookup.zip). Then unzip the file and move it to your working directory.

```R
# load in our naics lookup table
naics <- read_csv('naics_lookup.csv')
  ```

The data set we'll be joining is a lookup table using the first two digits of the NAICS code, which describes the top-level industry type.

First, let's make sure we have a column to match on.

```R
ppp_150k_clean_naics <- ppp_150k_clean %>%
  mutate(NAICS_initial = strtoi(substr(NAICSCode, start = 1, stop = 2)))
  ```

Then, let's join and group our industry codes accordingly. We can also use a nifty function from the `janitor` package to calculate totals and percentages.

```R
ppp_150k_clean_naics %>%
  left_join(naics, by = c('naics_initial_code')) %>% 
  group_by(naics_title) %>% 
  summarize(total = sum(InitialApprovalAmount)) %>% 
  arrange(desc(total)) %>% 
  adorn_totals() %>% 
  adorn_percentages('col') %>% 
  adorn_pct_formatting() %>%
  adorn_ns(position = "front") %>% 
  kable('simple')
  ```

## Additional resources
* [RStudio cheatsheets](https://rstudio.com/resources/cheatsheets/)