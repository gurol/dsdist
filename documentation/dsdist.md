# dsdist
@author Gürol Canbek, <gurol44@gmail.com>  
Copyright (C) 2017-2018 Gürol CANBEK  
@references <http://gurol.canbek.com>  
@keywords distributions, log-normal, power law, Poisson, exponential,
data sets, feature frequency  
@title dsdist - R Scripts for Distribution Fitting Testing  
@date 11 January 2018  
@version 1.1  
@note version history  
December 2017  
1.1 January 2018, Exponential and Poisson fits  
1.0 December 2017, The first version  
@description R scripts for comprehensive set of distribution testing to fit  
power law, log-normal, exponential, and Poisson statistical distribution  
into the feature frequency distributions (the truth). A part of dsanalysis  
 (Dataset Analysis)  
## libraries


```r
library(poweRlaw)
library(ggplot2)
library(magrittr)
library(dplyr)
```

```r
source('utils.R', chdir=TRUE)
```

## Distribution colors


```r
#               Power-law  Log-Normal Exponent   Poisson
cols_dist <- c('#fb8072', '#7f4e2C', '#80b1d3', '#49b960')
col_distpl <- 1
col_distln <- 2
col_distex <- 3
col_distpo <- 4

# Compare distribution Vuon's test statistics sign threshold
sign_threshold <- 0.001
```

### testLongTailDistributionsHypotheses
Test the various long tail distribution fit hypothesis for given truth given  
as a feature frequencies or counts per class (e.g. 'Positive' or 'Malign')  
with corresponding feature space names of one or more datasets (ds).  
**Parameters:**  
*class_name*: Class identifier (e.g. 'Positive' or 'Malign')  
*df_feat_freqs_or_counts*: Data frame holding feature frequencies or counts per dataset as a column vector  
*df_ds_feat_space_names*:  Data frame holding feature names per dataset as a column vector  
*df_ds_sample_sizes*: Data frame holding sample sizes if the feature distribution is given as a frequency (otherwise: default: NULL)  
*plot_graph_compare_two_dists*: Plot? (default: TRUE)  
*plot_graph_compare_all_dist*: Plot? (default: TRUE)  
*plot_graph_likelihood_ratios*: Plot? (default: FALSE)  
**Return:**  
none  
**Details:**
Select feature space names in the spreadsheet (hide other columns)  
`df_ds_feat_space_names <- rclip()`  
Select feature frequencies or counts in the spreadsheet (hide other columns)  
`df_feat_freqs_or_counts <- rclip()`  
Select sampe sizes if frequencies are provided  
`df_ds_sample_sizes <- rclip(header=FALSE)`  
ignore warning message: incomplete final line found by readTableHeader on 'pbpaste'
 
You can save the data frames for later use:
`save(df_feat_freqs_or_counts, df_ds_feat_space_names, df_ds_sample_sizes, file='Malign.RData')`  
 
*See the Examples*  
 
**Suggested plot file naming schema:**  
6 inch x 8 inch Landscape for Export | Save as PDF...  
800 x 600 for Export | Save as Image...  
Benign_DS0_VuongTest_fitted_powerlaw_vs_lognormal  
Benign_DS0_VuongTest_fitted_powerlaw_vs_poisson  
Benign_DS0_VuongTest_fitted_powerlaw_vs_exponential  
Benign_DS0_VuongTest_powerlaw_vs_fitted_lognormal  
Benign_DS0_VuongTest_powerlaw_vs_fitted_exponential  
Benign_DS0_VuongTest_lognormal_vs_fitted_exponential  
Benign_DS0_fitted_powerlaw_against_others  
Benign_DS0_fitted_lognormal_against_others  
Benign_DS0_fitted_exponential_against_others  
Benign_DS0_bootstrapt_powerlaw_fit  
Benign_DS0_bootstrapt_lognormal_fit  
Benign_DS5_bootstrapt_exponential_fit  
Benign_DS0_LogLikelihoodRatioDistribution_fitted_powerlaw_vs_lognormal  
Benign_DS0_LogLikelihoodRatioDistribution_powerlaw_vs_fitted_lognormal  
**Warning:**  
Unsolved exception when plot_graph_likelihood_ratios=TRUE  
**Examples:** `testLongTailDistributionsHypotheses('Benign', df_feat_freqs_or_counts, df_ds_feat_space_names, df_ds_sample_sizes, no_sim_count=50)`  


```r
testLongTailDistributionsHypotheses<-function(
  class_name, df_feat_freqs_or_counts, df_ds_feat_space_names,
  df_ds_sample_sizes=NULL, no_sim_count=NULL,
  plot_graph_compare_two_dists=TRUE, plot_graph_compare_all_dist=TRUE,
  plot_graph_likelihood_ratios=FALSE
  )
{
  # Dimensions check
  stopifnot(nrow(df_feat_freqs_or_counts) == nrow(df_ds_feat_space_names))
  stopifnot(ncol(df_feat_freqs_or_counts) == ncol(df_ds_feat_space_names))
  stopifnot(ncol(df_feat_freqs_or_counts) == ncol(df_ds_sample_sizes))
  stopifnot(ncol(df_ds_feat_space_names) == ncol(df_ds_sample_sizes))
  
  for (ds in 1:length(df_feat_freqs_or_counts)){
    feat_freqs_or_counts <- df_feat_freqs_or_counts[[ds]]
    feat_freqs_or_counts <- feat_freqs_or_counts[!is.na(feat_freqs_or_counts)]
    
    feat_space_names <- df_ds_feat_space_names[[ds]]
    feat_space_names <- feat_space_names[!is.na(feat_freqs_or_counts)]
    
    if (is.null(df_ds_sample_sizes))
      ds_sample_size <- NULL
    else
      ds_sample_size <- df_ds_sample_sizes[, ds]
    
    ds_name <- gsub('featureFreq', '', colnames(df_feat_freqs_or_counts)[ds])
    bs_p <- testLongTailDistributionsHypothesis(
      ds_name, class_name, feat_space_names, feat_freqs_or_counts,
      ds_sample_size,  no_sim_count=no_sim_count,
      plot_graph_compare_two_dists=plot_graph_compare_two_dists,
      plot_graph_compare_all_dist=plot_graph_compare_all_dist,
      plot_graph_likelihood_ratios=plot_graph_likelihood_ratios)
  }
}
```

