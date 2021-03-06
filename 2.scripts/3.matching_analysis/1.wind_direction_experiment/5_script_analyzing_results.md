---
title: "Analyzing Results"
description: |
  Comparing days with Wind Blowing from the North-East to Other Directions. Adjusting for calendar indicators and other weather variables.
author:
  - name: Léo Zabrocki 
    url: https://lzabrocki.github.io/
    affiliation: Paris School of Economics
    affiliation_url: https://www.parisschoolofeconomics.eu/fr/zabrocki-leo/
  - name: Anna Alari 
    url: https://scholar.google.com/citations?user=MiFY320AAAAJ&hl=fr
    affiliation: ISGlobal
    affiliation_url: https://www.isglobal.org/
  - name: Tarik Benmarhnia
    url: https://profiles.ucsd.edu/tarik.benmarhnia
    affiliation: UCSD & Scripps Institute
    affiliation_url: https://benmarhniaresearch.ucsd.edu/
date: "2022-02-24"
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



In this document, we take great care providing all steps and R codes required to estimate the influence of North-East winds on air pollutants. We compare days where:

* treated units are days where winds blow from the North-East in *t*.
* control units are day winds blow from other directions in *t*.

We adjust for calendar indicators and weather confouding factors.

**Should you have any questions, need help to reproduce the analysis or find coding errors, please do not hesitate to contact me at leo.zabrocki@psemail.eu**

# Required Packages

To reproduce exactly the `5_script_analyzing_results.html` document, we first need to have installed:

* the [R](https://www.r-project.org/) programming language 
* [RStudio](https://rstudio.com/), an integrated development environment for R, which will allow you to knit the `5_script_analyzing_results.Rmd` file and interact with the R code chunks
* the [R Markdown](https://rmarkdown.rstudio.com/) package
* and the [Distill](https://rstudio.github.io/distill/) package which provides the template for this document. 

Once everything is set up, we have to load the following packages:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load required packages</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://yihui.org/knitr/'>knitr</a></span><span class='op'>)</span> <span class='co'># for creating the R Markdown document</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://here.r-lib.org/'>here</a></span><span class='op'>)</span> <span class='co'># for files paths organization</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://tidyverse.tidyverse.org'>tidyverse</a></span><span class='op'>)</span> <span class='co'># for data manipulation and visualization</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://github.com/andytimm/retrodesign'>retrodesign</a></span><span class='op'>)</span> <span class='co'># for assessing type m and s errors</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='http://www.rforge.net/Cairo/'>Cairo</a></span><span class='op'>)</span> <span class='co'># for printing custom police of graphs</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://patchwork.data-imaginist.com'>patchwork</a></span><span class='op'>)</span> <span class='co'># combining plots</span>
<span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='https://github.com/rstudio/DT'>DT</a></span><span class='op'>)</span> <span class='co'># for tables</span>
</code></pre></div>

</div>


We finally load our custom `ggplot2` theme for graphs:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load ggplot custom theme</span>
<span class='kw'><a href='https://rdrr.io/r/base/source.html'>source</a></span><span class='op'>(</span><span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
  <span class='st'>"2.scripts"</span>,
  <span class='st'>"4.custom_ggplot2_theme"</span>,
  <span class='st'>"script_theme_tufte.R"</span>
<span class='op'>)</span><span class='op'>)</span>
<span class='co'># define nice colors</span>
<span class='va'>my_blue</span> <span class='op'>&lt;-</span> <span class='st'>"#0081a7"</span>
<span class='va'>my_orange</span> <span class='op'>&lt;-</span> <span class='st'>"#fb8500"</span>
</code></pre></div>

</div>


# Preparing the Data

We load the matched data:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load matched data</span>
<span class='va'>data_matched</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/base/readRDS.html'>readRDS</a></span><span class='op'>(</span><span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"5.matched_data"</span>, <span class='st'>"matched_data.rds"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


# Distribution of the Pair Differences in Concentration between Treated and Control units for each Pollutant

### Computing Pairs Differences in Pollutant Concentrations

We first compute the differences in a pollutant's concentration for each pair over time:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_matched_wide</span> <span class='op'>&lt;-</span> <span class='va'>data_matched</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>is_treated <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='va'>is_treated</span> <span class='op'>==</span> <span class='cn'>TRUE</span>, <span class='st'>"treated"</span>, <span class='st'>"control"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span>
    <span class='va'>is_treated</span>,
    <span class='va'>pair_number</span>,
    <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"mean_no2"</span><span class='op'>)</span>,
    <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"mean_o3"</span><span class='op'>)</span>,
    <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"mean_pm10"</span><span class='op'>)</span>,
    <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"mean_pm25"</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>pair_number</span>, <span class='va'>is_treated</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"variable"</span>,
    values_to <span class='op'>=</span> <span class='st'>"concentration"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    pollutant <span class='op'>=</span> <span class='cn'>NA</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"no2"</span><span class='op'>)</span>, <span class='st'>"NO2"</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"o3"</span><span class='op'>)</span>, <span class='st'>"O3"</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"pm10"</span><span class='op'>)</span>, <span class='st'>"PM10"</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"pm25"</span><span class='op'>)</span>, <span class='st'>"PM2.5"</span>, <span class='va'>.</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>time <span class='op'>=</span> <span class='fl'>0</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"lag_1"</span><span class='op'>)</span>,<span class='op'>-</span><span class='fl'>1</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"lead_1"</span><span class='op'>)</span>, <span class='fl'>1</span>, <span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>variable</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>pair_number</span>, <span class='va'>is_treated</span>, <span class='va'>pollutant</span>, <span class='va'>time</span>, <span class='va'>concentration</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_wider</span><span class='op'>(</span>names_from <span class='op'>=</span> <span class='va'>is_treated</span>, values_from <span class='op'>=</span> <span class='va'>concentration</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>unnest</span><span class='op'>(</span><span class='op'>)</span>

<span class='va'>data_pair_difference_pollutant</span> <span class='op'>&lt;-</span> <span class='va'>data_matched_wide</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>difference <span class='op'>=</span> <span class='va'>treated</span> <span class='op'>-</span> <span class='va'>control</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>treated</span>, <span class='va'>control</span><span class='op'>)</span><span class='op'>)</span> 
</code></pre></div>

</div>


### Pairs Differences in NO2 Concentrations

