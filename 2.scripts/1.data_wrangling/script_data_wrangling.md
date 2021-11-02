---
title: "Data Wrangling"
description: |
  Cleaning and Merging Air Pollution, Weather and Calendar Datasets.
author:
  - name: Léo Zabrocki 
    url: https://lzabrocki.github.io/
    affiliation: Paris School of Economics
    affiliation_url: https://www.parisschoolofeconomics.eu/fr/zabrocki-leo/
  - name: Anna Alari 
    url: https://scholar.google.com/citations?user=MiFY320AAAAJ&hl=fr
    affiliation: Sorbonne Université & INSERM
    affiliation_url: https://www.inserm.fr/
  - name: Tarik Benmarnhia
    url: https://profiles.ucsd.edu/tarik.benmarhnia
    affiliation: UCSD & Scripps Institute
    affiliation_url: https://benmarhniaresearch.ucsd.edu/
date: "2021-11-02"
output: 
    distill::distill_article:
      keep_md: true
      toc: true
      toc_depth: 2
---

<style>
body {
text-align: justify}
</style>



# Required Packages

We load the required packages:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load required packages</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://here.r-lib.org/'>here</a></span><span class='op'>)</span> <span class='co'># for files paths organization</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://tidyverse.tidyverse.org'>tidyverse</a></span><span class='op'>)</span> <span class='co'># for data manipulation and visualization</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://lubridate.tidyverse.org'>lubridate</a></span><span class='op'>)</span> <span class='co'># for manipulating date variables</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://github.com/mayer79/missRanger'>missRanger</a></span><span class='op'>)</span> <span class='co'># for missing values imputation</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='http://www.rforge.net/Cairo/'>Cairo</a></span><span class='op'>)</span> <span class='co'># for printing custom police of graphs</span>
</code></pre></div>

</div>


# Cleaning Air Pollution Data

We load and clean NO$_{2}$ data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># clean no2 data</span>
<span class='va'>data_no2</span> <span class='op'>&lt;-</span>
  <span class='fu'>data.table</span><span class='fu'>::</span><span class='fu'><a href='https://Rdatatable.gitlab.io/data.table/reference/fread.html'>fread</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
      <span class='st'>"1.data"</span>,
      <span class='st'>"1.air_pollutants_data"</span>,
      <span class='st'>"20080101_20200901-NO2_auto.csv"</span>
    <span class='op'>)</span>,
    check.names <span class='op'>=</span> <span class='cn'>TRUE</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, <span class='va'>PA01H</span>, <span class='va'>PA04C</span>, <span class='va'>PA06</span>, <span class='va'>PA07</span>, <span class='va'>PA12</span>, <span class='va'>PA13</span>, <span class='va'>PA15L</span>, <span class='va'>PA18</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># drop first line</span>
  <span class='fu'>slice</span><span class='op'>(</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># paste the elements of date together</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, sep <span class='op'>=</span> <span class='st'>"/"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># convert to date</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd_hms.html'>dmy_h</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>heure</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename_all</span><span class='op'>(</span> <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/chartr.html'>tolower</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&gt;</span> <span class='st'>"2008-01-01 00:00:00"</span> <span class='op'>&amp;</span>
           <span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2019-01-01 00:00:00"</span><span class='op'>)</span>

<span class='co'># select relevant measuring stations</span>
<span class='va'>data_no2</span> <span class='op'>&lt;-</span> <span class='va'>data_no2</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>pa07</span>, <span class='va'>pa12</span>, <span class='va'>pa13</span>, <span class='va'>pa18</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='st'>"mean_no2"</span>, <span class='va'>.</span>, sep <span class='op'>=</span> <span class='st'>"_"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


We load and clean O$_{3}$ data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># clean o3 data</span>
<span class='va'>data_o3</span> <span class='op'>&lt;-</span>
  <span class='fu'>data.table</span><span class='fu'>::</span><span class='fu'><a href='https://Rdatatable.gitlab.io/data.table/reference/fread.html'>fread</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
      <span class='st'>"1.data"</span>,
      <span class='st'>"1.air_pollutants_data"</span>,
      <span class='st'>"20080101_20200901-O3_auto.csv"</span>
    <span class='op'>)</span>,
    check.names <span class='op'>=</span> <span class='cn'>TRUE</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, <span class='va'>PA01H</span>, <span class='va'>PA04C</span>, <span class='va'>PA06</span>, <span class='va'>PA13</span>, <span class='va'>PA18</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># drop first line</span>
  <span class='fu'>slice</span><span class='op'>(</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># paste the elements of date together</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, sep <span class='op'>=</span> <span class='st'>"/"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># convert to date</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd_hms.html'>dmy_h</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>heure</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename_all</span><span class='op'>(</span> <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/chartr.html'>tolower</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&gt;</span> <span class='st'>"2008-01-01 00:00:00"</span> <span class='op'>&amp;</span>
           <span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2019-01-01 00:00:00"</span><span class='op'>)</span>

<span class='co'># select relevant measuring stations</span>
<span class='va'>data_o3</span> <span class='op'>&lt;-</span> <span class='va'>data_o3</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>pa13</span>, <span class='va'>pa18</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='st'>"mean_o3"</span>, <span class='va'>.</span>, sep <span class='op'>=</span> <span class='st'>"_"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


We load and clean PM$_{10}$ data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># clean pm10 data</span>
<span class='va'>data_pm10</span> <span class='op'>&lt;-</span>
  <span class='fu'>data.table</span><span class='fu'>::</span><span class='fu'><a href='https://Rdatatable.gitlab.io/data.table/reference/fread.html'>fread</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
      <span class='st'>"1.data"</span>,
      <span class='st'>"1.air_pollutants_data"</span>,
      <span class='st'>"20080101_20200901-PM10_auto.csv"</span>
    <span class='op'>)</span>,
    check.names <span class='op'>=</span> <span class='cn'>TRUE</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, <span class='va'>PA01H</span>, <span class='va'>PA04C</span>, <span class='va'>PA15L</span>, <span class='va'>PA18</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># drop first line</span>
  <span class='fu'>slice</span><span class='op'>(</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># paste the elements of date together</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, sep <span class='op'>=</span> <span class='st'>"/"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># convert to date</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd_hms.html'>dmy_h</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>heure</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename_all</span><span class='op'>(</span> <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/chartr.html'>tolower</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&gt;</span> <span class='st'>"2008-01-01 00:00:00"</span> <span class='op'>&amp;</span>
           <span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2019-01-01 00:00:00"</span><span class='op'>)</span>

<span class='co'># select relevant measuring stations</span>
<span class='va'>data_pm10</span> <span class='op'>&lt;-</span> <span class='va'>data_pm10</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>pa18</span><span class='op'>)</span>  <span class='op'>%&gt;%</span>
  <span class='fu'>rename_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='st'>"mean_pm10"</span>, <span class='va'>.</span>, sep <span class='op'>=</span> <span class='st'>"_"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


