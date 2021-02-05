hw1
================

``` r
library(data.table)
library(tidyverse)
library(ggplot2)
library(ggrepel)
```

### Downloaded all 14249 observations and 123 variables for Wisconsin in 2019

``` r
d19wi <- fread('https://www.fhwa.dot.gov/bridge/nbi/2019/delimited/WI19.txt')

fipscode <- paste0(d19wi$STATE_CODE_001,str_sub(paste0("00", d19wi$COUNTY_CODE_003), -3, -1)) # construct a 5-digit fipscode
coln <- cbind(names(d19wi)) # identify index of columns in interest
```

### A new file with selected variables

``` r
newdf <- data.frame(d19wi[,c(2, 27, 31, 96, 106, 117)],fipscode,d19wi[,c(67:69,24:25, 41:42,35)])
colnames(newdf) <- c("BridgeID","YearBLt","YearADT","YearIMP","YearRCT","YearFADT","Fipscode","Deck","Superstructure","Substructure","Maintenance","Owner","History","Navigation","DegreesSkew")
```

There are five variables relevant to year in the data-frame (newdf).
Based on the data description included in
[link](https://www.fhwa.dot.gov/bridge/mtguide.pdf), the meaning of the
five variables are different.  
1. YearBLT: the year the bridge was built.  
2. YearADT: the year represented by the recording of average daily
traffic.  
3. YearIMP: the base year for the total project cost.  
4. YearRCT: the year the bridge was reconstructed.  
5. YearFADT: the projected year of future average daily traffic.

### Interested data

``` r
indf <- filter(newdf, Deck == 9, Superstructure == 9, Substructure == 9)
all(indf$Maintenance==indf$Owner, indf$History==5)
```

    ## [1] TRUE

### Plot

``` r
means <- aggregate(DegreesSkew ~ Navigation, indf, mean)
medians <- aggregate(DegreesSkew ~ Navigation, indf, median)

ggplot(indf,aes(Navigation,DegreesSkew)) +
  geom_boxplot() +
  stat_summary(fun=mean, geom="point", shape=20, size=3, color="red", fill="red") +
  geom_text(data = means, aes(label = paste("Mean =", DegreesSkew), y = DegreesSkew + 2.5), size = 3) +
  geom_text(data = medians, aes(label = paste("Median =", DegreesSkew), y = DegreesSkew + 1.5), size = 3) +
  geom_text_repel(data=subset(indf, DegreesSkew>=40), aes(label=paste("(ID:",BridgeID, ", Fipscode:", Fipscode,")")), size = 3)
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->