Boxplots for NO2:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create the graph for no2</span>
<span class='va'>graph_boxplot_difference_pollutant_no2</span> <span class='op'>&lt;-</span>
  <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='st'>"NO2"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>.</span>, <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_boxplot</span><span class='op'>(</span>colour <span class='op'>=</span> <span class='va'>my_blue</span>, size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span> <span class='op'>~</span> <span class='va'>pollutant</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Pair Difference in \nConcentration (µg/m3)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># display the graph</span>
<span class='va'>graph_boxplot_difference_pollutant_no2</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-5-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_boxplot_difference_pollutant_no2</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"3.outputs"</span>,
    <span class='st'>"2.matching_analysis"</span>,
    <span class='st'>"graph_boxplot_difference_pollutant_no2.pdf"</span>
  <span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>18</span>,
  height <span class='op'>=</span> <span class='fl'>9</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>


### Pairs Differences in O3 Concentrations

Boxplots for O3:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create the graph for o3</span>
<span class='va'>graph_boxplot_difference_pollutant_o3</span> <span class='op'>&lt;-</span>
  <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='st'>"O3"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>.</span>, <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_boxplot</span><span class='op'>(</span>colour <span class='op'>=</span> <span class='va'>my_blue</span>, size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Pair Difference in \nConcentration (µg/m3)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># display the graph</span>
<span class='va'>graph_boxplot_difference_pollutant_o3</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-6-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_boxplot_difference_pollutant_o3</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"3.outputs"</span>,
    <span class='st'>"2.matching_analysis"</span>,
    <span class='st'>"graph_boxplot_difference_pollutant_o3.pdf"</span>
  <span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>18</span>,
  height <span class='op'>=</span> <span class='fl'>9</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>


### Pairs Differences in PM10 Concentrations

Boxplots for PM10:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create the graph for pm10</span>
<span class='va'>graph_boxplot_difference_pollutant_pm10</span> <span class='op'>&lt;-</span>
  <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='st'>"PM10"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>.</span>, <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_boxplot</span><span class='op'>(</span>colour <span class='op'>=</span> <span class='va'>my_blue</span>, size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span> <span class='op'>~</span> <span class='va'>pollutant</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Pair Difference in \nConcentration (µg/m3)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># display the graph</span>
<span class='va'>graph_boxplot_difference_pollutant_pm10</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-7-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_boxplot_difference_pollutant_pm10</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"3.outputs"</span>,
    <span class='st'>"2.matching_analysis"</span>,
    <span class='st'>"graph_boxplot_difference_pollutant_pm10.pdf"</span>
  <span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>18</span>,
  height <span class='op'>=</span> <span class='fl'>9</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>



### Pairs Differences in PM2.5 Concentrations

Boxplots for PM2.5:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create the graph for pm10</span>
<span class='va'>graph_boxplot_difference_pollutant_pm25</span> <span class='op'>&lt;-</span>
  <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='st'>"PM2.5"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>.</span>, <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_boxplot</span><span class='op'>(</span>colour <span class='op'>=</span> <span class='va'>my_blue</span>, size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span> <span class='op'>~</span> <span class='va'>pollutant</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Pair Difference in \nConcentration (µg/m3)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># display the graph</span>
<span class='va'>graph_boxplot_difference_pollutant_pm25</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-8-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_boxplot_difference_pollutant_pm25</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"3.outputs"</span>,
    <span class='st'>"2.matching_analysis"</span>,
    <span class='st'>"graph_boxplot_difference_pollutant_pm25.pdf"</span>
  <span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>18</span>,
  height <span class='op'>=</span> <span class='fl'>9</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>


# Neymanian Inference: Computing 99% and 95% Confidence Intervals for the Average Treatment Effects

We compute confidence intervals for the average treatment effect of North-East winds using Neyman's approach. We use the formula for the standard error of pair randomized experiment found in Imbens and Rubin (2015).

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># we first compute the average treatment effects for each pollutant and day</span>
<span class='va'>data_pair_mean_difference</span> <span class='op'>&lt;-</span> <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>pollutant</span> <span class='op'>!=</span> <span class='st'>"PM2.5"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>mean_difference <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># we store the number of pairs</span>
<span class='va'>n_pair</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span><span class='op'>(</span><span class='va'>data_matched</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span>

<span class='co'># compute the standard error</span>
<span class='va'>data_se_neyman_pair</span> <span class='op'>&lt;-</span>
  <span class='fu'>left_join</span><span class='op'>(</span>
    <span class='va'>data_pair_difference_pollutant</span>,
    <span class='va'>data_pair_mean_difference</span>,
    by <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"pollutant"</span>, <span class='st'>"time"</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>squared_difference <span class='op'>=</span> <span class='op'>(</span><span class='va'>difference</span> <span class='op'>-</span> <span class='va'>mean_difference</span><span class='op'>)</span> <span class='op'>^</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>standard_error <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>/</span> <span class='op'>(</span><span class='va'>n_pair</span> <span class='op'>*</span> <span class='op'>(</span><span class='va'>n_pair</span> <span class='op'>-</span> <span class='fl'>1</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>*</span> <span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='va'>squared_difference</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span>, <span class='va'>standard_error</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># merge the average treatment effect data witht the standard error data</span>
<span class='va'>data_neyman</span> <span class='op'>&lt;-</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_pair_mean_difference</span>,
            <span class='va'>data_se_neyman_pair</span>,
            by <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"pollutant"</span>, <span class='st'>"time"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
<span class='co'># compute the 95% and 99% confidence intervals</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    upper_bound_95 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>+</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.95</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>,
    lower_bound_95 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>-</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.95</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>,
    upper_bound_99 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>+</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.99</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>,
    lower_bound_99 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>-</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.99</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>
  <span class='op'>)</span>
</code></pre></div>

</div>


We plot below the point estimates and the associated 95% and 99% confidence intervals:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create an indicator to alternate shading of confidence intervals</span>
<span class='va'>data_neyman</span> <span class='op'>&lt;-</span> <span class='va'>data_neyman</span> <span class='op'>%&gt;%</span>
  <span class='fu'>arrange</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>stripe <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='op'>(</span><span class='va'>time</span> <span class='op'>%%</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>==</span> <span class='fl'>0</span>, <span class='st'>"Grey"</span>, <span class='st'>"White"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># make the graph</span>
<span class='va'>graph_ci</span> <span class='op'>&lt;-</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>data_neyman</span>,
         <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>mean_difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_rect</span><span class='op'>(</span>
    <span class='fu'>aes</span><span class='op'>(</span>fill <span class='op'>=</span> <span class='va'>stripe</span><span class='op'>)</span>,
    xmin <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>data_neyman</span><span class='op'>$</span><span class='va'>time</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>-</span> <span class='fl'>0.42</span>,
    xmax <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>data_neyman</span><span class='op'>$</span><span class='va'>time</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span> <span class='fl'>0.42</span>,
    ymin <span class='op'>=</span> <span class='op'>-</span><span class='cn'>Inf</span>,
    ymax <span class='op'>=</span> <span class='cn'>Inf</span>,
    color <span class='op'>=</span> <span class='cn'>NA</span>,
    alpha <span class='op'>=</span> <span class='fl'>0.4</span>
  <span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_hline</span><span class='op'>(</span>yintercept <span class='op'>=</span> <span class='fl'>0</span>,
             color <span class='op'>=</span> <span class='st'>"black"</span>,
             size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_vline</span><span class='op'>(</span>xintercept <span class='op'>=</span> <span class='st'>"0"</span>,
             color <span class='op'>=</span> <span class='st'>"black"</span>,
             size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_pointrange</span><span class='op'>(</span>
    <span class='fu'>aes</span><span class='op'>(</span>
      x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>,
      y <span class='op'>=</span> <span class='va'>mean_difference</span>,
      ymin <span class='op'>=</span> <span class='va'>lower_bound_95</span> ,
      ymax <span class='op'>=</span> <span class='va'>upper_bound_95</span>
    <span class='op'>)</span>,
    size <span class='op'>=</span> <span class='fl'>0.5</span>,
    colour <span class='op'>=</span> <span class='va'>my_blue</span>,
    lwd <span class='op'>=</span> <span class='fl'>0.8</span>
  <span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_pointrange</span><span class='op'>(</span>
    <span class='fu'>aes</span><span class='op'>(</span>
      x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>,
      y <span class='op'>=</span> <span class='va'>mean_difference</span>,
      ymin <span class='op'>=</span> <span class='va'>lower_bound_99</span> ,
      ymax <span class='op'>=</span> <span class='va'>upper_bound_99</span>
    <span class='op'>)</span>,
    colour <span class='op'>=</span> <span class='va'>my_blue</span>,
    lwd <span class='op'>=</span> <span class='fl'>0.3</span>
  <span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span> <span class='op'>~</span> <span class='va'>pollutant</span>, ncol <span class='op'>=</span> <span class='fl'>4</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_fill_manual</span><span class='op'>(</span>values <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'gray96'</span>, <span class='st'>"white"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>guides</span><span class='op'>(</span>fill <span class='op'>=</span> <span class='cn'>FALSE</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Average Increase in\n Concentrations (µg/m³)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme</span><span class='op'>(</span>axis.text.x <span class='op'>=</span> <span class='fu'>element_text</span><span class='op'>(</span>margin <span class='op'>=</span> <span class='fu'>ggplot2</span><span class='fu'>::</span><span class='fu'><a href='https://ggplot2.tidyverse.org/reference/element.html'>margin</a></span><span class='op'>(</span>t <span class='op'>=</span> <span class='fl'>0</span>, unit <span class='op'>=</span> <span class='st'>"cm"</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>


<span class='co'># print the graph</span>
<span class='va'>graph_ci</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-10-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_ci</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"3.outputs"</span>, <span class='st'>"2.matching_analysis"</span>, <span class='st'>"graph_ci.pdf"</span><span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>16</span>,
  height <span class='op'>=</span> <span class='fl'>8</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>


We display below the table with the point estimates and the 95% and 99% confidence intervals:

<div class="layout-chunk" data-layout="l-body-outset">

```{=html}
<div id="htmlwidget-4bd8990f8bfc5d69ded5" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-4bd8990f8bfc5d69ded5">{"x":{"filter":"none","data":[["1","2","3","4","5","6","7","8","9"],["NO2","NO2","NO2","O3","O3","O3","PM10","PM10","PM10"],[-1,0,1,-1,0,1,-1,0,1],[-1,-0.1,1.1,0,0,2.8,0.7,4.3,6.2],[-4.4,-3.4,-2.9,-3,-2.7,-1,0,2.5,3.9],[2.5,3.2,5.2,3.1,2.7,6.5,1.5,6,8.4],[-5.5,-4.5,-4.1,-4,-3.6,-2.2,-0.2,2,3.3],[3.5,4.3,6.4,4,3.6,7.7,1.7,6.5,9.1]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Pollutant<\/th>\n      <th>Time<\/th>\n      <th>Point Estimate<\/th>\n      <th>Lower Bound of the 95% Neymanian Interval<\/th>\n      <th>Upper Bound of the 95% Neymanian Interval<\/th>\n      <th>Lower Bound of the 99% Neymanian Interval<\/th>\n      <th>Upper Bound of the 99% Neymanian Interval<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[2,3,4,5,6,7]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

</div>


We can finally check if there is also an effect on North-East winds on PM2.5 concentrations. One issue is that Paris did not have measuring stations for background PM2.5 concentrations from 2009-09-22 to 2010-06-23, that is to say 274 days. We did not impute these missing concentrations but can nonetheless assess if the North-East winds influence the observed pollutant concentrations. We proceed as before but only work with pairs of days without missing PM2.5 recordings:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># we first only select pm2.5 pair differences</span>
<span class='va'>data_pair_difference_pollutant_pm25</span> <span class='op'>&lt;-</span> <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>pollutant</span> <span class='op'>==</span> <span class='st'>"PM2.5"</span><span class='op'>)</span>

<span class='co'># we then find pairs with missing PM2.5 concentrations</span>
<span class='va'>pairs_to_remove</span> <span class='op'>&lt;-</span> <span class='va'>data_pair_difference_pollutant_pm25</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span><span class='op'>(</span><span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span> 
  <span class='fu'>distinct</span><span class='op'>(</span><span class='va'>pair_number</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pull</span><span class='op'>(</span><span class='va'>pair_number</span><span class='op'>)</span>

<span class='co'># we remove those pairs</span>
<span class='va'>data_pair_difference_pollutant_pm25</span> <span class='op'>&lt;-</span> <span class='va'>data_pair_difference_pollutant_pm25</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='op'>!</span><span class='op'>(</span><span class='va'>pair_number</span> <span class='op'>%in%</span> <span class='va'>pairs_to_remove</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># we compute the average treatment effects for pm2.5 and by day</span>
<span class='va'>data_pair_mean_difference_pm25</span> <span class='op'>&lt;-</span>  <span class='va'>data_pair_difference_pollutant_pm25</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>mean_difference <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>difference</span>, na.rm <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># we store the number of pairs</span>
<span class='va'>n_pair</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/unique.html'>unique</a></span><span class='op'>(</span><span class='va'>data_pair_difference_pollutant_pm25</span><span class='op'>$</span><span class='va'>pair_number</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># we compute the standard error</span>
<span class='va'>data_se_neyman_pair_pm25</span> <span class='op'>&lt;-</span>
  <span class='fu'>left_join</span><span class='op'>(</span>
    <span class='va'>data_pair_difference_pollutant_pm25</span>,
    <span class='va'>data_pair_mean_difference_pm25</span>,
    by <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"time"</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>squared_difference <span class='op'>=</span> <span class='op'>(</span><span class='va'>difference</span> <span class='op'>-</span> <span class='va'>mean_difference</span><span class='op'>)</span> <span class='op'>^</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>standard_error <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>/</span> <span class='op'>(</span><span class='va'>n_pair</span> <span class='op'>*</span> <span class='op'>(</span><span class='va'>n_pair</span> <span class='op'>-</span> <span class='fl'>1</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>*</span> <span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='va'>squared_difference</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>time</span>, <span class='va'>standard_error</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># merge the average treatment effect data witht the standard error data</span>
<span class='va'>data_neyman_pm25</span> <span class='op'>&lt;-</span>
  <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_pair_mean_difference_pm25</span>,
            <span class='va'>data_se_neyman_pair_pm25</span>,
            by <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"time"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
<span class='co'># compute the 95% and 99% confidence intervals</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    upper_bound_95 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>+</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.95</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>,
    lower_bound_95 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>-</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.95</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>,
    upper_bound_99 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>+</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.99</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>,
    lower_bound_99 <span class='op'>=</span> <span class='va'>mean_difference</span> <span class='op'>-</span> <span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span> <span class='op'>-</span> <span class='fl'>0.99</span><span class='op'>)</span> <span class='op'>/</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>*</span> <span class='va'>standard_error</span><span class='op'>)</span>
  <span class='op'>)</span>
</code></pre></div>

</div>


We display below the estimates for the ATE and the associated 95% and 99% confidence intervals:

<div class="layout-chunk" data-layout="l-body-outset">

```{=html}
<div id="htmlwidget-d5f8316136f6902f9b2c" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-d5f8316136f6902f9b2c">{"x":{"filter":"none","data":[["1","2","3"],[-1,0,1],[0.3,2.3,5.4],[-0.4,1,3.9],[0.9,3.6,6.9],[-0.6,0.6,3.4],[1.1,4,7.4]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Time<\/th>\n      <th>Point Estimate<\/th>\n      <th>Lower Bound of the 95% Neymanian Interval<\/th>\n      <th>Upper Bound of the 95% Neymanian Interval<\/th>\n      <th>Lower Bound of the 99% Neymanian Interval<\/th>\n      <th>Upper Bound of the 99% Neymanian Interval<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

</div>



# Robustness Checks

### Missing Concentrations

We load non-imputed air pollution data and compute for each pollutant the 0-1 daily lags and leads:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load non-imputed data</span>
<span class='va'>data_not_imputed</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/base/readRDS.html'>readRDS</a></span><span class='op'>(</span><span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"4.data_for_analysis"</span>, <span class='st'>"data_pollutants_not_imputed.rds"</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># merge with matched data</span>
<span class='va'>data_missing</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_matched</span>, <span class='va'>data_not_imputed</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>

<span class='co'># display proportion of missing values</span>
<span class='va'>data_missing</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>not_imputed_mean_pm25</span><span class='op'>:</span><span class='va'>not_imputed_mean_pm10</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"Pollutant"</span>,
    values_to <span class='op'>=</span> <span class='st'>"concentration"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>Pollutant</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span><span class='st'>"Missing Proportion (%)"</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/Round.html'>round</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span><span class='op'>(</span><span class='va'>concentration</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>/</span>
                                               <span class='fu'>n</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>*</span> <span class='fl'>100</span>, <span class='fl'>1</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>knitr</span><span class='fu'>::</span><span class='fu'><a href='https://rdrr.io/pkg/knitr/man/kable.html'>kable</a></span><span class='op'>(</span><span class='va'>.</span>, align <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"l"</span>, <span class='st'>"c"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>


|Pollutant             | Missing Proportion (%) |
|:---------------------|:----------------------:|
|not_imputed_mean_no2  |          16.6          |
|not_imputed_mean_o3   |          10.7          |
|not_imputed_mean_pm10 |          10.1          |
|not_imputed_mean_pm25 |          28.4          |

</div>



<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load non-imputed data</span>
<span class='va'>data_not_imputed</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/base/readRDS.html'>readRDS</a></span><span class='op'>(</span><span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"4.data_for_analysis"</span>, <span class='st'>"data_pollutants_not_imputed.rds"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>not_imputed_mean_pm25</span><span class='op'>)</span>

<span class='co'># we first define data_not_imputed_leads and data_not_imputed_lags</span>
<span class='co'># to store leads and lags</span>

<span class='va'>data_not_imputed_leads</span> <span class='op'>&lt;-</span> <span class='va'>data_not_imputed</span>
<span class='va'>data_not_imputed_lags</span> <span class='op'>&lt;-</span> <span class='va'>data_not_imputed</span>

<span class='co'>#</span>
<span class='co'># create leads</span>
<span class='co'># </span>

<span class='co'># create a list to store dataframe of leads</span>
<span class='va'>leads_list</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/vector.html'>vector</a></span><span class='op'>(</span>mode <span class='op'>=</span> <span class='st'>"list"</span>, length <span class='op'>=</span> <span class='fl'>1</span><span class='op'>)</span>
<span class='fu'><a href='https://rdrr.io/r/base/names.html'>names</a></span><span class='op'>(</span><span class='va'>leads_list</span><span class='op'>)</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='fl'>1</span><span class='op'>)</span> 

<span class='co'># create the leads</span>
<span class='kw'>for</span><span class='op'>(</span><span class='va'>i</span> <span class='kw'>in</span> <span class='fl'>1</span><span class='op'>)</span><span class='op'>{</span>
  <span class='va'>leads_list</span><span class='op'>[[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>]</span> <span class='op'>&lt;-</span> <span class='va'>data_not_imputed_leads</span> <span class='op'>%&gt;%</span>
    <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span>  <span class='fu'>lead</span><span class='op'>(</span><span class='va'>.</span>, n <span class='op'>=</span> <span class='va'>i</span>, order_by <span class='op'>=</span> <span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
    <span class='fu'>rename_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>,<span class='kw'>function</span><span class='op'>(</span><span class='va'>x</span><span class='op'>)</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste0</a></span><span class='op'>(</span><span class='va'>x</span>,<span class='st'>"_lead_"</span>, <span class='va'>i</span><span class='op'>)</span><span class='op'>)</span>
<span class='op'>}</span>

<span class='co'># merge the dataframes of leads</span>
<span class='va'>data_leads</span> <span class='op'>&lt;-</span> <span class='va'>leads_list</span> <span class='op'>%&gt;%</span>
  <span class='fu'>reduce</span><span class='op'>(</span><span class='va'>left_join</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>

<span class='co'># merge the leads with the data_not_imputed_leads</span>
<span class='va'>data_not_imputed_leads</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_not_imputed_leads</span>, <span class='va'>data_leads</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>not_imputed_mean_no2</span><span class='op'>:</span><span class='va'>not_imputed_mean_pm10</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'>#</span>
<span class='co'># create lags</span>
<span class='co'># </span>

<span class='co'># create a list to store dataframe of lags</span>
<span class='va'>lags_list</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/vector.html'>vector</a></span><span class='op'>(</span>mode <span class='op'>=</span> <span class='st'>"list"</span>, length <span class='op'>=</span> <span class='fl'>1</span><span class='op'>)</span>
<span class='fu'><a href='https://rdrr.io/r/base/names.html'>names</a></span><span class='op'>(</span><span class='va'>lags_list</span><span class='op'>)</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='fl'>1</span><span class='op'>)</span> 

<span class='co'># create the lags</span>
<span class='kw'>for</span><span class='op'>(</span><span class='va'>i</span> <span class='kw'>in</span> <span class='fl'>1</span><span class='op'>)</span><span class='op'>{</span>
  <span class='va'>lags_list</span><span class='op'>[[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>]</span> <span class='op'>&lt;-</span> <span class='va'>data_not_imputed_lags</span> <span class='op'>%&gt;%</span>
    <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>, <span class='op'>~</span>  <span class='fu'><a href='https://rdrr.io/r/stats/lag.html'>lag</a></span><span class='op'>(</span><span class='va'>.</span>, n <span class='op'>=</span> <span class='va'>i</span>, order_by <span class='op'>=</span> <span class='va'>date</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
    <span class='fu'>rename_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='op'>-</span><span class='va'>date</span><span class='op'>)</span>,<span class='kw'>function</span><span class='op'>(</span><span class='va'>x</span><span class='op'>)</span> <span class='fu'><a href='https://rdrr.io/r/base/paste.html'>paste0</a></span><span class='op'>(</span><span class='va'>x</span>,<span class='st'>"_lag_"</span>, <span class='va'>i</span><span class='op'>)</span><span class='op'>)</span>
<span class='op'>}</span>

<span class='co'># merge the dataframes of lags</span>
<span class='va'>data_lags</span> <span class='op'>&lt;-</span> <span class='va'>lags_list</span> <span class='op'>%&gt;%</span>
  <span class='fu'>reduce</span><span class='op'>(</span><span class='va'>left_join</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>

<span class='co'># merge the lags with the initial data_not_imputed_lags</span>
<span class='va'>data_not_imputed_lags</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_not_imputed_lags</span>, <span class='va'>data_lags</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>

<span class='co'>#</span>
<span class='co'># merge data_not_imputed_leads with data_not_imputed_lags</span>
<span class='co'>#</span>

<span class='va'>data_not_imputed</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_not_imputed_lags</span>, <span class='va'>data_not_imputed_leads</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>
</code></pre></div>

</div>



We merge these data with the matched data and compute pair differences:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># merge with the matched_data</span>
<span class='va'>data_matched_with_missing_pollutants</span> <span class='op'>&lt;-</span> <span class='fu'>left_join</span><span class='op'>(</span><span class='va'>data_matched</span>, <span class='va'>data_not_imputed</span>, by <span class='op'>=</span> <span class='st'>"date"</span><span class='op'>)</span>

<span class='co'># compute pair differences</span>
<span class='va'>data_matched_wide_missing_pollutants</span> <span class='op'>&lt;-</span> <span class='va'>data_matched_with_missing_pollutants</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>is_treated <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='va'>is_treated</span> <span class='op'>==</span> <span class='cn'>TRUE</span>, <span class='st'>"treated"</span>, <span class='st'>"control"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>is_treated</span>, <span class='va'>pair_number</span>, <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"not_imputed_mean_no2"</span><span class='op'>)</span>, <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"not_imputed_mean_o3"</span><span class='op'>)</span>, <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"not_imputed_mean_pm10"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>cols <span class='op'>=</span> <span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>pair_number</span>, <span class='va'>is_treated</span><span class='op'>)</span>, names_to <span class='op'>=</span> <span class='st'>"variable"</span>, values_to <span class='op'>=</span> <span class='st'>"concentration"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>pollutant <span class='op'>=</span> <span class='cn'>NA</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"no2"</span><span class='op'>)</span>, <span class='st'>"NO2"</span>,<span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"o3"</span><span class='op'>)</span>, <span class='st'>"O3"</span>,<span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"pm10"</span><span class='op'>)</span>, <span class='st'>"PM10"</span>,<span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>time <span class='op'>=</span> <span class='fl'>0</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"lag_1"</span><span class='op'>)</span>, <span class='op'>-</span><span class='fl'>1</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"lead_1"</span><span class='op'>)</span>, <span class='fl'>1</span>, <span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>variable</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>pair_number</span>, <span class='va'>is_treated</span>, <span class='va'>pollutant</span>, <span class='va'>time</span>, <span class='va'>concentration</span><span class='op'>)</span> <span class='op'>%&gt;%</span> 
  <span class='fu'>pivot_wider</span><span class='op'>(</span>names_from <span class='op'>=</span> <span class='va'>is_treated</span>, values_from <span class='op'>=</span> <span class='va'>concentration</span><span class='op'>)</span>

<span class='va'>data_missing_pair_difference_pollutant</span> <span class='op'>&lt;-</span> <span class='va'>data_matched_wide_missing_pollutants</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>difference <span class='op'>=</span> <span class='va'>treated</span><span class='op'>-</span><span class='va'>control</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>treated</span>, <span class='va'>control</span><span class='op'>)</span><span class='op'>)</span> 
</code></pre></div>

</div>


We display below the proportion of missing differences by pollutant and day:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># make the graph</span>
<span class='va'>graph_missing_pollutants</span> <span class='op'>&lt;-</span> <span class='va'>data_missing_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>proportion_missing <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/NA.html'>is.na</a></span><span class='op'>(</span><span class='va'>difference</span><span class='op'>)</span><span class='op'>)</span><span class='op'>/</span><span class='fu'>n</span><span class='op'>(</span><span class='op'>)</span><span class='op'>*</span><span class='fl'>100</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>.</span>, <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>proportion_missing</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_segment</span><span class='op'>(</span><span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, xend <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='fl'>0</span>, yend <span class='op'>=</span> <span class='va'>proportion_missing</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_point</span><span class='op'>(</span>shape <span class='op'>=</span> <span class='fl'>21</span>, size <span class='op'>=</span> <span class='fl'>4</span>, colour <span class='op'>=</span> <span class='st'>"black"</span>, fill <span class='op'>=</span> <span class='va'>my_blue</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'><a href='https://rdrr.io/r/graphics/plot.window.html'>ylim</a></span><span class='op'>(</span><span class='fl'>0</span>, <span class='fl'>100</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span><span class='op'>~</span> <span class='va'>pollutant</span>, ncol <span class='op'>=</span> <span class='fl'>4</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Proportion of Pairs with \nMissing Concentrations (%)"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span>
  
<span class='co'># display the graph</span>
<span class='va'>graph_missing_pollutants</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-17-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_missing_pollutants</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span>
    <span class='st'>"3.outputs"</span>,
    <span class='st'>"2.matching_analysis"</span>,
    <span class='st'>"graph_missing_pollutants.pdf"</span>
  <span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>20</span>,
  height <span class='op'>=</span> <span class='fl'>10</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>


Clearly, many missing concentrations in the matched pairs were imputed. We compute the point estimates and confidence intervals for the pairs without missing values. Here, we use a Wilcoxon's test to also make the results less sensitive to potential outliers:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># carry out the wilcox.test </span>
<span class='va'>data_missing_rank_ci</span> <span class='op'>&lt;-</span> <span class='va'>data_missing_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'>drop_na</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span> <span class='va'>pair_number</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>nest</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>mean_difference <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>data</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/stats/wilcox.test.html'>wilcox.test</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>$</span><span class='va'>difference</span>, conf.int <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>$</span><span class='va'>estimate</span><span class='op'>)</span>,
         lower_bound_95 <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>data</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/stats/wilcox.test.html'>wilcox.test</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>$</span><span class='va'>difference</span>, conf.int <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>$</span><span class='va'>conf.int</span><span class='op'>[</span><span class='fl'>1</span><span class='op'>]</span><span class='op'>)</span>,
         upper_bound_95 <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>data</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/stats/wilcox.test.html'>wilcox.test</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>$</span><span class='va'>difference</span>, conf.int <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span><span class='op'>$</span><span class='va'>conf.int</span><span class='op'>[</span><span class='fl'>2</span><span class='op'>]</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>unnest</span><span class='op'>(</span>cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>mean_difference</span>, <span class='va'>lower_bound_95</span>, <span class='va'>upper_bound_95</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>data <span class='op'>=</span> <span class='st'>"Pairs without Missing Concentrations"</span><span class='op'>)</span>

<span class='co'># bind with analysis on imputed concentrations</span>
<span class='va'>data_imputed_nonimputed</span> <span class='op'>&lt;-</span> <span class='va'>data_neyman</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>data <span class='op'>=</span> <span class='st'>"Pairs with Imputed Concentrations"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>pollutant</span>,
         <span class='va'>time</span>,
         <span class='va'>mean_difference</span>,
         <span class='va'>lower_bound_95</span>,
         <span class='va'>upper_bound_95</span>,
         <span class='va'>data</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>bind_rows</span><span class='op'>(</span><span class='va'>.</span>,  <span class='va'>data_missing_rank_ci</span><span class='op'>)</span>

<span class='co'># create an indicator to alternate shading of confidence intervals</span>
<span class='va'>data_imputed_nonimputed</span> <span class='op'>&lt;-</span> <span class='va'>data_imputed_nonimputed</span> <span class='op'>%&gt;%</span>
  <span class='fu'>arrange</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>stripe <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='op'>(</span><span class='va'>time</span> <span class='op'>%%</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>==</span> <span class='fl'>0</span>, <span class='st'>"Grey"</span>, <span class='st'>"White"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># make the graph</span>
<span class='va'>graph_ci_missing</span> <span class='op'>&lt;-</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>data_imputed_nonimputed</span>,
         <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>mean_difference</span>, ymin <span class='op'>=</span> <span class='va'>lower_bound_95</span>, ymax <span class='op'>=</span> <span class='va'>upper_bound_95</span>, colour <span class='op'>=</span> <span class='va'>data</span>, shape <span class='op'>=</span> <span class='va'>data</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_rect</span><span class='op'>(</span>
    <span class='fu'>aes</span><span class='op'>(</span>fill <span class='op'>=</span> <span class='va'>stripe</span><span class='op'>)</span>,
    xmin <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>data_imputed_nonimputed</span><span class='op'>$</span><span class='va'>time</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>-</span> <span class='fl'>0.42</span>,
    xmax <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>data_imputed_nonimputed</span><span class='op'>$</span><span class='va'>time</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span> <span class='fl'>0.42</span>,
    ymin <span class='op'>=</span> <span class='op'>-</span><span class='cn'>Inf</span>,
    ymax <span class='op'>=</span> <span class='cn'>Inf</span>,
    color <span class='op'>=</span> <span class='cn'>NA</span>,
    alpha <span class='op'>=</span> <span class='fl'>0.4</span>
  <span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_hline</span><span class='op'>(</span>yintercept <span class='op'>=</span> <span class='fl'>0</span>,
             color <span class='op'>=</span> <span class='st'>"black"</span>,
             size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_vline</span><span class='op'>(</span>xintercept <span class='op'>=</span> <span class='st'>"0"</span>,
             color <span class='op'>=</span> <span class='st'>"black"</span>,
             size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_pointrange</span><span class='op'>(</span>position <span class='op'>=</span> <span class='fu'>position_dodge</span><span class='op'>(</span>width <span class='op'>=</span> <span class='fl'>1</span><span class='op'>)</span>, size <span class='op'>=</span> <span class='fl'>1.2</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_shape_manual</span><span class='op'>(</span>name <span class='op'>=</span> <span class='st'>"Dataset:"</span>, values <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='fl'>16</span>, <span class='fl'>17</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_color_manual</span><span class='op'>(</span>name <span class='op'>=</span> <span class='st'>"Dataset:"</span>, values <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>my_orange</span>, <span class='va'>my_blue</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span> <span class='op'>~</span> <span class='va'>pollutant</span>, ncol <span class='op'>=</span> <span class='fl'>4</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_fill_manual</span><span class='op'>(</span>values <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'gray96'</span>, <span class='st'>"white"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>guides</span><span class='op'>(</span>fill <span class='op'>=</span> <span class='cn'>FALSE</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Average Increase in\n Concentrations (µg/m³)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme</span><span class='op'>(</span>axis.text.x <span class='op'>=</span> <span class='fu'>element_text</span><span class='op'>(</span>margin <span class='op'>=</span> <span class='fu'>ggplot2</span><span class='fu'>::</span><span class='fu'><a href='https://ggplot2.tidyverse.org/reference/element.html'>margin</a></span><span class='op'>(</span>t <span class='op'>=</span> <span class='fl'>0</span>, unit <span class='op'>=</span> <span class='st'>"cm"</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>


<span class='co'># print the graph</span>
<span class='va'>graph_ci_missing</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-18-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_ci_missing</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"3.outputs"</span>, <span class='st'>"2.matching_analysis"</span>, <span class='st'>"graph_ci_missing.pdf"</span><span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>16</span>,
  height <span class='op'>=</span> <span class='fl'>8</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>

  
Results seem relatively robust to the imputation of missing values.

### Are results on PM$_{10}$ sensitive to hidden bias?

To assess whether the effects of North-East winds on PM$_{10}$ concentrations could be due to hidden bias, we implement the studentized sensitivity analysis for the ATE developed by Colin B. Fogarty (2019). We first load the relevant functions:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load fogarty's studentized Sensitivity Analysis functions</span>
<span class='co'># retrieved from http://www.mit.edu/~cfogarty/StudentizedSensitivity.R</span>

<span class='co'>#' StudentizedSensitivity</span>
<span class='co'>#'Function to perform a Studentized Sensitivity analysis on the sample average treatment</span>
<span class='co'>#'effect in a paired observational study</span>
<span class='co'>#'</span>
<span class='co'>#' @param PairedDiff: Vector of treated-minus-control paired differences.</span>
<span class='co'>#' @param null: Value of the sample average treatment effect under the null.</span>
<span class='co'>#' @param alpha: Desired Type I error rate.</span>
<span class='co'>#' @param alternative: Can be "less", "greater", or "two.sided".</span>
<span class='co'>#' @param Gamma: Vector of values for Gamma at which to perform the sensitivity</span>
<span class='co'>#'  analysis.</span>
<span class='co'>#' @param nperm: Number of permutations to perform permutation test.</span>
<span class='co'>#' @param Changepoint: If true, function returns the maximal Gamma for which the</span>
<span class='co'>#' test rejects at level alpha.</span>
<span class='co'>#' @param SensitivityInterval: If true, function returns (100-alpha) sensitivity</span>
<span class='co'>#' intervals. They will be one-sided if the alternative is less than or greater than,</span>
<span class='co'>#' and two-sided if the alternative is two-sided.</span>
<span class='co'>#'</span>
<span class='co'>#' @return Gamma: Vector of Gammas for which the sensitivity analysis was performed.</span>
<span class='co'>#' @return pval: P-values for each value of Gamma.</span>
<span class='co'>#' @return GammaPval: Matrix combining Gamma and pval.</span>
<span class='co'>#' @return Changepoint: Maximal Gamma for which the test rejected at level alpha.</span>
<span class='co'>#' @return SensitivityInterval: Upper and lower bounds for 100(1-alpha) sensitivity</span>
<span class='co'>#' intervals for each value of Gamma.</span>
<span class='co'>#' @export</span>


<span class='va'>StudentizedSensitivity</span> <span class='op'>=</span> <span class='kw'>function</span><span class='op'>(</span><span class='va'>PairedDiff</span>, <span class='va'>null</span> <span class='op'>=</span> <span class='fl'>0</span>, <span class='va'>alpha</span> <span class='op'>=</span> <span class='fl'>0.05</span>, <span class='va'>alternative</span> <span class='op'>=</span> <span class='st'>"greater"</span>, <span class='va'>Gamma</span> <span class='op'>=</span> <span class='fl'>1</span>, <span class='va'>nperm</span> <span class='op'>=</span> <span class='fl'>50000</span>, <span class='va'>Changepoint</span> <span class='op'>=</span> <span class='cn'>T</span>, <span class='va'>SensitivityInterval</span> <span class='op'>=</span> <span class='cn'>T</span><span class='op'>)</span>
<span class='op'>{</span>
   <span class='kw'>if</span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/any.html'>any</a></span><span class='op'>(</span><span class='va'>Gamma</span> <span class='op'>&lt;</span> <span class='fl'>1</span><span class='op'>)</span><span class='op'>)</span>
   <span class='op'>{</span>
     <span class='kw'><a href='https://rdrr.io/r/base/stop.html'>stop</a></span><span class='op'>(</span><span class='st'>"Values for Gamma must be &gt;= 1"</span><span class='op'>)</span>
   <span class='op'>}</span>
   <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span><span class='op'>!=</span><span class='st'>"less"</span> <span class='op'>&amp;</span> <span class='va'>alternative</span><span class='op'>!=</span> <span class='st'>"greater"</span> <span class='op'>&amp;</span> <span class='va'>alternative</span> <span class='op'>!=</span> <span class='st'>"two.sided"</span><span class='op'>)</span>
   <span class='op'>{</span>
     <span class='kw'><a href='https://rdrr.io/r/base/stop.html'>stop</a></span><span class='op'>(</span><span class='st'>"Values for alternative are `less', `greater', or `two.sided'"</span><span class='op'>)</span>
   <span class='op'>}</span>
   <span class='kw'>if</span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>null</span><span class='op'>)</span> <span class='op'>&gt;</span> <span class='fl'>1</span><span class='op'>)</span>
   <span class='op'>{</span>
     <span class='kw'><a href='https://rdrr.io/r/base/stop.html'>stop</a></span><span class='op'>(</span><span class='st'>"Value under the null must be a scalar"</span><span class='op'>)</span>
   <span class='op'>}</span>
      <span class='kw'>if</span><span class='op'>(</span><span class='va'>alpha</span> <span class='op'>&lt;</span> <span class='fl'>0</span> <span class='op'>|</span> <span class='va'>alpha</span> <span class='op'>&gt;</span> <span class='fl'>0.5</span><span class='op'>)</span>
   <span class='op'>{</span>
     <span class='kw'><a href='https://rdrr.io/r/base/stop.html'>stop</a></span><span class='op'>(</span><span class='st'>"alpha must be between 0 and 0.5"</span><span class='op'>)</span>
      <span class='op'>}</span>

  <span class='va'>PairedDifftrue</span> <span class='op'>&lt;-</span> <span class='va'>PairedDiff</span>
  <span class='va'>alphatrue</span> <span class='op'>&lt;-</span> <span class='va'>alpha</span>
  <span class='va'>I</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>PairedDiff</span><span class='op'>)</span>
  <span class='va'>Adjust</span> <span class='op'>&lt;-</span> <span class='va'>PairedDiff</span> <span class='op'>-</span> <span class='va'>null</span>

  <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span> <span class='op'>==</span> <span class='st'>"less"</span><span class='op'>)</span>
  <span class='op'>{</span>
    <span class='va'>Adjust</span> <span class='op'>&lt;-</span> <span class='op'>-</span><span class='va'>Adjust</span>
  <span class='op'>}</span>
  <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span> <span class='op'>==</span> <span class='st'>"two.sided"</span><span class='op'>)</span>
  <span class='op'>{</span>
    <span class='va'>alpha</span> <span class='op'>&lt;-</span> <span class='va'>alphatrue</span><span class='op'>/</span><span class='fl'>2</span>

    <span class='kw'>if</span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>Adjust</span><span class='op'>)</span> <span class='op'>&lt;</span> <span class='fl'>0</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='va'>Adjust</span> <span class='op'>&lt;-</span> <span class='op'>-</span><span class='va'>Adjust</span>
    <span class='op'>}</span>
  <span class='op'>}</span>

  <span class='va'>pval</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span><span class='op'>(</span><span class='fl'>0</span>, <span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>Gamma</span><span class='op'>)</span><span class='op'>)</span>

  <span class='kw'>for</span><span class='op'>(</span><span class='va'>i</span> <span class='kw'>in</span> <span class='fl'>1</span><span class='op'>:</span><span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>Gamma</span><span class='op'>)</span><span class='op'>)</span>

  <span class='op'>{</span>
  <span class='va'>D</span> <span class='op'>&lt;-</span> <span class='op'>(</span><span class='va'>Adjust</span><span class='op'>)</span> <span class='op'>-</span> <span class='op'>(</span><span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span><span class='op'>/</span><span class='op'>(</span><span class='fl'>1</span><span class='op'>+</span><span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>)</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>abs</a></span><span class='op'>(</span><span class='va'>Adjust</span><span class='op'>)</span>
  <span class='va'>obs</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>D</span><span class='op'>)</span><span class='op'>/</span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>D</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span><span class='op'>)</span>
  <span class='va'>Adjmat</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/matrix.html'>matrix</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>abs</a></span><span class='op'>(</span><span class='va'>Adjust</span><span class='op'>)</span>, <span class='va'>I</span>, <span class='va'>nperm</span><span class='op'>)</span>
  <span class='va'>Zmat</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/matrix.html'>matrix</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/stats/Uniform.html'>runif</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>*</span><span class='va'>nperm</span><span class='op'>)</span> <span class='op'>&lt;</span> <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>/</span><span class='op'>(</span><span class='fl'>1</span><span class='op'>+</span><span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>)</span>, <span class='va'>I</span>, <span class='va'>nperm</span><span class='op'>)</span>
  <span class='va'>Dmat</span> <span class='op'>&lt;-</span> <span class='op'>(</span><span class='fl'>2</span><span class='op'>*</span><span class='va'>Zmat</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span><span class='op'>*</span><span class='op'>(</span><span class='va'>Adjmat</span><span class='op'>)</span> <span class='op'>-</span> <span class='op'>(</span><span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span><span class='op'>/</span><span class='op'>(</span><span class='fl'>1</span><span class='op'>+</span><span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span><span class='op'>)</span><span class='op'>*</span><span class='va'>Adjmat</span>
  <span class='va'>perm</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/colSums.html'>colMeans</a></span><span class='op'>(</span><span class='va'>Dmat</span><span class='op'>)</span><span class='op'>/</span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='fu'>colVars</span><span class='op'>(</span><span class='va'>Dmat</span><span class='op'>)</span><span class='op'>/</span><span class='va'>I</span><span class='op'>)</span><span class='op'>)</span>
  <span class='va'>pval</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span> <span class='op'>&lt;-</span> <span class='op'>(</span><span class='fl'>1</span><span class='op'>+</span><span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='va'>perm</span><span class='op'>&gt;=</span><span class='va'>obs</span><span class='op'>)</span><span class='op'>)</span><span class='op'>/</span><span class='op'>(</span><span class='va'>nperm</span> <span class='op'>+</span> <span class='fl'>1</span><span class='op'>)</span>
  <span class='op'>}</span>
  <span class='va'>pvalret</span> <span class='op'>=</span> <span class='va'>pval</span>
  <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span> <span class='op'>==</span> <span class='st'>"two.sided"</span><span class='op'>)</span>
  <span class='op'>{</span>
    <span class='va'>pvalret</span> <span class='op'>=</span> <span class='fl'>2</span><span class='op'>*</span><span class='va'>pval</span>
  <span class='op'>}</span>
  <span class='va'>Pmatrix</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/cbind.html'>cbind</a></span><span class='op'>(</span><span class='va'>Gamma</span>, <span class='va'>pvalret</span><span class='op'>)</span>
  <span class='fu'><a href='https://rdrr.io/r/base/colnames.html'>colnames</a></span><span class='op'>(</span><span class='va'>Pmatrix</span><span class='op'>)</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"Gamma"</span>, <span class='st'>"P-value"</span><span class='op'>)</span>

  <span class='kw'>if</span><span class='op'>(</span><span class='va'>Changepoint</span> <span class='op'>==</span> <span class='cn'>T</span><span class='op'>)</span>
  <span class='op'>{</span>
    <span class='va'>proceed</span> <span class='op'>&lt;-</span> <span class='fu'>StudentizedSensitivity</span><span class='op'>(</span><span class='va'>PairedDifftrue</span>, <span class='va'>null</span>, <span class='va'>alphatrue</span>, <span class='va'>alternative</span>, Gamma<span class='op'>=</span><span class='fl'>1</span>, <span class='va'>nperm</span>,
                                      Changepoint <span class='op'>=</span> <span class='cn'>F</span>, SensitivityInterval <span class='op'>=</span> <span class='cn'>F</span><span class='op'>)</span><span class='op'>$</span><span class='va'>pval</span> <span class='op'>&lt;=</span> <span class='va'>alphatrue</span>

    <span class='va'>change</span> <span class='op'>&lt;-</span> <span class='fl'>1</span>

    <span class='kw'>if</span><span class='op'>(</span><span class='va'>proceed</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='va'>change</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>StudentizedChangepoint</span>, interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='fl'>1</span>, <span class='fl'>30</span><span class='op'>)</span>, PairedDiff <span class='op'>=</span> <span class='va'>PairedDifftrue</span>, null <span class='op'>=</span> <span class='va'>null</span>,
                        alpha <span class='op'>=</span> <span class='va'>alphatrue</span>, alternative <span class='op'>=</span> <span class='va'>alternative</span>, nperm <span class='op'>=</span> <span class='va'>nperm</span>,
                        extendInt <span class='op'>=</span> <span class='st'>"upX"</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>
    <span class='op'>}</span>
  <span class='op'>}</span>

  <span class='kw'>if</span><span class='op'>(</span><span class='va'>SensitivityInterval</span> <span class='op'>==</span> <span class='cn'>T</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='va'>lb</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span><span class='op'>(</span><span class='op'>-</span><span class='cn'>Inf</span>, <span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>Gamma</span><span class='op'>)</span><span class='op'>)</span>
      <span class='va'>ub</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span><span class='op'>(</span><span class='cn'>Inf</span>, <span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>Gamma</span><span class='op'>)</span><span class='op'>)</span>
      <span class='kw'>for</span><span class='op'>(</span><span class='va'>i</span> <span class='kw'>in</span> <span class='fl'>1</span><span class='op'>:</span><span class='fu'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='op'>(</span><span class='va'>Gamma</span><span class='op'>)</span><span class='op'>)</span>
      <span class='op'>{</span>
        <span class='co'># Warm Starts</span>
      <span class='va'>UB</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>BoundFinder</span>, <span class='va'>PairedDifftrue</span>, <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span>,
                   interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span>, <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>+</span><span class='fl'>4</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span><span class='op'>)</span>, extendInt <span class='op'>=</span> <span class='st'>"yes"</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>
      <span class='va'>LB</span> <span class='op'>=</span> <span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>BoundFinder</span>, <span class='op'>-</span><span class='va'>PairedDifftrue</span>, <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span>,
                    interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>-</span><span class='fl'>4</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span>, <span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>)</span>, extendInt <span class='op'>=</span> <span class='st'>"yes"</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>

      <span class='va'>SUB</span> <span class='op'>=</span> <span class='cn'>Inf</span>
      <span class='va'>SLB</span> <span class='op'>=</span> <span class='op'>-</span><span class='cn'>Inf</span>

      <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span> <span class='op'>==</span> <span class='st'>"greater"</span><span class='op'>)</span>
      <span class='op'>{</span>
        <span class='va'>SLB</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>StudentizedSI</span>, interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>UB</span><span class='op'>-</span><span class='fl'>4</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span>, <span class='va'>UB</span><span class='op'>)</span>, extendInt <span class='op'>=</span> <span class='st'>"yes"</span>,
                      Gamma <span class='op'>=</span> <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span>, PairedDiff<span class='op'>=</span><span class='va'>PairedDifftrue</span>, alternative <span class='op'>=</span> <span class='st'>"greater"</span>, alpha <span class='op'>=</span> <span class='va'>alpha</span>, nperm <span class='op'>=</span> <span class='va'>nperm</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>
      <span class='op'>}</span>

      <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span> <span class='op'>==</span> <span class='st'>"less"</span><span class='op'>)</span>
      <span class='op'>{</span>
        <span class='va'>SUB</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>StudentizedSI</span>, interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>LB</span>, <span class='va'>LB</span> <span class='op'>+</span> <span class='fl'>4</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span><span class='op'>)</span>, extendInt <span class='op'>=</span> <span class='st'>"yes"</span>,
                      Gamma <span class='op'>=</span> <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span>, PairedDiff<span class='op'>=</span><span class='va'>PairedDifftrue</span>, alternative <span class='op'>=</span> <span class='st'>"less"</span>, alpha <span class='op'>=</span> <span class='va'>alpha</span>, nperm <span class='op'>=</span> <span class='va'>nperm</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>
      <span class='op'>}</span>

      <span class='kw'>if</span><span class='op'>(</span><span class='va'>alternative</span> <span class='op'>==</span> <span class='st'>"two.sided"</span><span class='op'>)</span>
      <span class='op'>{</span>
       <span class='va'>SLB</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>StudentizedSI</span>, interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>UB</span><span class='op'>-</span><span class='fl'>4</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span>, <span class='va'>UB</span><span class='op'>)</span>, extendInt <span class='op'>=</span> <span class='st'>"yes"</span>,
                     Gamma <span class='op'>=</span> <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span>, PairedDiff<span class='op'>=</span><span class='va'>PairedDifftrue</span>, alternative <span class='op'>=</span> <span class='st'>"greater"</span>, alpha <span class='op'>=</span> <span class='va'>alpha</span>, nperm <span class='op'>=</span> <span class='va'>nperm</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>
       <span class='va'>SUB</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/stats/uniroot.html'>uniroot</a></span><span class='op'>(</span><span class='va'>StudentizedSI</span>, interval <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>LB</span>, <span class='va'>LB</span><span class='op'>+</span><span class='fl'>4</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/stats/sd.html'>sd</a></span><span class='op'>(</span><span class='va'>PairedDifftrue</span><span class='op'>)</span><span class='op'>/</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>I</span><span class='op'>)</span><span class='op'>)</span>, extendInt <span class='op'>=</span> <span class='st'>"yes"</span>,
                     Gamma <span class='op'>=</span> <span class='va'>Gamma</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span>, PairedDiff<span class='op'>=</span><span class='va'>PairedDifftrue</span>, alternative <span class='op'>=</span> <span class='st'>"less"</span>, alpha <span class='op'>=</span> <span class='va'>alpha</span>, nperm <span class='op'>=</span> <span class='va'>nperm</span><span class='op'>)</span><span class='op'>$</span><span class='va'>root</span>
      <span class='op'>}</span>

      <span class='va'>lb</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span> <span class='op'>=</span> <span class='va'>SLB</span>
      <span class='va'>ub</span><span class='op'>[</span><span class='va'>i</span><span class='op'>]</span> <span class='op'>=</span> <span class='va'>SUB</span>
      <span class='op'>}</span>

    <span class='va'>SImat</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/cbind.html'>cbind</a></span><span class='op'>(</span><span class='va'>Gamma</span>, <span class='va'>lb</span>, <span class='va'>ub</span><span class='op'>)</span>
    <span class='fu'><a href='https://rdrr.io/r/base/colnames.html'>colnames</a></span><span class='op'>(</span><span class='va'>SImat</span><span class='op'>)</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"Gamma"</span>, <span class='st'>"Lower Bound"</span>, <span class='st'>"Upper Bound"</span><span class='op'>)</span>
    <span class='op'>}</span>
    <span class='kw'>if</span><span class='op'>(</span><span class='va'>Changepoint</span> <span class='op'>==</span> <span class='cn'>F</span> <span class='op'>&amp;</span> <span class='va'>SensitivityInterval</span> <span class='op'>==</span> <span class='cn'>F</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='kw'><a href='https://rdrr.io/r/base/function.html'>return</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/list.html'>list</a></span><span class='op'>(</span>Gamma<span class='op'>=</span><span class='va'>Gamma</span>, pval <span class='op'>=</span> <span class='va'>pvalret</span>, GammaPval <span class='op'>=</span> <span class='va'>Pmatrix</span><span class='op'>)</span><span class='op'>)</span>
    <span class='op'>}</span>
    <span class='kw'>if</span><span class='op'>(</span><span class='va'>Changepoint</span> <span class='op'>==</span> <span class='cn'>F</span> <span class='op'>&amp;</span> <span class='va'>SensitivityInterval</span> <span class='op'>==</span> <span class='cn'>T</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='kw'><a href='https://rdrr.io/r/base/function.html'>return</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/list.html'>list</a></span><span class='op'>(</span>Gamma <span class='op'>=</span> <span class='va'>Gamma</span>, pval <span class='op'>=</span> <span class='va'>pvalret</span>, GammaPval <span class='op'>=</span> <span class='va'>Pmatrix</span>, SensitivityInterval <span class='op'>=</span> <span class='va'>SImat</span><span class='op'>)</span><span class='op'>)</span>
    <span class='op'>}</span>
    <span class='kw'>if</span><span class='op'>(</span><span class='va'>Changepoint</span> <span class='op'>==</span> <span class='cn'>T</span> <span class='op'>&amp;</span> <span class='va'>SensitivityInterval</span> <span class='op'>==</span> <span class='cn'>F</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='kw'><a href='https://rdrr.io/r/base/function.html'>return</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/list.html'>list</a></span><span class='op'>(</span>Gamma <span class='op'>=</span> <span class='va'>Gamma</span>, pval <span class='op'>=</span> <span class='va'>pvalret</span>, GammaPval <span class='op'>=</span> <span class='va'>Pmatrix</span>, Changepoint <span class='op'>=</span> <span class='va'>change</span><span class='op'>)</span><span class='op'>)</span>
    <span class='op'>}</span>
    <span class='kw'>if</span><span class='op'>(</span><span class='va'>Changepoint</span> <span class='op'>==</span> <span class='cn'>T</span> <span class='op'>&amp;</span> <span class='va'>SensitivityInterval</span> <span class='op'>==</span> <span class='cn'>T</span><span class='op'>)</span>
    <span class='op'>{</span>
      <span class='kw'><a href='https://rdrr.io/r/base/function.html'>return</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/list.html'>list</a></span><span class='op'>(</span>Gamma <span class='op'>=</span> <span class='va'>Gamma</span>, pval <span class='op'>=</span> <span class='va'>pvalret</span>, GammaPval <span class='op'>=</span> <span class='va'>Pmatrix</span>, Changepoint <span class='op'>=</span> <span class='va'>change</span>,
                  SensitivityInterval <span class='op'>=</span> <span class='va'>SImat</span><span class='op'>)</span><span class='op'>)</span>
    <span class='op'>}</span>

