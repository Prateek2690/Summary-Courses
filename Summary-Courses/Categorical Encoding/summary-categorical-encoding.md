# Categorical Encoding

## One-hot-encoding (OHE) - Dummy variable technique
            * When one hot encoding categorical variables, we create k - 1 binary variables 

            **Sample Code**
            - OHE also helps handling rare labels in test set
            # we create and train the encoder

            encoder = OneHotEncoder(categories='auto',
                                drop='first', # to return k-1, use drop=false to return k dummies
                                sparse=False,
                                handle_unknown='ignore') # helps deal with rare labels

            encoder.fit(X_train.fillna('Missing'))
            tmp = encoder.transform(X_train.fillna('Missing'))
            pd.DataFrame(tmp).head()