### testLongTailDistributionsHypothesis
Test the various long tail distribution fit hypothesis for given truth given  
as a feature frequencies or counts for given a class (e.g. 'Positive' or 'Malign')  
with corresponding feature space names of one dataset (ds).  
**Parameters:**  
*ds_name*: Dataset name (e.g. 'DS1')  
*ds_class_name*: Class identifier (e.g. 'Positive' or 'Malign')  
*ds_feat_space_names*:  Feature names  
*ds_feat_freqs_or_counts*: Feature frequencies or counts  
*ds_sample_size*: Sample size if the feature distribution is given as a frequency (otherwise: default: NULL)  
*threads*: Number of CPU cores (default: extractef from the system)  
*no_sim_count*: Number of simulation count for bootstrap test (default: NULL)  
*plot_graph_compare_two_dists*: Plot? (default: TRUE)  
*plot_graph_compare_all_dist*: Plot? (default: TRUE)  
*plot_graph_likelihood_ratios*: Plot? (default: FALSE)  
**Return:**  
list(bs_power_law, bs_log_normal, bs_exponential) bootstrap objects  
**Details:**

**Examples:** `testLongTailDistributionsHypothesis('DS0', 'Benign', ds_feat_space_names, ds_feat_freqs, 264303, no_sim_count=5)`  