<span class='op'>}</span>

<span class='co'>####These are auxiliary functions used for root finding and for calculating columnwise variances in StudentizedSensitivity</span>
<span class='va'>StudentizedChangepoint</span> <span class='op'>=</span> <span class='kw'>function</span><span class='op'>(</span><span class='va'>Gamma</span>, <span class='va'>PairedDiff</span>, <span class='va'>null</span>, <span class='va'>alternative</span>, <span class='va'>alpha</span>, <span class='va'>nperm</span><span class='op'>)</span>
<span class='op'>{</span>
  <span class='va'>alphachange</span> <span class='op'>=</span> <span class='va'>alpha</span>
  <span class='fu'>StudentizedSensitivity</span><span class='op'>(</span><span class='va'>PairedDiff</span>, <span class='va'>null</span>, <span class='va'>alpha</span>, <span class='va'>alternative</span>, <span class='va'>Gamma</span>, <span class='va'>nperm</span>, Changepoint <span class='op'>=</span> <span class='cn'>F</span>, SensitivityInterval <span class='op'>=</span> <span class='cn'>F</span><span class='op'>)</span><span class='op'>$</span><span class='va'>pval</span> <span class='op'>-</span> <span class='va'>alphachange</span>
<span class='op'>}</span>

<span class='va'>StudentizedSI</span> <span class='op'>=</span> <span class='kw'>function</span><span class='op'>(</span><span class='va'>null</span>,  <span class='va'>Gamma</span>, <span class='va'>PairedDiff</span>,  <span class='va'>alternative</span>, <span class='va'>alpha</span>, <span class='va'>nperm</span><span class='op'>)</span>
<span class='op'>{</span>
  <span class='fu'>StudentizedSensitivity</span><span class='op'>(</span><span class='va'>PairedDiff</span>, <span class='va'>null</span>, <span class='va'>alpha</span>, <span class='va'>alternative</span>, <span class='va'>Gamma</span>, <span class='va'>nperm</span>, Changepoint <span class='op'>=</span> <span class='cn'>F</span>, SensitivityInterval <span class='op'>=</span> <span class='cn'>F</span><span class='op'>)</span><span class='op'>$</span><span class='va'>pval</span> <span class='op'>-</span> <span class='va'>alpha</span>
<span class='op'>}</span>

