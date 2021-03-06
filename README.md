

---
output: html_document
---

TRAVEL TO PRIMARY ELECTION FOR 2016 Election contribution in Alabama by Jay Cheong
========================================================
```{r global_options, include=FALSE}
knitr::opts_chunk$set(echo=FALSE, warning=FALSE, message=FALSE, fig.width=10, fig.height=7)
```

```{r}
# knitr::opts_chunk$set(echo=TRUE, warning=FALSE, message=FALSE)

# ```{r, warning=FALSE, echo=FALSE}
#
# ```{r echo=FALSE, message=FALSE, warning=FALSE, packages}
# Load all of the packages that you end up using
# in your analysis in this code chunk.

# Notice that the parameter "echo" was set to FALSE for this code chunk.
# This prevents the code from displaying in the knitted HTML output.
# You should set echo=FALSE for all code chunks in your file.
library(ggplot2)
library(maps)
library(scales)
library(xlsx)
library(leaflet)
library(dplyr)
library(reshape2)
library(gridExtra)
library(stringr)
# library(ggally)
require(lubridate)


title_xy <- function(plt, title,xnm, ynm){
  plt <- plt +
    ggtitle(title)+
    labs(x = xnm, y = ynm)

  return(plt)
}


# Top ten function

topten <- function(name){
  val1 <- table(name)
  val1 <- val1[order(val1, decreasing = TRUE)][1:10]
  val1 <- as.data.frame(val1)
  return(val1)
}


# Load the Data
getwd()
setwd('~/desktop/udacity/project4')
dt <- read.csv('AL.csv', row.names = NULL)

# Change the column names
dt <- setNames(dt, c(names(dt)[2:19]))


#Generate modifide date column
dt$modifiedDate <- as.Date(dt$contb_receipt_dt, format = "%d-%b-%y")
# dt$Day <- format(dt$modifiedDate, "%d")
# dt$Month <- format(dt$modifiedDate, "%m")
# dt$Year <- format(dt$modifiedDate, "%y")

# short summary
# length(head(dt))

summary(dt)

# head(dt)
```

##### 23645 people choice recorded for election 23 columns information saved


# Univariate Plots Section

```{r}
# Alabama election candidate popularity

##### 21 candidates was on the election

##### Candidate popularity in Alabama

# cand_levels <- factor(levels = unique(dt$cand_id))

# 1. make the candidate list in order by popularity
cand_order <- table(dt$cand_nm)
# cand_order
# order(cand_order, decreasing = TRUE)
cand_order <- cand_order[order(cand_order, decreasing = TRUE)]
# cand_order
cand_order_name <- names(cand_order)
# cand_order_name
dt$cand_nm <- ordered(dt$cand_nm, levels = c(cand_order_name))

# cand_order_name
# dt$cand_nm
# levels(dt$cand_id)

# 2. plot the candidate popularity
cdt_ppl <- qplot(data=dt, x = cand_nm, main = 'Candidate Popularity in AL')+
  theme_bw()+
  theme(axis.text.x = element_text(angle = 90))+
  xlab('Candidate Name')

cdt_ppl + coord_cartesian(xlim = c(1,5))
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/1.png)

##### The result presented Top five candidates by contribution count. Ted Cruz was the first place and Bernard Sanders was the second place.Clinton Hillary took 4th place


##### Then, Let's see the economical index in AL, where is the most popular place to live in AL and how much they contributed for their candidate.


```{r}


##### Where is the biggest Contributor's City  
#make a table for the order of contributer's City

# tb_contbr_city <- table(dt$contbr_city)
# tb_contbr_city <- tb_contbr_city[order(tb_contbr_city, decreasing = TRUE)]
# dt$contbr_city <- ordered(dt$contbr_city, levels = names(tb_contbr_city))

contbr_ct <- topten(dt$contbr_city)
contbr_ct <- setNames(contbr_ct, c("City", "Count"))
ggplot(contbr_ct, aes(City, Count),
       main = "Top ten cities contribution")+geom_bar(stat = "identity")+
  theme(axis.text.x = element_text(angle = 90))

```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/2.png)


##### There are too many cities existed but I presented top ten big cities in AL from the data. So, I think we could figure out who is popular in AL by comparing those top ten cities because they could represent the Alabama people

```{r}

histogram <- ggplot(data = subset(dt, dt$contb_receipt_amt>0), aes(contb_receipt_amt))+
  geom_histogram(bins = 10)+
  xlab("Contribution")

shapiro.test(ggplot_build(histogram)$data[[1]]$count)

histogram1 <- histogram + scale_x_sqrt()
shapiro.test(ggplot_build(histogram1)$data[[1]]$count)

histogram2 <- histogram + scale_x_log10()
shapiro.test(ggplot_build(histogram2)$data[[1]]$count)

grid.arrange(histogram, histogram1, histogram2, nrow = 3)

```