We load and clean PM$_{2.5}$ data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># clean pm2.5 data</span>
<span class='va'>data_pm25</span> <span class='op'>&lt;-</span>
  <span class='fu'>data.table</span><span class='fu'>::</span><span class='fu'><a href='https://Rdatatable.gitlab.io/data.table/reference/fread.html'>fread</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
      <span class='st'>"1.data"</span>,
      <span class='st'>"1.air_pollutants_data"</span>,
      <span class='st'>"20080101_20200901-PM25_auto.csv"</span>
    <span class='op'>)</span>,
    check.names <span class='op'>=</span> <span class='cn'>TRUE</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, <span class='va'>PA01H</span>, <span class='va'>PA04C</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># drop first line</span>
  <span class='fu'>slice</span><span class='op'>(</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># paste the elements of date together</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste</a></span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>heure</span>, sep <span class='op'>=</span> <span class='st'>"/"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># convert to date</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd_hms.html'>dmy_h</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>heure</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename_all</span><span class='op'>(</span> <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/chartr.html'>tolower</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&gt;</span> <span class='st'>"2008-01-01 00:00:00"</span> <span class='op'>&amp;</span>
           <span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2019-01-01 00:00:00"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>cols <span class='op'>=</span> <span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span>,
               names_to <span class='op'>=</span> <span class='st'>"variable"</span>,
               values_to <span class='op'>=</span> <span class='st'>"concentration"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>mean_pm25 <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>concentration</span>, na.rm <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


Merge all pollutants variables together:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># merge pollutants</span>
<span class='va'>data_pollutants</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_no2</span>, <span class='va'>data_o3</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>.</span>, <span class='va'>data_pm10</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>.</span>, <span class='va'>data_pm25</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>
</code></pre></div>

</div>


Compute the average concentration for each day by pollutant:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_pollutants</span> <span class='op'>&lt;-</span> <span class='va'>data_pollutants</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>str_sub</span><span class='op'>(</span><span class='va'>date</span>, <span class='fl'>1</span>, <span class='fl'>10</span><span class='op'>)</span> <span class='op'>%&gt;%</span> <span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd.html'>ymd</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='va'>mean_no2_pa07</span><span class='op'>:</span><span class='va'>mean_pm25</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


# Cleaning Weather Data

We load and clean weather data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># read the data and rename the variables</span>
<span class='va'>weather_data</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/utils/read.table.html'>read.csv</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
      <span class='st'>"1.data"</span>,
      <span class='st'>"2.weather_data"</span>,
      <span class='st'>"daily_weather_data_paris_2008_2018.txt"</span>
    <span class='op'>)</span>,
    header <span class='op'>=</span> <span class='cn'>TRUE</span>,
    sep <span class='op'>=</span> <span class='st'>";"</span>,
    stringsAsFactors <span class='op'>=</span> <span class='cn'>FALSE</span>,
    dec <span class='op'>=</span> <span class='st'>","</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename</span><span class='op'>(</span>
    <span class='st'>"date"</span> <span class='op'>=</span> <span class='st'>"DATE"</span>,
    <span class='st'>"rainfall_height"</span> <span class='op'>=</span> <span class='st'>"RR"</span>,
    <span class='st'>"rainfall_duration"</span> <span class='op'>=</span> <span class='st'>"DRR"</span>,
    <span class='st'>"temperature_average"</span> <span class='op'>=</span> <span class='st'>"TM"</span>,
    <span class='st'>"humidity_average"</span> <span class='op'>=</span> <span class='st'>"UM"</span>,
    <span class='st'>"wind_speed"</span> <span class='op'>=</span> <span class='st'>"FXY"</span>,
    <span class='st'>"wind_direction"</span> <span class='op'>=</span> <span class='st'>"DXY"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>POSTE</span><span class='op'>)</span>

