Agent Based Modelling for Influenza Infection
================
Joshua Edefo
2024-12-14

Library

``` r
library(usethis)
```

    ## Warning: package 'usethis' was built under R version 4.3.2

The Agent Based Model simulates the spread of influenza with: 1. Agent
characteristics - Age, vaccinated status, and health status 2. Disease
characteristics - infection rate, recovery rate 3. Behavioural
modifications - wearing masks and social distancing

It tracks the number of infected individuals over time and calculates
the basic reproduction number(Ro) which is a measure of the average
number of secondary infections generated by an infected individual.

The model incorporates factors like vaccination, mask usage, and social
distancing to influence transmission probabilities.

``` r
# Set parameters
num_agents <- 1000              # Total population
initial_infected <- 5           # Initial number of infected agents
vaccination_rate <- 0.3         # Vaccination rate
infection_prob <- 0.3           # Probability of transmission between susceptible and infected individuals
recovery_prob <- 0.05           # Probability of recovery per time step
mask_effectiveness <- 0.5       # Effectiveness of mask in reducing transmission
social_distancing_effect <- 0.3 # Effectiveness of social distancing in reducing transmission
num_steps <- 50                 # Number of time steps for the simulation

# Initialize the population with agent characteristics
agents <- data.frame(
  status = rep("susceptible", num_agents), # All agents are initially susceptible
  age = sample(18:80, num_agents, replace = TRUE), # Random age between 18 and 80
  vaccinated = sample(c(TRUE, FALSE), num_agents, replace = TRUE, prob = c(vaccination_rate, 1 - vaccination_rate)),
  wearing_mask = sample(c(TRUE, FALSE), num_agents, replace = TRUE, prob = c(0.5, 0.5)), # Random chance of wearing a mask
  social_distancing = sample(c(TRUE, FALSE), num_agents, replace = TRUE, prob = c(0.3, 0.7)) # Random chance of social distancing
)

# Randomly infect a few individuals at the start
agents$status[sample(1:num_agents, initial_infected)] <- "infected"

# Function to update agent states
update_agents <- function(agents) {
  new_agents <- agents  # Copy current agents state
  # Loop through each agent and update their state
  for (i in 1:num_agents) {
    if (agents$status[i] == "infected") {
      # Transmission based on social behavior
      for (j in 1:num_agents) {
        if (agents$status[j] == "susceptible") {
          # Probability of infection depends on transmission probability, mask-wearing, and social distancing
          effective_infection_prob <- infection_prob
          if (agents$wearing_mask[j]) {
            effective_infection_prob <- effective_infection_prob * (1 - mask_effectiveness)
          }
          if (agents$social_distancing[j]) {
            effective_infection_prob <- effective_infection_prob * (1 - social_distancing_effect)
          }
          
          if (runif(1) < effective_infection_prob) {
            new_agents$status[j] <- "infected"  # Infect susceptible agent
          }
        }
      }
      
      # Recovery: Infected agents have a chance of recovery
      if (runif(1) < recovery_prob) {
        new_agents$status[i] <- "recovered"  # Agent recovers
      }
    }
  }
  
  return(new_agents)
}

# Simulate the infection spread and track the number of infections
history <- matrix(NA, nrow=num_steps, ncol=num_agents)  # Matrix to store agents' states over time
history[1,] <- agents$status  # Store the initial state of the agents
infected_count <- numeric(num_steps)  # Array to track number of infected individuals over time

# Track secondary infections
secondary_infections <- 1

for (t in 2:num_steps) {
  new_agents <- update_agents(agents)  # Update agents' states
  
  # Update agent state for this time step
  agents <- new_agents
  history[t,] <- agents$status  # Store the state for this timestep
  
  # Count the number of infected agents at this time step
  infected_count[t] <- sum(agents$status == "infected")
  
  # Track secondary infections between previous and current time step
  for (i in 1:num_agents) {
    if (history[t-1, i] == "infected" && history[t, i] == "susceptible") {
      secondary_infections <- secondary_infections + 1
    }
  }
}

# Plot the number of infected individuals over time
plot(1:num_steps, infected_count, type="l", col="red", xlab="Time", ylab="Number of Infected Agents", lwd=2, main="Influenza Infection Spread with Behavior Modifications")
```

![](Agent-Based-Modelling-for-Influenza-infection_files/figure-gfm/b-1.png)<!-- -->

``` r
# Calculate the reproduction number (R₀) estimate
num_infected_agents <- sum(history[num_steps, ] == "infected")  # Number of infected agents at the final time step

# Only calculate R₀ if there are any infected agents at the final time step
if (num_infected_agents > 0) {
  R0_estimate <- secondary_infections / num_infected_agents  # Reproduction number (R₀)
  cat("Estimated R₀ (Reproduction Number):", R0_estimate, "\n")
} else {
  cat("No infected agents at the final time step, R₀ cannot be calculated.\n")
}
```

    ## Estimated R₀ (Reproduction Number): 0.01162791

``` r
R0_estimate
```

    ## [1] 0.01162791

session information

``` r
sessionInfo()
```

    ## R version 4.3.1 (2023-06-16 ucrt)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 11 x64 (build 22631)
    ## 
    ## Matrix products: default
    ## 
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_United Kingdom.utf8 
    ## [2] LC_CTYPE=English_United Kingdom.utf8   
    ## [3] LC_MONETARY=English_United Kingdom.utf8
    ## [4] LC_NUMERIC=C                           
    ## [5] LC_TIME=English_United Kingdom.utf8    
    ## 
    ## time zone: Europe/London
    ## tzcode source: internal
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] usethis_2.2.2
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] digest_0.6.33     fastmap_1.2.0     xfun_0.40         magrittr_2.0.3   
    ##  [5] glue_1.6.2        knitr_1.44        htmltools_0.5.8.1 rmarkdown_2.25   
    ##  [9] lifecycle_1.0.3   cli_3.6.1         vctrs_0.6.5       compiler_4.3.1   
    ## [13] purrr_1.0.2       rstudioapi_0.15.0 tools_4.3.1       evaluate_0.21    
    ## [17] yaml_2.3.7        rlang_1.1.1       fs_1.6.3
