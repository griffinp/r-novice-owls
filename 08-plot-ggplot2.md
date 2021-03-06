---
layout: page
title: R for reproducible scientific analysis
subtitle: Creating publication quality graphics
minutes: 15
---

> # Learning Objectives {.objectives}
>
> * To be familiar with the base plotting system functions
> * To be able to use ggplot2 to generate publication quality graphics
> * To understand the basics of the grammar of graphics:
>   - The aesthetics layer
>   - The geometry layer
>   - Adding statistics
>   - Transforming scales
>   - Coloring or panelling by groups.
>

Plotting our data is one of the best ways to
quickly explore it and the various relationships
between variables.

There are three main plotting systems in R,
the [base plotting system][base], the [lattice][lattice]
package, and the [ggplot2][ggplot2] package.

[base]: http://www.statmethods.net/graphs/
[lattice]: http://www.statmethods.net/advgraphs/trellis.html
[ggplot2]: http://www.statmethods.net/advgraphs/ggplot2.html

Today we'll touch on the base plotting system so you are familiar
with some of the ways you can customise plots in R, and 
build on what you've learnt about accessing data elements, functions
and ***. 
Then we'll focus on the ggplot2 package, because it is the most 
effective for creating publication quality graphics.

Let's start off with an example:

~~~ {.r}
boxplot(owls$NegPerChick~owls$FoodTreatment)
~~~

There are lots of ways we can customise this plot. We could
add the result of a statistical test. Let's perform an 
analysis of variance to test whether there's a significant
effect of food treatment on the amount of negotiation per chick.

~~~ {.r}
aov1 <- aov(NegPerChick~FoodTreatment, data=owls)
summary(aov1)
               Df Sum Sq Mean Sq F value   Pr(>F)    
FoodTreatment   1   90.4   90.42   36.77 2.36e-09 ***
Residuals     597 1468.1    2.46                     
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
~~~

Say we want to access the P-value from this `aov1` object. 
Let's examine its structure:

~~~ {.r}
str(summary(aov1))
List of 1
 $ :Classes ‘anova’ and 'data.frame':	2 obs. of  5 variables:
  ..$ Df     : num [1:2] 1 597
  ..$ Sum Sq : num [1:2] 90.4 1468.1
  ..$ Mean Sq: num [1:2] 90.42 2.46
  ..$ F value: num [1:2] 36.8 NA
  ..$ Pr(>F) : num [1:2] 2.36e-09 NA
 - attr(*, "class")= chr [1:2] "summary.aov" "listof"
~~~

It is a list of length 1 that contains 5 other lists.

#### Challenge 1 {.challenge}
Figure out how to extract the P-value from the object `summary(aov1)`.
Assign it to a new variable called `pval`.
####

Now let's add this as text to our boxplot.

~~~ {.r}
text(x=1, y=1, labels=paste("P =", pval))
~~~

#INSERT IMAGE 2

~~~ {.r}
# Adjust the position
text(x=1.3, y=6, labels=paste("P =", pval))
~~~

INSERT IMAGE 3

~~~ {.r}
# Round the p-value down to 3 significant figures
text(x=1.3, y=6, labels=paste("P =", signif(pval, 3)))
~~~

INSERT IMAGE 4

~~~ {.r}
# Notice each text object has been added to the original plot.
# Re-plot the whole thing with just the text you want:
boxplot(owls$NegPerChick~owls$FoodTreatment)
text(x=1.3, y=6, labels=paste("P =", signif(pval, 3)))
~~~

INSERT IMAGE 5

We can use the `plot` command to make a scatterplot (its default).

~~~ {.r}
plot(owls$NegPerChick~owls$ArrivalTime)
~~~

INSERT IMAGE 6

~~~ {.r}
plot(owls$NegPerChick~owls$ArrivalTime, col="red", pch=19)
~~~

INSERT IMAGE 7

#Helpful tip#
Most of the parameters you can adjust by passing optional 
arguments to `plot` are not actually detailed in the 
`plot` help document! Instead they are listed in the help document 
for `par` ('parameters').

#### Challenge 2 {.challenge}
Consult the `par` help document and figure out how to orient
both sets of axis labels horizontally instead of parallel to 
their axis.
####

Now let's think again about the data we're plotting. 
`str(owls)` shows us that we've got data here from a lot of 
different nests - 27, in fact. Perhaps we want to explore
each nest separately. 

#### Challenge 3 {.challenge}
Discuss with your two neighbours how you might go about
exploring the data 'by nest'. What are your options?
####

There are a lot of ways you could choose to display
your data organised by nest. We will use the `ggplot2` library
to demonstrate one of them. 


~~~ {.r}
library(ggplot2)
plot1 <- ggplot(data=owls, aes(x=ArrivalTime, y=NegPerChick)) +
         geom_point()
plot1
~~~

![](img/ggplot-ex1.png)

Ggplot is based on two principles:

 * a grammar of graphics: a set of terms to define the basic components
   of a plot that can be used to produce figures in a coherent, consistent,
   and flexible way
 * Layers: Different basic components can be logically separated into layers
   which can be overlaid on top of each other.