<span class='co'># convert date variable in date format</span>
<span class='va'>weather_data</span> <span class='op'>&lt;-</span> <span class='va'>weather_data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd.html'>ymd</a></span><span class='op'>(</span><span class='va'>weather_data</span><span class='op'>$</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2018-12-31"</span><span class='op'>)</span>

<span class='co'># select relevant variables</span>
<span class='va'>weather_data</span> <span class='op'>&lt;-</span> <span class='va'>weather_data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span>
    <span class='va'>date</span>,
    <span class='va'>temperature_average</span>,
    <span class='va'>rainfall_duration</span>,
    <span class='va'>humidity_average</span>,
    <span class='va'>wind_speed</span>,
    <span class='va'>wind_direction</span>
  <span class='op'>)</span>
</code></pre></div>

</div>



# Cleaning Calendar Data

First, we create a dataframe with the date variable, the year, the month, and the day of the week:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create a dataframe with the date variable</span>
<span class='va'>dates_data</span> <span class='op'>&lt;-</span>
  <span class='fu'>tibble</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/seq.Date.html'>seq.Date</a></span><span class='op'>(</span>
    from <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/as.Date.html'>as.Date</a></span><span class='op'>(</span><span class='st'>"2008-01-01"</span><span class='op'>)</span>,
    to <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/as.Date.html'>as.Date</a></span><span class='op'>(</span><span class='st'>"2018-12-31"</span><span class='op'>)</span>,
    by <span class='op'>=</span> <span class='st'>"day"</span>
  <span class='op'>)</span><span class='op'>)</span>

<span class='co'># create julian data: the starting date is 2008-01-01</span>
<span class='va'>dates_data</span> <span class='op'>&lt;-</span> <span class='va'>dates_data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>julian_date <span class='op'>=</span> <span class='fl'>1</span><span class='op'>:</span><span class='fu'>n</span><span class='op'>(</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># create year, month and day of the week indicators</span>
<span class='va'>dates_data</span> <span class='op'>&lt;-</span> <span class='va'>dates_data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    year <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/year.html'>year</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span>,
    month <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/month.html'>month</a></span><span class='op'>(</span><span class='va'>date</span>, label <span class='op'>=</span> <span class='cn'>TRUE</span>, abbr <span class='op'>=</span> <span class='cn'>FALSE</span><span class='op'>)</span>,
    weekday <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/day.html'>wday</a></span><span class='op'>(</span><span class='va'>date</span>, label <span class='op'>=</span> <span class='cn'>TRUE</span>, abbr <span class='op'>=</span> <span class='cn'>FALSE</span><span class='op'>)</span>
  <span class='op'>)</span>

<span class='co'># reorder day of the week levels</span>
<span class='va'>dates_data</span> <span class='op'>&lt;-</span> <span class='va'>dates_data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>weekday <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>ordered</a></span><span class='op'>(</span>
    <span class='va'>weekday</span>,
    levels <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span>
      <span class='st'>"Monday"</span>,
      <span class='st'>"Tuesday"</span>,
      <span class='st'>"Wednesday"</span>,
      <span class='st'>"Thursday"</span>,
      <span class='st'>"Friday"</span>,
      <span class='st'>"Saturday"</span>,
      <span class='st'>"Sunday"</span>
    <span class='op'>)</span>
  <span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


