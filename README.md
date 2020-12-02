---
title: "ObesityModel"
author: "Kaiyan Li, Kirill Gavrilov, Victor Wang"
date: "28/11/2020"
output: pdf_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE,
                      tidy.opts = list(width.cutoff = 80), tidy = TRUE)
```

# Introduction
Center for Disease Control (CDC) collected data from distinct demographic groups throughout 2011 to 2016 in all states of America. Demographic groups include: age, income, education level, ethnicity and gender. Each group was sampled for 9 different categories: 

* Percent of adults aged 18 years and older who have obesity
* Percent of adults aged 18 years and older who have an overweight classification
* Percent of adults who report consuming fruit less than one time daily
* Percent of adults who report consuming vegetables less than one time daily
* Percent of adults who engage in muscle-strengthening activities on 2 or more days a week
* Percent of adults who achieve at least 150 minutes a week of moderate-intensity aerobic physical activity or 75 minutes a week of vigorous-intensity aerobic activity (or an equivalent combination)
* Percent of adults who achieve at least 150 minutes a week of moderate-intensity aerobic physical activity or 75 minutes a week of vigorous-intensity aerobic physical activity and engage in muscle-strengthening activities on 2 or more days a week
* Percent of adults who achieve at least 300 minutes a week of moderate-intensity aerobic physical activity or 150 minutes a week of vigorous-intensity aerobic activity (or an equivalent combination)
* Percent of adults who engage in no leisure-time physical activity


Our objective is to find whether or not there's relationship between any of these classifications. We would like to answer these questions: 

1. Is there relationship between *obesity rates* and *fruit/vegetable intake alongside physical activity*?
2. Is there evidence that *obesity rates* in the USA are growing?
3. Add more later


Firstly we need to analyze the original dataset, remove all the unnecessary variables and observations, add a new point to the dataset to make it unique. Since we are mainly going to be working with the obesity percentage data points, we will fill in the missing value for obesity % in Alabama in 2011, specifically for "Other" ethnicity. Particular value was chosen by looking at near by states at that year for that demographic. 
```{r message = FALSE}
library(tidyverse)
library(faraway)
cdc = read.csv("Nutrition__Physical_Activity__and_Obesity_-_Behavioral_Risk_Factor_Surveillance_System.csv")

# Removing LocationAbbr, and renaming LocationDesc, DataSource, Topic, TopicID, ClassID, GeoLocation, Data_Value_Unit, Data_Value_Type, DataValueTypeID, Data_Value_Footnote_Symbol, StratificationCategoryId1, StratificationID1
cdc_adjusted = cdc %>% select(-YearEnd, -LocationAbbr, -Datasource, -Topic, -TopicID, -ClassID, -GeoLocation, -Data_Value_Unit, -Data_Value_Type, - DataValueTypeID, -Data_Value_Footnote_Symbol, -StratificationCategoryId1, -StratificationID1, -Data_Value_Alt, -Data_Value_Footnote, -Low_Confidence_Limit, -High_Confidence_Limit) %>% rename(Year = YearStart, Location = LocationDesc)

#Removing Virigin Islands because they have observations for only 2016
cdc_adjusted = cdc_adjusted[!(cdc_adjusted$Location == "Virgin Islands"),]
cdc_adjusted[28,5] = 30.2
cdc_adjusted[28,6] = 64
```


```{r echo = FALSE}
#Percent of adults who report consuming fruit less than one time daily
Q18 <- cdc_adjusted %>% filter(QuestionID == 'Q018' )%>%select(-QuestionID) %>%
  rename(Class18=Class,Question18=Question, Data_Value18=Data_Value, Sample_Size18=Sample_Size)

#Percent of adults who report consuming vegetables less than one time daily
Q19 <- cdc_adjusted %>% filter(QuestionID == 'Q019' )%>%select(-QuestionID) %>%
  rename(Class19=Class,Question19=Question, Data_Value19=Data_Value, Sample_Size19=Sample_Size)

#Percent of adults aged 18 years and older who have obesity
Q36 <- cdc_adjusted %>% filter(QuestionID == 'Q036' )%>%select(-QuestionID) %>%
  rename(Class36=Class,Question36=Question, Data_Value36=Data_Value, Sample_Size36=Sample_Size)