```r
testLongTailDistributionsHypothesis<-function(
  ds_name, ds_class_name, ds_feat_space_names, ds_feat_freqs_or_counts,
  ds_sample_size=NULL, threads=getNumberOfCPUCores(), no_sim_count=NULL,
  plot_graph_compare_two_dists=TRUE, plot_graph_compare_all_dist=TRUE,
  plot_graph_likelihood_ratios=TRUE)
{
  if (all(ds_feat_freqs_or_counts == floor(ds_feat_freqs_or_counts)))
    # All elements are integer so the variable holds feature counts
    feat_counts <- ds_feat_freqs_or_counts
  else
    # Elements are fractinal convert into counts by multiplying the frequency
    # by dataset sample size
    feat_counts <- round(ds_sample_size*ds_feat_freqs_or_counts)
  
  feature_space_size <- length(feat_counts)
  
  # Determine simulation counts
  simulation_counts <- 100
  if (is.null(no_sim_count)) {
    if (ds_sample_size > simulation_counts) {
      if (ds_sample_size < 1000)
        simulation_counts <- 1000
      else if (ds_sample_size < 5000)
        simulation_counts <- ds_sample_size
      else
        simulation_counts <- 5000
    }
    # else
    #   simulation_counts <- 100
  }
  else
    simulation_counts <- no_sim_count
  
  xlabel_ds_class <- paste0(ds_name, ' (', ds_class_name, ') feature counts')
  
  # No scientific notation (e.g. 1e+05) in axis in plots
  scipen_option <- getOption('scipen')
  options(scipen=999)
  
  ############ Power-law #######################################################
  # (Discrete) Power-law fit? xmin is estimated
  fit_power_law <- displ$new(feat_counts)
  est <- estimate_xmin(fit_power_law)
  fit_power_law$setXmin(est)
  est <- estimate_pars(fit_power_law)
  fit_power_law$setPars(est)
  
  bs_power_law <- bootstrap_p(fit_power_law, xmax = 1e+06,
                              threads=threads, no_of_sims=simulation_counts)
  
  # Power-law, bootsptrap parameters
  xmin_pl <- fit_power_law$xmin
  ntail_pl <- get_ntail(fit_power_law, prop=FALSE)
  
  # (Discrete) Log-Normal fit?
  fit_log_normal_with_pl <- dislnorm$new(feat_counts)
  # Set Xmin from power-law (they should be the same in both distributions)
  # est <- estimate_xmin(fit_log_normal)
  fit_log_normal_with_pl$setXmin(xmin_pl)
  est <- estimate_pars(fit_log_normal_with_pl)
  fit_log_normal_with_pl$setPars(est)
  
  # Vuong's test,
  # A likelihood ratio test for model selection
  # using the Kullback-Leibler criteria
  compdist_pl_vs_ln_with_pl <- compare_distributions(
    fit_power_law, fit_log_normal_with_pl)
  if (plot_graph_compare_two_dists) {
    plot(compdist_pl_vs_ln_with_pl,
         col=ifelse(compdist_pl_vs_ln_with_pl$ratio$ratio < -sign_threshold,
                    cols_dist[col_distln],
                    ifelse(compdist_pl_vs_ln_with_pl$ratio$ratio > sign_threshold,
                           cols_dist[col_distpl], 'black')),
         main=paste0('Vuong\'s test: Power Law* vs. Log-Normal (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
    }
  
  # (Discrete) Poisson fit?
  fit_poisson_with_pl <- dispois$new(feat_counts)
  # Set Xmin from power-law (they should be the same in both distributions)
  # est <- estimate_xmin(fit_poisson)
  fit_poisson_with_pl$setXmin(xmin_pl)
  est <- estimate_pars(fit_poisson_with_pl)
  fit_poisson_with_pl$setPars(est)
  
  compdist_pl_vs_po_with_pl <- compare_distributions(
    fit_power_law, fit_poisson_with_pl)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_pl_vs_po_with_pl,
         col=ifelse(compdist_pl_vs_po_with_pl$ratio$ratio < -sign_threshold,
                    cols_dist[col_distpo],
                    ifelse(compdist_pl_vs_po_with_pl$ratio$ratio > sign_threshold,
                           cols_dist[col_distpl], 'black')),
         main=paste0('Vuong\'s Test: Power Law* vs. Poisson (', ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  # (Discrete) Exponential fit?
  fit_exponential_with_pl <- disexp$new(feat_counts)
  # Set Xmin from power-law (they should be the same in both distributions)
  # est <- estimate_xmin(fit_exponential)
  fit_exponential_with_pl$setXmin(xmin_pl)
  est <- estimate_pars(fit_exponential_with_pl)
  fit_exponential_with_pl$setPars(est)
  
  compdist_pl_vs_ex_with_pl <- compare_distributions(
    fit_power_law, fit_exponential_with_pl)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_pl_vs_ex_with_pl,
         col=ifelse(compdist_pl_vs_ex_with_pl$ratio$ratio < -sign_threshold,
                    cols_dist[col_distex],
                    ifelse(compdist_pl_vs_ex_with_pl$ratio$ratio > sign_threshold,
                           cols_dist[col_distpl], 'black')),
         main=paste0('Vuong\'s Test: Power Law* vs. Exponential (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  ############ Log-Normal ######################################################
  # (Discrete) Log-Normal fit ? xmin is estimated
  fit_log_normal_only <- dislnorm$new(feat_counts)
  est <- estimate_xmin(fit_log_normal_only)
  fit_log_normal_only$setXmin(est)
  est <- estimate_pars(fit_log_normal_only)
  fit_log_normal_only$setPars(est)
  
  bs_log_normal <- bootstrap_p(fit_log_normal_only, xmax = 1e+06,
                               threads=threads, no_of_sims=simulation_counts)
  
  # Log-Normal, bootsptrap parameters
  xmin_ln <- fit_log_normal_only$xmin
  ntail_ln <- get_ntail(fit_log_normal_only, prop=FALSE)
  
  # (Discrete) Power-law fit?
  fit_power_law_with_ln <- displ$new(feat_counts)
  # Set Xmin from Log-Normal (they should be the same in both distributions)
  # est <- estimate_xmin(fit_power_law_with_ln)
  fit_power_law_with_ln$setXmin(xmin_ln)
  est <- estimate_pars(fit_power_law_with_ln)
  fit_power_law_with_ln$setPars(est)
  
  # Vuong's test,
  # A likelihood ratio test for model selection
  # using the Kullback-Leibler criteria
  compdist_pl_with_ln_vs_ln <- compare_distributions(
    fit_power_law_with_ln, fit_log_normal_only)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_pl_with_ln_vs_ln,
         col=ifelse(compdist_pl_with_ln_vs_ln$ratio$ratio < -sign_threshold,
                    cols_dist[col_distln],
                    ifelse(compdist_pl_with_ln_vs_ln$ratio$ratio > sign_threshold,
                           cols_dist[col_distpl], 'black')),
         main=paste0('Vuong\'s Test: Power Law vs. Log-Normal* (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  # (Discrete) Poisson fit?
  fit_poisson_with_ln <- dispois$new(feat_counts)
  # Set Xmin from Log-Normal (they should be the same in both distributions)
  # est <- estimate_xmin(fit_poisson_with_ln)
  fit_poisson_with_ln$setXmin(xmin_ln)
  est <- estimate_pars(fit_poisson_with_ln)
  fit_poisson_with_ln$setPars(est)
  
  compdist_po_with_ln_vs_ln <- compare_distributions(
    fit_power_law_with_ln, fit_log_normal_only)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_po_with_ln_vs_ln,
         col=ifelse(compdist_po_with_ln_vs_ln$ratio$ratio < -sign_threshold,
                    cols_dist[col_distln],
                    ifelse(compdist_po_with_ln_vs_ln$ratio$ratio > sign_threshold,
                           cols_dist[col_distpo], 'black')),
         main=paste0('Vuong\'s Test: Poisson vs. Log-Normal* (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  # (Discrete) Exponential fit?
  fit_exponential_with_ln <- disexp$new(feat_counts)
  # Set Xmin from Log-Normal (they should be the same in both distributions)
  # est <- estimate_xmin(fit_exponential_with_ln)
  fit_exponential_with_ln$setXmin(xmin_ln)
  est <- estimate_pars(fit_exponential_with_ln)
  fit_exponential_with_ln$setPars(est)
  
  compdist_ex_with_ln_vs_ln <- compare_distributions(
    fit_exponential_with_ln, fit_log_normal_only)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_ex_with_ln_vs_ln,
         col=ifelse(compdist_ex_with_ln_vs_ln$ratio$ratio < -sign_threshold,
                    cols_dist[col_distln],
                    ifelse(compdist_ex_with_ln_vs_ln$ratio$ratio > sign_threshold,
                           cols_dist[col_distex], 'black')),
         main=paste0('Vuong\'s Test: Exponential vs. Log-Normal* (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  ############ Exponential #####################################################
  # (Discrete) Exponential fit ? xmin is estimated
  fit_exponential_only <- disexp$new(feat_counts)
  est <- estimate_xmin(fit_exponential_only)
  fit_exponential_only$setXmin(est)
  est <- estimate_pars(fit_exponential_only)
  fit_exponential_only$setPars(est)
  
  bs_exponential <- bootstrap_p(fit_exponential_only, xmax = 1e+06,
                               threads=threads, no_of_sims=simulation_counts)
  
  # Exponential, bootsptrap parameters
  xmin_ex <- fit_exponential_only$xmin
  ntail_ex <- get_ntail(fit_exponential_only, prop=FALSE)
  
  # (Discrete) Power-law fit?
  fit_power_law_with_ex <- displ$new(feat_counts)
  # Set Xmin from Exponential (they should be the same in both distributions)
  # est <- estimate_xmin(fit_power_law_with_ex)
  fit_power_law_with_ex$setXmin(xmin_ex)
  est <- estimate_pars(fit_power_law_with_ex)
  fit_power_law_with_ex$setPars(est)
  
  # Vuong's test,
  # A likelihood ratio test for model selection
  # using the Kullback-Leibler criteria
  compdist_pl_with_ex_vs_ex <- compare_distributions(
    fit_power_law_with_ex, fit_exponential_only)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_pl_with_ex_vs_ex,
         col=ifelse(compdist_pl_with_ex_vs_ex$ratio$ratio < -sign_threshold,
                    cols_dist[col_distln],
                    ifelse(compdist_pl_with_ex_vs_ex$ratio$ratio > sign_threshold,
                           cols_dist[col_distpl], 'black')),
         main=paste0('Vuong\'s Test: Power Law vs. Exponential* (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  # (Discrete) Log-Normal fit?
  fit_log_normal_with_ex <- dislnorm$new(feat_counts)
  # Set Xmin from Exponential (they should be the same in both distributions)
  # est <- estimate_xmin(fit_log_normal_with_ex)
  fit_log_normal_with_ex$setXmin(xmin_ex)
  est <- estimate_pars(fit_log_normal_with_ex)
  fit_log_normal_with_ex$setPars(est)
  
  # Vuong's test,
  # A likelihood ratio test for model selection
  # using the Kullback-Leibler criteria
  compdist_ln_with_ex_vs_ex <- compare_distributions(
    fit_log_normal_with_ex, fit_exponential_only)
  
  if (plot_graph_compare_two_dists) {
    plot(compdist_ln_with_ex_vs_ex,
         col=ifelse(compdist_ln_with_ex_vs_ex$ratio$ratio < -sign_threshold,
                    cols_dist[col_distln],
                    ifelse(compdist_ln_with_ex_vs_ex$ratio$ratio > sign_threshold,
                           cols_dist[col_distpl], 'black')),
         main=paste0('Vuong\'s Test: Log-Normal vs. Exponential* (',
                     ds_name, ')'),
         xlab=xlabel_ds_class)
  }
  
  # (Discrete) Poisson fit?
  fit_poisson_with_ex <- dispois$new(feat_counts)
  # Set Xmin from Exponential (they should be the same in both distributions)
  # est <- estimate_xmin(fit_poisson_with_ex)
  fit_poisson_with_ex$setXmin(xmin_ex)
  est <- estimate_pars(fit_poisson_with_ex)
  fit_poisson_with_ex$setPars(est)
  
  ############ Dump Results ####################################################
  ntail_ratio_pl <- get_ntail(fit_power_law, prop=TRUE)
  ntail_ratio_ln <- get_ntail(fit_log_normal_only, prop=TRUE)
  ntail_ratio_ex <- get_ntail(fit_exponential_only, prop=TRUE)
  
  # pl* vs. ln
  if (compdist_pl_vs_ln_with_pl$test_statistic < -sign_threshold)
    # Almost equal to double tilda ~
    closer_to_the_truth_pl_vs_ln_with_pl <- 'Log-Normal ≈ the truth'
  else if (compdist_pl_vs_ln_with_pl$test_statistic > sign_threshold)
    closer_to_the_truth_pl_vs_ln_with_pl <- 'no better or worse fit'
  else
    closer_to_the_truth_pl_vs_ln_with_pl <- 'Power Law ≈ the truth'
  
  # pl vs. ln*
  if (compdist_pl_with_ln_vs_ln$test_statistic < -sign_threshold)
    # Almost equal to double tilda ~
    closer_to_the_truth_pl_with_ln_vs_ln <- 'Log-Normal ≈ the truth'
  else if (compdist_pl_with_ln_vs_ln$test_statistic > sign_threshold)
    closer_to_the_truth_pl_with_ln_vs_ln <- 'no better or worse fit'
  else
    closer_to_the_truth_pl_with_ln_vs_ln <- 'Power Law ≈ the truth'
  
  # ex vs. ln*
  if (compdist_ex_with_ln_vs_ln$test_statistic < -sign_threshold)
    # Almost equal to double tilda ~
    closer_to_the_truth_ex_with_ln_vs_ln <- 'Log-Normal ≈ the truth'
  else if (compdist_ex_with_ln_vs_ln$test_statistic > sign_threshold)
    closer_to_the_truth_ex_with_ln_vs_ln <- 'no better or worse fit'
  else
    closer_to_the_truth_ex_with_ln_vs_ln <- 'Exponential ≈ the truth'
  
  # pl vs. ex*
  if (compdist_pl_with_ex_vs_ex$test_statistic < -sign_threshold)
    # Almost equal to double tilda ~
    closer_to_the_truth_pl_with_ex_vs_ex <- 'Exponential ≈ the truth'
  else if (compdist_pl_vs_ln_with_pl$test_statistic > sign_threshold)
    closer_to_the_truth_pl_with_ex_vs_ex <- 'no better or worse fit'
  else
    closer_to_the_truth_pl_with_ex_vs_ex <- 'Power Law ≈ the truth'
  
  # ln vs. ex*
  if (compdist_ln_with_ex_vs_ex$test_statistic < -sign_threshold)
    # Almost equal to double tilda ~
    closer_to_the_truth_ln_with_ex_vs_ex <- 'Exponential ≈ the truth'
  else if (compdist_pl_with_ln_vs_ln$test_statistic > sign_threshold)
    closer_to_the_truth_ln_with_ex_vs_ex <- 'no better or worse fit'
  else
    closer_to_the_truth_ln_with_ex_vs_ex <- 'Log-Normal ≈ the truth'
  
  # See (Clauset, Shalizi, and Newman, 2, pp.2) for typical values
  # See (Gillespie, 2015, pp.3) for moments in continous Power Law
  if (fit_power_law$pars > 1 && fit_power_law$pars <= 2) {
    pl_alpha_type <-
      '1 < typical <= 2'
    pl_alpha_moments <- 'all moments diverge 1:µ,2:σ2,3:skewness,4:kurtosis'
  }
  else if (fit_power_law$pars > 2 && fit_power_law$pars <= 3) {
    pl_alpha_type <-
      '2 < typical <= 3'
    pl_alpha_moments <- '1:µ,2:σ2 converge but >2nd moments diverge 3:skewness,4:kurtosis'
  }
  else {
    pl_alpha_type <-
      'atypical > 3'
    pl_alpha_moments <- '1:µ,2:σ2,3:skewness converge but >α moments diverge 4:kurtosis)'
  }
  
  print(paste(
    c('Dataset', 'Class', 'Feature count min', 'Feature space size',
      # Power-Law fit
      'xmin (pl)', 'Parameters (pl)[alpha]', 'Alpha Type', 'Alpha Moments',
      'ntail (fitted feature count) (pl)', 'ntail ratio (pl)',
      # Power-Law estimation
      'Bootstrap p-value (pl)', 'Bootstrap GoF (pl)',
      'Bootstrap Simulation Count (pl)', 'Simulation Time (pl)[minute]',
      'Package Version', 'GoF Distance Measure',
      # Comparison between Power-Law and the other 3 ditributions
      'Power Law* vs. Log-Normal: Test Statistics',
      'pl* vs. ln: p-value (1-sided)', 'pl* vs. ln: p-value (2-sided)',
      'pl* vs. ln: result',
      'Power Law* vs. Poisson: Test Statistics',
      'pl* vs. po: p-value (1-sided)', 'pl* vs. po: p-value (2-sided)',
      'Power Law* vs. Exponential: Test Statistics',
      'pl* vs. ex: p-value (1-sided)', 'pl* vs. ex: p-value (2-sided)',
      
      # Log-Normal fit
      'xmin (ln)', 'Parameters (ln)[mean]', 'Parameters (ln)[SD]',
      'ntail (fitted feature count) (ln)', 'ntail ratio (ln)',
      # Log-Normal estimation
      'Bootstrap p-value (ln)', 'Bootstrap GoF (ln)',
      'Bootstrap Simulation Count (ln)', 'Simulation Time (ln)[minute]',
      # Comparison between the other 3 distributions and Log-Normal
      'Power Law vs. Log-Normal*: Test Statistics',
      'pl vs. ln*: p-value (1-sided)', 'pl vs. ln*: p-value (2-sided)',
      'pl vs. ln*: result',
      'Poisson vs. Log-Normal*: Test Statistics',
      'po vs. ln*: p-value (1-sided)', 'po vs. ln*: p-value (2-sided)',
      'Exponential vs. Log-Normal*: Test Statistics',
      'ex vs. ln*: p-value (1-sided)', 'ex vs. ln*: p-value (2-sided)',
      'ex vs. ln*: result',
      
      # Exponential fit
      'xmin (ex)', 'Parameters (ex)[lambda]',
      'ntail (fitted feature count) (ex)', 'ntail ratio (ex)',
      # Exponential estimation
      'Bootstrap p-value (ex)', 'Bootstrap GoF (ex)',
      'Bootstrap Simulation Count (ex)', 'Simulation Time (ex)[minute]',
      # Comparison between Power-Law and other 2 ditributions
      'Power Law vs. Exponential*: Test Statistics',
      'pl vs. ex*: p-value (1-sided)', 'pl vs. ex*: p-value (2-sided)',
      'pl vs. ex*: result',
      'Log-normal vs. Exponential*: Test Statistics',
      'ln vs. ex*: p-value (1-sided)', 'ln vs. ex*: p-value (2-sided)',
      'ln vs. ex*: result',
      
      '\n'),
    collapse='\t'))
  print(paste(
    ds_name, ds_class_name, min(feat_counts), feature_space_size,
    
    # pl* POWER LAW
    xmin_pl, fit_power_law$pars, pl_alpha_type, pl_alpha_moments,
    ntail_pl, ntail_ratio_pl,
    # pl* bootstrap
    bs_power_law$p, bs_power_law$gof,
    simulation_counts, bs_power_law$sim_time,
    bs_power_law$package_version, bs_power_law$distance,
    # pl* vs. ln
    compdist_pl_vs_ln_with_pl$test_statistic,
    compdist_pl_vs_ln_with_pl$p_one_sided, compdist_pl_vs_ln_with_pl$p_two_sided,
    closer_to_the_truth_pl_vs_ln_with_pl,
    # pl* vs. po
    compdist_pl_vs_po_with_pl$test_statistic,
    compdist_pl_vs_po_with_pl$p_one_sided, compdist_pl_vs_po_with_pl$p_two_sided,
    # pl* vs. ex
    compdist_pl_vs_ex_with_pl$test_statistic,
    compdist_pl_vs_ex_with_pl$p_one_sided, compdist_pl_vs_ex_with_pl$p_two_sided,
    
    # ln* LOG-NORMAL
    xmin_ln, fit_log_normal_only$pars[1], fit_log_normal_only$pars[2],
    ntail_ln, ntail_ratio_ln,
    # ln* bootstrap
    bs_log_normal$p, bs_log_normal$gof,
    simulation_counts, bs_log_normal$sim_time,
    # pl vs. ln*
    compdist_pl_with_ln_vs_ln$test_statistic,
    compdist_pl_with_ln_vs_ln$p_one_sided, compdist_pl_with_ln_vs_ln$p_two_sided,
    closer_to_the_truth_pl_with_ln_vs_ln,
    # po vs. ln*
    compdist_po_with_ln_vs_ln$test_statistic,
    compdist_po_with_ln_vs_ln$p_one_sided, compdist_po_with_ln_vs_ln$p_two_sided,
    # ex vs. ln*
    compdist_ex_with_ln_vs_ln$test_statistic,
    compdist_ex_with_ln_vs_ln$p_one_sided, compdist_ex_with_ln_vs_ln$p_two_sided,
    closer_to_the_truth_ex_with_ln_vs_ln,
    
    # ex* EXPONENTIAL
    xmin_ex, fit_exponential_only$pars[1],
    ntail_ex, ntail_ratio_ex,
    # exp* bootstrap
    bs_exponential$p, bs_exponential$gof,
    simulation_counts, bs_exponential$sim_time,
    # pl vs. ex*
    compdist_pl_with_ex_vs_ex$test_statistic,
    compdist_pl_with_ex_vs_ex$p_one_sided, compdist_pl_with_ex_vs_ex$p_two_sided,
    closer_to_the_truth_pl_with_ex_vs_ex,
    # ln vs. ex*
    compdist_ln_with_ex_vs_ex$test_statistic,
    compdist_ln_with_ex_vs_ex$p_one_sided, compdist_ln_with_ex_vs_ex$p_two_sided,
    closer_to_the_truth_ln_with_ex_vs_ex,
    '[Estimation, simulations, and comparisons are completed]\n',
    sep='\t'))
  
  # Power-Law fitted and not-fitted features dump
  cat(paste('Fitted feature names (Power Law*) [>=xmin]: ',
            paste(ds_feat_space_names[feat_counts >= xmin_pl][1:ntail_pl],
                  collapse=', '), '\n'))
  print(paste(
    'Not fitted feature names (Power Law*) [<xmin]: ',
    paste(
      ds_feat_space_names[feat_counts < xmin_pl]
      [1:(feature_space_size-ntail_pl)],
      collapse=', '), '\n'))
  
  # Log-Normal fitted and not-fitted features dump
  print(paste('Fitted feature names (Log-Normal*) [>=xmin]: ',
              paste(ds_feat_space_names[feat_counts >= xmin_ln][1:ntail_ln],
                    collapse=', '), '\n'))
  print(paste(
    'Not fitted feature names (Log-Normal*) [<xmin]: ',
    paste(
      ds_feat_space_names[feat_counts < xmin_ln][1:(feature_space_size-ntail_ln)],
      collapse=', '), '\n'))
  
  # Exponential fitted and not-fitted features dump
  print(paste('Fitted feature names (Exponential*) [>=xmin]: ',
              paste(ds_feat_space_names[feat_counts >= xmin_ex][1:ntail_ex],
                    collapse=', '), '\n'))
  print(paste(
    'Not fitted feature names (Exponential*) [<xmin]: ',
    paste(
      ds_feat_space_names[feat_counts < xmin_ex][1:(feature_space_size-ntail_ex)],
      collapse=', '), '\n'))
  
  options(scipen=scipen_option)
  
  # return from the function as below for the following error
  # Error in grid.Call.graphics(C_setviewport, vp, TRUE) : 
  #   non-finite location and/or size for viewport
  # Called from: grid.Call.graphics(C_setviewport, vp, TRUE)
  # return(list(bs_power_law, bs_log_normal))
  
  # Plot pl* and other three distributions
  if (plot_graph_compare_all_dist) {
    plot(fit_power_law,
         xlab=xlabel_ds_class,
         ylab='Rank',
         main=paste0(
           'Power Law against other fits for feature distribution of ',
           ds_name, ' (', ds_class_name, ')')
         # sub =  '[Power-Law Fit]')
    )
    lines(fit_power_law,
          col=cols_dist[col_distpl], lty=1, lwd=4)
    lines(fit_log_normal_with_pl,
          col=cols_dist[col_distln], lty=2, lwd=2)
    lines(fit_exponential_with_pl,
          col=cols_dist[col_distex], lty=3, lwd=2)
    lines(fit_poisson_with_pl,
          col=cols_dist[col_distpo], lty=4, lwd=2)
    abline(v=xmin_pl, lty=3)
    mtext(paste('α:', round(fit_power_law$pars, digits=2),
                paste0(' where ', pl_alpha_type,
                       ' \n(', pl_alpha_moments, ')'),
                '\nVuong\'s test statistics (pl* vs. ln):',
                round(compdist_pl_vs_ln_with_pl$test_statistic, digits=3),
                paste0('\n(', closer_to_the_truth_pl_vs_ln_with_pl, ')'),
                '\np (1-sided):',
                round(compdist_pl_vs_ln_with_pl$p_one_sided, digits=2),
                '\np (2-sided):',
                round(compdist_pl_vs_ln_with_pl$p_two_sided, digits=2), ' '),
          adj=1, line=-6*0.75, side=3, cex=0.75)
    mtext(paste0('Xmin:', xmin_pl, ', ntail ratio (X>=Xmin):',
                 100*round(ntail_ratio_pl, digits=2), "%"),
          adj=1-ntail_ratio_pl, side=3, line=0, cex=0.75)
    legend(x='bottomleft', box.lty=0, col=cols_dist, lty=1:4, lwd=2,
           legend=c('Power Law*', 'Log-Normal', 'Exponential', 'Poisson'))
  }
  
  # Plot ln* and other three distributions
  if (plot_graph_compare_all_dist) {
    plot(fit_log_normal_only,
         xlab=xlabel_ds_class,
         ylab='Rank',
         main=paste0(
           'Log-Normal against other fits for feature distribution of ',
           ds_name, ' (', ds_class_name, ')')
         # sub='[Log-Normal Fit]')
    )
    lines(fit_power_law_with_ln,
          col=cols_dist[col_distpl], lty=1, lwd=2)
    lines(fit_log_normal_only,
          col=cols_dist[col_distln], lty=2, lwd=4)
    lines(fit_exponential_with_ln,
          col=cols_dist[col_distex], lty=3, lwd=2)
    lines(fit_poisson_with_ln,
          col=cols_dist[col_distpo], lty=4, lwd=2)
    abline(v=xmin_ln, lty=3)
    mtext(paste('µ (mean):', round(fit_log_normal_only$pars[1], digits=2),
                ', σ (SD):', round(fit_log_normal_only$pars[2], digits=2),
                '\nVuong\'s test statistics: (pl vs. ln*)',
                round(compdist_pl_with_ln_vs_ln$test_statistic, digits=3),
                paste0('\n(', closer_to_the_truth_pl_with_ln_vs_ln, ')'),
                '\np (1-sided):',
                round(compdist_pl_with_ln_vs_ln$p_one_sided, digits=2),
                '\np (2-sided):',
                round(compdist_pl_with_ln_vs_ln$p_two_sided, digits=2), ' '),
          adj=1, line=-5*0.75, side=3, cex=0.75)
    mtext(paste0('Xmin:', xmin_ln, ', ntail ratio (X>=Xmin):',
                 100*round(ntail_ratio_ln, digits=2), "%"),
          adj=1-ntail_ratio_ln, side=3, line=0, cex=0.75)
    legend(x='bottomleft', box.lty=0, col=cols_dist, lty=1:4, lwd=2,
           legend=c('Power Law', 'Log-Normal*', 'Exponential', 'Poisson'))
  }
  
  # Plot ex* and other three distributions
  if (plot_graph_compare_all_dist) {
    plot(fit_exponential_only,
         xlab=xlabel_ds_class,
         ylab='Rank',
         main=paste0(
           'Exponential against other fits for feature distribution of ',
           ds_name, ' (', ds_class_name, ')')
         # sub='[Exponential Fit]')
    )
    lines(fit_power_law_with_ex,
          col=cols_dist[col_distpl], lty=1, lwd=2)
    lines(fit_log_normal_with_ex,
          col=cols_dist[col_distln], lty=2, lwd=4)
    lines(fit_exponential_only,
          col=cols_dist[col_distex], lty=3, lwd=2)
    lines(fit_poisson_with_ex,
          col=cols_dist[col_distpo], lty=4, lwd=2)
    abline(v=xmin_ex, lty=3)
    mtext(paste('λ (rate):', round(fit_exponential_only$pars[1], digits=2),
                '\nVuong\'s test statistics: (ln vs. ex*)',
                round(compdist_ln_with_ex_vs_ex$test_statistic, digits=3),
                paste0('\n(', closer_to_the_truth_ln_with_ex_vs_ex, ')'),
                '\np (1-sided):',
                round(compdist_ln_with_ex_vs_ex$p_one_sided, digits=2),
                '\np (2-sided):',
                round(compdist_ln_with_ex_vs_ex$p_two_sided, digits=2), ' '),
          adj=1, line=-5*0.75, side=3, cex=0.75)
    mtext(paste0('Xmin:', xmin_ex, ', ntail ratio (X>=Xmin):',
                 100*round(ntail_ratio_ex, digits=2), "%"),
          adj=1-ntail_ratio_ex, side=3, line=0, cex=0.75)
    legend(x='bottomleft', box.lty=0, col=cols_dist, lty=1:4, lwd=2,
           legend=c('Power Law', 'Log-Normal', 'Exponential*', 'Poisson'))
  }
  
  plot(bs_power_law,
       main='bootstrapping hypothesis test: Power Law distribution is plausible?')
  plot(bs_log_normal,
       main='bootstrapping hypothesis test: Log-Normal distribution is plausible?')
  plot(bs_exponential,
       main='bootstrapping hypothesis test: Exponential distribution is plausible?')
  
  # reinstall gglot2 (install.packages('ggplot2')) for the following error
  # Error in grid.Call.graphics(C_setviewport, vp, TRUE) : 
  #   non-finite location and/or size for viewport
  # Called from: grid.Call.graphics(C_setviewport, vp, TRUE)
  
  # Comparison of likelihood ratios per feature
  # Based on N. Bertchinger
  # https://fias.uni-frankfurt.de/fileadmin/fias/bertschinger/CN/SolutionsWiSe1718_Ex2.pdf
  
  if (plot_graph_likelihood_ratios) {
    plotDistributionsLikelyhoodRatios(
      xlabel_ds_class, compdist_pl_vs_ln_with_pl,
      'Power Law*', 'Log-Normal',
      cols_dist[col_distpl], cols_dist[col_distln],
      closer_to_the_truth_pl_vs_ln_with_pl)
    
    # print(compdist_pl_vs_ln_with_pl$ratio %>%
    #         group_by(x, ratio) %>%
    #         summarize(cnt = n()) %>%
    #         ggplot(aes(x, ratio, color=ratio < 0, size=cnt)) +
    #         theme_bw() +
    #         geom_point(alpha=0.6) +
    #         # scale_color_discrete(name='Test results for distributions',
    #         #                      labels=c('Power Law*', 'Log-Normal')) +
    #         scale_color_manual(values=c(cols_dist[col_distpl],
    #                                     cols_dist[col_distln]),
    #                            name='Vuong\'s test on',
    #                            labels=c('Power Law* vs.', 'Log-Normal')) +
    #         labs(x=xlabel_ds_class,
    #              y=paste0('Log-likelihood ratios.',
    #                       ' Test statistics:',
    #                       round(compdist_pl_vs_ln_with_pl$test_statistic, digits=3),
    #                       '\n(', closer_to_the_truth_pl_vs_ln_with_pl, ')',
    #                       ', p (1-sided):',
    #                       round(compdist_pl_vs_ln_with_pl$p_one_sided, digits=2),
    #                       ', p (2-sided):',
    #                       round(compdist_pl_vs_ln_with_pl$p_two_sided, digits=2)),
    #              size='Count') +
    #         scale_x_log10())
  }
  
  if (plot_graph_likelihood_ratios) {
    plotDistributionsLikelyhoodRatios(
      xlabel_ds_class, compdist_pl_with_ln_vs_ln,
      'Power Law', 'Log-Normal*',
      cols_dist[col_distpl], cols_dist[col_distln],
      closer_to_the_truth_pl_with_ln_vs_ln)
    
    # print(compdist_pl_with_ln_vs_ln$ratio %>%
    #         group_by(x, ratio) %>%
    #         summarize(cnt = n()) %>%
    #         ggplot(aes(x, ratio, color=ratio < 0, size=cnt)) +
    #         theme_bw() +
    #         geom_point(alpha=0.6) +
    #         scale_color_manual(values=c(cols_dist[col_distpl],
    #                                     cols_dist[col_distln]),
    #                            name='Vuong\'s test on',
    #                            labels=c('Power Law vs.', 'Log-Normal*')) +
    #         labs(x=xlabel_ds_class,
    #              y=paste0('Log-likelihood ratios.',
    #                       ' Test statistics:',
    #                       round(compdist_pl_with_ln_vs_ln$test_statistic, digits=3),
    #                       '\n(', closer_to_the_truth_pl_with_ln_vs_ln, ')',
    #                       ', p (1-sided):',
    #                       round(compdist_pl_with_ln_vs_ln$p_one_sided, digits=2),
    #                       ', p (2-sided):',
    #                       round(compdist_pl_with_ln_vs_ln$p_two_sided, digits=2)),
    #              size='Count') +
    #         scale_x_log10())
  }
  
  if (plot_graph_likelihood_ratios) {
    plotDistributionsLikelyhoodRatios(
      xlabel_ds_class, compdist_ln_with_ex_vs_ex,
      'Log-Normal', 'Exponential*',
      cols_dist[col_distln], cols_dist[col_distex],
      closer_to_the_truth_ln_with_ex_vs_ex)
    
    # print(compdist_ln_with_ex_vs_ex$ratio %>%
    #         group_by(x, ratio) %>%
    #         summarize(cnt = n()) %>%
    #         ggplot(aes(x, ratio, color=ratio < 0, size=cnt)) +
    #         theme_bw() +
    #         geom_point(alpha=0.6) +
    #         scale_color_manual(values=c(cols_dist[col_distln],
    #                                     cols_dist[col_distex]),
    #                            name='Vuong\'s test on',
    #                            labels=c('Log-Normal vs.', 'Exponential*')) +
    #         labs(x=xlabel_ds_class,
    #              y=paste0('Log-likelihood ratios.',
    #                       ' Test statistics:',
    #                       round(compdist_ln_with_ex_vs_ex$test_statistic,
    #                             digits=3),
    #                       '\n(', closer_to_the_truth_ln_with_ex_vs_ex, ')',
    #                       ', p (1-sided):',
    #                       round(compdist_ln_with_ex_vs_ex$p_one_sided,
    #                             digits=2),
    #                       ', p (2-sided):',
    #                       round(compdist_ln_with_ex_vs_ex$p_two_sided, digits=2)),
    #              size='Count') +
    #         scale_x_log10())
  }
  
  options(scipen=scipen_option)
  
  return(list(bs_power_law, bs_log_normal, bs_exponential))
}
```