In the example, we told `ggplot` to use the owls dataset. As second argument
we gave it the `aes` function, which stands for *aesthetics*. The `aes` function
specifies how your data are represented visually: which variables to plot on 
each axis, as well as the color, size, shape, and transparency, which we'll change
soon. We then used the `+` operator to add a *geometry* layer,
specified by a function starting with `geom`,
in this case, a layer of points. 

This is a general rule.
Anything we specify inside the first function, `ggplot`, becomes a global setting for
the figure. Anything that we add with `+` is built up in layers (consecutively).

Next, lets color the points by nest:

~~~ {.r}
plot1 <- ggplot(data=owls, aes(x=ArrivalTime, y=NegPerChick)) +
         geom_point(aes(colour=Nest))
plot1
~~~

![](img/ggplot-ex2.png)

Here, we've specified that we want change the color *aesthetic* for the points layer.
We've told it to color the points based on the nest column. We can also change
the shape of the points (here by the sex of the parent owl):


~~~ {.r}
plot1 <- ggplot(data=owls, aes(x=ArrivalTime, y=NegPerChick)) +
  geom_point(aes(color=Nest, shape=SexParent))
plot1
~~~

![](img/ggplot-ex3.png)

Currently it's hard to see the relationship between the points because there are so many
levels of 'Nest'. 

Instead of plotting all nests on the same axes, we can add a facet

~~~ {.r}
ggplot(data = gapminder, aes(x = lifeExp, y = gdpPercap)) +
  geom_point(aes(color=continent, shape=continent)) +
  scale_y_log10()
~~~

![](img/ggplot-ex4.png)

We can also add statistical transformations and data summarisations to our plots. We'll 
add the fits of linear models for each group by adding the `geom_smooth` geometry layer:

~~~ {.r}
ggplot(data = gapminder, aes(x = lifeExp, y = gdpPercap)) +
  geom_point(aes(color=continent, shape=continent)) +
  scale_y_log10() + geom_smooth(method="lm")
~~~

![](img/ggplot-ex5.png)

Whoops that didn't quite do what we wanted. That's because the grouping has only been
applied to the `geom_point` layer. Let's change that so it's a global option:

~~~ {.r}
ggplot(data = gapminder, aes(x = lifeExp, y = gdpPercap, color=continent)) +
  geom_point() + scale_y_log10() + geom_smooth(method="lm")
~~~

![](img/ggplot-ex6.png)

It's still hard to see what's going on, That's because we're currently plotting all
years of collection. Another useful thing we can do with ggplot is *facet* our data:
create panels based on different groups. 

Let's take a look at how life expectancy has changed over time for each country,
and create a panel, or facet, for each continent:

~~~ {.r}
ggplot(data= gapminder, aes(x=year, y=lifeExp, color=country)) +
  geom_line() + facet_grid(. ~ continent)
~~~

![](img/ggplot-ex7.png)

We can see nice upward trends across all countries across all continents, with a 
few unfortunate downward spikes corresponding to wars.

The `facet_grid` layer took a formula as its argument, the `.` corresponds to 
"all variables in the data", and by putting "continent" on the right of the 
formular, we told ggplot to arrange the panels along the x axis. If we put
"continent" in the y-position of the formula, the panels would have been 
arranged along the y-axis.

Currently, there are too many different countries for the legend to render 
properly. Since we don't care so much about the particulars of each country,
lets turn the legend off:

~~~ {.r}
ggplot(data= gapminder, aes(x=year, y=lifeExp, color=country)) +
  geom_line() + facet_grid(. ~ continent) + theme(legend.position="none")
~~~

![](img/ggplot-ex8.png)

To save an image, we can either use the save button inside of RStudio,
or from the interactive console (or script) using the `ggsave` function:

~~~ {.r}
ggsave("lifeExp-vs-time.pdf")
~~~

This is just a taste of what you can do with `ggplot2`. A more detailed tutorial
can be found [here](ggplot.pdf). Once you get a handle of the different layers
and the grammar of graphics, the [reference list of functions][ref] will be 
useful.

[ref]: http://docs.ggplot2.org/current/

> #### Challenge 1 {.challenge}
>
> Create density plots of GDP per capita, colored by continent.
> Hints:
>  - Use `ggplot` to set up the basic plot.
>  - Use `aes` to tell ggplot what the axes of the plot are (you will only need the x-axis).
>  - Use `aes` to specify the color grouping.
>  - The geometry layer for density plots is `geom_density`.
> 
> Advanced:
>  - The `fill` aesthetic will color the area under the curve.
>  - Transform the scale of the x-axis to more easily visualise the difference 
>    between continents
>

> #### Challenge 2 {.challenge}
>
> Add a facet layer to panel the density plots by year. Hint: [facet_wrap][fw]
> will be more useful than `facet_grid`.
>

[fw]: http://docs.ggplot2.org/current/facet_wrap.html