contribution amount is highst arount $70 in total.
Almost of contribution amount below $2500


```{r}
# contribution amount

cnt_amt <- ggplot(data = dt, aes(cand_nm, contb_receipt_amt))+
  geom_bar(stat = "summary", fun.y = sum)+
  theme(axis.text.x = element_text(angle = 90))+
  xlab("candidate")+ylab("contribution amount")+
  labs(title = "Candidate contributed amount")+
  scale_y_continuous(labels = comma)


```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/3.png)

##### This plot shows how much each candidate were contributed and the order based on the popularity level. Ted took the first place as expected but sanders was second place in popularity but he didn't get much amount of money in the election even less than hillary, rubio.

```{r}
contbr_emp <- table(dt$contbr_employer)
contbr_emp <- contbr_emp[order(contbr_emp, decreasing = TRUE)][1:10]
contbr_emp <- as.data.frame(contbr_emp)
contbr_emp <- setNames(contbr_emp, c("Employer","Count"))

ggplot(contbr_emp, aes(Employer, Count), main = "Contributor Employer")+
  geom_bar(stat = "identity")+
  theme(axis.text.x = element_text(angle = 90))+
  scale_x_discrete(labels = function(Employer) str_wrap(Employer, width = 15))

```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/4.png)

#####Plot is for contributor's job. what kinds of people supported their candidate. Mostly, retired people supported a lot more than any other employees. it is kind of surprising.


```{r}


# Top ten Contributer name
# qplot(data = dt, x = contbr_nm, color = I('black'), fill = I('#099DD9'))+
#   theme(axis.text.x = element_text(angle = 90, size = 1))

contbr_name <- topten(dt$contbr_nm)
contbr_name <- setNames(contbr_name, c("Contributor","Count"))
ggplot(contbr_name, aes(Contributor, Count), main = "top ten contributor") +
  geom_bar(stat = "identity")+
  theme(axis.text.x = element_text(angle = 90))+
  scale_x_discrete(labels = function(Contributor) str_wrap(Contributor, width = 15))
```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/5.png)

#####This plot is for a contributor's name along with the frequency. I just could figure out one contributor, named GUEVARA, supported some candidates a lot more than others.

```{r}

his_pc <- hist(dt$modifiedDate, breaks = 30, xlab = "Date")
his_pc$counts

shapiro.test(his_pc$counts)


```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/6.png)


Tried to use shapiro on the histogram.
W = 0.84241, p-value = 0.001017
p-value not distributed well. almost closed to zero.


```{r}

# contribution by date
cont_date <- qplot(data = dt, x = modifiedDate,
                   geom = 'histogram', binwidth = 15, main = "Contribution date")
# head(cont_date)
cont_date + scale_x_date(labels = date_format("%y/%m/%d"),
                         breaks = date_breaks("month")) +
  theme(axis.text.x = element_text(angle = 90))

qplot(data = dt, x = modifiedDate, bins = 30, main = "Contribution date")

hist_date <- ggplot(data = dt, aes(modifiedDate, ..count..),
                    main = "Contribution date") + geom_histogram(binwidth = 3)

```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/7.png)

![alt tag](https://github.com/youngkil9999/Project4/blob/master/8.png)

#####Adjust several different bins of x scale of date. I could figure out end of each month contribution frequency was getting peak. As is on the first plot, I could know mostly much of the contributed money from end of every month. I am not sure why but I got my paystub almost end of month. So, they have money enough to send their candidate.


#Univariate Analysis

###What is the structure of your dataset?
#####There are 23 variables and 23645 objects. some of them are useful data.For example,There are candidate name, contributed cost, receipt date, contributor city, contributor job, contributor company etc.. and those data I mentioned above is useful for analyze it. the others like id, memo was not necessarily included in the analysis.

#####A lot Contributed >>>>>>>>>>>>>>>>>>>>>>>>>>> A few Contributed

#####Candidate : Ted cruze, Sanders, Carson, Hillary, Rubio... Stein

#####Year : 2016,2015

#####City : Birmingham, huntsville, Mobile, Montgomery....etc



###What is/are the main feature(s) of interest in your dataset?

#####I dont know politics well but just heard that rich people support hillary and the other side supports Sanders. Mostly, I will figure out the relation between cost and other variables such like candidate, contributor, city.

###What other features in the dataset do you think will help support your investigation into your feature(s) of interest?

#####Candidate, Contributor, Cost, City, Receipt date, Contributor company, Contributor Position

###Did you create any new variables from existing variables in the dataset?

#####Yes, I created Modified Date, Year, Month, Day.

###Of the features you investigated, were there any unusual distributions? Did you perform any operations on the data to tidy, adjust, or change the form of the data? If so, why did you do this?

#####I changed several times on the date bin and noticed that end of month normally people contributed a lot of times. It could be the date is the monthly salary date for me so that people may feel send money to their candidate when they have enough money.



# Bivariate Plots Section

```{r}
dt$year = format(dt$modifiedDate, "%Y")
dt$month = format(dt$modifiedDate, "%m")
dt$day = format(dt$modifiedDate, "%d")

date_plot1 <- qplot(data = subset(dt,year == 2015),
      x = day , color = I('black'), fill = I('red'))+
  facet_wrap(~month, ncol = 3) +
  theme(axis.text.x = element_text(angle = 90, size = 5))

date_plot2 <- qplot(data = subset(dt, year == 2016),
      x = day, color = I('black'), fill = I('red'))+
  facet_wrap(~month, ncol = 3) +
  theme(axis.text.x = element_text(angle = 90, size = 5))

grid.arrange(date_plot1, date_plot2, ncol = 2)

```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/9.png)