### plotDistributionsLikelyhoodRatios
Plot two compared distributions' log-likelihood ratio tests  
**Parameters:**  
*dataset_class*: Class identifier (e.g. 'Positive' or 'Malign')  
*compdist_dist1_dist2*: Distribution comparison object  
*dist1_name*:  The name of the 1st distribution  
*dist2_name*:  The name of the 2nd distribution  
*col_dist1*:  The color of the 1st distribution  
*col_dist2*:  The color of the 2nd distribution  
*which_dist_is_closer_the_truth*:  Which distribution is closer the truth?  
**Return:**  
none  


```r
plotDistributionsLikelyhoodRatios <- function(dataset_class,
                                              compdist_dist1_dist2,
                                              dist1_name, dist2_name,
                                              col_dist1, col_dist2,
                                              which_dist_is_closer_the_truth)
{
  print(compdist_dist1_dist2$ratio %>%
          group_by(x, ratio) %>%
          summarize(cnt = n()) %>%
          ggplot(aes(x, ratio, color=ratio < 0, size=cnt)) +
          theme_bw() +
          geom_point(alpha=0.6) +
          scale_color_manual(values=c(col_dist1, col_dist2),
                             name='Vuong\'s test on',
                             labels=c(paste0(dist1_name, ' vs.'), dist2_name)) +
          labs(x=dataset_class,
               y=paste0('Log-likelihood ratios.',
                        ' Test statistics:',
                        round(compdist_dist1_dist2$test_statistic,
                              digits=3),
                        '\n(', which_dist_is_closer_the_truth, ')',
                        ', p (1-sided):',
                        round(compdist_dist1_dist2$p_one_sided,
                              digits=2),
                        ', p (2-sided):',
                        round(compdist_dist1_dist2$p_two_sided, digits=2)),
               size='Count') +
          scale_x_log10())
}
```

