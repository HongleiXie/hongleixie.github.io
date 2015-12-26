---
layout: post
title:  "Simple Data Visualizations in R - Bar charts"
date:   2015-12-25
---

It's my first time intending to give up `ggplot2`, when I saw the amazing(狂霸酷炫拽) plotting galleries using `googleVis` and `rCharts`.

Here is a simple bar charts example and you can run the code locally to see the difference.

```r
library(ggplot2);
library(googleVis);
library(rCharts);

### sample data ###################################################
data <- data.frame(month = c('AUG', 'SEP', 'OCT', 'NOV', 'DEC'),
                   var1 = c(7 ,2,1,14,8),
                   var2 = c(2,1,4,1,2)
                    );

data$target <- 9;

#### googlevis ####################################################
googlevis <- gvisComboChart(data = data, xvar = 'month', 
                         yvar = c('var1', 'target','var2'),
                         
                         options=list(seriesType="bars",
                                      seriesType="bars",
                                      title="Test Title",
                                      
                                      series='{1: {type:"line"}}')
                          );

plot(googlevis);

### rCharts #######################################################

# same dataset but need more manipulation if using rCharts or ggplot2

data <- data.frame(month = c('AUG', 'SEP', 'OCT', 'NOV', 'DEC', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC'),
                   
                   type = c(rep('var1', 5), rep('var2', 5)),
                   count = c(7,2,1,14 ,8,2,1,4,1,2)
                   );


n2 <- nPlot(count ~ month, 
            group = "type", 
            data = data, 
            type = 'multiBarChart'
            );

n2$chart(showControls = F);
n2$set(width = 500, height=400);
n2$chart(reduceXTicks = FALSE);
n2$yAxis(axisLabel = 'count');
n2$templates$script <- "http://timelyportfolio.github.io/rCharts_nvd3_templates/chartWithTitle_styled.html";
n2$set(title = 'TITLE');
n2$yAxis(tickFormat = "#!
        function(d) {return d3.format('.')(d)}
        !#"
        );
n2;

### ggplot2 #######################################################
positions <- c('AUG', 'SEP', 'OCT', 'NOV');

ggplot(data=data, aes(x=month, y=count, fill=type, ymax = max(count)+1)) +  
  xlab("Meeting Date") + ylab("Number of Calls") +
  geom_bar(stat="identity", position=position_dodge())+
  scale_x_discrete(limits = positions)+
  theme_bw()+geom_hline(yintercept = 9)+
  geom_text(aes(label = count,y = count, x=month),size = 5, position = position_dodge(width=0.9));


```

Check out the example pages for package [`rCharts`](http://rcharts.io/gallery/) and [`googleVis`](http://rcharts.io/gallery/).

