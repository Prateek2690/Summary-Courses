# Missing data imputation

1. **CCA (Complete case analysis)**: Remove na/nan entries completely. 

## Which variables can I impute with CCA?
CCA can be applied to both categorical and numerical variables.

## Assumptions
CCA works well when the data are missing completely at random (MCAR). In fact, we should use CCA if we have reasons to believe that data is missing at random, and not otherwise. When data is MCAR, excluding observations with missing information is in essence the same as randomly excluding some observations from the dataset. Therefore the dataset after CCA is a fair representation of the original dataset.

## Advantages
Easy to implement
No data manipulation required
Preserves variable distribution (if data is MCAR, then the distribution of the variables of the reduced dataset should match the distribution in the original dataset)

## Disadvantages
It can exclude a large fraction of the original dataset (if missing data is abundant)
Excluded observations could be informative for the analysis (if data is not missing at random)
CCA will create a biased dataset if the complete cases differ from the original data (e.g., when missing information is in fact MAR or NMAR and not missing at random).
When using our models in production, the model will not know how to handle missing data

## When to use CCA
Data is missing completely at random
No more than 5% of the total dataset contains missing data

## Relevant Code
vars_cca = [var for var in data.columns if data[var].isnull().mean() < 0.05]
 - check the distribution of the variables once the nan entries are removed, to ensure no significant difference occurs.