#####Picture above is for 2015 and 2016 monthly contribution frequency. 2016 people contributed a lot more than 2015 because the election is coming almost around the corner

```{r}

# list(dt$contbr_nm)
dt_zero <- subset(dt, dt$contb_receipt_amt>=0)
# dt_zero
name_tb <- aggregate(dt_zero$contb_receipt_amt,
                     by = list(dt_zero$contbr_nm), sum)
name_tb <- as.data.frame(name_tb)
name_tb <- setNames(name_tb, c("name", "amount"))
name_tb <- arrange(name_tb, desc(amount), name)

# top 10
name_tb$name[1:10]
# Bottom 10
name_tb$name[(length(name_tb$name)-9):length(name_tb$name)]

# name_tb[1:10,]

# ggplot(data = subset(dt, contb_receipt_amt >3000),
#        aes(contbr_nm, contb_receipt_amt))+
#   geom_jitter(stat = "summary", fun.y = sum ,alpha = 1/10, color = 'red')+
#   theme(axis.text.x = element_text(angle = 90, size = 1))

# contribution top 10 bottom 10  
# table(dt$contb_receipt_amt)
# biplot1 <- ggplot(data = , aes(name_tb$name[1:10], contb_receipt_amt)) +
#   geom_bar(stat = "summary", fun.y = sum ,fill = 'red') +
#   theme(axis.text.x = element_text(angle = 90))+
#   xlab("Contributor")+ ylab("Amount")
biplot1 <- ggplot(data = name_tb[1:10,], aes(name, amount))+
  geom_bar(stat = "identity")+
  theme(axis.text.x = element_text(angle = 90))

#
# ggplot(data = subset(dt, contb_receipt_amt< 100 & contb_receipt_amt > 0), aes(contbr_nm, contb_receipt_amt)) +
#   geom_jitter(stat = "summary", fun.y = sum, alpha = 1/10, color = 'green')+
#   theme(axis.text.x = element_text(angle = 90, size = 1))

biplot2 <- ggplot(data = name_tb[(length(name_tb$name)-9):length(name_tb$name),],
                  aes(name,amount))+ geom_bar(stat = "identity")+
  theme(axis.text.x = element_text(angle = 90))

# biplot2 <- ggplot(data = subset(dt, contb_receipt_amt < 3 & contb_receipt_amt>0),
#                   aes(contbr_nm, contb_receipt_amt)) +
#   geom_bar(stat = "summary", fun.y = sum, fill = 'green')+
#   theme(axis.text.x = element_text(angle = 90, size = 1))+
#   xlab("Contributor") + ylab("Amount")


grid.arrange(biplot1, biplot2, ncol = 2)
```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/10.png)

##### Changed the raw data to subset of contribution amount more than $0 and removed the refunded amount of money from the data. I wanted to see countribution amount of 10 people from the top and from the bottom. top 10 people are almost 1000~3000 times much more amount of contribution than 10 people from the bottom

