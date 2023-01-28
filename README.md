---
output:
  html_document:
    toc: yes
    toc_float: yes
editor_options: 
  markdown: 
    wrap: sentence
---

## **Data analysis - basics**

### **Exercise 01: linear regression, linear models comparison (data: *cars*)**

Using the cars data set draw a scatter plot where the first variable is speed and the second is braking distance.

Fit a regression line and plot it on the graph.

Try to match the square and cubic function. Add them to the chart.

Suggest a model based on its quality.

\

##### **1.1 Check data set, assign to variable**

50 data points, 2 numeric columns: speed (mph) and stopping distance (ft).

```{r, message = FALSE}
# Summary statistic of data set
skimr::skim(cars)

# Assign data set to variable
dataset01 <- cars
```

\

##### **1.2 Scatter plot with line models**

3 models combined on one graph:

```{r, message = FALSE}
# ggplot and wesanderson libraries
library(ggplot2)
library(wesanderson)

ggplot(data = dataset01, aes(x = speed, y = dist)) +
  
  geom_point(aes(x = speed, y = dist)) +
  # Cubic model
  geom_smooth(method = 'lm',
              se = FALSE,
              formula = y ~ poly(x, 3),
              aes(colour = '03. Cubic')) +
  # Square model
  geom_smooth(method = 'lm',
              se = FALSE,
              formula = y ~ poly(x, 2),
              aes(colour = '02. Square')) +
  # Linear model
  geom_smooth(method = 'lm',
              se = FALSE,
              aes(colour = '01. Linear')) +
  # Lines color
  scale_color_manual(name = 'Model',
                     values = wes_palette('Moonrise2',
                                          type = "discrete",
                                          n = 4)) +
  # Axis labels
  labs(x = 'Speed (mph)',
       y = 'Distance (ft)')
```

Separate graphs:

```{r, figures-side, fig.show="hold", out.width="50%", echo = FALSE, warning = FALSE, message = FALSE}
# ggplot and wesanderson libraries
library(ggplot2)
library(wesanderson)

# Color palette
moon <- wes_palette('Moonrise2', n=4, type = 'discrete')

# Linear model
ggplot(data = dataset01, aes(x = speed, y = dist)) +
  geom_point(aes(x = speed, y = dist)) +
  geom_smooth(method = 'lm',
              se = FALSE,
              color = moon[1]) +
  labs(x = 'Speed (mph)',
       y = 'Distance (ft)') +
  ggtitle('Linear model')


# Square model
ggplot(data = dataset01, aes(x = speed, y = dist)) +
  geom_point(aes(x = speed, y = dist)) +
  geom_smooth(method = 'lm',
              se = FALSE,
              formula = y ~ poly(x, 2),
              color = moon[2])+
  labs(x = 'Speed (mph)',
       y = 'Distance (ft)') +
  ggtitle('Square model')

# Cubic model
ggplot(data = dataset01, aes(x = speed, y = dist)) +
  geom_point(aes(x = speed, y = dist)) +
  geom_smooth(method = 'lm',
              se = FALSE,
              formula = y ~ poly(x, 3),
              color = moon[3])+
  labs(x = 'Speed (mph)',
       y = 'Distance (ft)') +
  ggtitle('Cubic model')

```

\

##### **1.3 Linear models**

```{r, message = FALSE}
# Linear model (Model 1A)
model_1a = lm(formula = dist ~ speed, data = dataset01)

# Square model (Model 1B)
model_1b = lm(dist ~ poly(speed, 2, raw = TRUE), data = dataset01)

# Cubic model (Model 1C)
model_1c = lm(dist ~ poly(speed, 3, raw = TRUE), data = dataset01)
```

\

##### **1.4 Models summary and comparison**

```{r, echo = FALSE}
(a <- summary(model_1a))
(b <- summary(model_1b))
(c <- summary(model_1c))
```

**Multiple R-squared** - measure used to evaluate the fit of a linear model.
It ranges from 0 (model does not explain any of the variance) to 1 (model explains all of the variance in the dependent variable). In general, we aim for 0.6 or higher. Multiple R-squared is sensitive to number of variables and data points.

**Best result for model 1C: model 1C explains variance in stopping distance in 67%**. The more complex model, the better it fits into data points. (1A = `r a$r.squared`, 1B = `r b$r.squared`, 1C = `r c$r.squared`)

\

**Adjusted R-squared** - takes into account the number of predictors and data points in the model and adjusts the R-squared value accordingly. The adjusted R-squared value is always lower than the R-squared value and the difference between the two increases as the number of predictors in the model increases.

**Best result for model 1B** (1A = `r a$r.squared`, 1B = `r b$r.squared`, 1C = `r c$r.squared`, but there is no significant difference between the models)

\

**p-value** - represents the probability that the observed relationship between the predictor and the dependent variable is due to coincidence. A p-value of less than 0.05 is considered to be statistically significant.

