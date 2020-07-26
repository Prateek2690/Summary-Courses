# Missing data imputation

## CCA (Complete case analysis): Remove na/nan entries completely. 

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
    check the distribution of the variables once the nan entries are removed, to ensure no significant difference occurs.

##  Mean-median Imputation:
Imputation should be done over the training set, and then propagated to the test set. This means that the mean / median to be used to fill missing values both in train and test set, should be extracted from the train set only. And this is to avoid overfitting.

    ##Mean / median imputation with Scikit-learn ==> SimpleImputer
    The SimpleImputer class provides basic strategies for imputing missing values, including:
    Mean and median imputation for numerical variables
    Most frequent category imputation for categorical variables
    Arbitrary value imputation for both categorical and numerical variables
    
    ##Advantages
    Simple to use if applied to the entire dataframe
    Maintained by the Scikit-learn developers: good quality code
    Fast computation (it uses numpy for calculations)
    Allows for grid search over the various imputation techniques
    Allows for different missing values encodings (you can indicate if the missing values are np.nan, or zeroes, etc)
    
    ##Limitations
    Returns a numpy array instead of a pandas dataframe, inconvenient for data analysis
    Needs to use additional classes to select which features to impute ==>
    requires more lines of code
    additional classes still in beta (may change without warning)
    not so straightforward to use anymore.

    ##Relevant code
    #these are the objects we need to impute missing data with sklearn
    from sklearn.impute import SimpleImputer
    from sklearn.compose import ColumnTransformer
    from sklearn.pipeline import Pipeline

    ##we indicate that we want to impute with the median
    imputer = SimpleImputer(strategy='median')

    ##we fit the imputer to the train set
    imputer.fit(X_train[cols_to_use])

    ##we can look at the learnt medians like this:
    imputer.statistics_

    ##Why use ColumnTransformer? - to perform different imputation on diff. col
            numeric_features_mean = ['LotFrontage']
            numeric_features_median = ['MasVnrArea', 'GarageYrBlt']

            # then we instantiate the imputers, within a pipeline
            # we create one mean imputer and one median imputer
            # by changing the parameter in the strategy

            numeric_mean_imputer = Pipeline(steps=[
                ('imputer', SimpleImputer(strategy='mean')),
            ])

            numeric_median_imputer = Pipeline(steps=[
                ('imputer', SimpleImputer(strategy='median')),
            ])

            # then we put the features list and the transformers together
            # using the column transformer

            # we need to add remainder = True to indicate what we want
            # ALL the columns returned at the end of the transformation
            # and not just the engineered ones, which is the default
            # behaviour of ColumnTransformer. 

            preprocessor = ColumnTransformer(transformers=[
                ('mean_imputer', numeric_mean_imputer, numeric_features_mean),
                ('median_imputer', numeric_median_imputer, numeric_features_median)
            ], remainder='passthrough')

            # now we fit the preprocessor
            preprocessor.fit(X_train)

            # we can explore the transformers like this:
            preprocessor.transformers

            # and now we can impute the test data
            X_test = preprocessor.transform(X_test)

## Arbitary Value Imputation 
Imputation is the act of replacing missing data with statistical estimates of the missing values. The goal of any imputation technique is to produce a complete dataset that can be used to train machine learning models.

    ##Arbitrary value imputation consists of replacing all occurrences of missing values (NA) within a variable by an arbitrary value. Typically used arbitrary values are 0, 999, -999 (or other combinations of 9s) or -1 (if the distribution is positive).

    ##Which variables can I impute with an arbitrary value?
    Both categorical and numerical variables can be imputed by arbitrary values.

    ##For categorical variables, this is the equivalent of replacing all instances of NA by an additional label, which is very common practice, so I will treat it separately in a future notebook.

    ##Assumptions
    Data is not missing at random.
    If this is the case, we want to flag the missing values with a different (arbitrary) value, instead of replacing those occurrences with the mean or the median, which represent the most common value.

    ##Advantages
    Easy to implement
    Fast way of obtaining complete datasets
    Can be integrated in production (during model deployment)
    Captures the importance of "missingness" if there is one
    
    ##Limitations
    Distortion of the original variable distribution
    Distortion of the original variance
    Distortion of the covariance with the remaining variables of the dataset
    If the arbitrary value is at the end of the distribution it may mask or create outliers
    Need to be careful not to chose an arbitrary value too similar to the mean or median (or any other common value of the variable distribution)
    
    ##When to use arbitrary value imputation
    Replacing the NA by arbitrary values should be used when there are reasons to believe that the NA are not missing at random. In situations like this, we would not like to replace with the median or the mean, and therefore make the NA look like the majority of our observations. Instead, we want to flag the fact that the observation is missing.

    This approach is used widely both in organisations and in Kaggle competitions. It consists of replacing the NA by an arbitrary value. Any of your creation, but ideally different from the median/mean/mode, and not within the normal values of the variable.

    The problem consists in deciding which arbitrary value to choose.

    ##Final note
    When variables are captured by third parties, like credit agencies, they place arbitrary numbers already to signal the fact that the value for that observation is missing. So arbitrary value imputation is also common practice in real life data collections.

    ##using Sklearn
    imputer = SimpleImputer(strategy='constant', 
                       fill_value = 999)