```{r}
# geom_line(fun.y = mean, stat = "summary")

# ggplot(data = dt, aes(cand_nm, contbr_occupation)) + geom_point(alpha = 1/5)


dt_cand_cost <- setNames(aggregate(dt$contb_receipt_amt,
                                   list(dt$cand_nm), sum),
                         c("Candidate","Contribution"))

dt_cand_cost <- dt_cand_cost[order(dt_cand_cost$Contribution, decreasing = TRUE),]

dt_cand_cost$Candidate <- ordered(dt_cand_cost$Candidate,
                                  levels= dt_cand_cost$Candidate)

cont_point <- ggplot(data=dt_cand_cost, aes(Candidate, Contribution)) +
  geom_bar(stat = "identity", fill = "red")+
  theme(axis.text.x = element_text(angle = 90))+
  xlab('Candidate Name')+
  ylab('Money, candidate received from the election campaign($)')+
  scale_y_continuous(labels = comma, trans = 'sqrt')

cont_bar <- ggplot(data=dt, aes(cand_nm, contb_receipt_amt)) +
  stat_summary(geom = "bar", fun.y = median, mapping = aes(fill = "Median")) +
  geom_point(stat = "summary", fun.y = mean, mapping = aes(color = "Mean")) +
  theme(axis.text.x = element_text(angle = 90)) +
  xlab('Candidate Name') +
  ylab('Median, Mean Money, candidate \nreceived from the election campaign($)') +
  scale_y_continuous(labels = comma)+
  scale_color_manual(name = "", values = c("Blue"))+
  scale_fill_manual(name = "", values = c("red"))
  # geom_line(stat = "summary", fun.y = mean)

grid.arrange(cont_point, cont_bar, ncol = 2)
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/11.png)


#####Ted cruz was supported the most but average contribution amount is less than $100. on the contrary, Jeb bush was contributed from the 5th on the list but the average amount is around $1,000 and the median value seems like $250. maybe rich people like to support Bush.

##### Ted Cruze earned the most highest campaign contribution in AL but surprisingly, Hillary Clinton was 4th place on the popularity but she earned the second highest campaign contribution expense in AL. It explains small number of people contributed a lot of money for her. Sanders was No.3 in popularity plot but took over 6th place on the contributed money plot. It meant lots of people supported him but not actually contributed big money to him. So I could expect that might be true that rich people supported hillary and poor people supported Sanders. Trump is out of the rank in both plots

```{r}
ggplot(data = dt, aes(cand_nm, contb_receipt_amt)) +
  geom_boxplot()+
  theme(axis.text.x = element_text(angle = 90), legend.position = "none")+
  scale_y_continuous(labels = comma, trans = 'sqrt')+
  ggtitle("Candidate Contribution box plot")+
  labs(x = "Candidate", y = "Amount")
```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/12.png)

##### box plot which shows the similar plot as above, median, mean.

```{r}

# Contribution date and money relation with Scatter plot
log_date <- ggplot(dt, aes(modifiedDate, contb_receipt_amt)) +
  geom_jitter(alpha = 1/50)+
  theme(axis.text.x = element_text(angle = 90, hjust = 1))+
  xlab("Date")+ ylab("Amount")


log_date3 <- log_date + scale_y_continuous()
log_date1 <- log_date + scale_y_log10()
log_date2 <- log_date + scale_y_sqrt()

grid.arrange(log_date3, log_date1, log_date2, ncol = 3)
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/13.png)


##### High percentage of contribution amount stays below $1,000 and next strip is around $1,000 and small strip around $2,000 Next strip looks like around $3,000 and a few of line around $6,000.

```{r}
retired_plot1 <- ggplot(data = subset(dt, contbr_occupation == "RETIRED"),
                        aes(contb_receipt_amt, cand_nm)) +
  geom_point(alpha = 1/10, color = 'orange') +
  theme(axis.text.y = element_text(size = 7)) +
  scale_x_continuous(breaks = seq(-3000,5000,1000))+
  ggtitle("Contribution Amount by Retired people") +
  labs(x = "Amount", y = "Candidate")

retired_plot2 <- ggplot(data = subset(dt, contbr_occupation == "RETIRED"),
                        aes(modifiedDate, cand_nm)) +
  geom_point(alpha = 1/10, color = 'red') +
  ggtitle("Supported Date by retired people") +
  labs(x = "Date", y = "Candidate")
  # geom_line(stat = 'summary', fun.y = sum)

grid.arrange(retired_plot1, retired_plot2, nrow = 2)
```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/14.png)

##### Retired people mostly contributed a lot of money on the election. Clinton and Ted cruz are getting contributed as time goes by. Sanders and Rubio doesn't seem like contributed steadily.

```{r}
sumd <- function(y){
  z = (sum(y)/10)
  return(z)
}

retired_dt <- subset(dt, contbr_occupation == "RETIRED")

ggplot(data = subset(retired_dt, contb_receipt_amt>0),
       aes(modifiedDate, contb_receipt_amt)) +
  # stat_summary(fun.y = sum, geom = "area", color = 'black', alpha = 1/10)+
  geom_density(stat = "summary", fun.y = sumd,
               mapping = aes(fill = 'sum'), alpha = 3/10)+
  geom_density(stat = "summary", fun.y = mean,
               mapping = aes(fill = 'mean'), alpha = 3/10)+
  # scale_y_sqrt() +
  scale_fill_manual(name = "", values = c('blue', 'red'))+
  ggtitle("Daily Amount sum/10 & Mean")+
  labs(x = "Date", y = "Amount")

```

![alt tag](https://github.com/youngkil9999/Project4/blob/master/15.png)

##### Daily mean and sum/10 contribution amount. just wanted to try other plots.
##### nothing special can be noticed. mean value is almost similar of all time but summation of the contribution getting increased. so we could think people are participating more contribution for their candidate.

```{r}
dt_cand_cost <- setNames(aggregate(dt$contb_receipt_amt, list(dt$cand_nm), sum),
                         c("Candidate","Contributed"))

dt_cand_cost <- dt_cand_cost[order(dt_cand_cost$Contributed, decreasing = TRUE), ]

dt_cand_cost$Candidate <- ordered(dt_cand_cost$Candidate,
                                  levels= dt_cand_cost$Candidate)

# dt_cand_cost
# length(cand_order)
# cand_order
cand_order_table <- as.data.frame(cand_order)
# cand_order_table
cand_order_table <- cbind(Candidate = rownames(cand_order_table), cand_order_table)
# rownames(cand_order_table$Candidate) <- NULL
cand_order_table <- cand_order_table[,2:3]
colnames(cand_order_table)<- c("Candidate","Popularity")
# cand_order_table

# dt_cand_cost
Cost_Popular_Table <- merge(dt_cand_cost, cand_order_table, by = "Candidate")
Cost_Popular_Table$Mean <- as.integer(Cost_Popular_Table$Contributed/Cost_Popular_Table$Popularity)

Cost_Popular_Table <- arrange(Cost_Popular_Table, desc(Popularity))
# Cost_Popular_Table

ggplot(data = Cost_Popular_Table,
       aes(reorder(Candidate, -Popularity), Popularity)) +
  # geom_point(color = 'red')+
  stat_summary(fun.y = sum, geom = "bar")+
  theme(axis.text.x = element_text(angle = 90)) +
  geom_point(data = Cost_Popular_Table, aes(Candidate, Mean))+
  scale_y_continuous(trans = 'sqrt')
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/17.png)

#####Popularity and Mean value. Ted Cruze got contribution a lot compared to mean costs even though the amount of cost was small he got lots of contribution. As popularity goes lower, Mean value normally higher

```{r}


cand_order_name[1:5]
tt_city = levels(contbr_ct$City)

dt_seven = subset(dt, cand_nm %in% cand_order_name[1:5] & contbr_city %in% tt_city)

five_box <- ggplot(data = dt_seven, aes(cand_nm, contb_receipt_amt))+
  geom_boxplot() +
  theme(axis.text.x = element_text(angle = 90))

five_box2 <- ggplot(data = dt_seven, aes(cand_nm, contb_receipt_amt))+
  geom_boxplot()+
  theme(axis.text.x = element_text(angle = 90))+
  scale_y_continuous(limits = c(0,100))

grid.arrange(five_box, five_box2, ncol=2)
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/18.png)

#####As expected earlier, Ted and Benjamin got a lot of money. a few of them are over $10,000. Benjamin is mostly higher than the other four candidate. Other four candidate almost same amount of 75% of offset average

```{r}


city_amt1 <- ggplot(data = dt_seven, aes(contbr_city, contb_receipt_amt))+
  geom_boxplot(alpha = 1/10) +
  theme(axis.text.x = element_text(angle = 90))+
  scale_y_continuous(trans = 'log10')
title_xy(city_amt1,
         "Top five Contribution Amount box plot by City", "City", "Amount")

```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/19.png)

# Bivariate Analysis

### Talk about some of the relationships you observed in this part of the investigation. How did the feature(s) of interest vary with other features in the dataset?

##### candidate and contribution date was kind of interesting. I could track how some candidate popularity changed by time even though it could be indirect. Contribution for Ted Cruze keep increasing on the plot as well as hillary in retired people
##### also contribution amount and date was interesting. it could be possible to check how much contribution generally supported for a candidate. There were like stripe around below $800, $1,000 and $3,000


### Did you observe any interesting relationships between the other features (not the main feature(s) of interest)?
##### when I look through the contribution from Retired people. Mean contribution amount fluctuated a lot in the beginning of 2015 and getting converged in the beginning of 2016 even though total amount of contribution increased. people supported small amount of money but more people contributed for a candidate.

### What was the strongest relationship you found?

##### The big cities like birmingham, mobile, montgomey lived a lot more people than other countryside so that made contributed lots of money to a candidate they are supporting. also considering retired people dominated the most of contribution. we could say order people have an more interest on a politics.




# Multivariate Plots Section
```{r}
# by(dt$cand_id, dt$contbr_occupation, summary)

# summary(dt$cand_nm)
cand_list = unique(dt_seven$cand_nm)
republican_lst = cand_list[c(2,3,5)]
democrat = cand_list[c(1,4)]
cand_list
n = 1
for (a in dt_seven$cand_nm){
  if (a %in% republican_lst){
    dt_seven$Party[n] = "Republican"
  }else{
    dt_seven$Party[n] = "Democrat"
  }
  n = n + 1
}


# contribution by city with Scatter plot
amount_pp <- ggplot(dt_seven, aes(contbr_city, contb_receipt_amt))+
  geom_bar(aes(fill = cand_nm), stat = "summary",
           fun.y = sum, position = "dodge")+
  # stat_summary(fun.y = sum, geom = 'bar') +
  theme(axis.text.x = element_text(angle = 90),
        legend.position = "right")+
  scale_y_continuous(labels = comma)+
  ggtitle("Top five candidate by City")+
  labs(x = "City", y = "Amount")

amount_p <- ggplot(dt_seven, aes(contbr_city))+
  geom_bar(aes(fill = Party), stat = "count", position = "dodge")+
  theme(axis.text.x = element_text(angle = 90), legend.position = "right")+
  scale_y_continuous(labels = comma)+
  ggtitle("Top five candidate by City")+
  labs(x = "City", y = "Count")

amount_p
# grid.arrange(amount_pp, amount_p, nrow =2 )
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/20.png)

##### montgomery, Tuscaloosa supported democratic party more than republican and Hillary was supported the most in those cities. In other cities, mostly republican candidates were contributed highly.


```{r}
count_p <- ggplot(dt_seven, aes(contbr_city))+
  geom_bar(aes(fill = cand_nm), stat = "count", position = "dodge")+
  # stat_summary(fun.y = sum, geom = 'bar') +
  theme(axis.text.x = element_text(angle = 90), legend.position = "right")+
  scale_y_continuous(labels = comma)+
  facet_wrap(~Party)+
  scale_fill_manual(name = "Candidate",
                    values = c("lightblue1","indianred","khaki","orange1","honeydew"))+
  ggtitle("Top five candidate by City")+
  labs(x = "City", y = "Count")

grid.arrange(amount_pp,count_p, nrow =2 )
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/21.png)

##### In Democrat, Sanders mostly more often contributed than hillary even though the amounts were small. In Republican, Ted cruze dominated the contribution frequency mostly all other cities.

```{r}
# ggplot(data = dt_five, aes(contbr_city, contb_receipt_amt)) +
#   geom_jitter(aes(color = cand_nm), alpha = 1/5, stat = "summary", fun.y = sum)+
#   theme(axis.text.x = element_text(angle = 90, size = 1),
#         legend.position = "top") +
#   scale_y_continuous(labels = comma, trans = 'sqrt')
  # scale_y_continuous(labels = comma, trans = 'sqrt', limits = c(0,100000))

# grid.arrange(multi1, multi2, ncol=2)

# First plot is for all the candidate along with contribution amount.
# Second plot is for selected candidate like Hillary, Sanders, Ted..

city_month1 <- ggplot(data = subset(dt_seven, year == 2016), aes(month))+
  geom_bar(aes(fill = contbr_city), stat = "count", position = "dodge")+
  theme(axis.text.x = element_text(angle = 90))
city_month1 <- title_xy(city_month1,
                        "2016 Monthly City contribution", "City", "Count")

city_month2 <- ggplot(data = subset(dt_seven, year == 2015), aes(month))+
  geom_bar(aes(fill = contbr_city), stat = "count", position = "dodge")+
  theme(axis.text.x = element_text(angle = 90))
city_month2 <- title_xy(city_month2,
                        "2015 Monthly City contribution", "City", "Count")

# city_month1 <- qplot(data = subset(dt_seven, year == 2016),
#                      contbr_city, fill = month)+
#   theme(axis.text.x = element_text(angle = 90))
# city_month1 <- title_xy(city_month1,
#                         "2016 Monthly City contribution", "City", "Count")
#
# city_month2 <- qplot(data = subset(dt_seven, year == 2015),
#                   contbr_city, fill = month) +
#   theme(axis.text.x = element_text(angle = 90))
# city_month2 <- title_xy(city_month2,
#                         "2015 Monthly City contribution", "City", "Count")

grid.arrange(city_month2, city_month1, nrow=2)
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/22.png)

##### 2015, December Birmingham and huntsville contributed more than normal
##### In 2016, April seems like less than expected. it's because the data collected in the middle of the month I guess.


```{r}

