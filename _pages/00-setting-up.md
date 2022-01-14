---
layout: page
title: Setting up R
---

The statistical programming language R is quickly becoming one of the standard tools for data journalists around the world.

In this walkthrough, you'll learn:
- How to install R and R Studio on your machine
- Basics about R vocabulary
- How to import data and generate simple summary stats
- Use keyboard shortcuts to save time

## Installing software

There's a variety of ways to work with R, which is the actual __language__. For most of the projects here, we'll be using RStudio, a popular integrated development environment (IDE) that essentially functions as a user interface for the R language itself.

### Step 1: Install R

Download and install the [latest version of R from CRAN](https://cran.rstudio.com/), the "Comprehensive R Archive Network," for your operating system:

https://cran.rstudio.com/

No need to change any of the settings in the default install wizard.

*NOTE: If you're on a Mac, and you get an error about not being able to open an application downloaded from the Internet, right-click (or control-click) on the file and click "Open" on the dialogue to override Apple's sometimes finicky security settings.*

### Step 2: Install RStudio

Download and install the latest version of RStudio (the website should automatically suggest the version for your operating system):

https://www.rstudio.com/products/rstudio/download/#download

*NOTE: If you're on a Mac, you may have to drag and drop the RStudio icon into your Applications folder to complete the install.*

### Step 3: Open RStudio

To make sure everything is installed correctly, navigate to wherever your applications are stored and launching RStudio.

You may see an icon for R – we won't actually be launching this application, since we're using the interface instead.

## A few R basics

When you open RStudio, for the first time, you'll see a few different panes in your workspace. The largest is your console, which you can use to directly enter commands using the R language. Some commands, like simple arithmetic, are pretty obvious!

![Simple addition in the RStudio console](/assets/screenshots/r000_simple_addition.gif)

For the most part though, we won't be working in the console directly. We'll be writing our R code in an *R script*, a separate file we can save, reload and share.

At the top menu, click `File > New File > R Script` to start a new script. You'll see your blank file appear in a new pane in RStudio. Go ahead and `Save As...` to give it a name. You should get in the habit of saving your work often.

![Starting a new script](/assets/screenshots/r000_studio_start.png)

At the top of your script, write a quick comment that tells you something about what your new script does. Starting each line with a `#` character will ensure this line is not executed when you run your code.

```R
    #R script for the Duke Data Journalism Lab
    #workshop on Feb. 19, 2022
```

To save us some headaches down the road, we want to tell RStudio where we want to do our work by setting our working directory. It's also good practice to comment your code as you go for readability.

```R
    #set working directory to the data folder of the downloaded repository
    setwd("~/Desktop/data-journalism-with-r/data")
```

Execute the code by clicking `Run` or with <kbd>Command</kbd> + <kbd>Enter</kbd>.

You can also store variables using an assignment function. The most basic variable types are *characters*, *numerics*, *dates* and *logicals*. There are a few more, that we'll get to later.

```R
current_year <- 2022

current_date <- as.Date('2022-01-10')

current_month <- 'January'

is_winter <- TRUE
```

R has a lot of great, basic functionality built in. But an entire community of R developers has created a long list of packages that give R a wealth of additional tricks.

We'll talk about some of those in another tutorial.

For a full (and continually updating) list of common R terms, [check out the glossary](/_pages/glossary.html).

## Handy keyboard shortcuts

A few common shortcuts in RStudio can save you from typing a lot of extra keystrokes.

__NOTE: We're using <kbd>Command</kbd> here for Macs, but if you're using a Windows machine, substitute <kbd>Control</kbd>.__

### Execute a command
<kbd>Command</kbd> + <kbd>Enter</kbd>

Run a section of code in your R script.

### Use an assignment
<kbd>Option</kbd> + <kbd>-</kbd>

Input a `<-` at your cursor to assign output to a variable.

### Comment multiple lines
<kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd>