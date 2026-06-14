---
link: https://tylercipriani.com/blog/2023/05/18/chartjunk/
byline: Tyler Cipriani
excerpt: |-
  For many people the first word that comes to mind when they think
  about statistical charts is “lie.”
slurped: 2026-06-01T08:26
title: What I've learned about data visualization
---

> For many people the first word that comes to mind when they think about statistical charts is “lie.”
> 
> – Edward R. Tufte

![William Playfair, 1801, Statistical map showing the extent, population and income of the principal nations of Europe The birth of the pie chart. Playfair’s attempt to show the size of European countries by relating them to planets, saying: “we have a more accurate idea of the sizes of the planets, which are spheres, than of the nations of Europe […] all of which are irregular forms”](https://photos.tylercipriani.com/thumbs/4d/7b7093596f72b025439eda463f52ca/large.jpg)

When I moved from engineering to management, people expected me to make charts.

After spending some time learning about data visualization, I’ve come to two important conclusions:

1. Good data visualization is powerful
2. Powerful data visualization is rare

But creating a compelling chart is an underrated superpower for engineers. Here are some ideas that helped me learn how to do that.

## Learning from experts

The best books I’ve read on data visualization are:

- “[The Visual Display of Quantitative Information](https://openlibrary.org/books/OL3966475M/The_Visual_Display_of_Quantitative_Information)” by Tufte, and
- “[Storytelling with Data](https://openlibrary.org/works/OL17659741W/Storytelling_with_Data)” by Cole Nussbaumer Knaflic

Between those two books, you get a balance: the ideal (from Tufte) and the PowerPoint world (from Knaflic).

**Tufte**

[Edward R. Tufte](https://en.wikipedia.org/wiki/Edward_Tufte) is professor emeritus of statistics, political science, and computer science at Yale.

And he’s a man upset by pie charts.

> the only worse design than a pie chart is several of them.
> 
> – Edward R. Tufte

Tufte’s rules:

- 🦑**Maximize the data-ink ratio** – Most of your chart should be data. Erase the parts that convey nothing.
- 🍫**Avoid chart junk** – Avoid needless colors, shading, tickmarks, and gridlines: remove distractions from data.

**Knaflic**

Cole Nussbaumer Knaflic was the former manager of Google’s People Analytics team. She knows her way around a PowerPoint[1](https://tylercipriani.com/blog/2023/05/18/chartjunk/#fn1).

Knaflic’s rules:

- 📈**Choose an appropriate visual display** – Know your context, then pick your chart
- 🏰**Tell a story** – Charts ought to communicate something; you should know what that something is.

## Tufte’s principles: a case study

Take a look at this chart from the Wikimedia Foundation’s [2023–2024 budget projections](https://meta.wikimedia.org/wiki/Wikimedia_Foundation_Annual_Plan/2023-2024/Foundation_Details):

![Wikimedia Foundation 2023–2024 Draft Budget](https://photos.tylercipriani.com/2023-05-17_Wikimedia-foundation-annual-plan-2023-2024-draft-budget-overview.png)

This is a default Google Sheets chart for this data.

What I’m able to glean from this chart:

- “Building analytics & ML Services”: the biggest, > 25%
- “Features and functionality” and “Supporting volunteers”: smaller, ~20% each
- “Fundraising,” “Protecting access,” and “General & Admin”: smaller still, ~10% each
- The colors seem meaningless

Now consider what we glean from the table used to generate it:

|Program|Budget (millions)|Percent|
|---|---|---|
|Building analytics & ML services|$46.4|26.2%|
|Features and functionality|$39.7|22.4%|
|Supporting volunteers|$35.1|19.8%|
|General & Admin|$21.3|12.0%|
|Fundraising|$17.9|10.1%|
|Protecting access|$16.6|9.4%|
|Total|$177.0|100.0%|

This table gives us more information than the chart in a similar amount of space.

From the table, we learn:

- Exact percentages—no need to guess
- Exact dollar amounts
- The total budget as a dollar figure

And we get rid of the meaningless colors.

OK—confession time: there were percentages on the original pie chart. I edited them out.

But I did that to prove a point. **The pie chart is doing less work than the numbers**. The slices of the pie add almost nothing to the numbers.

Tufte would prefer the table above to the pie chart above because the table:

- Shows the data
- Maximizes the data-ink ratio
- Avoids [chartjunk](https://en.wikipedia.org/wiki/Chartjunk) (like the meaningless colors)

And in this case, I agree: I like the table.

But choosing data visualizations is difficult.

## How to choose a chart: visual perception accuracy ranked

If you intend to communicate, then people must be able to read your chart.

In 1985, Cleveland & McGill conducted what is still the most cited experiment [on what charts are easiest to read](https://doi.org/10.2307/2288400). Their purpose was simple: rank standard charts by the number of errors people make while reading them.

Here’s their ranking of charts—from easiest to hardest to read:

|Icon|Name|Example|
|---|---|---|
|![](https://photos.tylercipriani.com/bar-chart-icon.png)|Position along a common scale|[scatterplots](https://datavizcatalogue.com/methods/scatterplot.html), [bar charts](https://datavizcatalogue.com/methods/bar_chart.html), [sparklines](https://en.wikipedia.org/wiki/Sparkline)|
|![](https://photos.tylercipriani.com/2023-05-18_stacked-bar-chart-icon.png)|Positions along nonaligned scales|[stacked bar charts](https://datavizcatalogue.com/methods/stacked_bar_graph.html)|
|![](https://photos.tylercipriani.com/2023-05-18_pie-chart-icon.png)|Length, direction, angle|[pie charts](https://datavizcatalogue.com/methods/pie_chart.html), [donut charts](https://datavizcatalogue.com/methods/donut_chart.html)|
|![](https://photos.tylercipriani.com/2023-05-18_bubble-chart-icon.png)|Area|[bubble charts](https://datavizcatalogue.com/methods/bubble_chart.html), [treemaps](https://datavizcatalogue.com/methods/treemap.html)|
|![](https://photos.tylercipriani.com/2023-05-18_3d-volume-chart-icon.png)|Volume, curvature|3d charts|
|![](https://photos.tylercipriani.com/2023-05-18_heatmap-icon.png)|Shading, color saturation|[Heatmaps](https://datavizcatalogue.com/methods/heatmap.html)|

But I love heatmaps! You can use heatmaps. Just understand that only a sophisticated audience can interpret heatmaps correctly.

Know your audience, then use something like [the data viz catalog](https://datavizcatalogue.com/) to choose your chart.

## Tell a story

Charts are communication.

At their worst, as Tufte said, they’re little more than “devices for showing the obvious to the ignorant.”

But at their best, they’re a powerful way to intuitively communicate a lot of data in a small space.

---

1. Tufte will not help you with PowerPoint. In his essay, “[The Cognitive Style of PowerPoint](https://www.inf.ed.ac.uk/teaching/courses/pi/2016_2017/phil/tufte-powerpoint.pdf)” (which is included in the book “[Beautiful Evidence](https://openlibrary.org/works/OL2824008W/Beautiful_Evidence)”), he says: “bulleted outlines make us stupid” before going on to blame PowerPoint for [the space shuttle Columbia disaster](https://en.wikipedia.org/wiki/Space_Shuttle_Columbia_disaster).[↩︎](https://tylercipriani.com/blog/2023/05/18/chartjunk/#fnref1)