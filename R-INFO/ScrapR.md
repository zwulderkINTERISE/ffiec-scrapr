---
layout: page
title: FFIEC ScrapR
permalink: /ffiec-scrapr-guide/
---

# FFIEC Address Scraper

## Overview
Purpose
    : The goal of this document is to make it as easy as possible to automate the process of searching addresses on the FFIEC website in order to retrieve census tract income data.

Future
    : The ultimate goal is to develop a more sophisticated process for gathering this data that will not rely on the FFIEC geocoding search, after which this scraping process will be deprecated. The current process is meant to be an intermediary step between manually searching every address and the eventual sophisticated process. The final form will essentially take each address and convert it to a latitude-longitude point. Census tracts will then be overlayed on these points to determine the census tract that corresponds to each point. Income data will then be merged on by census tract.

## Set Up
A few programs are needed to run the process: R, RStudio, and Docker. If any are already installed, you can skip to the next program. We are also going to install Firefox because it can be less problematic than Chrome for this process.

### R
R is the programming language and underlying software that is used to run this process.
1. Navigate to the [R website](https://cloud.r-project.org/) and download the newest version of R for Windows. 
2. Once the installer is downloaded, run it.

### RStudio
Even though R is the programming language, we interact with it through a much more user-friendly IDE called RStudio.
1. Navigate to the [RStudio website](https://www.rstudio.com/products/rstudio/download/) and download the newest version of RStudio for Windows. There may be a few options, in which case download the RStudio Desktop version that is free. 
2. Once the installer is downloaded, run it.

### Docker
Docker is a bit harder to explain, but it's essentially a program that lets you run a virtual environment on your computer. It is within this virtual environment that we will be running our automation.
1. Navigate to the [Docker website](https://hub.docker.com/) and download the newest version of Docker Desktop for Windows. You may need to create an account with Docker to do this.
2. Once the installer is downloaded, run it.
3. Go to Task Manager (right click on task bar or ctrl+alt+delete) 
4. Go to the "Performance" tab and see if "Virtualization" is enabled. If so, skip to the next section. If not, continue.
5. Go to the Windows menu (Windows icon button) and type "turn windows features on or off." Select that option from the results.
6. Find the check box called "Hyper-V" and make sure it is checked.
7. Go back to the Windows menu (Windows icon button) and type "Settings." Select that option from the results.
8. Select "Update & Security."
9. Select "Recovery" from the side bar.
10. **Note: This step will restart your computer. Read the following steps to ensure you know what to do next.** Under "Advanced startup," select "Restart now."
11. Select "Troubleshoot."
12. Select "Advanced options."
13. Select "UEFI Firmware Settings."
14. Select "Restart."
15. Your computer will restart in BIOS mode, which will enable you to enable Virtualization. If you are given a startup menu, choose BIOS Setup. 
16. Use the arrow keys to go to the "System Configuration" tab.
17. Change the "Virtualization Technology" option to `<Enabled>` by pressing the Enter key and selecting `<Enabled>.` 
18. Save and Exit using the key specified at the bottom of the screen (likely F10). 

### Firefox
1. Navigate to the [Firefox website](https://www.mozilla.org/en-US/firefox/) and download the newest version of Firefox for Windows.
2. Once the installer is downloaded, run it.

## The Scrape
### Preparing RStudio Environment
1. Open RStudio.
2. Press ctrl+o (or File >> Open) and navigate to 'S:\Evaluation and Graduate Data\R\Address Automation\Code'.
3. Open "2019.05.15 FFIEC Search Address ScrapR.R"
4. If you have never done so before on this computer, type the following in the Console (bottom left):
```
install.packages("pacman")
```

### (R)Selenium
In order to make this process less manual, we will be using Selenium, which is designed to do exactly what we want to do here: automate a repetitive process that involves a web browser. In particular, we will be using RSelenium, which is essentially just Selenium for R (it's actually a package of R bindings for the Selenium Webdriver API). 

Before actually running the script to do the scrape, an important programming lesson on for-loops in R...

### For-loops in R
A for-loop is a common programming feature that essentially just says "Take this list and do the following thing over and over again until you've done it for the entire list." For example, if you had a list of names like `"amy, bob, chris"`, you may want to capitalize the first letter of each. You could create a for-loop that would say "For each item in the list of names, capitalize the first letter." The special thing about for-loops is that they go one-by-one. This lets us ensure that every item is being considered in order. 

As a caveat, for-loops in R are often frowned upon because they aren't always the most efficient way of doing things, but a for-loop works well for our purposes here.

The way you tell R to reference a particular item is by using an iterator variable. If our list of names is called `name_list`, we could get the first item in the list (`"amy"`) by saying `name_list[1]`, the second by saying `name_list[2]`, and so on. In a for loop, your iterator is often the variable `i` (by convention). `i` takes on whatever value the for loop tells it to. In this example, the first time the for-loop is run, `i` = 1, the second time `i` = 2, etc. It's basically a way for R to keep track of where it is in your list. You would see this written, for example, as `name_list[i]`, where `i` can take on different numbers in the for-loop. Other iterators can be specified in the for-loop, but `i` is among the most common and is what I use in this code.

In this case, our for-loop is slightly more complex, but what it is essentially doing is saying "For each address in our list of addresses, go to the FFIEC website and pull down the corresponding census tract income data." When `i` is 1 (the first time through the loop), it will look up the first address in our list. When `i` is 2 (the second time through the loop), it will look up the second address in our list. And so on. 

### Running the Script: A New Run
The FFIEC Address Scraping Script is heavily commented to explain what each line is doing, but please ask me (Zach) if you have any questions. If at any point you want to look at your data in a way that more closely resembles Excel, type (case-sensitive):
```
View(df)
```
in the console, where df is the name of the object you want to see. (This will only work with some types of objects (e.g., dataframes, lists, vectors, etc.))

I would recommend running everything prior to the PREPARE SCRAPE section in one run, and then running through each line of code individually until you hit the RERUN section.

It is important to set your working directory first, which will tell R which folder it is working out of on your computer/the server. The command to set your working directory is toward the top of the script and looks like this:
```
setwd("S:/Evaluation and Graduate Data/R/Address Automation")
```
Replace the folder path inside the quotes if you will be working out of a different location. This may also require you to update the read.csv command later in the code in order to point R to the data you wish to load in. If you are unsure what path to use, simply leave the path as is.

### Running the Script: Return of Doing It Manually (kind of)
Inevitably we will still have some addresses for which we were unable to return a census tract income level. Assuming you have not changed your working directory or, if you have, you have a parallel folder structure, the missing results will be in a timestamped Excel file in the "Intermediate" folder. 

Now for the manual part: you can go through the Excel file and, in the `address` column, try to see if you can fix any places where the formatting, spelling, etc. would confuse the FFIEC search. If you notice a systemic issue in the address (i.e. the same type of issue over and over), please let me (Zach) know and I will try to figure out a way to deal with it before this step. This is also the time when you can lookup the addresses manually for a business, etc.

Once that file is updated, you can run the RERUN section, which reads in the most recently modified file in the "Intermediate" folder and reruns the for-loop for it. If you are still experiencing issues, simply go back to the Excel file and try to clean up the address again. Then, rerun the RERUN section. Repeat the editing and rerunning process until you're satisfied/unable to figure out why the FFIEC isn't accepting the remaining addresses.

### Running the Script: Finalizing
The FINALIZING section simply combines all of the dataframes we've made (the data with found addresses, the data without found addresses, the PO box data, and the non-50 states data) and writes it to a timestamped Excel file in the "Output" folder.

## Appendix 1: R Glossary

Here are a few definitions and pieces of information that might be useful for more novice R users.  

* `<-`  
  * This assigns the results of everything to the right of the `<-` to the object with the name on the left of the `<-`. For example, `foo <- 2 + 2` will create an object called `foo` with a value of `4`.  
* `%>%`  
  * This is called a pipe (from the magrittr package). It tells R to feed the results/value of anything to the left of the `%>%` into whatever is to the right of the `%>%`. For example, `3 %>% sum(2)` will calculate `sum(3, 2)` and yield a result of `5`.   
* `%<>%`  
  * This is a compound pipe (from the magrittr package). It is basically a hybrid of `<-` and `%>%`. It tells R to take the object to the left of the `%<>%`, feed it into everything to the right of the `%<>%`, and then store the results with the same name as the original object. For example, if `foo` is equal to `4`, `foo %<>% sum(7)` will create an object called `foo` (overwriting the old `foo`) that contains the result of `sum(foo, 7)`, which is `11` because `sum(4, 7)` is `11`.  
* `$`  
  * This is an extraction operator. It tells R to look inside the object to the immediate left of the `$` for the object with the name to the right. For example, if we have a dataframe `foo`, `foo$var1` refers to "var1" (a variable or column) inside the dataframe `foo`. A similar example would be if we have a dataframe containing Salesforce data called `df`, `df$first_name` would return the column containing all of the first names from that data.  
* _indentations_  
  * This is an important way of keeping the code organized and following its steps. In the example below,  `2` and `7` are indented because they are the children of the parent `sum`. If something is indented, it means it is not the ultimate parent in this step. Similarly, when I say "to the right" in other definitions, this includes "below and to the right." `foo` is not simply equal to `sum(`, it is equal to `sum(2,7)`.  
~~~~ 
foo <- sum(
    2,
    7)
~~~~