### getCommonFeatures
Return the intersection of feature names given as a comma seperated list per dataset (i.e. Copy)  
**Parameters:**  
*ds_fit_unfit_features*: Performance metric  
*split*: Seperator between column values (default: ', ')  
**Return:**  
common_features as character vector  
**Examples:**  


```r
# 1) Open datasets.ods file  
# 2) Select Features (Permissions) cells in dsdist (fit features) worksheet  
#      for the following filters  
#         "Malign" in Class and  
#         1 in Plausibility Degree and  
#         "Fitted (>=xmin)" in Type and  
#         exclude "NA" rows  
#      Don't forget to include the first header row cell  
#         "Features (Permissions)" into your selection  
# 3) Copy the selected cells  
# 4) Run the following commands  
# `malign_fit <- rclip()`  
# `malign_fit_common <- getCommonFeatures(malign_fit)`  
getCommonFeatures<-function(ds_fit_unfit_features, split=', ')
{
  common_features <- character()
  ds_count <- nrow(ds_fit_unfit_features)
  
  if (ds_count == 0) {
    return(common_features)
  }
  
  common_features <- strsplit(ds_fit_unfit_features[1, 1], split=split)[[1]]
  
  for (i in 1:(ds_count-1)) {
    features <- strsplit(ds_fit_unfit_features[i+1, 1], split=split)[[1]]
    common_features <- intersect(common_features, features)
  }
  
  return(common_features)
}
```

