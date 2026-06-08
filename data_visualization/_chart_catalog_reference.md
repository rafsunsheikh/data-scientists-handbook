# Data Visualization Catalog — Complete Reference

> Source: [Data Visualization Catalogue](https://datavizcatalogue.com)
>
> This file contains detailed information about all 60 chart types, organized by category. Use it to enrich visualization notebooks with chart selection guidance, data requirements, and best practices.

---

## Table of Contents

1. [Distributions](#distributions)
2. [Comparisons](#comparisons)
3. [Relationships / Correlations](#relationships--correlations)
4. [Compositions / Part-to-Whole](#compositions--part-to-whole)
5. [Data Over Time / Trends](#data-over-time--trends)
6. [Geospatial / Location](#geospatial--location)
7. [Networks / Hierarchies](#networks--hierarchies)
8. [Flows / Processes](#flows--processes)
9. [Text / Content](#text--content)
10. [Financial / Ranges](#financial--ranges)
11. [Brainstorming / Conceptual](#brainstorming--conceptual)
12. [Dashboards / Reference](#dashboards--reference)

---

## Distributions

Charts that show how data is spread across a range.

### Box & Whisker Plot

- **Description:** Displays data distribution through quartiles. Rectangular boxes frame central data ranges; whiskers (parallel lines) indicate spread beyond bounds. Can be oriented vertically or horizontally.
- **Data Required:** Statistical summaries — median, mean, 25th percentile, extreme values (min/max).
- **Use When:** Space is limited but you need to evaluate multiple datasets simultaneously. Excels at comparing statistical distributions across numerous groups — a compact alternative to histograms or density curves.
- **Insights Revealed:** Key statistical markers (medians, percentiles), extreme data points, symmetry/asymmetry, data clustering density, directional skewness.
- **Variations:** Variable-width box plots, notched box plots.
- **Similar Charts:** Span Chart, Violin Plot.
- **Tools:** D3.js, Plotly, Seaborn, Vega, Excel, Stata, Tableau, Flourish, RAWGraphs, PlotDB.

### Density Plot (Kernel Density Plot)

- **Description:** Visualizes distribution of data over a continuous interval using a smooth curve. Applies kernel smoothing to eliminate noise — unlike histograms, it produces a fluid curve without arbitrary bin boundaries. Elevated sections show where data points cluster most heavily.
- **Data Required:** Continuous numerical measurements tracked across a defined timeframe or measurable range.
- **Use When:** Assessing distribution shapes and identifying trends. Provides a stable visual representation that consistently reveals the true data shape, regardless of how intervals are grouped. Avoids the visual distortion caused by varying bin counts in histograms.
- **Advantage:** Unaffected by bin quantity; delivers a clearer view of the data's true shape than histograms.
- **Similar Charts:** Histogram.
- **Tools:** R, D3.js, Python (Seaborn/Matplotlib), React, ggplot2, Vega, Vega-Lite, Datylon, Everviz, Plotivy, Tableau.

### Histogram

- **Description:** Visualizes the distribution of data over a continuous interval. Each bar corresponds to the calculated frequency within specific bins/ranges.
- **Data Required:** Continuous numerical data that can be grouped into ranges.
- **Use When:** Analyzing distributions and patterns. Helps viewers quickly estimate value concentrations, identify extremes, detect gaps or anomalies, and approximate probability distributions.
- **Similar Charts:** Bar Chart, Density Plot, Population Pyramid.
- **Tools:** D3.js, Plotly, Python, R, Vega, Flourish, Tableau, Infogram, Microsoft Office, Apple Numbers.

### Stem & Leaf Plot

- **Description:** A hybrid between a table and histogram that preserves individual data values while showing distribution shape. Each data point is split into a "stem" (leading digit(s)) and a "leaf" (trailing digit).
- **Data Required:** Numerical data with multiple digits (typically 2-4 digit numbers).
- **Use When:** Small datasets where you need to see both the distribution shape and the actual data values. Useful in educational settings and for quick exploratory analysis.
- **Limitations:** Becomes unwieldy with large datasets or data with many decimal places.
- **Similar Charts:** Histogram, Density Plot.
- **Tools:** Manual/paper, R (stem() function), Python (stemgraphic library).

### Violin Plot

- **Description:** A hybrid visualization merging a box plot and density plot. Shows data distribution and probability density. Visual markers: central white dot = median, thick black bar = interquartile range, thin lines = adjacent min/max values.
- **Data Required:** Numerical datasets used to calculate median, interquartile range, adjacent min/max points, and overall probability density.
- **Use When:** Comparing how values distribute across categories, particularly when identifying **bimodal or multimodal** patterns that simpler charts typically hide.
- **Trade-off:** More detail than box plots, but can appear "visually noisier."
- **Similar Charts:** Box & Whisker Plot.
- **Tools:** D3.js, Highcharts, Python/seaborn, R/ggplot2, Vega, Flourish, Plotly, Plotivy.

---

## Comparisons

Charts designed to compare values across categories or groups.

### Bar Chart

- **Description:** Uses horizontal or vertical bars to display discrete numerical comparisons across categories. Answers the question "how many?" in each category.
- **Data Required:** Discrete, numerical data organized by specific categories. Must be categorical (not continuous).
- **Use When:** Comparing values across distinct groups. Identify patterns across categories.
- **Avoid When:** Showing trends over continuous intervals.
- **Also Known As:** Bar graph, column graph.
- **Variations:** Multi-set, Population Pyramid, Radial, Stacked formats.
- **Tools:** D3.js, Plotly, Highcharts, Datawrapper, Flourish, Excel, Tableau.

### Bullet Graph

- **Description:** Performance-focused visualization that expands on the standard bar chart by adding contextual markers. Designed by Stephen Few to replace cluttered dashboard meters.
- **Data Required:** Primary metric ("Feature Measure"), target threshold ("Comparative Measure"), and qualitative performance bands (e.g., poor/average/great).
- **Use When:** Tracking goal progress, comparing values, displaying range data efficiently. Optimized for dashboard use.
- **Best Practice:** Limit qualitative bands to five maximum.
- **Similar Charts:** Standard Bar Chart.
- **Tools:** D3.js, Vega-Lite, Plotly, Datawrapper, Flourish, Tableau.

### Marimekko Chart (Mosaic Plot)

- **Description:** Maps categorical data across two variables. Percentage scales on both axes control segment dimensions — operates like a dual-axis 100% Stacked Bar Graph. Highlights connections between primary categories and their subdivisions.
- **Data Required:** Categorical information organized around two variables, where both axes represent variables measured on a percentage scale.
- **Use When:** Broad overviews of category relationships across two dimensions. Supports comparisons, part-to-whole breakdowns, proportions, and relational trends.
- **Limitations:** Interpretation grows difficult as segment count increases. Blocks lack a unified baseline — better for general insights than precise measurements.
- **Similar Charts:** Stacked Bar Graph, Treemap.
- **Tools:** AnyChart, FusionCharts, JSCharting, D3.js, Vega-Lite, plotDB, Plotivy, Tableau.

### Multi-set Bar Chart (Grouped/Clustered Bar Chart)

- **Description:** Arranges multiple data series on a shared axis under broader parent categories. Bar length represents discrete numerical values. Each series gets a distinct color; bars in the same group are placed together and spaced from other groupings.
- **Data Required:** Parent categories containing identical subcategories. Handles comparisons of shared sub-variables, contrasts mini-histograms, or tracks metrics across time intervals.
- **Use When:** Comparing categories sharing the same sub-variables, visualizing time-based trends, or contrasting frequency distributions.
- **Best Practice:** Limit the number of bars within each cluster for clarity.
- **Analytical Functions:** Comparisons, distribution tracking, pattern identification, relationship analysis.
- **Similar Charts:** Standard Bar Chart, Histogram, Stacked Bar Graph.
- **Tools:** Various programming libraries, web platforms, desktop applications.

### Pictogram Chart

- **Description:** Uses repeated icons or images to represent data values. Each icon stands for a specific quantity.
- **Data Required:** Numerical data that divides evenly by the icon value, or requires partial-icon rendering.
- **Use When:** Presenting data to general audiences, simplifying complex statistics, adding visual appeal to reports.
- **Limitations:** Difficult to represent partial values accurately. Not suitable for precise quantitative analysis.
- **Similar Charts:** Dot Matrix Chart.
- **Tools:** Various design tools, web generators.

### Radar Chart (Spider/Web/Polar/Star Chart)

- **Description:** Maps multiple quantitative metrics by assigning each a radial axis from a central origin. Axes share uniform spacing and scale. Data points are linked to create a polygon with reference grid lines.
- **Data Required:** Multiple quantitative variables or performance metrics for each category/subject.
- **Use When:** Identifying similar values or outliers, tracking performance levels, analyzing comparisons, patterns, and relationships across multiple dimensions.
- **Limitations:** Multiple polygons become hard to read and cluttered (especially when filled). Too many variables complicate readability. Not as good for comparing values across each variable as linear layouts.
- **Best Practice:** Use transparency to reduce overlap; keep layout simple.
- **Tools:** amCharts, Chart.js, Highcharts, D3.js, Plotly, Vega, Flourish, Tableau.

### Radial Bar Chart

- **Description:** Maps a standard bar chart onto polar coordinates. Bars grow outward from the center along concentric rings.
- **Data Required:** Categorical data paired with numerical values suitable for comparison.
- **Use When:** **Only for decorative/aesthetic purposes.** The chart is misleading for data comparison because bar lengths at larger radii visually expand compared to identical values at smaller radii.
- **Warning:** Straight-line bar charts are far more accurate for interpretation.
- **Similar Charts:** Standard Bar Chart, Radial Column Chart.
- **Tools:** amCharts, AnyChart, Apache ECharts, JSCharting, Everviz, Infogram, Vizzlo.

### Radial Column Chart (Star Graph)

- **Description:** Maps bars onto a polar grid. Concentric rings indicate magnitude; lines radiating from the core denote distinct groups. Bars grow outward from the middle, scaling with each ring. Negatives: zero positioned at any ring, inner rings = negative magnitudes.
- **Data Required:** Category labels for radial divisions and numerical values for bar lengths.
- **Use When:** Comparisons in a circular layout.
- **Similar Charts:** Bar Chart, Radial Bar Chart.
- **Tools:** amCharts, Bokeh, D3.js, jChartFX, matplotlib, Flourish, Plotivy.

### Tally Chart

- **Description:** Records and displays data frequency using tally marks. Categories/values/intervals placed in one axis/column, with a mark added per occurrence. Tallies are summed into a total column.
- **Data Required:** Simple frequency data — how often specific categories, numerical values, or defined intervals occur.
- **Use When:** Making direct comparisons or analyzing how data is distributed across groups.
- **Construction:** Two-step process: mark occurrences per category, then calculate totals.
- **Similar Charts:** Histogram.
- **Tools:** Paper and pen, Plotivy.

---

## Relationships / Correlations

Charts that reveal connections between variables.

### Bubble Chart

- **Description:** A cross between a Scatterplot and a Proportional Area Chart. Maps two metrics on standard axes while circle size represents a third value. Colors can denote categories or a fourth metric; time can be mapped or animated.
- **Data Required:** At least 3 numerical variables for positioning and sizing. Categorical labels for grouping. Optional 4th variable for color coding, plus temporal data.
- **Use When:** Comparisons, tracking data over time, showing distributions, identifying patterns, displaying proportions, revealing relationships between categorized circles.
- **Critical Rule:** Circle sizes must be drawn based on **area**, not radius or diameter, to prevent visual distortion.
- **Limitations:** Large datasets quickly become cluttered. Interactive features (hover tooltips, filtering) help manage complexity.
- **Similar Charts:** Scatterplot, Proportional Area Chart.
- **Tools:** AmCharts, Chart.js, ggplot2, Flourish, Excel, Tableau, D3.js, Plotly.

### Parallel Coordinates Plot

- **Description:** Displays multivariate data using parallel vertical axes. Each line represents a data point, crossing each axis at the value for that variable. Makes it possible to see clusters and outliers.
- **Data Required:** Multiple numerical variables for each data point. Tabular data where each row is an observation and each column is a variable.
- **Use When:** Comparing multiple variables across many data points, identifying patterns and clusters in high-dimensional data, spotting outliers.
- **Limitations:** Can become cluttered with too many lines. Axes must be scaled similarly for fair comparison.
- **Similar Charts:** Parallel Sets, Radar Chart.
- **Tools:** R (ggplot2, parallel), Python (pandas, matplotlib), D3.js.

### Scatterplot

- **Description:** Plots numerical pairs across a coordinate system (X-Y plot). Shows how two variables relate to each other.
- **Data Required:** Paired numerical data representing two distinct variables.
- **Use When:** Analyzing whether changes in one metric correspond to changes in another — "to see if one variable impacts the other."
- **Analytical Features:**
  - **Correlation Types:** Positive, negative, or null patterns
  - **Shape & Strength:** Linear, exponential, or U-shaped curves; point density indicates strength
  - **Outliers:** Points far from the main group
  - **Trend Analysis:** "Line of Best Fit" overlays for interpolation estimates
- **Critical Note:** "Correlation is not causation" — hidden variables may skew results.
- **Similar Charts:** Bubble Chart.
- **Tools:** D3.js, Chart.js, matplotlib, ggplot2, Python, R, Excel, Tableau.

### Span Chart (Range Bar/Floating Bar/High-Low/Difference Graph)

- **Description:** Shows dataset ranges between a minimum and maximum value. Focuses on the range rather than individual points.
- **Data Required:** Paired min/max values for each category. No intermediate data points required.
- **Use When:** Comparing ranges between categories (e.g., temperature ranges, budget ranges, score ranges).
- **Limitations:** Focuses only on extreme values — gives no information on data points between min and max (averages, distribution).
- **Similar Charts:** Box & Whisker Plot, Candlestick Chart, Gantt Chart.
- **Tools:** AnyChart, D3.js, JSCharting, Vega-Lite, ZingChart, Plotivy, Tableau.

---

## Compositions / Part-to-Whole

Charts showing how a whole divides into parts.

### Pie Chart

- **Description:** Splits a circle into proportional slices. Each slice's arc length indicates a category's share; the entire circle = 100%.
- **Data Required:** Categorical data where all values sum to a single total (100%). Best with a limited number of categories.
- **Use When:** Delivering a rapid overview of proportional distribution or comparing a single category against the total.
- **Limitations:**
  - Slices become unreadable as category count increases
  - Occupies more space than 100% stacked bar charts; typically requires a legend
  - Accurate comparisons across multiple pie charts are challenging — humans are bad at judging area differences
- **Similar Charts:** Donut Chart, Nightingale Rose Chart, Sunburst Diagram.
- **Tools:** D3.js, Chart.js, Highcharts, R, Python, Excel, Tableau, Flourish.

### Donut Chart

- **Description:** A Pie Chart with the center area cut out. Shifts attention from slice area to **arc length**, helping viewers track overall value changes. The empty middle allows additional information to be placed inside.
- **Data Required:** Categorical data representing distinct segments of a total, suitable for measuring proportions.
- **Use When:** Comparisons, part-to-a-whole relationships, proportions.
- **Similar Charts:** Pie Chart, Sunburst Diagram.
- **Tools:** D3.js, Chart.js, Plotly, Vega, Datawrapper, Flourish, Tableau, Microsoft Office.

### Stacked Bar Graph

- **Description:** Layers segments vertically within bars. Shows how a larger category divides into subcategories alongside each component's share of the aggregate. Simple formats stack values sequentially; percentage-based layouts display proportional shares.
- **Data Required:** Categorical groupings containing multiple subcategories, paired with numerical values for individual segments and combined sums.
- **Use When:** Comparisons and proportions. Simple layouts excel for "total amounts across each segmented bar." The 100% variant suits part-to-a-whole analysis.
- **Limitations:** Clarity drops with numerous divisions. Cross-segment evaluation suffers because bars aren't aligned on a common baseline.
- **Similar Charts:** Bar Chart, Marimekko Chart.
- **Tools:** D3.js, Chart.js, Highcharts, R, Python, Excel, Tableau, Flourish.

### Treemap

- **Description:** Displays hierarchical data using nested rectangles. Each shape's size reflects a numerical value. Layouts use tiling algorithms (commonly "squarified") prioritizing near-square shapes. Originally designed by Ben Shneiderman for file directory visualization.
- **Data Required:** Hierarchical/category data (parent-child relationships) + numerical values for part-to-whole comparisons (required for accurate sizing; if omitted, space splits equally among siblings).
- **Use When:** Space is limited and you need a quick overview of how categories relate proportionally. Best for comparisons, hierarchy mapping, part-to-a-whole relationships, and proportions.
- **Strengths:** Highly compact, screen-space efficient, effective for proportional relationships.
- **Weaknesses:** Doesn't show hierarchical levels as clearly as tree or sunburst diagrams.
- **Similar Charts:** Circle Packing, Marimekko Chart, Sunburst Diagram.
- **Tools:** D3.js, Plotly, ECharts, amCharts, AnyChart, Vega, Flourish, RAWGraphs, Infogram, Tableau, Microsoft Office.

---

## Data Over Time / Trends

Charts for tracking changes across time.

### Area Graph

- **Description:** A line graph variant where the space beneath the line is filled with color or texture. Created by plotting coordinates, connecting them, and shading the resulting shape.
- **Data Required:** Numerical measurements tracked across a timeline or continuous interval.
- **Use When:** Displaying the development of quantitative values over an interval or time period. Ideal for highlighting general trajectories rather than reporting precise figures.
- **Variations:** Grouped (shared baseline) and Stacked (each series builds on the previous endpoint).
- **Similar Charts:** Line Graph, Stacked Area Graph, Stream Graph.
- **Tools:** D3.js, Chart.js, Highcharts, Plotly, Excel, Tableau, Flourish.

### Calendar

- **Description:** Displays periods of time and organizes events using monthly grids with 7 columns for weekdays and 5-6 rows. Must clearly show a chronological sequence of dates or time units.
- **Data Required:** Date-specific data to track events or periods across days, weeks, months, and years.
- **Use When:** Visualizing data over time and serving as a reference tool.
- **Similar Charts:** Timetable.
- **Tools:** AnyChart, Google Charts, JSCharting, Vega, ZingChart, Canva, Google Docs, Plotivy, RAWGraphs, Excel.

### Gantt Chart

- **Description:** Project management tool that visualizes task schedules across a timeline. Displays activities with their duration over time. Grid layout: rows = activities, columns = timescale.
- **Data Required:** Task names, start/end dates, duration values, completion percentages, categorical groupings, dependency mappings, milestone markers, current date reference.
- **Use When:** Scheduling, tracking parallel workflows, managing task dependencies. Best for data over time, processes & methods, ranges, and reference tools.
- **Visual Elements:** Horizontal bars for task durations (color-coded by category, partially shaded for progress). Connecting arrows for dependencies, highlighted arrows for critical paths, milestone symbols, vertical line for present date.
- **Similar Charts:** Timeline, Stream Graph.
- **Tools:** Asana, ClickUp, Smartsheet, Flourish, Excel, Tableau, amCharts, DHTMLX, Google Charts, Vega-Lite, Plotly.

### Line Graph

- **Description:** Maps quantitative values across continuous intervals or time periods. Points plotted on a grid and connected by lines. Upward slopes = growth, downward = decline. The path reveals underlying dataset patterns.
- **Data Required:** Quantitative metrics paired with a timescale or sequential intervals. Supports negative values beneath the horizontal axis.
- **Use When:** Tracking temporal changes and comparing multiple datasets. **Best chart for showing trends.**
- **Critical Best Practice:** Avoid using more than 3-4 lines per graph to maintain readability. For additional series, split into smaller separate charts (small multiples).
- **Similar Charts:** Area Graph, Stacked Area Graph, Stream Graph.
- **Tools:** D3.js, Chart.js, matplotlib, ggplot2, Datawrapper, Flourish, RAWGraphs, Google Docs, Tableau, Microsoft Office.

### Spiral Plot

- **Description:** Plots time-based data along an Archimedean spiral originating at the center and expanding outward. Can render bars, lines, or points along the curved path. Also called a "Time Series Spiral."
- **Data Required:** Time-series datasets, particularly large volumes of records spanning extended durations.
- **Use When:** Showing large datasets to highlight long-term trends and recurring cycles. Assigning distinct colors to each cycle improves visual segmentation.
- **Similar Charts:** Line Graph, Stacked Area Graph.
- **Tools:** D3.js (Arpit Narechania), R/ggplot2 (coord_polar).

### Stacked Area Graph

- **Description:** Each data point is built on top of the previous series, layering multiple sets. The entire graph represents the total of all data plotted. Relies on filled areas to communicate magnitude.
- **Data Required:** Multiple continuous datasets tracking whole numbers across a sequential progression (typically chronological). **Cannot display negative values.**
- **Use When:** Comparing multiple variables changing over time while tracking cumulative totals.
- **Functions:** Comparisons, data over time, pattern recognition.
- **Similar Charts:** Area Graph, Line Graph, Stream Graph.
- **Tools:** D3.js, Chart.js, Highcharts, R/ggplot2, Python/Plotly, Datawrapper, Flourish, Tableau.

### Stream Graph

- **Description:** A modified stacked area plot shifted to fluctuate around a center line rather than a flat baseline. Uses fluid, organic bands to show how categories evolve. Band width = value magnitude; horizontal axis = time.
- **Data Requirements:** High-volume datasets tracking multiple distinct categories across a continuous timeline, with values that rise and fall.
- **Use When:** Spotting broad trends, seasonal cycles, or volatility across numerous groups. Recommended for interactive displays and high-level overview audiences.
- **Limitations:** Legibility issues with dense datasets. Smaller categories get visually overwhelmed by larger ones. Missing value axis makes reading exact figures impossible.
- **Example:** New York Times — "Box Office Receipts 1986-2008."
- **Similar Charts:** Stacked Area Graph.
- **Tools:** Apache ECharts, NVD3.js, Vega-Lite, R packages, Flourish, RAWGraphs, Infogram, Datylon.

### Timeline

- **Description:** A graphical way of displaying a list of events in chronological order. Some versions use proportional scales; others simply show sequence. Communicates time-related information for analysis or visual storytelling.
- **Data Required:** A sequence of events tied to specific dates or durations. Scaled versions need precise time intervals. Quantitative metrics may be added to show changes alongside the chronological layout.
- **Use When:** Tracking data over time, distribution, or patterns. Analyzing historical sequences, spotting recurring trends, illustrating how measured values evolve chronologically.
- **Similar Charts:** Gantt Chart, Timetable.
- **Tools:** AnyChart, Google Charts, KronoGraph, Vega, Creately, Flourish, Plotivy, Tiki-Toki, Venngage, Vizzlo, yEd live, Adobe Illustrator.

---

## Geospatial / Location

Charts for mapping data to geographic areas.

### Bubble Map

- **Description:** Circles placed on a map, with the area of each circle proportional to its value.
- **Data Required:** Geographic coordinates and proportional numerical values for each location.
- **Use When:** Comparing regional values **without the size-bias inherent in choropleth maps**.
- **Limitations:** Oversized markers can overlap other data points or map features — requires careful sizing adjustments.
- **Similar Charts:** Dot Map, Proportional Area Chart.
- **Tools:** amCharts, AnyChart, Google Charts, Plotly, Polymaps, Tableau, CARTO, Columns.ai, Datawrapper, Flourish.

### Choropleth Map

- **Description:** Geographic territories are shaded or patterned to reflect a specific metric. Helps visualize values over a geographical area, showing variation or patterns across the displayed location. Color scales shift from light to dark or blend hues.
- **Data Required:** Spatial boundary data paired with a corresponding numerical variable. **Values must be normalized** (e.g., density per square kilometer) rather than raw counts to prevent misleading visual emphasis on larger zones.
- **Use When:** Analyzing location-based trends, comparing metrics across regions, identifying spatial patterns.
- **Limitations & Best Practices:** Color progressions make precise numerical comparisons difficult. Larger regions naturally draw more attention. **Always normalize data** to avoid misleading density representations.
- **Common Error:** Encoding raw data values instead of normalized values produces a misleading map (not a true density map).
- **Similar Charts:** Bubble Map, Connection Map, Flow Map.
- **Tools:** D3.js, Plotly, R, Python, Datawrapper, Flourish, Tableau, ArcGIS.

### Connection Map

- **Description:** Also known as a Link Map or Ray Map. Points placed on a map are connected by straight or curved lines.
- **Data Required:** Geographically plotted points along with link or route data that connects them.
- **Use When:** Showing connections and relationships geographically, tracking specific pathways. Highlights spatial patterns through the distribution of connections and shows where links are most dense.
- **Functions:** Distribution, location, movement, patterns, relationships.
- **Similar Charts:** Flow Map.
- **Tools:** amCharts, AnyChart, D3.js, JSCharting, Python/R Graph Galleries, yFiles, Zoomcharts, Flourish, FlowmapBlue, Plotivy.
- **Example:** Busiest Routes From Heathrow Airport (2014).

### Dot Map (Point Map / Dot Density Map)

- **Description:** Uses uniformly sized markers across a geographic area to visualize spatial data. Easy to grasp and better for overview of data, but not great for retrieving exact values.
- **Data Required:** Geographic boundaries or coordinates paired with numerical counts or scaled units.
- **Use When:** Analyzing distribution, pinpointing locations, identifying spatial patterns.
- **Variations:**
  - **Single-count format:** Each dot = one individual item
  - **Scaled format:** Each dot = a defined quantity (e.g., 10 trees per dot)
- **Similar Charts:** Bubble Map.
- **Tools:** D3.js, Plotly, Vega-Lite, AnyChart, Flourish, Datawrapper, CARTO, Plotivy, Tableau.

### Flow Map

- **Description:** Geographically shows the movement of information or objects from one location to another. Line thickness represents volume or magnitude.
- **Data Required:** Origin and destination coordinates, directional indicators, and quantitative values for flow volume.
- **Use When:** Analyzing distribution, location, and movement & flow patterns.
- **Design Guidelines:** Lines branch from origins. Use arrows for directional/incoming/outgoing data, or omit for bidirectional trade. Bundle overlapping lines; avoid crossings to minimize visual clutter.
- **Examples:** Historical migration routes, trade routes.
- **Similar Charts:** Connection Map, Sankey Diagram.
- **Tools:** AnyChart (JS), FlowmapBlue, Plotivy.

---

## Networks / Hierarchies

Charts for showing relationships and hierarchical structures.

### Arc Diagram

- **Description:** Maps network data onto a single axis. Nodes positioned linearly, linked with curved lines.
- **Data Needed:** Paired node data representing connections, plus optional metrics for link thickness (interaction frequency or co-occurrence).
- **Use When:** Identifying co-occurrences and relationship patterns within datasets.
- **Limitations:** Lacks structural clarity of traditional 2D network charts. High link density creates visual clutter.
- **Categories:** Patterns and Relationships.
- **Similar Charts:** Network Diagram.
- **Tools:** Data to Viz (R), Highcharts, Protovis, D3.js, Plotivy, RAWGraphs.

### Chord Diagram

- **Description:** Maps relationships between entities arranged around a circle. Links use curved ribbons; widths scaled to match numerical weights. Color coding separates distinct categories.
- **Data Requirements:** Entity names for circular markers, paired connection data, numerical values for link strength, categorical labels for coloring.
- **Use When:** Comparing similarities within a dataset or between different groups. Tracking comparisons and relationships.
- **Limitations:** Over-cluttering with too many connections.
- **Similar Charts:** Non-Ribbon Chord Diagram, Network Diagram.
- **Tools:** D3.js, amCharts, plotly (Python), circlize (R), Flourish, Circos, ArcGIS Insights.

### Circle Packing

- **Description:** Also known as a "Circular Treemap." Replaces rectangles with circles to map nested data. Parent nodes enclose sub-branches; circle area reflects a metric (quantity, file size); color denotes categories or additional variables.
- **Data Requirements:** Hierarchical/tree data, proportional numerical values for node sizing, optional categorical/continuous data for color mapping.
- **Use When:** Clarifying nested relationships — "reveals hierarchical structure better than a Treemap."
- **Trade-off:** Sacrifices pixel efficiency due to empty space within circles.
- **Similar Charts:** Treemap, Sunburst Diagram.
- **Tools:** D3.js, Vega, R/ggraph, matplotlib, Flourish, RAWGraphs, Plotivy.

### Network Diagram (Network Graph / Node-Link Diagram)

- **Description:** Shows how things are interconnected using nodes (circles/icons) and connecting lines (links). Additional metrics can adjust node dimensions or link thickness. Maps connected systems to interpret structural patterns like clustering and connection density.
- **Data Needed:** Entities (nodes), connection data (links), optional numerical values for node sizes or link weights.
- **Use When:** Analyzing relationship structures, identifying natural clusters, measuring how densely a group connects.
- **Variations:** Undirected (only connections) vs. Directed (arrows for one/two-way flow).
- **Limitations:** Becomes hard to read with too many nodes — limited data capacity.
- **Similar Charts:** Arc Diagram, Brainstorm, Chord Diagram, Connection Map, Tree Diagram.
- **Tools:** D3.js, Apache ECharts, Plotly, Vega, graph-tool, Sigma.js, Gephi, Cytoscape, Flourish, Cosmograph, yEd live.

### Sunburst Diagram (Sunburst Chart / Ring Chart / Multi-level Pie Chart / Radial Treemap)

- **Description:** Displays hierarchical data using concentric rings divided into slices. Center = root node; levels expand outward. Slices sized equally or proportionally to a value; colors emphasize categories.
- **Data Requirements:** Hierarchical datasets with multiple parent-child levels. Optional numerical values to scale slice angles proportionally.
- **Use When:** Illustrating hierarchy and part-to-a-whole relationships within complex datasets.
- **Similar Charts:** Donut Chart, Pie Chart, Treemap.
- **Tools:** amCharts, AnyChart, Apache ECharts, FusionCharts, D3.js, Plotly, Vega, Flourish, RAWGraph, Microsoft Office.

### Tree Diagram (Organizational Chart / Linkage Tree)

- **Description:** Maps hierarchical structures by layering segments vertically (not adjacently). Shows how a larger category divides into smaller subcategories alongside each component's share.
- **Data Needed:** Information structured to show parent-child relationships, classifications, or lineage.
- **Use When:** Tracking family descent, biological taxonomy, evolutionary origins, computer science/mathematical structures, corporate management hierarchies.
- **Key Components:** Root node (top element without superior), Nodes & Branches (connected elements with relationship lines), Leaf nodes (terminal elements without children).
- **Variations:** Dendrogram, Radial Tree Diagram.
- **Similar Charts:** Network Diagram, Circle Packing.
- **Tools:** Apache ECharts, Google Chart, JSCharting, yFiles, ZingChart, ZoomCharts, Creately, Gliffy, Google Docs, Vizzlo, yEd Live.

---

## Flows / Processes

Charts for tracking movement, transitions, and processes.

### Flow Chart

- **Description:** Maps sequential steps using connected shapes. Shows the sequential steps of a process. Clarifies how complex procedures, systems, or algorithms function.
- **When to Use:** Planning, improving, or communicating workflows. Requires details about action sequences, decision points, and procedural flow (not numerical datasets).
- **Structure & Symbols:** Curved rectangles = start/end, Lines/arrows = movement direction, Rectangles = simple instructions, Diamonds = decision moments. Labels inside each shape.
- **Orientation:** Horizontal or vertical.
- **Similar Charts:** Sankey Diagram.
- **Tools:** draw.io, Lucidchart, Visio, GoJS.

### Parallel Sets

- **Description:** Maps categorical data across multiple dimensions using colored bands to represent flow and proportions. Similar to Sankey Diagrams but omits directional arrows and splits paths at each dimension line. Band widths reflect category proportions; colors enable cross-category distribution comparisons.
- **Data Needed:** Multi-dimensional categorical data with corresponding values or proportional weights for each category combination across displayed dimensions.
- **Use When:** Tracking flow, proportions, and distributions across sequential or related categorical dimensions.
- **Academic Foundation:** Robert Kosara's 2006 and 2010 publications on interactive categorical data exploration.
- **Similar Charts:** Sankey Diagram, Parallel Coordinates Plot.
- **Tools:** amCharts, AnyChart, ECharts, D3.js, Plotly, Flourish, plotDB, SankeyMATIC, EagerEyes.
- **Example:** Energy flow mapping.

### Sankey Diagram

- **Description:** Visualizes proportional flows and quantities. Line thickness represents magnitude; paths can merge or split across stages. Color differentiates categories or tracks state changes.
- **Use When:** Tracking resource transfers or sequential workflows. Represents the transfer of energy, money, materials, or flow of any isolated system or process.
- **Data Required:** Flow metrics, quantity values, sequential stages, categorical/state-transition labels.
- **Clarification:** Frequently mistaken for Parallel Sets and Alluvial Diagrams, though they are distinct.
- **Similar Charts:** Flow Maps, Parallel Sets.
- **Tools:** D3.js, Plotly, Matplotlib, R, ECharts, FusionCharts, Google Charts, Flourish, RAWGraphs, SankeyMATIC, Plotivy, e!Sankey, SDraw, Vizzlo.

---

## Text / Content

Charts for visualizing text data.

### Word Cloud (Tag Cloud)

- **Description:** Scales text size to match occurrence rates, arranging terms in clusters or custom layouts. Color typically serves decorative or categorical roles.
- **Data Requirements:** Source text corpus to calculate frequency, plus optional attached metrics like population figures or keyword tags.
- **Use When:** Blog keyword displays, comparing separate text collections, decorative web designs.
- **Limitations:** Long words are emphasized over short words. Not great for analytical accuracy — used more for aesthetic reasons.
- **Similar Charts:** Dot Matrix Chart.
- **Tools:** Various code libraries, web generators, Tableau.

---

## Financial / Ranges

Specialized charts for financial and range data.

### Candlestick Chart (Japanese Candlestick)

- **Description:** Tracks financial price movements over time for assets (stocks, bonds, commodities). Each symbol compresses trading activity into a single interval plotted along a time axis.
- **Data Required:** Open price, closing price, highest traded price, lowest traded price, and a defined time period for the horizontal axis.
- **Use When:** Spotting market trends, interpreting daily sentiment, recognizing psychological shifts between buyers and sellers. Aids in forecasting future price action.
- **Anatomy:** Central rectangle = "real body" (gap between open/close). Upper/lower lines = "shadows" (peak/trough). Green/white = bullish (close > open); Red/black = bearish (close < open). Extended bodies = strong pressure; Compact ones = consolidation.
- **Limitations:** Does not express events between open and close. Cannot measure intra-period volatility. Must not be mistaken for box plots.
- **Similar Charts:** Kagi Chart, OHLC Chart.
- **Tools:** plotly, Vega-Lite, amCharts, Excel, Tableau.

### Kagi Chart

- **Description:** Maps an asset's supply and demand through vertical line patterns. **Time-independent** — filters out market noise to emphasize major price shifts. X-axis = key action dates (not continuous timeline); Y-axis = value.
- **Data Needed:** Historical price action data and a predetermined "reversal" threshold to trigger directional changes.
- **Use When:** Traders prioritizing significant trend analysis over time-based data. Isolates key price movements and identifies supply/demand imbalances.
- **Anatomy:** Lines extend vertically until hitting reversal threshold, then U-turn. "Shoulder" joins rising/declining lines; "waist" connects declining/rising lines. Yang line (thicker/green) = demand outpaces supply. Yin line (thinner/red) = supply outpaces demand.
- **Trading Signals:** Yin to Yang = buy signal. Yang to Yin = sell signal.
- **Similar Charts:** Candlestick Chart, OHLC Chart, Point & Figure Chart.
- **Tools:** D3.js, FusionCharts, Wolfram, Plotivy.

### Open-High-Low-Close (OHLC) Chart (Price/Bar Chart)

- **Description:** Tracks financial values across intervals. Plots a symbol representing two ranges: highest/lowest prices traded, and opening/closing price on that time period. Central vertical line = high-low span; left/right ticks = open/close values.
- **Data Required:** Open, high, low, and close prices per interval. Vertical axis = monetary value; horizontal axis = time.
- **Use When:** Tracking data over time, spotting patterns, evaluating ranges. Interpret day-to-day market sentiment and forecast future price changes.
- **Color Coding:** Bullish (close > open) vs. Bearish (close < open).
- **Similar Charts:** Candlestick Chart, Kagi Chart.
- **Tools:** Plotly, AnyChart, ECharts, plotDB, Plotivy.

### Point & Figure (P&F) Chart

- **Description:** Visualizes asset supply and demand using alternating vertical columns of Xs and Os. **Time-independent** — horizontal axis marks specific price-action dates; vertical axis tracks value. Xs = upward movements (buyers dominate); Os = downward shifts (sellers take control).
- **Data & Parameters:** Historical price data (closing, high, or low values). Two settings: **Box size** (minimum price change to add a symbol, filters minor fluctuations) and **Reversal amount** (price movement against trend to start new column).
- **Use When:** Identifying breakouts, mapping support/resistance zones, drawing trend lines. Detecting shifts in supply and demand dynamics.
- **Rules:** Columns contain either Xs or Os, never mixed. Numerical markers (1-9 for Jan-Sep, A-C for Oct-Dec) may denote monthly starts.
- **Similar Charts:** Kagi Chart.
- **Tools:** Python (mplfinance, pypnf), R (rpnf), Plotivy, StockCharts.

---

## Brainstorming / Conceptual

Charts for idea organization and conceptual mapping.

### Brainstorm (Mind Map)

- **Description:** Visual diagram organizing interconnected ideas, words, images, and concepts. Method for generating ideas, identifying associations, categorizing thoughts, structuring information, and supporting study. Commonly functions as a note-taking aid during early project phases, collaborative work, and team-building.
- **Data Requirements:** Central topic or project title, followed by related categories, subcategories, and associated keywords or concepts.
- **Use When:** Initial project planning, idea generation, organizing information, studying, collaborative sessions, boosting team morale.
- **Structure:** Central topic in a shape (circle/cloud) at center → Categories radiating outward → Subcategories on each branch → Repeat for deeper levels.
- **Functions:** Tracks concepts and relationships.
- **Tools:** yFiles (code), Coggle, MindMup, Lucidchart, Plotivy, yEd live (web), Adobe Illustrator, Apple Pages/Keynote, Microsoft Office (desktop).

### Illustration Diagram

- **Description:** Combines images with textual guides (notes, labels, legends). Graphics can be symbolic, pictorial, or realistic. May incorporate enlargements and cross-sections for deeper analysis.
- **Data Needed:** Visual inputs (illustrations, rough sketches, wireframes, photographs) combined with accompanying annotations or a legend.
- **Use When:** Illustrating concepts, showing how things work, describing objects/places, showing how things move/change, providing additional insight.
- **Tools:** Plotivy (web), Adobe Illustrator, Affinity (desktop).

---

## Dashboards / Reference

Charts for dashboards, scheduling, and reference tools.

### Bullet Graph — see Comparisons section above.

### Error Bars

- **Description:** Visual overlay for Cartesian plots communicating data variability and measurement uncertainty. Shorter markers = tightly clustered values/higher precision. Longer markers = wider dispersion/reduced reliability.
- **Data Requirements:** Ranged datasets processable into statistical metrics. Markers typically represent standard deviation, standard error, confidence intervals, or min/max values.
- **Use When:** Contextualizing precision alongside quantitative comparisons. Can be added to scatterplots, dot plots, bar charts, line graphs, area graphs, histograms, multi-set bar charts, and span charts.
- **Visual:** Cap-tipped lines extending from data points or chart edges. Parallel to the relevant quantitative axis (vertical or horizontal). Can be symmetrical or asymmetrical for skewed distributions.
- **Tools:** amCharts, AnyChart, Highcharts, Vega, Vega-Lite, ZingChart, Google Docs, Plotivy, Plotly, Apple Numbers, Microsoft Office, Tableau.

### Timetable

- **Description:** Referencing and management tool for scheduled events, tasks, and actions. Structuring information chronologically or alphabetically enables faster lookup. Commonly used for arrival/departure times of transportation.
- **Data Required:** Information on planned events, tasks, actions, and their corresponding times or sequences.
- **Use When:** Tracking data over time and functioning as a reference tool.
- **Similar Charts:** Calendar.
- **Tools:** Google Docs, Plotivy (web), Adobe Illustrator, Apple Numbers, Microsoft Office (desktop).

---

## Quick Reference: Chart Selection Guide

### By Data Type

| Data Type | Best Charts |
|---|---|
| **Single categorical + numerical** | Bar Chart, Radial Bar, Tally Chart |
| **Two numerical (distribution)** | Histogram, Density Plot, Violin Plot, Box Plot |
| **Two numerical (relationship)** | Scatterplot, Bubble Chart |
| **Three numerical** | Bubble Chart, Parallel Coordinates |
| **Four numerical** | Bubble Chart (with color), Parallel Coordinates |
| **Time series (single)** | Line Graph, Area Graph, Spiral Plot |
| **Time series (multiple)** | Line Graph (max 3-4), Stacked Area, Stream Graph |
| **Time series (tasks)** | Gantt Chart, Timeline |
| **Part-to-whole (single)** | Pie Chart, Donut Chart, Treemap |
| **Part-to-whole (hierarchical)** | Sunburst, Circle Packing, Tree Diagram |
| **Part-to-whole (multiple groups)** | Stacked Bar, Marimekko |
| **Geographic** | Choropleth, Bubble Map, Dot Map, Connection Map, Flow Map |
| **Network/Relationships** | Network Diagram, Chord Diagram, Arc Diagram |
| **Flow/Process** | Sankey Diagram, Parallel Sets, Flow Chart |
| **Text/Frequency** | Word Cloud, Tally Chart |
| **Financial/Range** | Candlestick, OHLC, Kagi, Point & Figure |
| **Ranges (non-financial)** | Span Chart, Box Plot |
| **Multivariate comparison** | Radar Chart, Parallel Coordinates |
| **Demographic** | Population Pyramid |
| **Dashboard/Metrics** | Bullet Graph |

### By Analysis Purpose

| Purpose | Best Charts |
|---|---|
| **Compare values** | Bar Chart, Bullet Graph, Radar Chart |
| **Show distribution** | Histogram, Density Plot, Violin Plot, Box Plot |
| **Show relationship** | Scatterplot, Bubble Chart, Chord Diagram |
| **Show composition** | Pie Chart, Treemap, Stacked Bar |
| **Show trend over time** | Line Graph, Area Graph, Stream Graph |
| **Show geographic pattern** | Choropleth, Dot Map, Flow Map |
| **Show network structure** | Network Diagram, Arc Diagram |
| **Show flow** | Sankey Diagram, Parallel Sets |
| **Show hierarchy** | Tree Diagram, Sunburst, Treemap |
| **Show process** | Flow Chart, Gantt Chart |
| **Show text frequency** | Word Cloud |
| **Show financial data** | Candlestick, OHLC, Kagi |
| **Show uncertainty** | Error Bars, Box Plot |
| **Show ranges** | Span Chart, Box Plot, Gantt Chart |

---

## Tools Reference by Category

### Code Libraries

| Library | Best For |
|---|---|
| **D3.js** | Nearly all chart types — most comprehensive |
| **Plotly** | Interactive charts, scientific/financial |
| **ggplot2 (R)** | Statistical plots, distributions |
| **Matplotlib/Seaborn (Python)** | Distributions, scientific plots |
| **Chart.js** | Web dashboards, common charts |
| **Vega/Vega-Lite** | Declarative visualization grammar |
| **ECharts** | Rich chart types, flows, networks |

### Web Apps

| Platform | Best For |
|---|---|
| **Flourish** | Nearly all types, animations |
| **Datawrapper** | Clean, publication-ready charts |
| **RAWGraphs** | Complex charts (Sankey, Treemap, Chord) |
| **Plotivy** | Quick prototyping, many chart types |

### Desktop Software

| Software | Best For |
|---|---|
| **Excel** | Common charts (bar, line, pie, scatter) |
| **Tableau** | Dashboards, geospatial, advanced analytics |
| **Power BI** | Business dashboards |
| **Adobe Illustrator** | Custom illustration diagrams |

---

*Generated from [Data Visualization Catalogue](https://datavizcatalogue.com) — 60 chart types cataloged.*