<span class='va'>BoundFinder</span> <span class='op'>=</span> <span class='kw'>function</span><span class='op'>(</span><span class='va'>null</span>,  <span class='va'>PairedDiff</span>, <span class='va'>Gamma</span><span class='op'>)</span>
<span class='op'>{</span>
  <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>PairedDiff</span> <span class='op'>-</span> <span class='va'>null</span> <span class='op'>-</span> <span class='op'>(</span><span class='va'>Gamma</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span><span class='op'>/</span><span class='op'>(</span><span class='fl'>1</span><span class='op'>+</span><span class='va'>Gamma</span><span class='op'>)</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>abs</a></span><span class='op'>(</span><span class='va'>PairedDiff</span><span class='op'>-</span><span class='va'>null</span><span class='op'>)</span><span class='op'>)</span>
<span class='op'>}</span>

<span class='va'>colVars</span> <span class='op'>&lt;-</span> <span class='kw'>function</span><span class='op'>(</span><span class='va'>x</span><span class='op'>)</span> <span class='op'>{</span>
  <span class='va'>N</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span><span class='op'>(</span><span class='va'>x</span><span class='op'>)</span>
  <span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/colSums.html'>colSums</a></span><span class='op'>(</span><span class='va'>x</span><span class='op'>^</span><span class='fl'>2</span><span class='op'>)</span> <span class='op'>-</span> <span class='fu'><a href='https://rdrr.io/r/base/colSums.html'>colSums</a></span><span class='op'>(</span><span class='va'>x</span><span class='op'>)</span><span class='op'>^</span><span class='fl'>2</span><span class='op'>/</span><span class='va'>N</span><span class='op'>)</span> <span class='op'>/</span> <span class='op'>(</span><span class='va'>N</span><span class='op'>-</span><span class='fl'>1</span><span class='op'>)</span>
<span class='op'>}</span>
</code></pre></div>

</div>


We select the pair differences for PM$_{10}$ concentrations in $t$ and run the function for $\Gamma$=2:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># we select the relevant pair differences</span>
<span class='va'>PairedDiff</span> <span class='op'>&lt;-</span> <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>pollutant</span> <span class='op'>==</span> <span class='st'>"PM10"</span> <span class='op'>&amp;</span> <span class='va'>time</span> <span class='op'>==</span> <span class='fl'>0</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pull</span><span class='op'>(</span><span class='va'>difference</span><span class='op'>)</span>

<span class='co'># we run the function</span>
<span class='fu'>StudentizedSensitivity</span><span class='op'>(</span>
  <span class='va'>PairedDiff</span>,
  null <span class='op'>=</span> <span class='fl'>0</span>,
  alpha <span class='op'>=</span> <span class='fl'>0.05</span>,
  alternative <span class='op'>=</span> <span class='st'>"two.sided"</span>,
  Gamma <span class='op'>=</span> <span class='fl'>2</span>,
  nperm <span class='op'>=</span> <span class='fl'>50000</span>,
  Changepoint <span class='op'>=</span> <span class='cn'>T</span>,
  SensitivityInterval <span class='op'>=</span> <span class='cn'>T</span>
<span class='op'>)</span><span class='op'>$</span><span class='va'>SensitivityInterval</span> <span class='op'>%&gt;%</span>
  <span class='fu'>as_tibble</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_all</span><span class='op'>(</span><span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/Round.html'>round</a></span><span class='op'>(</span><span class='va'>.</span>, <span class='fl'>2</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/pkg/DT/man/datatable.html'>datatable</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span>
</code></pre></div>

```{=html}
<div id="htmlwidget-05afe28c2c585e170fe5" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-05afe28c2c585e170fe5">{"x":{"filter":"none","data":[["1"],[2],[0.79],[7.77]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Gamma<\/th>\n      <th>Lower Bound<\/th>\n      <th>Upper Bound<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[1,2,3]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

</div>


We then implement the same procedure but for concentrations in $t+1$:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># we select the relevant pair differences</span>
<span class='va'>PairedDiff</span> <span class='op'>&lt;-</span> <span class='va'>data_pair_difference_pollutant</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>pollutant</span> <span class='op'>==</span> <span class='st'>"PM10"</span> <span class='op'>&amp;</span> <span class='va'>time</span> <span class='op'>==</span> <span class='fl'>1</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pull</span><span class='op'>(</span><span class='va'>difference</span><span class='op'>)</span>

<span class='co'># we run the function</span>
<span class='fu'>StudentizedSensitivity</span><span class='op'>(</span>
  <span class='va'>PairedDiff</span>,
  null <span class='op'>=</span> <span class='fl'>0</span>,
  alpha <span class='op'>=</span> <span class='fl'>0.05</span>,
  alternative <span class='op'>=</span> <span class='st'>"two.sided"</span>,
  Gamma <span class='op'>=</span> <span class='fl'>2</span>,
  nperm <span class='op'>=</span> <span class='fl'>50000</span>,
  Changepoint <span class='op'>=</span> <span class='cn'>T</span>,
  SensitivityInterval <span class='op'>=</span> <span class='cn'>T</span>
<span class='op'>)</span><span class='op'>$</span><span class='va'>SensitivityInterval</span> <span class='op'>%&gt;%</span>
  <span class='fu'>as_tibble</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_all</span><span class='op'>(</span> <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/Round.html'>round</a></span><span class='op'>(</span><span class='va'>.</span>, <span class='fl'>2</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/pkg/DT/man/datatable.html'>datatable</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span>
</code></pre></div>

```{=html}
<div id="htmlwidget-262ee2b0cfc56a7340ed" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-262ee2b0cfc56a7340ed">{"x":{"filter":"none","data":[["1"],[2],[1.62],[10.88]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Gamma<\/th>\n      <th>Lower Bound<\/th>\n      <th>Upper Bound<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[1,2,3]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

</div>


### Does pairing improve precision?

To check that our pair matching procedure improves precsision, we compare the estimate of the variance for a pair experiment with the one of a complete experiment (the formula can be found in Imbens & Rubin 2015's textbook):

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># compute estimates of the sampling variability</span>
<span class='co'># for a complete experiment</span>
<span class='va'>sampling_variability_complete</span> <span class='op'>&lt;-</span> <span class='va'>data_matched</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>is_treated</span>, <span class='va'>mean_no2</span>, <span class='va'>mean_o3</span>, <span class='va'>mean_pm10</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>mean_no2</span><span class='op'>:</span><span class='va'>mean_pm10</span><span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"pollutant"</span>,
    values_to <span class='op'>=</span> <span class='st'>"concentration"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>is_treated</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    n_obs <span class='op'>=</span> <span class='fu'>n</span><span class='op'>(</span><span class='op'>)</span>,
    average_concentration <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>concentration</span><span class='op'>)</span>,
    squared_difference <span class='op'>=</span> <span class='op'>(</span><span class='va'>concentration</span> <span class='op'>-</span> <span class='va'>average_concentration</span><span class='op'>)</span> <span class='op'>^</span>
      <span class='fl'>2</span>,
    variance_group <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='va'>squared_difference</span><span class='op'>)</span> <span class='op'>/</span> <span class='op'>(</span><span class='va'>n_obs</span> <span class='op'>-</span> <span class='fl'>1</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>variance_component <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/mean.html'>mean</a></span><span class='op'>(</span><span class='va'>variance_group</span> <span class='op'>/</span> <span class='va'>n_obs</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>summarise</span><span class='op'>(</span>
    variance <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/sum.html'>sum</a></span><span class='op'>(</span><span class='va'>variance_component</span><span class='op'>)</span>,
    standard_error_complete <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='op'>(</span><span class='va'>variance</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>variance</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    pollutant <span class='op'>=</span> <span class='fu'>case_when</span><span class='op'>(</span>
      <span class='va'>pollutant</span> <span class='op'>==</span> <span class='st'>"mean_no2"</span> <span class='op'>~</span> <span class='st'>"NO2"</span>,
      <span class='va'>pollutant</span> <span class='op'>==</span> <span class='st'>"mean_o3"</span> <span class='op'>~</span> <span class='st'>"O3"</span>,
      <span class='va'>pollutant</span> <span class='op'>==</span> <span class='st'>"mean_pm10"</span> <span class='op'>~</span> <span class='st'>"PM10"</span>
    <span class='op'>)</span>
  <span class='op'>)</span>

<span class='co'># estimates of the sampling variability for a pair experiment</span>
<span class='va'>sampling_variability_pair</span> <span class='op'>&lt;-</span> <span class='va'>data_se_neyman_pair</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>time</span> <span class='op'>==</span> <span class='fl'>0</span> <span class='op'>&amp;</span> <span class='va'>pollutant</span> <span class='op'>!=</span> <span class='st'>"PM2.5"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename</span><span class='op'>(</span>standard_error_pair <span class='op'>=</span> <span class='va'>standard_error</span><span class='op'>)</span>

<span class='co'># merge the two datasets and display results</span>
<span class='fu'>left_join</span><span class='op'>(</span><span class='va'>sampling_variability_pair</span>,
          <span class='va'>sampling_variability_complete</span>,
          by <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>"pollutant"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    <span class='st'>"Precision Improvement (%)"</span> <span class='op'>=</span> <span class='op'>(</span><span class='va'>standard_error_pair</span> <span class='op'>-</span> <span class='va'>standard_error_complete</span><span class='op'>)</span> <span class='op'>/</span>
      <span class='va'>standard_error_complete</span> <span class='op'>*</span> <span class='fl'>100</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate_at</span><span class='op'>(</span><span class='fu'>vars</span><span class='op'>(</span><span class='va'>standard_error_pair</span>, <span class='va'>standard_error_complete</span><span class='op'>)</span>, <span class='op'>~</span> <span class='fu'><a href='https://rdrr.io/r/base/Round.html'>round</a></span><span class='op'>(</span><span class='va'>.</span>, <span class='fl'>2</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>`Precision Improvement (%)` <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/Round.html'>round</a></span><span class='op'>(</span><span class='va'>`Precision Improvement (%)`</span>, <span class='fl'>0</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>rename</span><span class='op'>(</span>
    <span class='st'>"Pollutant"</span> <span class='op'>=</span> <span class='va'>pollutant</span>,
    <span class='st'>"S.E Pair Experiment"</span> <span class='op'>=</span> <span class='va'>standard_error_pair</span>,
    <span class='st'>"S.E Complete Experiment"</span> <span class='op'>=</span> <span class='va'>standard_error_complete</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/pkg/DT/man/datatable.html'>datatable</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span>
</code></pre></div>

```{=html}
<div id="htmlwidget-47cd3a244ea542a428c1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-47cd3a244ea542a428c1">{"x":{"filter":"none","data":[["1","2","3"],["NO2","O3","PM10"],[1.7,1.39,0.87],[1.18,2,0.98],[44,-30,-11]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Pollutant<\/th>\n      <th>S.E Pair Experiment<\/th>\n      <th>S.E Complete Experiment<\/th>\n      <th>Precision Improvement (%)<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[2,3,4]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

</div>



### Outcome Regression Analysis on Matching Data

Finally, we ran a simple time-stratified regression model on the matching data to see how the results differ with those found in our matched data analysis. We only adjust for calendar indicator and weather covariate measured at time $t$. We load the matching data:


<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># load matching data</span>
<span class='va'>data_matching</span> <span class='op'>&lt;-</span>
  <span class='fu'><a href='https://rdrr.io/r/base/readRDS.html'>readRDS</a></span><span class='op'>(</span><span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"1.data"</span>, <span class='st'>"5.matched_data"</span>, <span class='st'>"matching_data.Rds"</span><span class='op'>)</span><span class='op'>)</span>


<span class='co'># reshape in long according to pollutants</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_matching</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>
    cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span>
      <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"no2"</span><span class='op'>)</span>,
      <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"o3"</span><span class='op'>)</span>,
      <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"pm10"</span><span class='op'>)</span>,
      <span class='fu'>contains</span><span class='op'>(</span><span class='st'>"pm25"</span><span class='op'>)</span>
    <span class='op'>)</span>,
    names_to <span class='op'>=</span> <span class='st'>"variable"</span>,
    values_to <span class='op'>=</span> <span class='st'>"concentration"</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    pollutant <span class='op'>=</span> <span class='cn'>NA</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"no2"</span><span class='op'>)</span>, <span class='st'>"NO2"</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"o3"</span><span class='op'>)</span>, <span class='st'>"O3"</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"pm10"</span><span class='op'>)</span>, <span class='st'>"PM10"</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
      <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"pm25"</span><span class='op'>)</span>, <span class='st'>"PM2.5"</span>, <span class='va'>.</span><span class='op'>)</span>
  <span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>time <span class='op'>=</span> <span class='fl'>0</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"lag_1"</span><span class='op'>)</span>,<span class='op'>-</span><span class='fl'>1</span>, <span class='va'>.</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
           <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>variable</span>, <span class='st'>"lead_1"</span><span class='op'>)</span>, <span class='fl'>1</span>, <span class='va'>.</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>variable</span><span class='op'>)</span>

<span class='co'># we nest the data by pollutant and time</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>nest</span><span class='op'>(</span><span class='op'>)</span>
</code></pre></div>

</div>

  
We run the regression model:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># running the model</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>
    <span class='co'># regression model</span>
    model <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>data</span>, <span class='op'>~</span><span class='fu'><a href='https://rdrr.io/r/stats/lm.html'>lm</a></span><span class='op'>(</span><span class='va'>concentration</span> <span class='op'>~</span> <span class='va'>is_treated</span> <span class='op'>+</span> 
                                 <span class='va'>temperature_average</span> <span class='op'>+</span> <span class='fu'><a href='https://rdrr.io/r/base/AsIs.html'>I</a></span><span class='op'>(</span><span class='va'>temperature_average</span><span class='op'>^</span><span class='fl'>2</span><span class='op'>)</span> <span class='op'>+</span> 
                                 <span class='va'>temperature_average_lag_1</span> <span class='op'>+</span> <span class='fu'><a href='https://rdrr.io/r/base/AsIs.html'>I</a></span><span class='op'>(</span><span class='va'>temperature_average_lag_1</span><span class='op'>^</span><span class='fl'>2</span><span class='op'>)</span> <span class='op'>+</span>
                                 <span class='va'>rainfall_duration</span> <span class='op'>+</span> <span class='va'>rainfall_duration_lag_1</span> <span class='op'>+</span>
                                 <span class='va'>humidity_average</span> <span class='op'>+</span> <span class='va'>humidity_average_lag_1</span> <span class='op'>+</span>
                                 <span class='va'>wind_speed</span> <span class='op'>+</span> <span class='va'>wind_speed_lag_1</span> <span class='op'>+</span>
                                 <span class='va'>weekday</span> <span class='op'>+</span> <span class='va'>holidays_dummy</span> <span class='op'>+</span>
                                 <span class='va'>bank_day_dummy</span> <span class='op'>+</span> <span class='va'>month</span><span class='op'>*</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>year</span><span class='op'>)</span>, data <span class='op'>=</span> <span class='va'>.</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># transform in long according to models</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>pivot_longer</span><span class='op'>(</span>cols <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='va'>model</span><span class='op'>)</span>, names_to <span class='op'>=</span> <span class='st'>"model"</span>, values_to <span class='op'>=</span> <span class='st'>"coefficients"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='op'>-</span><span class='va'>data</span><span class='op'>)</span>

<span class='co'># tidy regression ouputs</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>models_dfs <span class='op'>=</span> <span class='fu'>map</span><span class='op'>(</span><span class='va'>coefficients</span>, <span class='op'>~</span> <span class='fu'>broom</span><span class='fu'>::</span><span class='fu'><a href='https://generics.r-lib.org/reference/tidy.html'>tidy</a></span><span class='op'>(</span><span class='va'>.</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>

<span class='co'># unnest results and select coefficient for total gross tonnage</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>  
  <span class='fu'>unnest</span><span class='op'>(</span><span class='va'>models_dfs</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>dplyr</span><span class='fu'>::</span><span class='fu'><a href='https://dplyr.tidyverse.org/reference/filter.html'>filter</a></span><span class='op'>(</span><span class='va'>term</span><span class='op'>==</span><span class='st'>"is_treatedTRUE"</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>select</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span>, <span class='va'>estimate</span>, <span class='va'>std.error</span><span class='op'>)</span>

<span class='co'># we compute the 95% confidence intervals</span>
<span class='va'>interval_95</span> <span class='op'>&lt;-</span> <span class='op'>-</span><span class='fu'><a href='https://rdrr.io/r/stats/Normal.html'>qnorm</a></span><span class='op'>(</span><span class='op'>(</span><span class='fl'>1</span><span class='op'>-</span><span class='fl'>0.95</span><span class='op'>)</span><span class='op'>/</span><span class='fl'>2</span><span class='op'>)</span>

<span class='co'># compute lower and upper bound of each interval</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>ci_lower_95 <span class='op'>=</span> <span class='va'>estimate</span> <span class='op'>-</span> <span class='va'>std.error</span><span class='op'>*</span><span class='va'>interval_95</span>,
         ci_upper_95 <span class='op'>=</span> <span class='va'>estimate</span> <span class='op'>+</span> <span class='va'>std.error</span><span class='op'>*</span><span class='va'>interval_95</span><span class='op'>)</span>
</code></pre></div>

</div>


We draw the graph of results:

<div class="layout-chunk" data-layout="l-body-outset">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># create an indicator to alternate shading of confidence intervals</span>
<span class='va'>data_regression_analysis</span> <span class='op'>&lt;-</span> <span class='va'>data_regression_analysis</span> <span class='op'>%&gt;%</span>
  <span class='fu'>arrange</span><span class='op'>(</span><span class='va'>pollutant</span>, <span class='va'>time</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>stripe <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/ifelse.html'>ifelse</a></span><span class='op'>(</span><span class='op'>(</span><span class='va'>time</span> <span class='op'>%%</span> <span class='fl'>2</span><span class='op'>)</span> <span class='op'>==</span> <span class='fl'>0</span>, <span class='st'>"Grey"</span>, <span class='st'>"White"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='co'># make the graph</span>
<span class='va'>graph_regression_ci</span> <span class='op'>&lt;-</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='va'>data_regression_analysis</span>,
         <span class='fu'>aes</span><span class='op'>(</span>x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>, y <span class='op'>=</span> <span class='va'>estimate</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_rect</span><span class='op'>(</span>
    <span class='fu'>aes</span><span class='op'>(</span>fill <span class='op'>=</span> <span class='va'>stripe</span><span class='op'>)</span>,
    xmin <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>data_regression_analysis</span><span class='op'>$</span><span class='va'>time</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>-</span> <span class='fl'>0.42</span>,
    xmax <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/numeric.html'>as.numeric</a></span><span class='op'>(</span><span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>data_regression_analysis</span><span class='op'>$</span><span class='va'>time</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span> <span class='fl'>0.42</span>,
    ymin <span class='op'>=</span> <span class='op'>-</span><span class='cn'>Inf</span>,
    ymax <span class='op'>=</span> <span class='cn'>Inf</span>,
    color <span class='op'>=</span> <span class='cn'>NA</span>,
    alpha <span class='op'>=</span> <span class='fl'>0.4</span>
  <span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_hline</span><span class='op'>(</span>yintercept <span class='op'>=</span> <span class='fl'>0</span>,
             color <span class='op'>=</span> <span class='st'>"black"</span>,
             size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_vline</span><span class='op'>(</span>xintercept <span class='op'>=</span> <span class='st'>"0"</span>,
             color <span class='op'>=</span> <span class='st'>"black"</span>,
             size <span class='op'>=</span> <span class='fl'>0.3</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_pointrange</span><span class='op'>(</span>
    <span class='fu'>aes</span><span class='op'>(</span>
      x <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/factor.html'>as.factor</a></span><span class='op'>(</span><span class='va'>time</span><span class='op'>)</span>,
      y <span class='op'>=</span> <span class='va'>estimate</span>,
      ymin <span class='op'>=</span> <span class='va'>ci_lower_95</span> ,
      ymax <span class='op'>=</span> <span class='va'>ci_upper_95</span>
    <span class='op'>)</span>,
    size <span class='op'>=</span> <span class='fl'>0.5</span>,
    colour <span class='op'>=</span> <span class='va'>my_blue</span>,
    lwd <span class='op'>=</span> <span class='fl'>0.8</span>
  <span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_y_continuous</span><span class='op'>(</span>breaks <span class='op'>=</span> <span class='fu'>scales</span><span class='fu'>::</span><span class='fu'><a href='https://scales.r-lib.org/reference/breaks_pretty.html'>pretty_breaks</a></span><span class='op'>(</span>n <span class='op'>=</span> <span class='fl'>8</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span> <span class='op'>~</span> <span class='va'>pollutant</span>, ncol <span class='op'>=</span> <span class='fl'>4</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>scale_fill_manual</span><span class='op'>(</span>values <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'gray96'</span>, <span class='st'>"white"</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>guides</span><span class='op'>(</span>fill <span class='op'>=</span> <span class='cn'>FALSE</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>ylab</span><span class='op'>(</span><span class='st'>"Average Increase in\n Concentrations (µg/m³)"</span><span class='op'>)</span> <span class='op'>+</span> <span class='fu'>xlab</span><span class='op'>(</span><span class='st'>"Day"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme_tufte</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>theme</span><span class='op'>(</span>axis.text.x <span class='op'>=</span> <span class='fu'>element_text</span><span class='op'>(</span>margin <span class='op'>=</span> <span class='fu'>ggplot2</span><span class='fu'>::</span><span class='fu'><a href='https://ggplot2.tidyverse.org/reference/element.html'>margin</a></span><span class='op'>(</span>t <span class='op'>=</span> <span class='fl'>0</span>, unit <span class='op'>=</span> <span class='st'>"cm"</span><span class='op'>)</span><span class='op'>)</span><span class='op'>)</span>


<span class='co'># print the graph</span>
<span class='va'>graph_regression_ci</span>
</code></pre></div>
![](5_script_analyzing_results_files/figure-html5/unnamed-chunk-25-1.png)<!-- --><div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'># save the graph</span>
<span class='fu'>ggsave</span><span class='op'>(</span>
  <span class='va'>graph_regression_ci</span>,
  filename <span class='op'>=</span> <span class='fu'>here</span><span class='fu'>::</span><span class='fu'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='op'>(</span><span class='st'>"3.outputs"</span>, <span class='st'>"2.matching_analysis"</span>, <span class='st'>"graph_regression_ci.pdf"</span><span class='op'>)</span>,
  width <span class='op'>=</span> <span class='fl'>16</span>,
  height <span class='op'>=</span> <span class='fl'>8</span>,
  units <span class='op'>=</span> <span class='st'>"cm"</span>,
  device <span class='op'>=</span> <span class='va'>cairo_pdf</span>
<span class='op'>)</span>
</code></pre></div>

</div>


We display below the values of the point estimates and 95% confidence intervals:

<div class="layout-chunk" data-layout="l-body-outset">

```{=html}
<div id="htmlwidget-88f377da48d4ac0fc7ff" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-88f377da48d4ac0fc7ff">{"x":{"filter":"none","data":[["1","2","3","4","5","6","7","8","9","10","11","12"],["NO2","NO2","NO2","O3","O3","O3","PM10","PM10","PM10","PM2.5","PM2.5","PM2.5"],[-1,0,1,-1,0,1,-1,0,1,-1,0,1],[0.5,-0.5,0.8,-0.9,-0.9,1.1,3.9,5.5,7.4,2.8,4,5.7],[0.1,-0.9,0.4,-1.4,-1.4,0.6,3.4,5,6.9,2.1,3.3,5],[0.9,-0.2,1.2,-0.4,-0.5,1.7,4.5,6,8,3.6,4.7,6.4]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Pollutant<\/th>\n      <th>Time<\/th>\n      <th>Point Estimate<\/th>\n      <th>Lower Bound of the 95% Confidence Interval<\/th>\n      <th>Upper Bound of the 95% Confidence Interval<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[2,3,4,5]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

</div>

```{.r .distill-force-highlighting-css}
```