#Percent of adults aged 18 years and older who have an overweight classification
Q37 <- cdc_adjusted %>% filter(QuestionID == 'Q037' )%>%select(-QuestionID)

#Percent of adults who achieve at least 150 minutes a week of moderate-intensity aerobic physical activity or 75 minutes a week of vigorous-intensity aerobic activity (or an equivalent combination)
Q43 <- cdc_adjusted %>% filter(QuestionID == 'Q043' )%>%select(-QuestionID)

#Percent of adults who achieve at least 150 minutes a week of moderate-intensity aerobic physical activity or 75 minutes a week of vigorous-intensity aerobic physical activity and engage in muscle-strengthening activities on 2 or more days a week
Q44 <- cdc_adjusted %>% filter(QuestionID == 'Q044' )%>%select(-QuestionID)

#Percent of adults who achieve at least 300 minutes a week of moderate-intensity aerobic physical activity or 150 minutes a week of vigorous-intensity aerobic activity (or an equivalent combination)
Q45 <- cdc_adjusted %>% filter(QuestionID == 'Q045' )%>%select(-QuestionID)

#Percent of adults who engage in muscle-strengthening activities on 2 or more days a week
Q46 <- cdc_adjusted %>% filter(QuestionID == 'Q046' )%>%select(-QuestionID)

#Percent of adults who engage in no leisure-time physical activity
Q47 <- cdc_adjusted %>% filter(QuestionID == 'Q047' )%>%select(-QuestionID) %>%
  rename(Class47=Class,Question47=Question, Data_Value47=Data_Value, Sample_Size47=Sample_Size)


total <- inner_join(Q18,Q19,by=c("Year","Location","Total", "Age.years.", "Education", "Gender", "Income", "Race.Ethnicity","LocationID","StratificationCategory1","Stratification1"))
total <-inner_join(total,Q36,by=c("Year","Location","Total", "Age.years.", "Education", "Gender", "Income", "Race.Ethnicity","LocationID","StratificationCategory1","Stratification1"))
total <-inner_join(total,Q47,by=c("Year","Location","Total", "Age.years.", "Education", "Gender", "Income", "Race.Ethnicity","LocationID","StratificationCategory1","Stratification1"))
total <- total%>% select(-Class18,-Class19,-Class36,-Class47, -Question18,-Question19,-Question36,-Question47)%>%drop_na()%>%rename(Age=Age.years.)
```

We now want to analyze the ordinary least squares model that relates obesity rates to people who report eating fruit and vegetable less than 1 time a day and engage in no physical activity 
```{r}
model <- lm(Data_Value36 ~ Data_Value18 + Data_Value19 + Data_Value47, data=total)
summary(model)
#Using Backward Elimination method, remove the variable Data_Value19
model <- lm(Data_Value36 ~ Data_Value18 + Data_Value47, data=total)
summary(model)
#Test if collinearity problem exists
vif(model)  # < 10, no serious collinearity problem

plot(model)
# From the Residuals vs Fitted plot, the residuals are uncorrelated and the constant variance assumption is satisfied.
# From the Normal Q-Q plot, the residuals are not normally distributed.
library(MASS)
# Let's try to do robust model to see if maybe that model is more applicable in our case
model_robust = rlm(Data_Value36 ~ Data_Value18 + Data_Value47, data=total)
summary(model_robust)$coefficient

detach("package:MASS", unload = TRUE)
```

# Try to fit different models
$$ obesity =\beta_0 + \beta_1fruit + \beta_2exercise + \beta_iIndicator_i$$
```{r}
model_age <- lm(Data_Value36 ~ Data_Value18 + Data_Value47 + Age, data=total)
summary(model_age)
# The result is significantly different for each age group.
ggplot(data = total) + 
  geom_point(mapping = aes(x = Data_Value47, y = Data_Value36, color = Age))
ggplot(data = total) + 
  geom_point(mapping = aes(x = Data_Value18, y = Data_Value36, color = Age))
# The plot indicates different models should be used for each age group.

model_gender <- lm(Data_Value36 ~ Data_Value18 + Data_Value47 + Gender, data=total)
summary(model_gender) 

model_income <- lm(Data_Value36 ~ Data_Value18 + Data_Value47 + Income, data=total)
summary(model_income)