Then, we load two datasets on holidays and bank days indicators:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># cleaning holidays data</span>
<span class='va'>data_holidays</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/utils/read.table.html'>read.csv</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"3.calendar_data"</span>, <span class='st'>"data_holidays.csv"</span><span class='op'>)</span>,
    stringsAsFactors <span class='op'>=</span> <span class='cn'>FALSE</span>,
    encoding <span class='op'>=</span> <span class='st'>"UTF-8"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>date</span>, <span class='va'>vacances_zone_c</span>, <span class='va'>nom_vacances</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename</span><span class='op'>(</span>holidays_dummy <span class='op'>=</span> <span class='va'>vacances_zone_c</span>, holidays_name <span class='op'>=</span> <span class='va'>nom_vacances</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>holidays_dummy <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='va'>holidays_dummy</span> <span class='op'>==</span> <span class='st'>"True"</span>, <span class='fl'>1</span>, <span class='fl'>0</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd.html'>ymd</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&gt;=</span> <span class='st'>"2008-01-01"</span> <span class='op'>&amp;</span> <span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2018-12-31"</span><span class='op'>)</span>

<span class='co'># cleaning bank days data</span>
<span class='va'>data_bank_days</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/utils/read.table.html'>read.csv</a></span><span class='op'>(</span>
    <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"3.calendar_data"</span>, <span class='st'>"data_bank_days.csv"</span><span class='op'>)</span>,
    stringsAsFactors <span class='op'>=</span> <span class='cn'>FALSE</span>,
    encoding <span class='op'>=</span> <span class='st'>"UTF-8"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename</span><span class='op'>(</span>bank_day_dummy <span class='op'>=</span> <span class='va'>est_jour_ferie</span>, name_bank_day <span class='op'>=</span> <span class='va'>nom_jour_ferie</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>bank_day_dummy <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='va'>bank_day_dummy</span> <span class='op'>==</span> <span class='st'>"True"</span>, <span class='fl'>1</span>, <span class='fl'>0</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>date <span class='op'>=</span> <span class='fu'>lubridate</span><span class='fu'>::</span><span class='fu'><a href='https://lubridate.tidyverse.org/reference/ymd.html'>ymd</a></span><span class='op'>(</span><span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>date</span> <span class='op'>&gt;=</span> <span class='st'>"2008-01-01"</span> <span class='op'>&amp;</span> <span class='va'>date</span> <span class='op'>&lt;=</span> <span class='st'>"2018-12-31"</span><span class='op'>)</span>
</code></pre></div>

</div>

  
We merge the three datasets together:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># merge the three datasets together</span>
<span class='va'>data_calendar</span> <span class='op'>&lt;-</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>dates_data</span>, <span class='va'>data_holidays</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>.</span>, <span class='va'>data_bank_days</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>
</code></pre></div>

</div>


# Merging All Datasets

We merge all datasets together:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_calendar</span>, <span class='va'>data_pollutants</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>.</span>, <span class='va'>weather_data</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>
</code></pre></div>

</div>


# Imputing Missing Values

We impute missing values using the `missRanger` package:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># imputation of missing values</span>
<span class='va'>data</span> <span class='op'>&lt;-</span> <span class='fu'>missRanger</span><span class='fu'>::</span><span class='fu'><a href='https://rdrr.io/pkg/missRanger/man/missRanger.html'>missRanger</a></span><span class='op'>(</span>
  <span class='va'>data</span>,
  <span class='co'># variables to impute</span>
  <span class='va'>mean_no2_pa07</span> <span class='op'>+</span>
    <span class='va'>mean_no2_pa12</span> <span class='op'>+</span>
    <span class='va'>mean_no2_pa13</span> <span class='op'>+</span>
    <span class='va'>mean_no2_pa18</span> <span class='op'>+</span>
    <span class='va'>mean_o3_pa13</span> <span class='op'>+</span>
    <span class='va'>mean_o3_pa18</span> <span class='op'>+</span>
    <span class='va'>mean_pm10_pa18</span> <span class='op'>+</span>
    <span class='va'>rainfall_duration</span> <span class='op'>+</span>
    <span class='va'>humidity_average</span> <span class='op'>+</span>
    <span class='va'>wind_speed</span> <span class='op'>+</span>
    <span class='va'>wind_direction</span> <span class='op'>-</span>
    <span class='va'>mean_pm25</span> <span class='op'>~</span>
    <span class='co'># variables used for the imputation</span>
    <span class='va'>.</span> <span class='op'>-</span> <span class='va'>date</span> <span class='op'>-</span> <span class='va'>julian_date</span> <span class='op'>-</span> <span class='va'>holidays_name</span> <span class='op'>-</span> <span class='va'>name_bank_day</span>,
  pmm.k <span class='op'>=</span> <span class='fl'>10</span>,
  num.trees <span class='op'>=</span> <span class='fl'>100</span>
<span class='op'>)</span>
</code></pre></div>

```

Missing value imputation by random forests

  Variables to impute:		mean_no2_pa07, mean_no2_pa12, mean_no2_pa13, mean_no2_pa18, mean_o3_pa13, mean_o3_pa18, mean_pm10_pa18, rainfall_duration, humidity_average, wind_speed, wind_direction
  Variables used to impute:	year, month, weekday, holidays_dummy, bank_day_dummy, mean_no2_pa07, mean_no2_pa12, mean_no2_pa13, mean_no2_pa18, mean_o3_pa13, mean_o3_pa18, mean_pm10_pa18, temperature_average, rainfall_duration, humidity_average, wind_speed, wind_direction
iter 1:	...........
iter 2:	...........
iter 3:	...........
iter 4:	...........
iter 5:	...........
iter 6:	...........
```

</div>


# Last Cleaning Steps

We cut the rainfall duration into quartiles and wind direction into the main four directions:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># add rainfall duration quartiles</span>
<span class='va'>data</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>rainfall_duration <span class='op'>=</span> <span class='fu'>Hmisc</span><span class='fu'>::</span><span class='fu'><a href='https://rdrr.io/pkg/Hmisc/man/cut2.html'>cut2</a></span><span class='op'>(</span><span class='va'>rainfall_duration</span>, g <span class='op'>=</span> <span class='fl'>4</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='co'># create wind direction categories</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    wind_direction_categories <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/cut.html'>cut</a></span><span class='op'>(</span>
      <span class='va'>wind_direction</span>,
      breaks <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/seq.html'>seq</a></span><span class='op'>(</span><span class='fl'>0</span>, <span class='fl'>360</span>, by  <span class='op'>=</span> <span class='fl'>90</span><span class='op'>)</span>,
      include.lowest <span class='op'>=</span> <span class='cn'>TRUE</span>
    <span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'>recode</span><span class='op'>(</span>
        <span class='va'>.</span>,
        <span class='st'>"[0,90]"</span> <span class='op'>=</span> <span class='st'>"North-East"</span>,
        <span class='st'>"(90,180]"</span> <span class='op'>=</span> <span class='st'>"South-East"</span>,
        <span class='st'>"(180,270]"</span> <span class='op'>=</span> <span class='st'>"South-West"</span>,
        <span class='st'>"(270,360]"</span> <span class='op'>=</span> <span class='st'>"North-West"</span>
      <span class='op'>)</span>
  <span class='op'>)</span> 
