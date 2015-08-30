# The impact of weather events across the USA
Sunday, June 22, 2014  

## Synopsis
This document briefly analyzes the impact of different weather events that occured
in the United States between 1950 and 2011. More specifically, the economic
damage as represented by the damage to property and the damage to population 
health as represented by the number of injuries and fatalities are considered. 
Since the purpose of the analysis is to help in understanding differences in
weather events, the events are compared based on the results of the damage analysis. 
By far the largest harm to population health was done by tornados. A second 
group of events that are relatively harmful to health is comprised of Excessive Heat,
Thunderstorms, Floods, and Lightning.

## Data processing

```r
setwd(dir = "~\\GitHub\\Storms\\")
# download.file(url = "http://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", 
# destfile = "stormdata.csv.bz2")
# unzip
# library(R.utils)
# datadir <- paste0(getwd(), "/data")
# bunzip2("stormdata.csv.bz2", destname=paste0(datadir, "/stormdata.csv"), remove=F)
stormdata <- read.csv("~/GitHub/Storms/data/stormdata.csv")

# For economic damage the EXP in the data frame needs to be taken into account.
# The damage has to be multiplied accordingly (e.g. k = multiply by 1000)
propertymulti <- rep(0, times = nrow(stormdata))
for (i in 1:nrow(stormdata)){
      if (stormdata$PROPDMGEXP[i] == "h") propertymulti[i] <- 100
      if (stormdata$PROPDMGEXP[i] == "H") propertymulti[i] <- 100
      if (stormdata$PROPDMGEXP[i] == "k") propertymulti[i] <- 1000
      if (stormdata$PROPDMGEXP[i] == "K") propertymulti[i] <- 1000
      if (stormdata$PROPDMGEXP[i] == "m") propertymulti[i] <- 1000000
      if (stormdata$PROPDMGEXP[i] == "M") propertymulti[i] <- 1000000
      if (stormdata$PROPDMGEXP[i] == "b") propertymulti[i] <- 1000000000
      if (stormdata$PROPDMGEXP[i] == "B") propertymulti[i] <- 1000000000
}
propertydamage <- stormdata$PROPDMG * propertymulti
stormdata <- cbind(stormdata, propertydamage)
```

## Results

```r
# Which are the most harmful events regarding health?
fatalities <- tapply(X = stormdata$FATALITIES, INDEX = stormdata$EVTYPE, FUN = sum)
fatalities2 <- fatalities[which(fatalities>100)]
fatalities2 <- sort(fatalities2, decreasing = T)

fatality_evtypes <- names(fatalities2)

injuries <- tapply(X = stormdata$INJURIES, INDEX = stormdata$EVTYPE, FUN = sum)
injuries2 <- injuries[names(injuries) %in% fatality_evtypes] 
# same EVTYPE as in fatalities
injuries2 <- sort(injuries2, decreasing = T)

# Plot fatality and injury values depending on event type
library(ggplot2)
library(reshape2)
m_injuries2 <- melt(injuries2, value.name="Injuries")
m_fatalities2 <- melt(fatalities2, value.name="Fatalities")
plotdata <- merge(m_fatalities2, m_injuries2, by = "Var1")
colnames(plotdata)[1] <- "Eventtype"
plotdata <- melt(plotdata, id.vars = c("Eventtype"), value.name = "Amount", 
variable.name = "Damagetype")
#plotdata <- plotdata[order(plotdata$Amount, decreasing = T), ] # order

ybreaks <- 100 * 2^(0:10)
healthp <- ggplot(plotdata, aes(x=reorder(plotdata$Eventtype, plotdata$Amount, FUN=sum, order=T), y=Amount)) + 
geom_point(aes(colour=Damagetype), stat="identity", size=5) +
scale_y_sqrt(breaks = ybreaks) + xlab("Eventtype") + 
opts(axis.text.x=theme_text(angle=-90)) + 
ggtitle("Amount of fatalities and injuries depending on the event type")
```

```
## 'opts' is deprecated. Use 'theme' instead. (Deprecated; last used in version 0.9.1)
## theme_text is deprecated. Use 'element_text' instead. (Deprecated; last used in version 0.9.1)
```

```r
print(healthp)
```

![plot of chunk unnamed-chunk-2](./Stormpaper_files/figure-html/unnamed-chunk-2.png) 
As already pointed out in the synopsis the most harmful events are tornados,
excessive heat, thunderstorms, floods, and lightning. The events are a little
different with respect to the relative amounts of fatalities and injuries.
In general, injuries and fatalities are positively correlated and usually the
number of injuries is higher than the number of fatalities. 


```r
## Which are the most economically harmful events?
econdamage <- tapply(X = stormdata$propertydamage, INDEX = stormdata$EVTYPE, FUN = sum)
econdamage2 <- econdamage[which(econdamage>10**10)]
econdamage2 <- sort(econdamage2, decreasing = T)
econdamage2
```

```
##             FLOOD HURRICANE/TYPHOON           TORNADO       STORM SURGE 
##         1.447e+11         6.931e+10         5.694e+10         4.332e+10 
##       FLASH FLOOD              HAIL         HURRICANE 
##         1.614e+10         1.573e+10         1.187e+10
```
With respect to the damage to property tornados are less important. Floods 
caused roughly twice the damage of the second most dangerous event. 