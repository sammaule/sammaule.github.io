---
layout: post
title: "Visualising the European Energy Mix"
date: 2019-05-24
tags: test1 test2 test3
---

Countries typically generate electricity from multiple sources, which can make it hard to visually identify which countries have similar characteristics because they cannot easily be displayed in two dimensions.

One technique that can help to resolve this is called t-SNE, which was [introduced around a decade ago](https://lvdmaaten.github.io/tsne/) and has producing some stunning results since then (see the Examples section in the previous link - I especially like the MNIST ones). The general aim of this technique is to find a representation of data in lower dimensional space (typically two dimensional) that is faithful to the data in its original higher-dimensional space.

Applying this technique to European energy data makes it possible to quickly identify similarities between countries. Plotting the data as a bubble plot also makes it easy to visualise the relative amounts of energy produced by these countries. Whilst something similar could be done with a much simpler stacked bar chart, it’s much easier to spot clusters of countries this way. 

Specifically, I decided to look at the energy sources for electricity (and heat) generation because I am mainly interested in general similarities and differences between countries energy mixes. Looking at gross energy, whilst giving a better view of the bigger picture, has lower diversity due to the dominance of oil in powering transportation, which makes up a significant chunk of overall energy production.

The code to run t-SNE and generate the visualisation are available on my [GitHub page](https://github.com/sammaule/europe-electricity-tsne) along with the cleansed data.

<iframe width="900" height="800" frameborder="0" scrolling="no" src="//plot.ly/~sam.maule/116/european-electricity-energy-mix-t-sne-representation/#/"></iframe>

The visualisation reveals three main extremes in the data. On the far left are countries with high renewable energy generation, towards the bottom are countries more reliant on solid fossil fuels,  and on the far right are nuclear-heavy countries, with a large cluster of countries with a more balanced mix in the middle. One thing that stood out to me (admittedly because I was looking for it) and gave me a great fact about my holiday destination this year, Albania, is the only country in Europe to produce electricity solely from renewable sources (even more specifically hydropower) after [a gas and oil powered plant financed by the World Bank failed to operate due to technical failures](https://bankwatch.org/beyond-coal/energy-sector-in-albania). Other countries doing well at this side of the plot are, variously, hilly, lucky and forward-thinking but unfortunately small. 

t-SNE also successfully identifies the more nuclear inclined countries to the east of the plot. France everyone knows about, but Finland, Hungary, Belgium and Sweden are not all that dissimilar. Estonia, which is out on its own in the north, is the only country where oil shale is popular choice, who knew!

The middle of the plot is where things get a little more messy as it is where all the countries with more mixed electricity generation lie. Without wanting to commit one of the [cardinal sins of reading t-SNE plots](https://distill.pub/2016/misread-tsne/), I’ll end the description with the safe statements that the UK looks quite like Spain which both look a lot more like France or Austria than they do Poland, while Turkey, Ukraine and Germany have a lot of work to do. 

Overall taking the time to make this plot has taught me a lot about European electricity generation, the relative size of each of the countries electricity markets (Ukraine is surprisingly big) and the relative unimportance of many others (sorry Albania). I’m sure there are many other interesting nuggets of information to be gained by hovering over points, toggling on and off different regions, and generally having a play. The rest of the post is concerned with some of the more technical aspects of the data and the plot, and is entirely optional :). 

### The data

The data used in this visualisation comes from Eurostat. The data is updated on an annual basis and the latest figures are from 2017 for the vast majority of countries in this study.

The data were somewhat difficult to extract from the original spreadsheets, since there was an individual file for each country, and with 39 countries in total, copying and pasting the relevant rows into a separate excel file is a pretty painful exercise. To make matters worse, the data is stored in .xlsb files, which do not work with the popular pandas and openpyxl packages.

To get around this, I made use of a somewhat more niche package called pyxlsb, read the relevant row of the raw data for each country (fortunately formatting was constant across all country files), saved these as csv files before concatenating everything into a single pandas data frame, which is the neat processed_data.csv file in the [GitHub repository](https://github.com/sammaule/europe-electricity-tsne).

### Energy sources

While the Eurostat data provides statistics for every individual energy source, I decided to look one level up (e.g. solid fossil fuels instead of lignite and coal) as I was particularly interested in understanding the clusters of countries doing relatively better/worse at decarbonisation, and at the more aggregated level renewables are all treated together so won’t be separated on the plot. An examination of the data at the more granular level would be equally valid, and could help understand in more detail which countries rely more on e.g. hydropower versus wind and solar boom, however I leave this as an exercise for the interested reader.

### t-SNE specification

Whilst t-SNE Is a powerful tool for visualisation, that same power gives it lots of flexibility and therefore ability to be misused and/or misread. This post on the great [distill.pub site](https://distill.pub/2016/misread-tsne/) goes into detail on how to use and interpret t-SNE plots effectively, guidelines which I have attempted to follow when producing this.

One of the main conclusions of the distill paper is that the hyperparameters for t-SNE really matter. Specifically, the number of iterations must be set high enough to converge to a stable configuration. From experimenting with this value I found that 5,000 was plenty to achieve this. The perplexity, which (roughly) decides whether local or global aspects of the data dominate,  also needs to be tuned carefully. Fortunately this is not too difficult to do in this case, as the results can be compared to our knowledge of the energy systems in European countries, and iterated until they seem to be a faithful representation of reality. In this case I find that 30 seems to give a good result, but slightly lower and higher values also work well.