</code></pre></div>

</div>


We finally aggregate air pollutant concentrations at the city level:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># aggregation of each pollutant's concentrations at the city level</span>
<span class='va'>data</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rowwise</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>mean_no2 <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span>
    <span class='va'>mean_no2_pa07</span>, <span class='va'>mean_no2_pa12</span>, <span class='va'>mean_no2_pa13</span>, <span class='va'>mean_no2_pa18</span>
  <span class='op'>)</span><span class='op'>)</span>,
  mean_o3 <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>mean_o3_pa13</span>, <span class='va'>mean_o3_pa18</span><span class='op'>)</span><span class='op'>)</span>,
  mean_pm10 <span class='op'>=</span> <span class='va'>mean_pm10_pa18</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>
</code></pre></div>

</div>


We finally select relevant variables and save the data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span>
    <span class='va'>date</span><span class='op'>:</span><span class='va'>holidays_dummy</span>,
    <span class='va'>bank_day_dummy</span>,
    <span class='va'>mean_no2</span>,
    <span class='va'>mean_o3</span>,
    <span class='va'>mean_pm10</span>,
    <span class='va'>mean_pm25</span>,
    <span class='va'>temperature_average</span><span class='op'>:</span><span class='va'>wind_speed</span>,
    <span class='va'>wind_direction</span>,
    <span class='va'>wind_direction_categories</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/base/readRDS.html'>saveRDS</a></span><span class='op'>(</span><span class='va'>.</span>,
          <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"5.data_for_analysis"</span>, <span class='st'>"data_for_analysis.RDS"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>




```{.r .distill-force-highlighting-css}
```