model_education <- lm(Data_Value36 ~ Data_Value18 + Data_Value47 + Education, data=total)
summary(model_education)

model_race <- lm(Data_Value36 ~ Data_Value18 + Data_Value47 + Race.Ethnicity, data=total)
summary(model_race)
ggplot(data = total) + 
  geom_point(mapping = aes(x = Data_Value47, y = Data_Value36, color = Race.Ethnicity))
ggplot(data = total) + 
  geom_point(mapping = aes(x = Data_Value18, y = Data_Value36, color = Race.Ethnicity))
```


## Analysis of total obesity rates for every state
Analyzing the ordinary least squares model of how obesity rates relate to the year in order to conclude growing obesity rate in the country. Specifically, we will look at the "Total" category which represents the final obesity percentage for each state in a particular year.

$$ obesity =\beta_0 + \beta_1year + \epsilon$$
```{r}
Q36 = Q36 %>% select(Year, Location, Class36, Question36, Data_Value36, Sample_Size36, Total) %>% arrange(Location)
Q36[Q36 == ""] = NA
Q36 = Q36 %>% drop_na()
model_all_states = lm(Data_Value36 ~ Year, data = Q36)
summary(model_all_states)
```

P-value for the model does seem to indicated that there's evidence of significant relationship between obesity rates and year, however the R-squared is extremely low. Let's take a look at the residuals.

```{r fig.show = "hold", out.width = "50%"}
par(mar = c(4, 4, 0.1, 0.1))
plot(model_all_states, which = c(2,2))
plot(model_all_states, which = c(4,4))
```
Standardized residuals do not look out of place for the most part. There are a couple of observations that fall out of the straight normality line like #307 and #35, however that is expected. Out of 319 observations, 1% of those are expected to be potential outliers according Normal Distribution. Taking a look at the Cook's distance to inspect any influential points, we find that gladly there aren't any. Obeservation #307 appears again with the largest Cook's distance of just about 0.025. In order for the point to be considered influential, it's Cook's distance should be greater than 1. 

Let's reduce our set of obeservations to only the states that show significant evidence of increase obesity rates. Visualizing will also help us draw any conclusions. 


```{r message = FALSE}
#ggplot(Q36, aes(x = Year, y = Data_Value36, colour = Location)) + geom_line()

reg_coef_all_states = Q36 %>% group_by(Location) %>%
  summarize(slope = lm(Data_Value36 ~ Year)$coef["Year"],
            pvalue = coef(summary(lm(Data_Value36 ~ Year)))[2,4]) 

Q36_reduced = reg_coef_all_states %>% filter(pvalue<0.05) %>% left_join(Q36)

ggplot(Q36, aes(x = Year, y = Data_Value36)) + geom_line(aes(group = Location), colour = "grey", alpha = 0.4) +
  geom_smooth(Q36_reduced, mapping = aes(x = Year, y = Data_Value36), colour = "red") +
  geom_line(Q36_reduced, mapping = aes(x = Year, y = Data_Value36, colour = Location), alpha = 0.35) + 
  geom_smooth(colour = "black") +
  ggtitle("2011-2016 National Obesity Rates") + xlab("Year") + ylab("Obesity %")
```
Particular image represents total obesity rates graph of observations for all states. Lines in grey colour are the states that do not show significant increase in obesity rates in 6 years. That was deducted by building an individual linear model for each state versus year. If the p-value was greater than 0.05, that state is considered to not have increasing obesity rates. In contrary, coloured lines indicate growing obesity rates. Further more, red and black lines show the full model slope for both, all states and states with growing obesity rates respectively. 

We can investigate this further by looking more closely at the observations. States that have relatively low obesity rates in 2011, tend to maintain that trend and hold obesity rates constant. On the other hand, states with higher obesity rates initially, show evidence of increasing rates.

```{r}
model_updated = lm(Data_Value36 ~ Year, data = Q36_reduced)
summary(model_updated)
```


```{r fig.show = "hold", out.width = "50%"}
par(mar = c(4, 4, 0.1, 0.1))
plot(model_updated, which = c(2,2))
plot(model_updated, which = c(4,4))
```
Taking a look at the updated model for obesity rates over the years, there's a substantial increase in the value of intercept coefficient and slope remains relatively the same. Residuals appear to be in good shape with slight deviation from the Normal line and do not violate the independence assumption. 