**Best result for model 1A = 1.49e-12** (1A = `r pf(a$fstatistic[1],a$fstatistic[2],a$fstatistic[3],lower.tail = FALSE)`, 1B = `r pf(b$fstatistic[1],b$fstatistic[2],b$fstatistic[3],lower.tail = FALSE)`, 1C = `r pf(c$fstatistic[1],c$fstatistic[2],c$fstatistic[3],lower.tail = FALSE)`, but it's ok for all models.)

```{r}
summary(model_1a)
summary(model_1b)
summary(model_1c)
```

\

**MSE (Mean Squared Error)** - measure of the quality of a prediction model. The MSE is calculated as the average of the squared differences between the predicted values and the actual values, lower MSE indicates a better fit of the model to the data. **Best result for 1C.**

```{r}
mean(resid(model_1a)^2)
mean(resid(model_1b)^2)
mean(resid(model_1c)^2)
```

\

**Akaike information criterion (AIC)** and **Bayesian information criterion (BIC)** are measures of the relative quality of a statistical model.

The main difference is that BIC penalizes models with more parameters more heavily than the AIC.

The model with the lowest AIC or BIC value is considered to be the best model.

```{r}
# Akaike
AIC(model_1a, model_1b, model_1c)
# Bayesian
BIC(model_1a, model_1b, model_1c)
```

**Akaike chooses 1B, BIC chooses 1A.** Differences between models are minor (\<2).

------------------------------------------------------------------------

**PROPOSED MODEL:** In terms of the fit **model 1C** is the best, since more complex models are more adaptive. From the other side they tend to cause over fitting (tendency to adjust to random "noise" and outliers). Although in terms of MSE and multiple R-squared model 1C has the best results, other parameters and measures (which takes into consideration complexity and number of predictors, such as adjusted R-squared or AIC criterion) have much worse results.

For prediction the best is **model 1A**, as it's the simplest one, shows pattern in data and have good parameters comparing to square & cubic models.

\
\

### **Exercise 02: linear regression, prediction, working with outliers (data: *manual update*)**

Make a correlation diagram and find a regression line for the house price and number of rooms data.

With significance level = 5%, does the number of rooms have an impact on the price?

What will be the price of the two-room apartment according to the estimated model?

Are the assumptions of the model fulfilled?

\

##### **2.1 Create data set**

```{r, message = FALSE}
price <- c(300, 250, 400, 550, 317, 389, 425, 289, 389, 559) 
rooms <- c(3, 3, 4, 5, 4, 3, 6, 3, 4, 5)

dataset02 <- data.frame(price,rooms)
```

\

##### **2.2 Scatter plot**

```{r, message = FALSE,}
library(ggplot2)
library(wesanderson)

# Color palette
moon <- wes_palette('Moonrise2', n=4, type = 'discrete')

# Graph
ggplot(data = dataset02, aes(x = rooms, y = price)) +
  
  geom_point(data = dataset02, aes(x = rooms, y = price)) +
  
  geom_smooth(data = dataset02,
              method = 'lm',
              color = moon[2], 
              aes(x = rooms, y = price)) +
  
  labs(x = 'Number of rooms',
       y = 'Price (kPLN)')

```
\

##### **2.3 Model summary**

**Residuals**: we aim at symmetrical residuals (difference between model and observed values).
Not bad, but there might be some outliers. More detailed analysis can be done with residuals histogram, normal Q-Q, residuals vs fitted and residuals vs leverage plots.

**Adjusted R-squared = 0.4846** (variance in price of the apartment depends on 48% on the number of rooms).
Not good enough, we aim for at least 0.6.

**p-value**: 0.01521 (OK, less than significance level = 0.05)

**Coefficient estimates**:

-   **Intercept**: defines a point of crossing y axis.
    In theory, 0-rooms apartment would cost 94.4k PLN.

-   **Rooms**: defines the slope. In our case: with each extra room price will increase by 73.1k PLN.

**Residual standard error**: on average, actual price deviates by 75.15k PLN from linear model.

```{r, message = FALSE,}
model_2a = lm(price ~ rooms, data = dataset02)
summary(model_2a)
```

**Significance of Pearson's coeff.** - measure of the linear correlation between two variables.
It ranges from -1 to 1, where -1 indicates a strong negative correlation, 0 indicates no correlation, and 1 indicates a strong positive correlation (0 - 0.3 weak, 0.3 - 0.5 moderate, >0.5 - strong).

**cor = 0.7361** (strong positive correlation)

```{r, message = FALSE,}
cor.test(~ price + rooms, data = dataset02) 
```

**Residuals histogram**: histogram suggests that the residuals are not normally distributed (left skewed) and there are some outliers.

```{r, message = FALSE,}
hist(resid(model_2a), col = moon[2])
```
\

##### **2.4 Predict price of 2-rooms apartment**

Linear regression function: $y = a * x + b$, where y = price, x = number of rooms, a = coeff. rooms (from model_2a summary), b = coeff. estimate (from model_2a summary).

```{r, message=FALSE}
# Create data set with known variable
tworooms <- data.frame(rooms=c(2))

# Use predict function using linear model
predict(model_2a,tworooms)
```

***

**ANSWER**: With significance level = 0.05 assumptions of model are met. There is strong positive correlation between number of rooms and the price (price depends on 73% on number of the rooms). Possibily, with remong outlier observation, general parameters of the model (adjusted R-squared, p-value, normality etc.) can improve.

Predicted price of 2 rooms apartment: 240.6 k PLN.

\

##### **2.5 Additional analysis for outlier**

Let's check, which observation is outlier - based on scatter plot it might be apartment with 6 rooms (obs. #7), which is far from other points. What are other symptoms of outliers in data set?

**Residuals vs. fitted plot** is mainly used to check if there is linearity in data (red line should be close to grey dashed line) or whether there are outliers ('extreme' residuals are far from the center part of the plot).

Points are not symmetrically distributed, there are potential outliers (observation #7, #10 and #4)

```{r, message = FALSE,}
# Residuals vs fitted plot
plot(model_2a, 1, pch = 20) 
```

**Normal probability plot (Q-Q)** shows every data point against normal distribution for the same amount of observations. Points closer to the straight dotted line are closer to standard distribution.
Other cases are shown [here](https://sherrytowers.com/2013/08/29/aml-610-fall-2013-module-ii-review-of-probability-distributions/qqplot_examples/).

Observations #10 and #7 are potential outliers.

```{r, message = FALSE,}
plot(model_2a, 2, pch = 20) 
```

**Residuals vs. leverage plot** allows us to spot observations that highly influence the regression model. Points that are close to or outside of the Cook's distance (grey dashed lines) are considered as influential observations.
By removing them we can expect the coefficients of the model to change noticeably.

Obs. #7 highly influences the regression line.

```{r, message = FALSE,}
plot(model_2a, 5, pch = 20) 
```
***

Based on above analysis, we will remove observation #7 from data set and check parameters of new model:

1. Divide data set: observations w/o outlier + outlier (#7),
2. Create new model w/o outlier,
2. Create scatter plot to compare linear models (with all observations and w/o outlier) and influential observation,
4. Compare parameters of those models.

```{r, message = FALSE,}
dataset02A <- dataset02[-7,]  # data set w/o outlier
dataset02B <- dataset02[7,]   # outlier

ggplot(data=dataset02, aes(x = rooms, y = price))+
  # Data points w/o outlier
  geom_point(data=dataset02A, aes(x = rooms, y = price),
             size = 2,
             colour = 1) +
  # Outlier
  geom_point(data=dataset02B, aes(x = rooms, y = price),
             size = 2,
             colour = moon[2]) +
  # Linear model w/o outlier
  geom_smooth(data = dataset02A,
              method = 'lm',
              fullrange = TRUE,
              se = FALSE,
              aes(x = rooms,
                  y = price,
                  colour = 'Model w/o outlier',
                  linetype = 'Model w/o outlier')) +
  # Linear model with all observations
  geom_smooth(data = dataset02,
              method = 'lm',
              se = FALSE,
              aes(x = rooms,
                  y = price,
                  colour = 'All data points',
                  linetype = 'All data points')) +
  # Color & line types
  scale_color_manual(name = 'Model', values = c(moon[1],moon[2]))+
  scale_linetype_manual(name = 'Model', values = c(2,1)) +
  # Axis labels
  labs(x = 'Number of rooms', y = 'Price [k PLN]')
```

**New model summary:**

**Residuals**: more symmetrical distribution, from -91.4 to 96.9 (vs -108.00 to 99.10 in 2A, improvement)

**Adjusted R-squared**: 0.7425 (vs 0.4846 in 2A, improvement)

**p-value**: 0.001741 (vs 0.01521 in 2A, improvement)

```{r, message = FALSE,}
model_2b = lm(price ~ rooms, data = dataset02A)
summary(model_2b)
```

**Significance of Pearson's coeff.**: cor = 0.88 (vs 0.7361 in 2A)

```{r, message = FALSE,}
cor.test(~ price + rooms, data = dataset02A) 
```

**Residuals histogram**: residuals are now normally distributed.

```{r, fig.show="hold", out.width="50%"}
hist(resid(model_2b), col = moon[2], main = 'Model w/o outlier')
hist(resid(model_2a), col = moon[2], main = 'Model with all data points')
```

**Residuals vs fitted**: points are more symmetrically distributed, red line is more straight. New potential outliers: #5 and #6.

```{r, fig.show="hold", out.width="50%"}
plot(model_2b, 1, pch = 20, main = 'Model w/o outlier')
plot(model_2a, 1, pch = 20, main = 'Model with all data points') 
```

**Normality**: OK in our case - rather symmetrical, potential outliers: #5, #6. Similar shape to model with all data points.

```{r, fig.show="hold", out.width="50%"}
plot(model_2b, 2, pch = 20, main = 'Model w/o outlier')
plot(model_2a, 2, pch = 20, main = 'Model with all data points') 
```

**Influential points**: points are closer to the center, graph looks much better than in previous model. Observation #6 another potential outlier.

```{r, fig.show="hold", out.width="50%"}
plot(model_2b, 5, pch = 20, main = 'Model w/o outlier')
plot(model_2a, 5, pch = 20, main = 'Model with all data points') 
```

\
\

### **Exercise 03 - linear regression, working with outliers (data: *emissions* from the UsingR package)**

In the emissions data set from the package UsingR (CO2 emissions vs. GDP level, 26 countries) there is at least one outlier.
Draw a correlation diagram for variables GDP and CO2. Based on it, determine the outlier.
Accepting variable CO2 behind the dependent variable, find the regression line with an outlier and without it.
How have the results changed? Add both lines to the graph.

\

##### **3.1 Check data set, assign to variable**

No missing values, 26 data points, 3 numeric variables (GDP, GDP per Capita and CO2 emmision)

```{r, message = FALSE}
# Library:
library(UsingR)

# Summary statistic:
skimr::skim(emissions)

# Assign data set to variable
dataset03 <- emissions
```

\

##### **3.2 Scatter plot with linear model**

```{r, message = FALSE}
# Color palette
moon <- wes_palette('Moonrise2', n=4, type = 'discrete')

ggplot(data = dataset03, aes(x = GDP, y=CO2)) +
  # Linear model
  geom_smooth(method = 'lm',
              data = dataset03,
              color = moon[2],
              aes(x = GDP, y=CO2)) +
  # Data points
  geom_point(data = dataset03,
             size = 2,
             aes(x = GDP, y=CO2)) +
  # Data point labels
  geom_text(label = rownames(dataset03),
            hjust=0.5,
            vjust=-0.5) +
  # Axis labels
  labs(x = 'GDP',
       y = 'CO2 emmission')
```

**Outliers based on scatter plot:** USA, Russia and Japan. According to [Practical Statistics for Data Scientists by Peter Bruce, Andrew Bruce and Peter Gedeck](https://www.oreilly.com/library/view/practical-statistics-for/9781491952955/), *"an outlier is any value that is very distant from the other values in a data set"*. Although USA falls within 95% confidence interval, its GDP and  CO2 emission are much higher than median* values for whole data set.

/
* /*Unlike the mean, median (or [trimmed mean](https://www.r-bloggers.com/2021/12/how-to-find-a-trimmed-mean-in-r/), which is calculated after removing extreme values) is not affected by outliers and allows us spot influential observations.*

**Outlier chosen: USA.**

```{r, message = FALSE}
# USA values
dataset03['UnitedStates',]

# Median for data set
library(magrittr)
skimr::skim(emissions) %>% dplyr::select(numeric.p50)
```

\

##### **3.3 Model summary**

**Adjusted R-squared:** 0.8988

**p-value:** 1.197e-13 (ok with significance level = 0.05)

```{r, message = FALSE}
model_3a <- lm(CO2 ~ GDP, data = dataset03)
summary(model_3a)
```

\

##### **3.4 Removing outlier (USA), creating new model**

```{r, message = FALSE}
countries_remove <- c('UnitedStates')

# w/o USA:
dataset03_A <- dataset03[!(rownames(dataset03) %in% countries_remove),]

# USA only:
dataset03_USA <- dataset03[(rownames(dataset03) %in% countries_remove),]
```

Scatter plot:

```{r, message = FALSE}
(g <- ggplot(data = dataset03, aes(x = GDP, y=CO2)) +
  # Data points:
  geom_point(data = dataset03_A,
             color = 1,
             size = 2,
             aes(x = GDP, y=CO2)) +
  # Outlier:
  geom_point(data = dataset03_USA,
             color = moon[2],
             size = 2,
             aes(x = GDP, y=CO2)) +
  # Point labels:
  geom_text(label = rownames(dataset03),
            hjust = 0.5,
            vjust = -0.5) +
  # Linear model w/o outlier:
  geom_smooth(method = 'lm',
              data = dataset03_A,
              fullrange = TRUE,
              se = FALSE,
              aes(x = GDP, y=CO2,
                  color = 'Model w/o outlier',
                  linetype = 'Model w/o outlier')) +
  # Linear model with all data points:
  geom_smooth(method = 'lm',
              data = dataset03,
              fullrange = TRUE,
              se = FALSE,
              aes(x = GDP, y=CO2,
                  color = 'Model with all data points',
                  linetype = 'Model with all data points')) +
  # Lines color & type:
  scale_color_manual(name = 'Model',
                     values=c(moon[2],moon[1])) +
  scale_linetype_manual(name = 'Model',
                        values=c(1,2)) +
  # Axis labels:
  labs(x = 'GDP',
       y = 'CO2 emmission'))
```

Zoom in to countries clustered at the beginning of coordinates system:

```{r, message = FALSE, warning = FALSE}
# Set limits on axis w/o removing data points outside the range (USA):
g + coord_cartesian(xlim = c(NA,3200000), ylim = c(NA,2100)) 
```

\

##### **3.5 Summary of new model (w/o USA)**

**Adjusted R-squared:** 0.4679 (vs 0.8988 in model_3a)

**p-value:** 9.802e-05 (vs 1.197e-13 in model_3a)

```{r, message = FALSE}
model_3b <- lm(data = dataset03_A, CO2 ~ GDP)
summary(model_3b)
```

**Residuals histogram**: not symmetrical, right skewed

```{r, fig.show="hold", out.width="50%"}
hist(resid(model_3b), col = moon[2], main = 'New model (w/o USA)')
hist(resid(model_3a), col = moon[2], main = 'Model with all data points')
```

**Residuals vs fitted**: line is more parallel to the center, but there are still extreme values (Russia, Germany)

```{r, fig.show="hold", out.width="50%"}
plot(model_3b, 1, pch = 20, main = 'New model (w/o USA)') 
plot(model_3a, 1, pch = 20, main = 'Model with all data points') 
```

**Normality**: a little bit more symmetrical, higher impact from outliers: Russia, Germany

```{r, fig.show="hold", out.width="50%"}
plot(model_3b, 2, pch = 20, main = 'New model (w/o USA)') 
plot(model_3a, 2, pch = 20, main = 'Model with all data points') 
```

**Residuals vs leverage**: points closer to center, new influential points: Japan, Germany, Russia

```{r, fig.show="hold", out.width="50%"}
plot(model_3a, 5, pch = 20, main = 'Model with all data points') 
plot(model_3b, 5, pch = 20, main = 'New model (w/o USA)') 
```

\

**SUMMARY**: With removing USA from the model, general parameters (R-squared and p-value) have worsen. Even though USA was far away from other points, it was relatively close to regression line. Now, impact from other outliers (Germany and Russia) is more visible. 

\
\

### **Excercise 04 - linear models criteria validation (data: *homeprice* from the UsingR package)**

The UsingR homeprice dataset contains information on homes sold in New Jersey in 2001.

Did the number of toilets (half) affect price (sale)?

Are the assumptions of the model met?

\

##### **4.1 Check data set, assign to variable**

29 observations and 7 variables, no missing values.

```{r, message = FALSE}
# Load library
library(UsingR)

# Summary statistic
skimr::skim(homeprice)

# Assign data set to variable
dataset04 <- homeprice
```

##### **4.2 Scatter plot**

```{r, message = FALSE, warning = FALSE}
# Color palette
moon <- wes_palette('Moonrise2', n=4, type = 'discrete')

# Scatter plot
ggplot(data = dataset04, aes(x = half, y = sale)) +

  geom_point(data = dataset04, aes(x = half, y = sale)) +

  geom_smooth(data = dataset04,
              method = 'lm',
              color = moon[2],
              aes(x = half, y = sale)) +

  labs(x = 'Number of toilets',
       y = 'Sale price (k$)')
```

##### **4.3 Model summary**

**Adjusted R-squared**: 0.1241 (below 0.6) - number of toilets (half) explains variance in the price in 12%.
 
**p-value**: 0.03436 (below, but very close to significance level = 0.05)

Residuals are not symmetrical.

```{r}
model_4a <- lm(data = dataset04, sale ~ half)
summary(model_4a)
```

**Pearson's correlation coefficient**: 0.3941621 (weak positive correlation)

```{r}
cor.test(data = dataset04, ~ sale + half) 
```

**Residuals histogram**: not symmetrical (right skewed), residuals are not normally distributed

```{r, message = FALSE}
hist(resid(model_4a), col = moon[2])
```

**Residuals vs fitted**: red line is parallel to center, points are symmetrically distributed

```{r, message = FALSE}
plot(model_4a, 1, pch = 20) 
```

**Normality**: points are rather symmetrically distributed

```{r, message = FALSE}
plot(model_4a, 2, pch = 20) 
```

**ANSWER**: assumptions of the model are not met (p value is very close to 0.05 and adjusted R-squared is below 0.6), also correlation between number of half-bathrooms and final house price is very weak (cor = 0.39). Possibly, other variables explain price difference better.

\

##### **4.4 Additional analysis**

We start with all variables (except listing price):

```{r}
model_4b <- lm(sale ~ .-list, data = dataset04)
summary(model_4b)
```

**Adjusted R-squared**: 0.8879 (vs 0.1241 in model_4a)

**p-value**: 3.686e-11 (vs 0.03436 in model_4a)

Most influential variables are *neighborhood*\*\*\*, *half*\*\* and in some part *full*.
With significance level at 5% we should remove *rooms* and *bedrooms* from the model.

```{r}
# Model w/o listing price, rooms and bedrooms variables
model_4c <- lm(sale ~ full + half + neighborhood, data = dataset04)
summary(model_4c)
```

**Adjusted R-squared**: 0.8577 (vs 0.8879 in model_4b and 0.1241 in model_4a)

**p-value**: 2.436e-11 (vs 3.686e-11 in model_4b and 0.03436 in model_4a)

Comparison of models using Akaike and Bayesian information criterion: both method choose model_4b (with all variables), although difference between 4b and 4c are not significant.

```{r}
AIC(model_4a,model_4b,model_4c)
BIC(model_4a,model_4b,model_4c)
```

**SUMMARY:** The best model is model_4b (model with all variables, except listing price).

\
\

### **Exercise 05 - non-linear regression model (data: *USPop* from the carData package)**

Match the classic model of population growth (logistic model: $y = \frac{a}{1 + e ^ {\frac{b-x}{c}}}$ with the USPop data from the carData package containing information about the US population from 1790 to 2000.

Draw a fitted regression function.

Are the assumptions of the model met?

\

##### **5.1 Check data set, assign to variable**

22 observations and 2 numeric variables (year and population), no missing values.

```{r, message = FALSE}
# Load library
library(carData)

# Summary statistic
skimr::skim(USPop)

# Assign data set to variable
dataset05 <- USPop
```
\

##### **5.2 Create models, main differences between linear and non-linear models**

```{r, message = FALSE}
# Logistic model
model_5log <- nls(population ~ SSlogis(year, a, b, c), dataset05)

# Square model (for comparison)
model_5sq <- lm(population ~ poly(year, 2, raw = TRUE), data = dataset05)

# Models summary:
summary(model_5log)
summary(model_5sq)
```
\

Both linear and non-linear models can fit to data with curved regression lines. In linear model equation is:

$[population] = 21617.39 - 24.03 * [year] + 0.0066811 * [year]^{2}$, in logistic model:

$[population] = \frac{440.833}{1 + e ^ {\frac{1976.634 - [year]}{46.28364}}}$.

The difference between them is that in linear regression **parameters** are linear, even if predictors ($X$'s) are not).General formula for linear regression is:

$Y = \beta_{0} + \beta_{1}*X_{1} + ... + \beta_{k}*X_{k}$

(in our case it's $Y = \beta_{0} + \beta_{1}*X + \beta_{2}*X^{2}$)

\
By standard, whenever it's possible, it's better to fit linear model. It's less complex and much easier to interpret impact of each predictor on dependent variable. Also, we can use p-values and R-squared to check how model fits the data.

**For logistic model, there are no measures as such as R-squared or p-value.**

By definition, R-squared is proportion of variance explained by the model to total variance and it ranges from 0 to 100% (0 to 1). In case of non-linear models R-squared isn't always between 0 and 100% and it is not correct measure to check goodness of the fit in non-linear models. Based on [Spiess & Neumeyer study](https://bmcpharma.biomedcentral.com/articles/10.1186/1471-2210-10-6) using R-squared as measure to pick best model often leads to choosing worse model.

Instead of R-squared we should use **residual standard error** (smaller values represent better models). In our case it's 4.901.

p-value is calculated for each of parameters to form null hypothesis. It is probability that data was generated randomly: if p-value is lower than significance level we reject null hypothesis and assume that changes in independent variable are related to dependent variables. 

p-value cannot be calculated for non-linear models, since they are not restricted in terms how parameters and predictors can be used and they allow non-linear parameters set up (there are infinite number of forms of those models). Therefore, we cannot form single null hypothesis that will work for all parameters in a model.

##### **5.3 Draw scatter plot**

```{r, fig.show="hold", out.width="50%", warning=FALSE, message=FALSE}
# Color palette
moon <- wes_palette('Moonrise2', n=4, type = 'discrete')

# square model
ggplot(data = dataset05, aes(x = year, y = population)) +
  geom_point(aes(x = year, y = population)) +
  geom_smooth(method = 'lm', se = FALSE, formula = y ~ poly(x, 2), size =1, color = moon[2], fullrange = TRUE) +
  ggtitle('Linear model') +
  xlab('Year') + ylab('Population')

# Logistic model:
ggplot(data = dataset05, aes(x = year, y = population)) +
  geom_point(aes(x = year, y = population)) +
  # Create regression line based on predictions using logistic model:
  geom_line(y = predict(model_5log, dataset05$year), size = 1, color = moon[3]) +
  ggtitle('Non-linear model') +
  xlab('Year') + ylab('Population')

```
\
\

In our case, square model won't work - although it fits observations a little bit better than logistic model (residual standard error is lower for square model), it won't be useful for prediction of population before year 1790 and after year 2000 (compare graphs below).
\

```{r, fig.show="hold", out.width="50%", warning=FALSE, message=FALSE}
# Basic plot:
p <- ggplot(data = data.frame(x = 0), mapping = aes(x = x)) +
  geom_point(data = dataset05, aes(x = year, y = population)) +
  xlim(1500,2500) + ylim(0,500) +
  xlab('Year') + ylab('Population')

# Square function:
fun_1 <- function(x) (coef(model_5sq)[1] + coef(model_5sq)[2]*x + coef(model_5sq)[3]*x**2)
p + stat_function(fun = fun_1, color = moon[2], size = 1) +
  ggtitle('Linear model')
  

# Logistic function:
fun_2 <- function(x) (coef(model_5log)[1]/(1 + exp(1) ^ ((coef(model_5log)[2]-x)/coef(model_5log)[3])))

p + stat_function(fun = fun_2, color = moon[3], size = 1) +
  ggtitle('Non-linear model')
```
\

##### **5.4 Examine residual values to validate non-linear model assumptions**

Non-linear model assumptions are met if:
1. Residuals are normally distributed,
2. Residuals have the same variance,
3. Residuals are independent.

\

Residuals are not normally distributed:
```{r, warning=FALSE, message=FALSE}
hist(resid(model_5log), col = moon[2])
```


Variance is not homogeneous (points are not symmetrically distributed) and show [positive correlation](https://www.originlab.com/doc/en/Origin-Help/Residual-Plot-Analysis):

```{r, warning=FALSE, message=FALSE}
# Library for nls tools:
library(nlstools)
plot(nlsResiduals(model_5log), 1)
```

**Shapiro-Wilk normality test** is used to check normality of residuals. Null hypothesis is that residuals are normally distributed. If p-value is lower than 0.05 we can reject null hypothesis.

**p-value = 0.01252** < 0.05, we reject null hypothesis, residuals are not normally distributed.



**The Runs Test** is a test for randomness of residuals. Low p-value (below 0.05) suggests that the residuals are not randomly distributed.

**p-value = 0.000896** < 0.05, residuals are not normally distributed.

```{r, warning=FALSE, message=FALSE}
test.nlsResiduals(nlsResiduals(model_5log))
```

\

**SUMMARY**: Residuals indicates that assumptions of logistic model are not met (there is correlation between residuals and they are not normally distributed).

\
\


### **Exercise 06 - Michaelis-Menten model (data: *manual update*)**

A skydiver jumps with a parachute from a hot air balloon.

Below are his velocities [m/s] in successive moments of time [s], starting from 1s.

Fit the Michaelis-Menten model to this data.

Draw a scatter plot with a fitted regression curve.

What speed the jumper will reach in the 17th second of the flight?


\

##### **6.1 Create data set**

```{r, message = FALSE}
input <- ('t	v
          1	10
          2	16.3
          3	23
          4	27.5
          5	31
          6	35.6
          7	39
          8	41.5
          9	42.9
          10	45
          11	46
          12	45.5
          13	46
          14	49
          15	50')

dataset06 <- read.table(textConnection(input), header = TRUE)
```

\

##### **6.1 Create model**

```{r, message = FALSE}
# Michaelis-Menten:
model_6 <- nls(v ~ SSmicmen(t, a, b), dataset06)

# Models summary:
summary(model_6)
```
\

##### **6.3 Draw scatter plot**

```{r, warning=FALSE, message=FALSE}
# Color palette
moon <- wes_palette('Moonrise2', n=4, type = 'discrete')

# Graph
ggplot(data = dataset06, aes(x = t, y = v)) +
  geom_point(aes(x = t, y = v)) +
  # Regression line based on predictions from Michaelis-Menten model
  geom_line(y = predict(model_6, dataset06$t), size = 1, color = moon[2]) +
  xlab('Time [s]') + ylab('Speed [m/s]')

```
\

##### **6.4 Speed prediction**

```{r, warning=FALSE, message=FALSE}
(speed = predict(model_6, data.frame(t=17)))
```
\

**SUMMARY**: In 17th second the diver will reach speed = `r speed` [m/s].

\
\

## **Data analysis - PCA**

### **Exercise 07 - loadings of components, ggbiplot (data: *painters* from the MASS package)**

Using the painters set (subjective assessments of painters) from the MASS package, perform a principal component analysis.

Investigate the loadings of the first three principal components.

Draw a scatter diagram for the first two principal components using different colors or symbols to distinguish schools of painting.

\

##### **7.1 Check & prepare data set**

1 factor (School), 4 numeric variables (Composition, Drawing, Color, Expression), no missing values.

PCA analysis can be done only on quantitative variables and data set must be complete (complete rate = 1 for each).

```{r, warning=FALSE, message=FALSE}
library(MASS)
skimr::skim(painters)
dataset07 <- painters

```

\

'School' is a qualitative variable, indicated by a factor level A-H. The school to which a painter belongs:

1. A: Renaissance
2. B: Mannerist
3. C: Seicento
4. D: Venetian
5. E: Lombard
6. F: Sixteenth Century
7. G: Seventeenth Century
8. H: French

\

```{r, warning=FALSE, message=FALSE}
# Assign factor to new variable
school <- c(dataset07$School)

# Levels in "School' factor
levels(school)
```

```{r, warning=FALSE, message=FALSE}
# Library for mapvalues function
library(plyr)

# Re-assign level names for easier analysis later
school <- mapvalues(school,
          from = c('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'),
          to = c('Renaissance', 'Mannerist', 'Seicento', 'Venetian', 'Lombard', '16th Century', '17th Century', 'French'))

# Remove "school" variable from data set (PCA is done for quantitative variables only)
dataset07 <- dataset07[,-5]
```

\

##### **7.2 PCA analysis**

Check data set using boxplot - in our case, there is no need to scale the data (boxes are close to each other, scale is unified.)

```{r, warning=FALSE, message=FALSE}
boxplot(dataset07, col = moon[1])
```
\

Using **pcromp** function on our data set we get 4 principal components (same amount as number of variables). Each component (PC1, PC2...) explains some part of variation in initial data set: PC1 explains 56% of total variance (*Proportion of Variance*), first 2 components explain in total 84,5% of total variance (*Cumulative Proportion*) and so on.

**Rotation** explains how each variable contributes to each principal component, in example:

-   To PC1 mostly contribute Expression (0.66), then in some part Composition (0.48),
-   PC2 has strong negative weight for Colour (-0.84),
-   PC3 has strong negative weight for Compostiion (-0.78) and positive weight of Expression (0.513)

```{r, warning=FALSE, message=FALSE}
# PCA
dataset07_pca <- prcomp(dataset07)

# Summary
summary(dataset07_pca)
dataset07_pca$rotation 
```

\

##### **7.3 Biplots**

Biplots are scatter plots which visualize outcome of PCA analysis. They help us to understand how specified principal components (usually we choose first 2 to get 2 dimensional graph) are related to independent variables. 

```{r, warning=FALSE, message=FALSE}
# Libraries
library(dplyr)
library(ggbiplot)
library(devtools)
install_github("vqv/ggbiplot")

# Create new palette (we need at least 8 colors to map all groups):
my_pal <- c(wes_palette('Moonrise2', n=4, type = 'discrete'),
          wes_palette('Cavalcanti1', n=2, type = 'discrete'),
          wes_palette('IsleofDogs1', n=2, type = 'discrete'))

# ggbiplot
ggbiplot(dataset07_pca,
         ellipse = TRUE,
         groups = school,
         labels = rownames(dataset07)) +
  scale_color_manual(name = 'School', values = c(my_pal))

```

\

**SUMMARY**: in terms of PCs, there is no correlation between Composition and Colour (angle close to 90 degrees). There is strong positive correlation between Composition and Expression (small angle). Colour and Drawing variables have negative correlation.

\
Painters assessment:


-   Venetian painters were highly valued for Color, while on other characteristics they were received poorly (especially Drawing).
-   Manerists were valued mostly for Drawing. Group is tightly clustered: there are small discrepancies between ratings they've received.
-   Besides Bourdon, French painters were highly valued for Drawing, as well as Expression and Composition, all of them received rather low ratings for Colour.
-   16th Century Painters received rather low ratings in terms of Expression and Composition. 
-   High rated painters: Raphael, Rubens; low rated painters: Fr. Penni, Belini.

\
\

### **Exercise 08 - loadings of components, ggbiplot (data: *Cars93* from the MASS package)**

Using categorical and continuous variables from the Cars93 data set from the MASS package, perform principal component analysis.

Compare (in separate charts):

-   American and other cars (Origin),
-   Car types (Type).

\

##### **8.1 Check & prepare data set**

9 factor variables, 18 numeric. 93 rows of data. Missing values in columns: Rear.seat.room and Luggage.room

We need to omit missing values and remove factors from data set.

```{r, warning=FALSE, message=FALSE}
library(MASS)
skimr::skim(Cars93)
dataset08 <- Cars93
```

\
Data set preparation:

```{r, warning=FALSE, message=FALSE}
# Exclude missing values:
dataset08 <- na.omit(dataset08)

# Take 2 groups in which we will compare data later
# (it needs to be done before removing factor variables)
origin <- c(dataset08$Origin)
type <- c(dataset08$Type)

# For labels on chart
make <- c(dataset08$Make)

# Distinguish continuous variables
continuous <- c()

for (i in 1:ncol(dataset08)){
  if(sapply(dataset08[i], class) != 'factor'){
    continuous <- append(continuous,colnames(dataset08[i]))
  }
}

# Remove categorical variables from data set
dataset08 <- dataset08[ , continuous]
```
\
After preparation we have 18 numeric columns and no missing values:

```{r, warning=FALSE, message=FALSE}
skimr::skim(dataset08)
```
\

##### **8.2 PCA analysis**

There is high discrepancy in scale of variables, we need to normalize data (we will set parameter **scale. = TRUE** in prcomp function).

```{r, warning=FALSE, message=FALSE}
boxplot(dataset08, col = moon[1])
```
\

We get 18 principal components. First 3 PCs explain 83,3% of variance.

```{r, warning=FALSE, message=FALSE}
dataset08_pca <- prcomp(dataset08, scale. = TRUE)
summary(dataset08_pca) 
```
\

**Screeplot (elbow plot)** is useful to determine which components are most important (explain most of the variance). As of PC4 curve flattens out, so we should consider first 3 PCs.

```{r, warning=FALSE, message=FALSE}
screeplot(dataset08_pca, type = 'l', pch = 20) 
```
\

##### **8.3 Biplots**

```{r, warning=FALSE, message=FALSE}
# Libraries
library(dplyr)
library(ggbiplot)
library(devtools)
install_github("vqv/ggbiplot")

# Biplot on PC1/PC2 grid:
ggbiplot(dataset08_pca, choices = c(1,2)) +
  ggtitle('PC1 & PC2')

# Biplot on PC3/PC4 grid:
ggbiplot(dataset08_pca, choices = c(2,3))+
  ggtitle('PC2 & PC3')
```

\
```{r, warning=FALSE, message=FALSE}
# palette
my_pal <- c(wes_palette('Moonrise2', n=4, type = 'discrete'),
          wes_palette('Cavalcanti1', n=2, type = 'discrete'),
          wes_palette('IsleofDogs1', n=2, type = 'discrete'))

# ggbiplot
ggbiplot(dataset08_pca,
         ellipse = TRUE,
         groups = origin,
         labels = make) +
  scale_color_manual(name = 'Origin', values = c(my_pal))

```
\
**SUMMARY**: 
USA cars are bigger: they can fit more passengers, size of engine and car itself (length and width) are bigger than in non-USA cars. USA cars have also lower RPM value - engines are less dynamic, meaning they are fitted to drive long distance, w/o big acceleration.

Non-USA cars are more efficient in terms of gas usage (but based purely on labels it's rather for Japan cars - Subaru, Mazda, Suzuki). European cars (BMW, VW, Volvo) have worse parameters in terms of MPG. Also, non-USA cars are more expensive.

\
\

Bi-plots grouped by origin AND type:

```{r, warning=FALSE, message=FALSE}
ggbiplot(dataset08_pca,
         ellipse = TRUE,
         groups = type,
         labels = make) +
  geom_point(aes(color = type, shape = origin), size = 2.5) +
  scale_color_manual(name = 'Type', values = c(my_pal)) +
  scale_shape_manual(name = 'Origin', values = c(16,17))

```
\
**SUMMARY**: 
Midsize ans Sporty cars are the most diverse group (midsize in terms of price - cars from USA are cheaper than those non-USA originated). 

The most clustered groups (with homogeneous parameters) are Small and Large cars.

All cars classified as Large are from USA. 

There is no surprise that Small cars have the best parameters in terms of gas usage.
\
\

## **Data analysis - classification**

### **Exercise 09 - classification - 1NN, LDA, QDA, NB. Visualization of prediction using PCA (data: *Vehicles* from the mlbench package)**

The Vehicle dataset from the mlbench package contains information (18 features, 846 observations) about silhouettes of cars (4 types).

-   Construct classification models: LDA, QDA, 1NN & NB.

-   Estimate the classification error for the constructed methods.

-   Which of the constructed classifiers is recommendable for this data set? Justify your answer.

-   Try to visualize the results in the space formed by the first two principal components.

\

##### **9.1 Check & prepare data set**

1 factor (Class), 18 numeric variables, no missing values.

```{r, warning=FALSE, message=FALSE}
library(mlbench)
data(Vehicle)
dataset09 <- Vehicle
skimr::skim(dataset09)
```

Levels of "Class" variable: bus, opel, saab, van

```{r, warning=FALSE, message=FALSE}
levels(dataset09$Class)
```


Classes are distributed equally, enough data for QDA

```{r, warning=FALSE, message=FALSE}
table(dataset09$Class)
```



For 1NN method we need to divide data set to training and testing data sets: in order to create model, we need 2 **equally** sized data sets.

```{r, warning=FALSE, message=FALSE}
# To get reproducible result
set.seed(123)

# Indices for training
n = nrow(dataset09)
train = sample(1:n, size = round(0.5*n), replace=FALSE)
train <- sort(train)
length(train)

# Indices for testing
idx = seq(n)
test <- idx[!idx %in% train]
length(test)
```
\

##### **9.2 Models - 1NN, LDA, QDA, NB (normal and kernel)**

```{r, warning=FALSE, message=FALSE}
# Libraries
  # For kNN
  library(class)
  # For LDA and QDA
  library(MASS) 
  # For NaiveBayes() command
  library(klaR)

                # train data - index + numeric variables
model_1NN <- knn(dataset09[train, 1:18],
                # test data - index + numeric variables
                dataset09[test, 1:18],
                # train data classes 
                dataset09$Class[train],
                # number of neighbors
                k = 1)

model_LDA <- lda(Class ~ .,
                 data = dataset09)

model_QDA <- qda(Class ~ .,
                 data = dataset09)

model_nb_normal <- NaiveBayes(Class ~ ., 
                              data = dataset09) 

model_nb_kernel <- NaiveBayes(Class ~ ., 
                              data = dataset09,
                              usekernel = TRUE) 
```

\

##### **9.3 Prediction errors**

Prediction errors calculated based on **confusion matrix**: [prediction error] = [1 - accuracy]

Confusion matrix interpretation (based on results for 1NN method):

-   'Bus' class was predicted correctly 88 times. 12 times 'bus' was mistaken with 'saab', 5 times with 'opel' and 6 times with 'van'.
-   Most accurate prediction was for 'van' class - it was predicted correctly 84 times. 'Van' was mistaken 3 times with 'bus', 4 times with 'opel' and 2 times with 'saab'.
-   'Opel' and 'saab' were confused most of the times.

**Kappa** is a metric that compares an Observed Accuracy with an Expected Accuracy (random chance). It can be compared to the kappa statistic for any other model used for the same classification task.
Kappa ranges from -1 (classification is much worse than random), through 0 (classification is as good as random) to 1 (classification is much better than random).

```{r, warning=FALSE, message=FALSE}
# Libraries for confusion matrix
library(caret)
library(lattice)

# 1NN
(nn_cm <- confusionMatrix(data = model_1NN, reference = dataset09$Class[test]))

# LDA
lda_cm <- confusionMatrix(data = predict(model_LDA, dataset09)$class, reference = dataset09$Class)

# QDA
qda_cm <- confusionMatrix(data = predict(model_QDA, dataset09)$class, reference = dataset09$Class)

# NB Normal
nbn_cm <- confusionMatrix(data = predict(model_nb_normal, dataset09)$class, reference = dataset09$Class)

# NB Kernel
nbk_cm <- confusionMatrix(data = predict(model_nb_kernel, dataset09)$class, reference = dataset09$Class)
```

\

Prediction errors calculated using **errorest** (2 methods: cross validation and bootstrap). For NB kernel we will use only bootstrap method, CV will return error.

```{r, warning=FALSE, message=FALSE}
# Library for errorest:
library(ipred) 

# Cross validation

# 1NN CV
# We need create mod function, so CV parameter (k = number of rows in data set) is not mistaken with 1NN parameter (k = 1), otherwise errorest will return error 
mymod <- function(formula, 
                  data, 
                  l = 1) {
  ipredknn(formula = formula, data = data, k = l)
  
}
(nn_cv <- errorest(Class ~ ., 
         data = dataset09, 
         model = mymod, 
         estimator = 'cv', 
         predict = function(o, newdata) predict(o, newdata,  'class'), 
         est.para = control.errorest(k = nrow(dataset09)), 
         l = 1))

# LDA CV
lda_cv <- errorest(Class ~ ., 
         data = dataset09, 
         model = lda, 
         estimator = 'cv', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(k = nrow(dataset09)))

# QDA CV
qda_cv <- errorest(Class ~ ., 
         data = dataset09, 
         model = qda, 
         estimator = 'cv', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(k = nrow(dataset09)))

# NB Normal CV 
nbn_cv <- errorest(Class ~ ., 
         data = dataset09, 
         model = NaiveBayes, 
         estimator = 'cv', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(k = nrow(dataset09)), 
         use.kerrnel = FALSE)


# Bootstrap

# 1NN bootstrap
(nn_boot <- errorest(Class ~ ., 
         data = dataset09, 
         model = ipredknn, 
         estimator = 'boot', 
         predict = function(o, newdata) predict(o, newdata, 'class'), 
         est.para = control.errorest(nboot = 100), 
         k = 1))

# LDA bootstrap
lda_boot <- errorest(Class ~ ., 
         data = dataset09, 
         model = lda, 
         estimator = 'boot', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(nboot = 100))

# QDA bootstrap 
qda_boot <- errorest(Class ~ ., 
         data = dataset09, 
         model = qda, 
         estimator = 'boot', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(nboot = 100))


# NB Normal bootstrap 
nbn_boot <- errorest(Class ~ ., 
         data = dataset09, 
         model = NaiveBayes, 
         estimator = 'boot', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(nboot = 100), 
         use.kerrnel = FALSE)

# NB Kernel bootstrap
nbk_boot <- errorest(Class ~ ., 
         data = dataset09, 
         model = NaiveBayes, 
         estimator = 'boot', 
         predict = function(o, newdata) predict(o, newdata)$class, 
         est.para = control.errorest(nboot = 100), 
         use.kerrnel = TRUE)
```

Summary of prediction errors:

| Error rate             | 1NN                               | LDA                                | QDA                                | NB Normal                          | NB Kernel                          |
|------------------------|-----------------------------------|------------------------------------|------------------------------------|------------------------------------|------------------------------------|
| **Confusion matrix**   | `r round((1-nn_cm$overall[1]),3)` | `r round((1-lda_cm$overall[1]),3)` | `r round((1-qda_cm$overall[1]),3)` | `r round((1-nbn_cm$overall[1]),3)` | `r round((1-nbk_cm$overall[1]),3)` |
| **Errorest CV**        | `r round(nn_cv$error,3)`          | `r round(lda_cv$error,3)`          | `r round(qda_cv$error,3)`          | `r round(nbn_cv$error,3)`          | -                                  |
| **Errorest bootstrap** | `r round(nn_boot$error,3)`        | `r round(lda_boot$error,3)`        | `r round(qda_boot$error,3)`        | `r round(nbn_boot$error,3)`        | `r round(nbk_boot$error,3)`        |
|                        |                                   |                                    |                                    |                                    |                                    | 
| **Kappa**              | `r round(nn_cm$overall[2],3)`     | `r round(lda_cm$overall[2],3)`     | `r round(qda_cm$overall[2],3)`     | `r round(nbn_cm$overall[2],3)`     | `r round(nbk_cm$overall[2],3)`     |


**SUMMARY**: Chosen model: QDA (lowest prediction error, kappa closest to 1). There is enough data in each class to correctly work with that method.

\

##### **9.4 Data visualization**

```{r, warning=FALSE, message=FALSE}
# Libraries for biplot
library(ggplot2)
library(ggbiplot)
library(devtools)
install_github("vqv/ggbiplot")
library(wesanderson)

# Two groups to compare data
real <- c(dataset09$Class)
predicted <- c(predict(model_LDA, dataset09)$class)

# PCA
dataset09_pca <- prcomp(dataset09[,-19], scale. = TRUE)

# Biplot
ggbiplot(dataset09_pca, groups = real, labels = dataset09$Class) +
  geom_point(aes(shape = predicted, colour = real), size = 2.5) +
  labs(shape = 'Predicted', colour = 'Actual') +
  scale_color_manual(values= wes_palette('Moonrise2', n = 4))
```
\
\

## **Data analysis - cluster analysis**

### **Exercise 10 - number of clusters, hierarchical & non-hierarchical cluster analysis. Clusters visualization on dendrogram and PCA (data: *mtcars*)**

For the mtcars data set, perform a hierarchical cluster analysis (mean binding method) and a non-hierarchical cluster analysis (k-means).

Draw a dendrogram.

How many clusters make sense? Determine the CH index and the silhouette factor.

How much clusters you can suggest based on them?

\

##### **10.1 Check & prepare data set**

11 numeric variables, no missing values. 

```{r, warning=FALSE, message=FALSE}
dataset10 <- mtcars
skimr::skim(dataset10)
```

High discrepancy in data set scale: data needs to be scaled.

```{r, warning=FALSE, message=FALSE}
boxplot(dataset10, col = moon[1])
```

```{r, warning=FALSE, message=FALSE}
dataset10_sc <- as.data.frame(scale(dataset10))
```

##### **10.2 Number of clusters**

Based on Calinski criterion we should choose 10 clusters (the bigger value the better). While increasing max number of clusters in arguments, cascade model suggests more and more clusters as outcome (i.e with max set to 20, model proposes 20 clusters). 10 clusters seems a lot for just 32 different cars.


With 5 clusters CH index increases comparing to 4 or 6 clusters.


After scaling the data, we get highest CH index with 4 clusters. Final number of clusters to be confirmed based on dendrogram and silhouette factors (4 or 5 clusters).

```{r, message = FALSE, warning = FALSE, fig.show="hold", out.width="50%"}
# For cascadeKM() command
library(vegan)

# For initial data set
                          # dataset, min # of cluster, max # of clusters
model_cascade <- cascadeKM(dataset10, 2, 10)
model_cascade$results
plot(model_cascade, main = 'k-means for initial data set') 

# For scaled data set
model_cascade_sc <- cascadeKM(dataset10_sc, 2, 10)
model_cascade_sc$results
plot(model_cascade_sc) 
```


##### **10.3 Hierarchical clustering**

```{r, warning=FALSE, message=FALSE}
# For fancy dendrogram:
library(dendextend)

# Distance matrix
dist_mat <- dist(dataset10_sc, method = 'euclidean')

# Hierarchical clustering & dendrogram
hclust_avg <- hclust(dist_mat, method = 'average')
plot(hclust_avg)
# Based on dendrogram >> 5 clusters

# Clusters marked on dendrogram:
rect.hclust(hclust_avg , k = 5, border = 2:6)

# Height of the cut:
abline(h = 3.5, col = 1, lwd = 2)

# Dendrogram with colorful branches:
avg_dend_obj <- as.dendrogram(hclust_avg)
avg_col_dend <- color_branches(avg_dend_obj, h = 3.5)
plot(avg_col_dend)
```

##### **10.4 k-means clustering**

For k = 4 silhouettes have different thickness (max for cluster 3), but all of them have similar height >> we aim for most unity in terms of shape.

```{r, warning=FALSE, message=FALSE}
# For silhouette() command
library(cluster)

# To get same results we need to set.seed():
set.seed(500)

# k-means model with 4 clusters:
model_kmeans_4 <- kmeans(dataset10, 
                       centers = 4, 
                       nstart = 100) 

sil_index_4 <- silhouette(model_kmeans_4$cluster, 
                        dist = dist(dataset10,
                                    method = 'euclidean'))
plot(sil_index_4)
```

For k = 5 we get much better shape in terms of thickness, height remains similar.

```{r, warning=FALSE, message=FALSE}
# To get same results we need to set.seed():
set.seed(500)

# k-means model with 4 clusters:
model_kmeans_5 <- kmeans(dataset10, 
                       centers = 5, 
                       nstart = 100) 

sil_index_5 <- silhouette(model_kmeans_5$cluster, 
                        dist = dist(dataset10,
                                    method = 'euclidean'))
plot(sil_index_5)
```

With more clusters silhouettes are getting more diverse, so we will keep 5 clusters.

```{r, warning=FALSE, message=FALSE}
# To get same results we need to set.seed():
set.seed(500)

# k-means model with 4 clusters:
model_kmeans_6 <- kmeans(dataset10, 
                       centers = 6, 
                       nstart = 100) 

sil_index_6 <- silhouette(model_kmeans_6$cluster, 
                        dist = dist(dataset10,
                                    method = 'euclidean'))
plot(sil_index_6)
```
Each cluster mean s(i) is close to mean s(i) for whole data set:

```{r, warning=FALSE, message=FALSE}
# mean s(i) for each cluster
summary(sil_index_5)

# mean s(i)
mean(sil_index_5[, 3]) 
```

##### **10.5 Visualize data with PCA**

```{r, warning=FALSE, message=FALSE}
# Libraries for biplot
library(ggplot2)
library(ggbiplot)
library(devtools)
install_github("vqv/ggbiplot")
library(wesanderson)

#Add new column with k_means cluster:
dataset10$Cluster <- as.factor(model_kmeans_5$cluster)

# Create PCA model (w/o cluster column):
dataset10_pca <- prcomp(dataset10[,-12], scale. = TRUE)

# Graph:
ggbiplot(dataset10_pca, choices = c(1,2), ellipse=TRUE, groups = dataset10$Cluster, labels = rownames(dataset10)) +
  labs(colour = 'k-means cluster') +
  scale_color_manual(values= wes_palette('Cavalcanti1', n = 5))
```

**SUMMARY**: Based on CH index for scaled and not-scaled data, dendrogram and silhouette factor we should divide data set into 5 clusters. Such clustering seems natural, while visualized on PCA grid.