mnthly_amt1 <- ggplot(data = dt_seven, aes(modifiedDate, contb_receipt_amt))+
  geom_density(aes(fill = cand_nm), stat = "summary", fun.y = mean, alpha = 3/10) +
  scale_fill_manual(name = "",
                    values = c("lightblue1","indianred","khaki","orange1","seashell"))+
  theme(axis.text.x = element_text(angle = 90))+
  xlab("date") + ylab("contribution Amount")

mnthly_amt1
```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/23.png)

##### Monthly mean value for top five candidates

#####Benjamin and Sanders steadily got contributed from the beginning before Mar 2016. Rubio Marco high amount of mean amount from Nov 2015 Opposingly Hillary got high amount contribution efore 2015 Nov.

```{r}

# require(maps)
# require(ggmap)
#
# # Practice Group by
# practice <- dt %>%
#   group_by(contbr_city) %>%
#   summarise(mean = mean(contb_receipt_amt),
#             sum = sum(contb_receipt_amt),
#             n = n()) %>%
#   arrange(contbr_city)
#
#
# practice
# summary(practice)
# head(practice)
#
# practice$colorBuckets <- as.numeric(cut(practice$n,
#                                         c(0,100,300,500,1000,2000,3500)))
#
# colors = c("#F1EEF6", "#D4B9DA", "#C994C7", "#DF65B0", "#DD1C77",
#     "#980043")
# map("county","AL", col = colors[practice$colorBuckets],
#     fill = TRUE, resolution =0,lty = 0, projection = "polyconic")
#
# map("county", col = "white", fill = FALSE, add = TRUE, lty = 1, lwd = 0.2,
#     projection = "polyconic")
#

```


# Multivariate Analysis

### Talk about some of the relationships you observed in this part of the investigation. Were there features that strengthened each other in terms of looking at your feature(s) of interest?
##### There are some trend between the date and candidate contribution amount.
##### Monthly mean value for top five candidates
##### Benjamin and Sanders steadily got contributed from the beginning before Mar 2016. Rubio Marco high amount of contribution suddenly increased from Nov 2015 Opposingly Hillary got high amount contribution before 2015 Nov and getting small amount of money after then. Between Oct2015 and Dec2015 for these three months. trend changed a lot.

### Were there any interesting or surprising interactions between features?
##### Pupularity and mean contribution amount is not necessarily same. even though Ted highly supported in Alabama. he only got small amount of mean contribution. Bush Jep opposedely supported by small people but average contribution amount is the top. funny thing is as many people support their candidate, normally the mean contribution amount is small. I just curious about the relation ship between popularity and total contribution amount.


### OPTIONAL: Did you create any models with your dataset? Discuss the strengths and limitations of your model.


# Final Plots and Summary

```{r}

### Plot One


ggplot(data = dt, aes(dt$cand_nm, contb_receipt_amt))+
  geom_bar(fun.y = mean, stat ='summary', aes(fill = "Mean"),
           position = position_nudge(x = -0.2), width = 0.4)+
  geom_bar(fun.y = median, stat = 'summary' , aes(fill = "Median"),
           position = position_nudge(x = 0.2), width = 0.4)+
  theme(axis.text.x = element_text(angle = 90))+
  xlab('Candidate Name')+
  ylab('Contributed Amount ($)')+
  scale_y_continuous(labels = comma)+
  scale_fill_manual(name = "", values = c("Mean" = "lightblue1","Median" = "indianred"))+
  labs(title = "Candidate Contributed Amount")

# dt_cont <- ggplot(data=dt_cand_cost, aes(Candidate, Contributed)) +
#   # geom_freqpoly(fun.y = sum, stat = "summary")+
#   geom_point(fun.y = mean, stat = "summary")+
#   theme(axis.text.x = element_text(angle = 90))+
#   xlab('Candidate Name')+
#   ylab('Contributed Amount ($)')+
#   scale_y_continuous(labels = comma)+
#   labs(title = "Candidate Contributed Amount")
# # options(scipen = 5)
# dt_cont
# grid.arrange(dt_pop, dt_cont, ncol = 2)

