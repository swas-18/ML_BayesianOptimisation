# Bayesian Black-Box Optimisation

This project uses a Gaussian Process based Bayesian optimisation pipeline to identify high-performing input combinations for eight unknown black-box functions and select the next point to evaluate under a limited query budget.

The main surrogate model is a Gaussian Process with:

- a Matérn kernel;
- a separate length scale for each input dimension;
- output transformations where required.


The Gaussian Process produces:

- a predicted mean for each candidate;
- a predictive uncertainty estimate;
- an acquisition score used to rank candidates.



The optimiser combines candidates from several sources (Sobol, local, neural-network), and then primarily ranked using the UCB acquisition functions:

### Upper Confidence Bound

The Upper Confidence Bound (UCB) acquisition function is defined as:

$$
\mathrm{UCB}(x) = \mu(x) + \kappa \sigma(x)
$$

where:

- $\mu(x)$ is the predicted mean;
- $\sigma(x)$ is the predictive uncertainty;
- $\kappa$ controls the balance between exploration and exploitation.

A larger value of $\kappa$ places more weight on uncertain candidates, encouraging exploration. A smaller value places more weight on the predicted mean, encouraging exploitation.
An alternative acquisition function using Expected Improvement can also be specified (where a larger value of $\xi$ encourages more exploration/ smaller value places more emphasis on exploiting regions that already have a high predicted mean). 

The final candidate is selected using the Gaussian Process acquisition score. Several additional models are fitted as diagnostic check, and predictions for candidates are compared with the Gaussian Process recommendation. These models do not directly determine the final ranking. Instead, they help identify cases where the GP may be unsuitable or overly dependent on uncertainty.

- neural-network ensemble;
- Extra Trees;
- XGBoost.

## Instructions

To run the pipeline, follow the instructions in this notebook: https://github.com/swas-18/ML_BayesianOptimisation/blob/main/Hybrid_Bayesian_Optimisation.ipynb

You can view performance over rounds with some visuals in this notebook: https://github.com/swas-18/ML_BayesianOptimisation/blob/main/Performance_summary.ipynb

More details about the model and functions used can be found here: https://github.com/swas-18/ML_BayesianOptimisation/blob/main/Model_card.md

More details about the underlying data this is built on are here: https://github.com/swas-18/ML_BayesianOptimisation/blob/main/Datasheet.md
