---
layout: page
title: Mapping with R
---

This workshop expands on the functionality of the `ggplot` package even further, incorporating features on doing __geospatial analysis__. We'll also look at a few other geographic information system (or __GIS__) tools for mapping that will come in handy for visualizing stories in data.

## Getting started

Just like [last time](https://www.dropbox.com/s/kwrsme2oe4e0rum/nc_census_data.zip?dl=1), let's set up a place to store our data and working files.

Somewhere in your computer's directory (it could be your Desktop or Documents folder, for example), create a new folder and give it a name (for example: `data_journalism_with_r`).

After starting RStudio, click "File" > "New File" > "R Script" in the menu to start a new script. Then go ahead and "Save As..." to give it a name.

Add a comment so we know what our script does. And set your working directory. We'll also turn off scientific notation (more on this later).

<div class="alert alert-warning"><b>NOTE:</b> The exact path here &#x2935; depends on where you created your new folder.</div>

```R
#R script for the Duke Data Journalism Lab
#workshop on Feb. 11, 2022 on data viz

#set working directory to the data folder of the downloaded repository
setwd("~/Desktop/data_journalism_with_r")

#turn off scientific notation
options(scipen = 999)
```

Execute the code by clicking "Run" or with <kbd>CMD</kbd> + <kbd>Enter</kbd>.

We should have most of our packages already installed. But we will be using [a new one](https://r-spatial.github.io/sf/) called `sf`, or "simple features."

<div class="alert alert-warning"><b>NOTE:</b> This step &#x2935; we'll only have to do ONCE for each package.</div>

```R
#install other packages
install.packages('sf') #handy set of functions for geospatial analysis
```

Load our [Tidyverse](https://www.tidyverse.org/) package and a few others.

<div class="alert alert-warning"><b>REMEMBER:</b> This step &#x2935; we'll have to do EACH TIME we start R or start a new workspace.</div>

```R
#load our packages from our library into our workspace
library(tidyverse)
library(sf)
```

## About the data

The data we'll be working with today comes from the 2020 census, the once-in-a-decade count of every United States resident required by the Constitution.

Data for the 2020 census [was published (late) in August 2021](https://www.census.gov/programs-surveys/decennial-census/about/rdo/summary-files.html) for the purposes of __redistricting__, when state lawmakers redraw political lines for Congress and state legislatures across the country.

Before we load the data though, it's helpful to understand a little more about it.

Because it's intended specifically for political line-drawing, data from the 2020 redistricting data summary (P.L. 94-171) is a little more limited than you normally expect from the Census. It's [made up of tables](https://www2.census.gov/programs-surveys/decennial/2020/technical-documentation/complete-tech-docs/summary-file/2020Census_PL94_171Redistricting_StatesTechDoc_English.pdf#page=99) counting the following topics:
* Race
* Race/ethnicity
* Race, 18 and up
* Race/ethncity, 18 and up
* Housing units (occupied or vacant)
* Group quarters (student housing, jails, etc.)

![Census summary levels]({{ site.baseurl }}/assets/images/r004_census_levels.jpg)

These counts are gathered on different __summary levels__ (state, county, city) described by a __summary level code__.

| Area type&nbsp;&nbsp;&nbsp; | summary level code |
|:--|:--|
| State&nbsp;&nbsp;&nbsp; | 040 |
| County&nbsp;&nbsp;&nbsp; | 050 |
| Consolidated city&nbsp;&nbsp;&nbsp; | 170 |
| Place&nbsp;&nbsp;&nbsp; | 160 |
| Census tract&nbsp;&nbsp;&nbsp; | 140 |
| Block&nbsp;&nbsp;&nbsp; | 750 |
| Congressional district&nbsp;&nbsp;&nbsp; | 500 |
| NC Senate district&nbsp;&nbsp;&nbsp; | 610 |
| NC House district&nbsp;&nbsp;&nbsp; | 620 |

Every one of the geographies on each of these levels has a unique [__geographic identifier__](https://www.census.gov/programs-surveys/geography/guidance/geo-identifiers.html), or GEOIDS. The position of the number in every ID corresponds to a specific subdivision of a shape (think nesting dolls).

| 37 | 183 | 052404&nbsp;&nbsp;&nbsp; | 1017 |
|:--|:--|:--|:--|
| State&nbsp;&nbsp;&nbsp; | County&nbsp;&nbsp;&nbsp; | Tract&nbsp;&nbsp;&nbsp; | Block&nbsp;&nbsp;&nbsp; |

More on that in a minute.

But there's another thing we should know: Census geographies aren't always stable over time.

Sure, state and counties are more or less the same decade to decade. But there are subtle shifts in other levels of geography – namely tracts and blocks – that mean you can't just compare data from one decennial census to the next.

We'll have to __weight__ data from past years to make comparisons to the latest one. There are a few ways to do that, and the details are a bit outside the scope of this tutorial.

Just know that we'll be using __weighted data__ from 2010 – so don't be surprised if you see some decimals when you might expect whole numbers!

And one last thing.

Census data itself doesn't include any actual _shapes_. We'll need to combine it with a separate set of shapefiles that function a little different than the data we've worked with so far. They're called [TIGER/Line files](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html), and they use the same GEOIDs as our census data.

## Loading the data

[Download this zip file of datasets]({{ site.baseurl }}/assets/data/) below by __right clicking__ the link and selecting "Save link as..."

Save the file to your working directory and unzip it.

Inside, you'll see a folder with several files:
- A CSV of weighted race and ethnicity by block for 2010
- A CSV of race and ethnicity by block for 2020
- A folder containing block-level shapefiles

We can load in our CSVs like we have before using the `read_csv` function.

<div class="alert alert-success"><b>PRO TIP:</b> Instead of typing in full file paths by hand, type the beginning of the path and hit <kbd>TAB</kbd> to autocomplete.</div>

```R
#load in our race and ethnicity data for 2010 and 2020
race_eth2010 <- read_csv('nc_census_data/nc_blocks_2010_rc_eth_weighted.txt')
race_eth2020 <- read_csv('nc_census_data/nc_blocks_2020_rc_eth.txt')
```

Now we'll load in our shapefile data, but that's going to use the `st_read` function from our `sf` package to read in our shapefile. Don't worry if it takes a minute to load – it's a big file!

```R
#load the block shapefile for nc
nc_blocks <- st_read('nc_census_data/nc_blocks/tl_2020_37_tabblock20.shp')
```

When it's finished, the function will output a few things of interest, notably:
* The number of __features__ or shapes
* The coordinates of the bounding box
* The coordinate reference system (or CRS) for the shapefile.

## An aside on maps

"NAD83" stands for the North American Datum of 1983. It's the system geographers used to translate an approximation of a globe into the latitude and longitude coordinates of this particular map.

![Latitude and longitude]({{ site.baseurl }}/assets/images/r004_latlng.png)

Because here's the thing. It's hard to know the shape of a sphere like this:

![Our pale blue dot.]({{ site.baseurl }}/assets/images/r004_earth.gif)

Which is actually more of an ellipsoid – it's a little flattened on the top and bottom!

Other CRS systems (NAD27 or WGS84) do the same thing, but they can differ from each other by as much as 300 meters!

Just remember: latitude and longitudes aren't absolute. So make sure if you're using multiple shapefiles, their coordinate systems match.

If they don't, it will cause problems with some of the "atomic units" of geography: your points, lines and polygons.

The shapefiles we'll be working with are __polygons__ that describe each of the 236,638 blocks in North Carolina.

![]({{ site.baseurl }}/assets/images/r004_geo_elements.png)

## Joining our data

You'll notice that after you loaded in your shapefile data, the `nc_blocks` dataframe looks a lot like your other dataframes. You can even click on it and see that there is a good bit of data embedded there.

But there's a column that looks a little weird – `geometry` – that is a little too complete to show in that traditional two-dimensional table.

Mapping 200,000+ blocks is a little resource intensive. You can do it, but it might take a while to render.

Instead, let's just look at Durham County as a test using the FIPS code as a filter – that's one of those GEOID levels we talked about earlier.

We'll use a new function from `ggplot` called `geom_sf()` to add in our _geometry_ instead of a line or a scatterplot.

```R
#do a test map for durham county
nc_blocks %>%
  filter(COUNTYFP20 == '063') %>% 
  ggplot() +
  geom_sf()
```

![An ugly map of Durham.]({{ site.baseurl }}/assets/images/r004_ugly_durham.png)

Beautiful it ain't, but at least we know our geometry is working!

We can add in a few style elements to strip away the stuff we know we're not interested in, like latitude coordinates and the background and add labels. We can also use a site like [Colorbrewer](https://colorbrewer2.org/) to help us select nice palette.

```R
#improve the syling a bit
nc_blocks %>%
  filter(COUNTYFP20 == '063') %>% 
  ggplot() +
  geom_sf(fill = '#3182bd',
          size = 0.1,
          color = '#f0f0f0') + #add a color and a stroke width
  theme_void() + #add in a preset theme to simplify our map
  labs(caption = "SOURCE: U.S. Census Bureau",
       title = 'Durham County census blocks')
```

But there's not a lot of data associated with the map yet. For that, we'll need to join it up with our actual census data.

Let's start with a few fields from the 2020 data, specifically, the ethnicity data. We'll store the joined data into a new dataframe so we can work with it a little easier.

```R
#join map data with our ethnicity data
nc_blocks_joined <- nc_blocks %>%
  left_join(
    race_eth2020 %>%
      mutate(geocode = as.character(geocode)) %>% #convert our match id to a character
      select(geocode, total, starts_with('eth_')), #subset our columns
    by = c('GEOID20' = 'geocode') #match the ids from the two tables
  )
```

## Creating choropleth maps

Now that we've got some joined data, let's make a __choropleth map__. "Choropleth" is Greek for "multitude of areas," which makes sense here.

We want to show each of the blocks, color-coded by some aspect of our data.

Let's start with something simple: a look at total population. We'll add in `scale_fill_distiller()` to give us some control over the color scale of this variable, which is __continuous__ (it goes from 0 to some whole number).

```R
#map durham population
nc_blocks_joined %>%
  filter(COUNTYFP20 == '063') %>% #filter for durham
  ggplot() +
  geom_sf(aes(fill = total), #add aesthetic based on total population
          size = 0, #adjust the stroke
          ) +
  scale_fill_distiller(
    direction = 1 #change the direction of the scale
  ) +
  theme_void() + #add in a preset theme to simplify our map
  labs(caption = "SOURCE: U.S. Census Bureau",
       title = 'Durham County population by block',
       fill = 'Total population') #label our legend
```

![An ugly map of Durham.]({{ site.baseurl }}/assets/images/r004_durham_population.png)

That's... not great?

The blocks are so small (and there's one particularly dense one), that it's blowing up our scale and making any real trend pretty hard to see.

We could play around with normalizing the data a little bit by dividing the population into quartiles or quantiles. But we're not actually all that interested in the count as much as some of the data _within_ the count.

Let's instead create a new column, calculating the percentage of Hispanic residents and map that instead.

```R
#map durham's hispanic population
nc_blocks_joined %>%
  filter(COUNTYFP20 == '063') %>% #filter for durham
  mutate(hisp_pct = round(eth_hispanic/total * 100, 2)) %>% #calculate our new column
  ggplot() +
  geom_sf(aes(fill = hisp_pct), #add aesthetic based on total population
          size = 0, #adjust the stroke
  ) +
  scale_fill_distiller(
    direction = 1, #change the direction of the scale
    na.value = '#f0f0f0' #what to color the block when it's zero
  ) +
  theme_void() + #add in a preset theme to simplify our map
  labs(caption = "SOURCE: U.S. Census Bureau",
       title = 'Durham County Hispanic population by block',
       fill = '% Hispanic') #label our legend
```
![Durham's Hispanic population.]({{ site.baseurl }}/assets/images/data_journalism_with_r.png)

That's a little more illuminating! We can see concentrations of the Hispanic population in certain places with a little more fidelity, especially if we zoom in.

## Know thy data

There's something of a catch here, though.

For its 2020 census, the bureau introduced a new algorithm to create "noise" in the data to protect the privacy of those who answer. These privacy protections are required by law, and are especially important in places where the population is so small, you can basically reverse engineer the answers.

Because of this change in methodology, the U.S. Census Bureau encourages us to aggregate this data into larger groups so this "fuzziness" disappears.

We can construct our own block groups, of course, but a "block group" is an actual summary level in census data. We can redownload all the data on this level specifically. Or, as a proof of concept, we can aggregate those figures ourselves with the `sf` package.

A block group ID is the first 12 digits of our geographic identifier, so like we've done in the past, we can pair `mutate()` with `substr()` to create a new column that serves as our block group ID.

Then, we can `group_by()` and `summarize()`. We know how to do that already with things like totals and averages. But what about geometry?

We need to merge these grouped block ids together – turn multiple polygons into one. For that, we'll use the `st_union()` function much in the same way we've used functions like `sum()` or `mean()`. Don't forget to recalculate your Hispanic percentage so you can map it.

```R
nc_blockgroups <- nc_blocks_joined %>% 
  filter(COUNTYFP20 == '063') %>% #filter for durham
  mutate(block_group_id = substr(GEOID20, 1, 12)) %>% #create a block group id
  group_by(block_group_id) %>% #group by our new column
  summarize( total = sum(total), #add up the total population
             eth_hispanic = sum(eth_hispanic), #add up the hispanic population
             geometry = st_union(geometry) #merge our geometry
  ) %>% 
  mutate(hisp_pct = round(eth_hispanic/total * 100, 2)) #calculate our new column
```

Then, we can plot that brand new, merged dataset just like we did before. 

```R
nc_blockgroups %>% 
  ggplot() +
  geom_sf(aes(fill = hisp_pct), #an an aesthetic based on our total population column
          size = 0, #adjust the stroke
  ) +
  scale_fill_distiller(
    direction = 1, #change the direction of the scale
    na.value = '#f0f0f0' #what to color the block when it's zero
  ) +
  theme_void() + #add in a preset theme to simplify our map
  labs(caption = "SOURCE: U.S. Census Bureau",
       title = 'Durham County Hispanic population by block',
       fill = '% Hispanic') #label our legend
```

![Durham Hispanic population by block group.]({{ site.baseurl }}/assets/images/r004_blockgroups_hispanic.png)

## Wrapping up

There's a lot more we can do here with just these tools.

Try combining what you've learned so far to explore:

* The distribution of other ethnic and racial groups
* The makeup of other counties
* Historical change from 2010 to 2020

What questions can we ask – and answer – with the data we have? 

## More resources

* [Spacial manipulation with sf – cheat sheet](https://github.com/rstudio/cheatsheets/blob/main/sf.pdf)
* [@everytract](https://twitter.com/everytract)
* [Census Reporter](https://censusreporter.org/)
* [Carolina Demography](https://www.ncdemography.org/)
* [IPUMS NHGIS](https://www.nhgis.org/)