```
![alt tag](https://github.com/youngkil9999/Project4/blob/master/24.png)

### Description One
##### There are relationship between the candidate popularity and contribution amount for sure. just doesn't match well for some candidate. Normally, we could think if there are many people support a candidate, the candidate will get more contribution which is true. As you can check left and right plot. Mostly, high ranked candidates who received a lot of contribution are popular in Alabama except for Bush Jeb.


```{r}
### Plot Two
plot_two1 <- ggplot(data = dt, aes(modifiedDate, cand_nm)) +
  geom_point(alpha = 1/10, color = 'orange') +
  labs(x = "Receipt Date", y = "Candidate Name",
       title = "Candidate Contributed Frequency by Receipt Date")+
  scale_x_date() +
  theme(plot.title = element_text(size = 8))

plot_two1
# plot_two2 <- ggplot(data = dt_seven, aes(modifiedDate, contb_receipt_amt)) +
#   geom_bar(aes(color = cand_nm), stat = "summary", alpha = 1/3, fun.y = mean) +
#   labs(x = "Receipt Date", y = "Candidate name",
#        title = "Five popular Candidate Contributed Mean Amount by Receipt Date") +
#   theme(axis.text.x = element_text(size = 3), plot.title = element_text(size = 8))
#  
# grid.arrange(plot_two1, plot_two2, ncol=2)



![alt tag](https://github.com/youngkil9999/Project4/blob/master/25.png)

### Description Two

##### Easily noticed five popular candidate year long. Sanders, Rubio, Cruz, Hillary, Benjamin from the top. Interesting thing is End of April, It looks like three candidates left on the election, Sanders, Cruz, Hillary, which are not true. There still three more candidates faintly contributed, Donald, Rubio, Kasick.
##### On the right side of the plot, Benjamin was steadily contributed but there are not big amount of money. He might be supported by blue collar as well as Sanders. As I mentioned before I don't know politics well. Evertyhing is just as I see in the plot. Otherwise, Clinton and Rubio sometimes received high amount of money might be supported by white collar.


#
#
# if (dt_seven$cand_nm %in% republican_lst == TRUE){
#   dt_seven$Party = "Republican"
# } else {
#   dt_seven$Party = "Democrat"
# }



# amount_p <- ggplot(dt_seven, aes(contbr_city, contb_receipt_amt))+
#   geom_bar(aes(fill = cand_nm), stat = "summary", fun.y = sum)+
#   # stat_summary(fun.y = sum, geom = 'bar') +
#   theme(axis.text.x = element_text(angle = 90), legend.position = "right")+
#   scale_y_continuous(labels = comma)+
#   ggtitle("Top five candidate by City")+
#   labs(x = "City", y = "Amount")
#
grid.arrange(count_p, amount_p, nrow = 2)


![alt tag](https://github.com/youngkil9999/Project4/blob/master/26.png)

###Description Three.
#####Top ten cities have most of people so that it could reflect the index of AL state  candidate support. Birmingham, Huntsville, Mobile, Montgomery. I could say Republican candidates are more popular than democrat in AL and sanders and Ted supported the most in the big cities but the contributed amount almost same as other candidates such as Benjamin, Hillary.



### Reflection

##### Looking through 2016 election in Alabama. There were 23645 object and 23 columns including candidate name, contributor name, city, employer, contribution amount, date,..I do not strongly insist that the ratio of the contribution amount not related to the candidate popularity because there is always exceptional candidate like Trump but generally, it is true. There are too many cities in Alabama even though population concentrated into five cities. Birmingham, Huntville, Mobile, Montgomery.. So, I could think the candidate win in those big city will take up the alabama.

##### Those plots above show that people supported their candidate end of every month when they have enough money I think. I am not sure when retired people got their 401k monthly salary but should be on end of month and the amount of contribution is getting increased when it gets closer to the election. I could see there are some stripes on the contribution amount $3,000, $1,000, $700, $500, $300.


##### The candidates could be separated by three groups. Popular and received small amount contribution from many people like Ted Cruz, Sanders, benjamin. Popular and received big amount money like, Hillary, Marco Rubio. Not popular but received big amount of contribution like, Jeb Bush.

##### There are contribution trend. It could be one way to think of candidate popularity. Some candidate contribution frequency is getting increased such like Ted Cruz, Hillary, Sanders when it close to the election, some didn't at some point like Rubio, Benjamin.

##### Three people left for the finalized election so far as going through the data plot, Hillary, Sanders, Ted Cruz. But in reality, There is Donald Trump and not the Ted Cruz. So, It is not easy to say that the contribution amount not directly connected to the popularity even though Donal Drump is a special case, the richest guy in USA no need contribution. If I could have a chance to analyze the whole USA data, It could be more interesting than just Alabama but still satisfied with the result and it was fun.
```{r}