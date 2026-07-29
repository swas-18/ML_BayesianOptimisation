# Model Card for the Bayesian Optimisation Pipeline

## 1. Model overview

This project uses a Gaussian Process based Bayesian optimisation pipeline to identify high-performing input combinations for eight unknown black-box functions.

The model is not intended to predict outcomes for deployment. Its purpose is to select the next point to evaluate under a limited query budget.

## 2. Model details

**Primary model:** Gaussian Process Regressor

**Primary kernel:** Matérn kernel with (\nu=1.5)

**Input:** Continuous variables bounded between 0 and 1

**Output:** Predicted function value and predictive uncertainty

**Optimisation objective:** Maximisation

**Implementation:** Python using NumPy, SciPy, scikit-learn and scikit-optimize components

## 3. Intended use

The model is intended to:

* approximate each unknown objective function;
* estimate uncertainty in unobserved regions;
* rank candidate points;
* balance exploration and exploitation;
* recommend the next expensive black-box query.

It is designed for an educational Bayesian optimisation exercise rather than production deployment.

## 4. Model architecture

The main surrogate is a Gaussian Process with the following kernel structure:

```text
ConstantKernel × Matérn or RBF + WhiteKernel
```

Automatic Relevance Determination is implemented by estimating a separate length scale for each input variable.

Short length scales indicate that the output may change rapidly along a dimension. Long length scales indicate that the output changes slowly or that the variable may have limited relevance over the observed range.

The model is fitted using normalised outputs and multiple optimiser restarts.

## 5. Output transformations

Most functions are fitted on their original output scale.

Two function-specific transformations are used:

| Function   | Transformation     | Reason                                                                       |
| ---------- | ------------------ | ---------------------------------------------------------------------------- |
| Function 1 | `asinh(y / scale)` | Handles positive and negative outputs spread across many orders of magnitude |
| Function 5 | `log1p(y)`         | Compresses a strongly right-skewed output distribution                       |

Recommendations are selected in transformed space. Approximate predictions may then be converted back to the original scale for interpretation.

## 6. Candidate generation

The model evaluates a finite pool of candidate points.

Candidates are generated using:

* global Sobol low-discrepancy sampling;
* local Gaussian perturbations around the best observed point;
* optional manually included boundary points.

The global pool protects against missing a better region elsewhere. The local pool provides finer resolution near an identified promising basin.

Candidate points are bounded between 0 and 1 and should be deduplicated after generation.

## 7. Acquisition strategy

The base acquisition function is Upper Confidence Bound:

```text
UCB(x) = predicted mean(x) + kappa × predicted uncertainty(x)
```

A higher `kappa` gives more weight to exploration. A lower `kappa` gives more weight to exploitation.

Additional optional terms include:

* an SVM score that rewards candidates resembling historically good observations;
* an ARD-weighted distance penalty that discourages movement far from the current best point.

The ARD term is best understood as a trust-region or local-exploitation adjustment. It does not discourage points from being too close to the incumbent.

## 8. Supporting models

The following models are used as sensitivity checks:

* Extra Trees Regressor;
* neural network;
* XGBoost Regressor;
* RBF Support Vector Machine classifier (only in earlier versions).

These models do not replace the GP. They are primarily used to assess whether a promising region is robust to different modelling assumptions.

Greater confidence is placed on location agreement across models than on agreement in exact predicted values.

## 9. Evaluation

The model is evaluated using leave-one-out cross-validation.

For every observation:

1. the observation is removed;
2. a new GP is fitted to the remaining data;
3. the omitted output is predicted;
4. the prediction error and uncertainty are recorded.

Main metrics include:

* RMSE;
* MAE;
* RMSE relative to a constant-mean baseline;
* predictive skill;
* standardised residuals;
* (E[z^2]) calibration;
* 50%, 80% and 95% interval coverage;
* prediction-variation ratio.

## 10. Current performance

The leave-one-out diagnostics indicate:

| Function   | Assessment                                                             |
| ---------- | ---------------------------------------------------------------------- |
| Function 1 | Raw-scale GP performed poorly; transformed model preferred             |
| Function 2 | Good predictive performance and calibration                            |
| Function 3 | Strong predictive performance and reasonable calibration               |
| Function 4 | Excellent predictive performance; slightly conservative uncertainty    |
| Function 5 | Raw-scale GP performed poorly; log-transformed model preferred         |
| Function 6 | Useful predictions but some overconfidence                             |
| Function 7 | Moderate predictive value but severe overconfidence                    |
| Function 8 | Excellent predictive performance and slightly conservative uncertainty |

These assessments should be updated as additional queries are observed.

## 11. Limitations

The model has several important limitations:

* very small training datasets;
* sparse coverage in higher dimensions;
* reliance on stationary kernel assumptions;
* unstable ARD length scales when data are limited;
* possible overconfidence in narrow or poorly sampled regions;
* sensitivity to output scale and transformation;
* candidate recommendations depend on the generated candidate pool;
* auxiliary models may overfit because of the limited sample size.

A high acquisition value does not guarantee that a candidate will outperform the current best observation.

## 12. Risks

The main risk is wasting a limited query on a model artefact.

Examples include:

* selecting a highly uncertain but very poor global candidate;
* trusting a raw-scale GP fitted to a heavily skewed output;
* treating an SVM probability as well calibrated;
* overreacting to an unstable ARD length scale;
* treating agreement between several models trained on the same small dataset as independent confirmation.

## 13. Mitigations

The project uses the following safeguards:

* output transformations;
* alternative kernels;
* comparison with Extra Trees, neural networks and XGBoost;
* hybrid global and local candidate pools;
* candidate deduplication;
* function-specific exploration parameters;
* manual review of top-ranked candidates;
* comparison of predicted means, uncertainties and incumbent values.

## 14. Ethical considerations

The model does not process personal or demographic information and is not used to make decisions about individuals.

The main ethical requirement is transparent reporting of uncertainty and limitations. Model recommendations should not be presented as guaranteed optima.

## 15. Out-of-scope uses

This model should not be used for:

* safety-critical optimisation;
* medical, financial or legal decisions;
* causal inference;
* optimisation where failed evaluations may cause harm;
* production deployment without further validation;
* claiming discovery of the true global optimum.

## 16. Reproducibility

A separate random seed is used for each function:

```python
random_state = 42 + function_number
```

When comparing kernels or acquisition settings for the same function, the same random state should be used to keep the candidate pool consistent.

Core parameters to record include:

* kernel type;
* noise level;
* length-scale bounds;
* `kappa`;
* number of candidates;
* global/local candidate split;
* local perturbation scale;
* SVM weight;
* ARD penalty strength;
* output transformation;
* random seed.

## 17. Future improvements

Possible future improvements include:

* local or non-stationary Gaussian Processes;
* better uncertainty calibration;
* expected improvement as an alternative acquisition function;
* repeated cross-validation for auxiliary models;
* performance-based ensemble weights;
* explicit modelling of model disagreement;
* adaptive trust-region sizes;
* automated transformation selection.

## 18. Ownership

* Developer: `Swasti Gupta`
* Course or programme: `Imperial AI & Machine Learning`
* Repository: `[[REPOSITORY LINK]](https://github.com/swas-18/ML_BayesianOptimisation-/new/main)`
* Version: `week 7`
* Last updated: `July 2026`

