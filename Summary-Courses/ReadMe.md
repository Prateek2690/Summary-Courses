# Summary - Feature Engineering and Feature Selection
* **Data Analysis**: 
    - Understand the variables (whether it is numerical, categorical, date/time, mixed, anything else) and Understand what we are trying to predict
        -  Also, check whether we have problem of missing data; if we have categorical (then we want to encode them into numbers); if the variables follow linear model assumptions, if the distribution is normal, skewed (if skewed, we want to perform transformations); if the variables contain outliers; if the features have different magnitudes (then we want to scale them)
    
*  **Feature Engineering**: 
    - We can create new features from dates, mixed vars
        -   we may impute missing values
            - we may perfrom numerical variable transform  
                -  we can discretisation
                    - we can handle outliers
                        -also feature scaling
    - Build a Feature engineering pipeline, to analyze new data, to produce reproducible research, to deploy code.


 ## Missing data: 
    There could be 3 different categories of missing data
    - MCAR - No relationship with other variables or given information
    - MAR - some relationship with other variable
    - MNAR - systematic relationship with other variables
    - relevant formulaes: df.isnull().sum() or df.isnull().mean()

## Cardinality: 
    High cardinality tend to:
    - overfit the linear models (some of the labels may only be present in the test set)
    - create bias in tree based models 
    - so feature engineer the high cardinality variables
    - club the labels with low frequency into RARE labels
    - relevant formulaes: pd.Series(data[col].value_counts() / length_of_data)

**plotting ex**:

    temp_df = pd.Series(data[col].value_counts() / total_houses)

    # make plot with the above percentages
    fig = temp_df.sort_values(ascending=False).plot.bar()
    fig.set_xlabel(col)

    # add a line at 5 % to flag the threshold for rare categories
    fig.axhline(y=0.05, color='red')
    fig.set_ylabel('Percentage of houses')
    plt.show()

**How is the target, "target var", related to these categories?**

    def calculate_mean_target_per_category(df, var, target):

            # total number of houses
            total_houses = len(df)

            # percentage of houses per category
            temp_df = pd.Series(df[var].value_counts() / total_houses).reset_index()
            temp_df.columns = [var, 'perc_houses']

            # add the mean target
            temp_df = temp_df.merge(df.groupby([var])[target].mean().reset_index(),
                                    on=var,
                                    how='left')

            return temp_df

        def plot_categories(df, var):
            
            fig, ax = plt.subplots(figsize=(8, 4))
            plt.xticks(df.index, df[var], rotation=90)

            ax2 = ax.twinx()
            ax.bar(df.index, df["perc_houses"], color='lightgrey')
            ax2.plot(df.index, df["SalePrice"], color='green', label='Seconds')
            ax.axhline(y=0.05, color='red')
            ax.set_ylabel('percentage of houses per category')
            ax.set_xlabel(var)
            ax2.set_ylabel('Average Sale Price per category')
            plt.show()

### Replace Rare labels

def group_rare_labels(df, var):

    total_houses = len(df)

    # first I calculate the % of houses for each category
    temp_df = pd.Series(df[var].value_counts() / total_houses)

    # now I create a dictionary to replace the rare labels with the
    # string 'rare' if they are present in less than 5% of houses

    grouping_dict = {
        k: ('rare' if k not in temp_df[temp_df >= 0.05].index else k)
        for k in temp_df.index
    }

    # now I replace the rare categories
    tmp = df[var].map(grouping_dict)

    return tmp


## Outlier detection and removal: 
    Outliers can lead to bad gen. or bad model, as some model can put more weight on outliers. These ways can be adopted to detect and remove outliers.

**Extreme value analysis**: 
### If the the variable is Normally distributed (Gaussian), then the values that lie outside the mean plus or minus 3 times the standard deviation of the variable are considered outliers.

    __outliers__ = mean +/- 3* std
    If the variable is skewed distributed, a general approach is to calculate the quantiles, and then the inter-quantile range (IQR), as follows:

    IQR = 75th quantile - 25th quantile
    An outlier will sit outside the following upper and lower boundaries:

    Upper boundary = 75th quantile + (IQR * 1.5)

    Lower boundary = 25th quantile - (IQR * 1.5)

    or for extreme cases:

    Upper boundary = 75th quantile + (IQR * 3)

    Lower boundary = 25th quantile - (IQR * 3)

    relevant code:
        def find_normal_boundaries(df, variable):

        # calculate the boundaries outside which sit the outliers
        # for a Gaussian distribution

        upper_boundary = df[variable].mean() + 3 * df[variable].std()
        lower_boundary = df[variable].mean() - 3 * df[variable].std()

        return upper_boundary, lower_boundary

### if the variable is skewed,
def find_skewed_boundaries(df, variable, distance):

    # Let's calculate the boundaries outside which sit the outliers
    # for skewed distributions

    # distance passed as an argument, gives us the option to
    # estimate 1.5 times or 3 times the IQR to calculate
    # the boundaries.

    IQR = df[variable].quantile(0.75) - df[variable].quantile(0.25)

    lower_boundary = df[variable].quantile(0.25) - (IQR * distance)
    upper_boundary = df[variable].quantile(0.75) + (IQR * distance)

    return upper_boundary, lower_boundary

# Variable Magnitude
- feature scaling affect linear-like models or distance based models (linear, logistic, SVMs, KNN)

