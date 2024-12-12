# Logistic Growth

# Question 1

## Plotting data

Script 1 (plot_data.R) plots the dataset from experiment.csv (a simulated experiment that visualises the logistic model). The x-axis shows time (t) and the y-axis shows the number of bacteria (N). Plotting was conducted using ggplot2 package on R to produce publication-quality figures. 

The second plot in this script is a semi-log plot with a linear x-axis and a log-transferred y-axis. This semi-log plot shows that at early time points, there is an increasing linear relationship, while at later time points, population size remains constant. This semi-log plot is useful for future linear model analysis. 

```{r}
#Code to plot logistic growth data
install.packages("ggplot2")
library(ggplot2)
growth_data <- read.csv("experiment.csv")

#Logistic plot
ggplot(aes(t,N), data = growth_data) +
  geom_point() +
  xlab("Time (minutes)") +
  ylab("Population Size (number of bacteria)") +
  theme_bw()

#Semi-log plot
ggplot(aes(t,N), data = growth_data) +
  geom_point() +
  xlab("Time (minutes)") +
  ylab("Population Size (number of bacteria)") +
  scale_y_continuous(trans='log10') +
  theme_bw()
  
#Semi-log plot - distinguishing when t is small or large 

growth_data$threshold <- ifelse(
  growth_data$t<=2000, 'Below', 'Above'
)

ggplot(aes(t,N, color = threshold), data = growth_data) +
  geom_point() +
  xlab("Time (minutes)") +
  ylab("Population Size (number of bacteria)") +
  scale_y_continuous(trans='log10') +
  theme_bw() +
  theme(legend.position = 'none')
```

#### INSERT Logistic growth plot

#### INSERT Semi-log plot showing relationship between t and N

#### INSERT Semi-log plot showing relationship between t and N (with colour demarcating when t is small (light blue) or large (red)).

## Fitting linear models

Script 2 (fit_linear_model.R) generates two different linear models for the semi-logged data. In the first model, t is small, while in the second model, t is large. In these models, two distinct relationships occur as interpreted from the semi-log plot. For this, dplyr package was used to filter the dataset (filter () when t is large or small), enabling prediction of underlying logistic parameters. 

To generate linear models (t as explanatory variable and log_N as response variable), lm() function was used. For parameter estimates of slopes and intercepts in the two distinct linear models, summary() function was used. 

The code below shows the subsetting of data and generation of linear models. 

```{r}
install.packages("dplyr")
library(dplyr)

# Scenario 1. K >> N0, t is small

data_subset1 <- growth_data %>% 
  filter(t<1500) %>% 
  mutate(N_log = log(N))

linearmodel1 <- lm(N_log ~ t, data_subset1)
summary(linearmodel1)

#Scenario 2. N(t) = K

data_subset2 <- growth_data %>% filter(t>2500)

linearmodel2 <- lm(N ~ 1, data_subset2)
summary(linearmodel2)
```
## Plotting my own models

Script 3 (plot_data_and_model.R) produces the 'logistic_function' function. This allows generation of my own logistic curve, in which logistic parameters (N0, r and K) can be modified. This logistic curve can be superimposed onto "experiment.csv" dataset to enable evaluation of whether my own logistic parameters are consistent with the logistic parameters underlying the logistic curve in "experiment.csv" dataset. 

## Results

## Linear Model 1 (small t)

#### INSERT summary table for Linear Model 1

The summary table provides the parameters for Linear Model 1 (in the form 'y = a + bx') where $$ln(N) = ln(N0) + rt$$

Per the summary table, intercept (ln(N0)) is 6.8941709; therefore, **N0 is 986** (since e^6.8941709 = 986.5074723).

Per the summary table, t (r) is 0.0100086; therefore, **r is 0.01**. 

## Linear Model 2 (large t)

#### INSERT summary table for Linear Model 2

Per the summary table, intercept (K) is 6 x 10^10; therefore, **K is 60000000000**.

Confirmation of these logistic parameters was achieved by superimposing my linear function (with these logistic parameters) over experimental datapoints. This is shown by the figure below, generated using filled-in Script 3 (plot_data_and_model.R):

```{r}
N0 <- 986 # Initial bacterial population size

r <- 0.01 # Growth rate

K <- 6.00e+10 # Environmental carrying capacity 

logistic_function <- function(t) {
  
  N <- (N0*K*exp(r*t))/(K-N0+N0*exp(r*t))
  
  return(N)
  
}

# Superimposing model over experimental data

ggplot(aes(t,N), data = growth_data) +
  
  geom_function(fun=logistic_function, colour="red") +
  
  geom_point()
```
#### INSERT figure of logistic function with logistic parameters obtained from linear model superimposed over experimental datapoints

This figure validates determination by linear model analyses of the logistic parameters underlying the experimental dataset, since the superimposed linear function is consistent with experimental datapoints. 

This enables use of my logistic function for interpolation of population size at specified timepoints. 

# Question 2

Under exponential population growth, N(t) = N0 * e^(r*t)

Substituting my estimates of N0 and r to calculate bacterial population size at t=4980 min: N(4980) = 986 * e^(0.01*4980) = 4.19 x 10^24

Under logistic population growth, bacterial population size at t=4980 min would be 6 x 10^10 since by this timepoint the bacterial population would have already reached and be held constant at the environmental carrying capacity.

Therefore, at time = 4980 minutes, bacterial population size would be 6.98x10^13 times larger under exponential growth than under logistic growth (since (4.19 x 10^24)/(6 x 10^10) = 6.98x10^13). 

# Question 3

Code used to construct a graph comparing the exponential and logistic growth curves is shown below.

```{r}
# Defining logistic function
logistic_function <- function(t) {
  N <- (N0 * K * exp(r * t)) / (K - N0 + N0 * exp(r * t))
  return(N)
}

# Defining exponential function
exponential_function <- function(t) {
  N <- (N0 * exp(r * t))
  return(N)
}

# Setting parameters estimated from linear models for both exponential and logistic functions
N0 <- 986
r <- 0.01
K <- 60000000000

# Plotting both functions
ggplot() +
  geom_function(aes(colour = "Logistic"), fun = logistic_function, linewidth = 1.2) +  # Increasing line width
  geom_function(aes(colour = "Exponential"), fun = exponential_function, linewidth = 1.2) +  # Increasing line width
  xlim(0, 5000) +
  scale_y_continuous(trans = 'log10') +
  scale_colour_manual(values = c("Logistic" = "darkgreen", "Exponential" = "darkblue")) +
  xlab("Time (minutes)") +
  ylab("Population size (number\nof bacteria, log10 transformed)") +  # Stacking y-axis label
  ggtitle("Comparison of exponential and logistic\nmodels of bacterial population growth") +  # Stacking title
  theme_bw() +
  theme(
    legend.title = element_blank(),
    plot.title = element_text(face = "bold", hjust = 0.5),  # Centering title with hjust
    axis.title = element_text(face = "bold"), # Emboldening axes labels
    legend.text = element_text(face = "bold") # Emboldening legend labels
  )
```

#### INSERT figure comparing exponential and logistic growth curves.

This graph illustrates the difference between exponential and logistic models of E. coli population growth. The graph suggests that over short time periods (t<1750 minutes), accurate forecasting of E. coli population size in-vitro is possible using an exponential growth model. The graph suggests that over long time periods, accurate forecasting of E. coli population size in-vitro can only be achieved using a logistic growth model that accounts for limited resources in the growth environment in-vitro, defined by carrying capacity (K). This is because the exponential growth model inaccurately predicts continual population growth under limited resource availability in the culture environment.
