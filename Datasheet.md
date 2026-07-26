# Datasheet for the Bayesian Black-Box Optimisation Dataset

## 1. Dataset overview

This dataset supports a Bayesian black-box optimisation capstone project. It contains observations from eight unknown objective functions of increasing dimensionality.

Each observation consists of:

* a set of numerical input variables;
* one numerical output from the relevant black-box function.

The objective is to identify input combinations that maximise the output of each function using a limited query budget.

## 2. Motivation

The dataset was created for an optimisation exercise in which the underlying mathematical forms of the functions are unknown. The task is therefore not conventional supervised prediction. Instead, the observed data are used to build surrogate models that guide the selection of new, expensive function evaluations.

The main learning objectives are:

* applying Bayesian optimisation under a limited query budget;
* balancing exploration and exploitation;
* assessing uncertainty;
* comparing different surrogate models;
* adapting modelling decisions as new observations become available.

## 3. Dataset composition

The dataset contains eight separate functions.

| Function   | Input dimensions | Initial observations | Current observations |
| ---------- | ---------------: | -------------------: | -------------------: |
| Function 1 |                2 |                   10 |                   15 |
| Function 2 |                2 |                   10 |                   15 |
| Function 3 |                3 |                   15 |                   20 |
| Function 4 |                4 |                   30 |                   35 |
| Function 5 |                4 |                   20 |                   25 |
| Function 6 |                5 |                   20 |                   25 |
| Function 7 |                6 |                   30 |                   35 |
| Function 8 |                8 |                   40 |                   45 |

All input variables are continuous and bounded between 0 and 1.

The output ranges differ substantially across functions. Some outputs are positive, some are negative, and some vary across several orders of magnitude.

## 4. Data instances

Each row represents one evaluated input combination.

Example structure:

```text
x1    x2    ...    output
0.35  0.42  ...    0.68
```

The inputs are candidate decisions or coordinates within the search space. The output is the value returned by the unknown black-box function.

There are no individual people, organisations or demographic groups represented in the dataset.

## 5. Data generation

The initial observations were supplied as part of the capstone exercise.

Additional observations were generated sequentially by submitting queries selected using Bayesian optimisation and alternative machine-learning models.

The main query-selection pipeline included:

* a Gaussian Process surrogate;
* a Matérn or RBF kernel;
* Automatic Relevance Determination length scales;
* Upper Confidence Bound acquisition;
* Sobol global candidate generation;
* local perturbations around the best observed point;
* an optional SVM good-region score;
* an optional distance-from-incumbent penalty.

Extra Trees, neural-network and XGBoost models were used as sensitivity checks.

## 6. Data preprocessing

Inputs were already bounded between 0 and 1, so no additional input scaling was required for the Gaussian Process.

The output was normally modelled on its original scale. Two exceptions were introduced after diagnostic testing:

* Function 1: an inverse hyperbolic sine transformation was tested because outputs were concentrated near zero but varied across many orders of magnitude and included both signs;
* Function 5: a `log1p` transformation was tested because the output distribution was strongly right-skewed.

The transformed outputs preserve ordering, so maximising the transformed output also maximises the original output.

## 7. Missing and invalid data

The dataset contains no intentionally missing values.

Before fitting, the pipeline checks that:

* the `output` column is present;
* all inputs and outputs are finite;
* input values lie between 0 and 1;
* sufficient observations are available.

Duplicate or near-duplicate candidate points may occur when local perturbations are clipped at the boundaries. Candidate deduplication is therefore recommended.

## 8. Dataset quality

Dataset quality differs across functions because the number of observations is small relative to the dimensionality of the search spaces.

Coverage is strongest for the lower-dimensional functions and weakest for Functions 7 and 8.

The observations are also adaptive rather than independently sampled. Later points are deliberately concentrated in regions predicted to perform well. The resulting dataset therefore becomes increasingly unrepresentative of the full input space.

This is appropriate for optimisation, but it means the data should not be treated as a random sample from the unit hypercube.

## 9. Important patterns and anomalies

Several functions have unusual characteristics:

* Function 1 contains outputs extremely close to zero, with values spanning many orders of magnitude.
* Function 4 appears to contain a relatively smooth interior optimum.
* Function 5 has a highly skewed output distribution and a sharp increase near the upper corner.
* Function 6 shows evidence that strong performance is associated with the fifth input being close to zero.
* Function 7 may contain a narrow nonlinear basin.
* Function 8 appears comparatively smooth and predictable despite having eight dimensions.

These patterns were discovered progressively and may change as new observations are added.

## 10. Biases and limitations

The principal limitations are:

* small sample sizes;
* sparse coverage in high dimensions;
* adaptive sampling concentrated near promising regions;
* sensitivity to unusual or extreme observations;
* possible narrow peaks that have not yet been observed;
* limited ability to distinguish true function structure from surrogate-model assumptions.

The dataset contains no demographic or social data, so conventional fairness concerns are not directly applicable. However, model-selection bias remains relevant: different surrogate models may favour different regions because of their assumptions about smoothness and extrapolation.

## 11. Recommended uses

The dataset is suitable for:

* Bayesian optimisation experiments;
* surrogate-model comparison;
* acquisition-function analysis;
* uncertainty-calibration exercises;
* optimisation under a fixed evaluation budget;
* educational demonstrations of exploration and exploitation.

## 12. Uses to avoid

The dataset should not be used:

* to claim that one surrogate model is universally superior;
* to make real-world safety-critical decisions;
* to infer causal relationships between inputs and outputs;
* to estimate population-level relationships;
* as evidence that a predicted optimum is the true global optimum.

## 13. Maintenance

The dataset should be updated after each submitted query.

Recommended versioning information includes:

* query round;
* input coordinates;
* observed output;
* model and acquisition strategy used;
* random seed;
* whether the point was global, local or manually included;
* transformation applied to the output.

## 14. Reproducibility

Random seeds are set separately for each function, generally using:

```python
random_state = 42 + function_number
```

The same seed should be retained when comparing alternative model specifications for one function so that differences are not driven by different candidate pools.

## 15. Licence and attribution

This dataset was supplied and extended for an educational capstone project.

Add the applicable course, repository and licence information here:

* Author: `Swasti Gupta`
* Course or programme: `Imperial Machine Learning`
* Repository: `https://github.com/swas-18/ML_BayesianOptimisation-/new/main`
* Licence: `[LICENCE]`
* Last updated: `July 2026`
