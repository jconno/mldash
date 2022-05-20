
# mldash

<!-- badges: start -->
<!-- badges: end -->

The goal of `mldash` is to provide a framework for evaluating the
performance of many predictive models across many datasets. The package
includes common predictive modeling procedures and datasets. Details on
how to contribute additional datasets and models is outlined below. Both
datasets and models are defined in the Debian Control File (dcf) format.
This provides a convenient format for storing both metadata about the
datasets and models but also R code snippets for retrieving data,
training models, and getting predictions. The `run_models` function
handles executing each model for each dataset (appropriate to the
predictive model type, i.e. classification or regression), splitting
data into training and validation sets, and calculating the desired
performance metrics utilizing the
[`yardstick`](https://yardstick.tidymodels.org) package.

**WARNING** This is very much a alpha project as I explore this approach
to evaluating predictive models.

## Installation

You can install the development version of `mldash` using the `remotes`
package like so:

``` r
remotes::install_github('jbryer/mldash')
```

## Running Predictive Models

To begin, we read in the datasets using the `read_ml_datasets()`
function. There are two parameters:

-   `dir` is the directory containing the metadata files. The default is
    to look in the package’s installation directory.
-   `cache_dir` is the directory where datasets can be stored locally.

This lists the datasets currenlty included in the package (more to come
soon).

``` r
ml_datasets <- mldash::read_ml_datasets(dir = 'inst/datasets',
                                        cache_dir = 'data-raw')
ml_datasets
#>                name           type
#> abalone.dcf abalone     regression
#> ames.dcf       ames     regression
#> titanic.dcf titanic classification
#>                                                                                                       description
#> abalone.dcf                                             Predicting the age of abalone from physical measurements.
#> ames.dcf                                                                                       Ames Housing Data.
#> titanic.dcf The original Titanic dataset, describing the survival status of individual passengers on the Titanic.
#>                                                                               source
#> abalone.dcf                          https://archive.ics.uci.edu/ml/datasets/Abalone
#> ames.dcf                                      http://jse.amstat.org/v19n3/decock.pdf
#> titanic.dcf https://www.openml.org/search?type=data&sort=runs&id=40945&status=active
#>                                                                                                                                                                                                                          reference
#> abalone.dcf Nash, Warwick J. & Tasmania. Marine Research Laboratories. (1994). The Population biology of abalone (Haliotis species) in Tasmania. Hobart: Sea Fisheries Division, Dept. of Primary Industry and Fisheries, Tasmania
#> ames.dcf                                                  De Cock, D. (2011). "Ames, Iowa: Alternative to the Boston Housing Data as an End of Semester Regression Project," Journal of Statistics Education, Volume 19, Number 3.
#> titanic.dcf                                                                                                                                                                    Harrell, F.E., & Cason, T. (2017). Titanic Dataset.
#>                                                                                        url
#> abalone.dcf https://archive.ics.uci.edu/ml/machine-learning-databases/abalone/abalone.data
#> ames.dcf                          http://jse.amstat.org/v19n3/decock/DataDocumentation.txt
#> titanic.dcf                        https://www.openml.org/data/download/16826755/phpMYEkMl
#>                                                                                   model
#> abalone.dcf                                                        rings ~ length + sex
#> ames.dcf    Sale_Price_log ~ Longitude + Latitude + Lot_Area + Neighborhood + Year_Sold
#> titanic.dcf            survived ~  pclass + sex + age + sibsp + parch + fare + embarked
#>             note
#> abalone.dcf <NA>
#> ames.dcf    <NA>
#> titanic.dcf <NA>
```

Similarly, the `read_ml_models` will read in the models. The `dir`
parameter defines where to look for model files.

``` r
ml_models <- mldash::read_ml_models(dir = 'inst/models')
ml_models
#>                                                        name           type
#> bag_mars_classification.dcf         bag_mars_classification classification
#> bag_mars_regression.dcf                 bag_mars_regression     regression
#> lm.dcf                                                   lm     regression
#> logistic.dcf                                       logistic classification
#> randomForest_classification.dcf randomForest_classification classification
#> randomForest_regression.dcf         randomForest_regression     regression
#>                                                                                                             description
#> bag_mars_classification.dcf     Ensemble of generalized linear models that use artificial features for some predictors.
#> bag_mars_regression.dcf         Ensemble of generalized linear models that use artificial features for some predictors.
#> lm.dcf                                                                  Linear regression using the stats::lm function.
#> logistic.dcf                                                         Logistic regression using the stats::glm function.
#> randomForest_classification.dcf                        Random forest prediction model usign the randomForest R package.
#> randomForest_regression.dcf                            Random forest prediction model usign the randomForest R package.
#>                                 note          packages
#> bag_mars_classification.dcf     <NA> parsnip, baguette
#> bag_mars_regression.dcf         <NA> parsnip, baguette
#> lm.dcf                          <NA>              <NA>
#> logistic.dcf                    <NA>              <NA>
#> randomForest_classification.dcf           randomForest
#> randomForest_regression.dcf               randomForest
```

Once the datasets and models have been loaded, the `run_models` will
train and evaluate each model for each dataset as appropriate for the
model type.

``` r
ml_results <- mldash::run_models(datasets = ml_datasets, models = ml_models)
#> [1 / 3] Loading abalone data...
#>    [1 / 3] Running bag_mars_regression model...
#>    [2 / 3] Running lm model...
#>    [3 / 3] Running randomForest_regression model...
#> [2 / 3] Loading ames data...
#>    [1 / 3] Running bag_mars_regression model...
#>    [2 / 3] Running lm model...
#>    [3 / 3] Running randomForest_regression model...
#> [3 / 3] Loading titanic data...
#>    [1 / 3] Running bag_mars_classification model...
#> Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred
#>    [2 / 3] Running logistic model...
#>    [3 / 3] Running randomForest_classification model...
ml_results
#>   dataset                           model           type base_accuracy
#> 1 abalone         bag_mars_regression.dcf     regression            NA
#> 2 abalone                          lm.dcf     regression            NA
#> 3 abalone     randomForest_regression.dcf     regression            NA
#> 4    ames         bag_mars_regression.dcf     regression            NA
#> 5    ames                          lm.dcf     regression            NA
#> 6    ames     randomForest_regression.dcf     regression            NA
#> 7 titanic     bag_mars_classification.dcf classification          0.62
#> 8 titanic                    logistic.dcf classification          0.62
#> 9 titanic randomForest_classification.dcf classification          0.62
#>   time_user time_system time_elapsed accuracy kappa sensitivity specificity
#> 1     0.210       0.012        0.226       NA    NA          NA          NA
#> 2     0.001       0.000        0.001       NA    NA          NA          NA
#> 3     1.300       0.054        1.361       NA    NA          NA          NA
#> 4     0.336       0.006        0.342       NA    NA          NA          NA
#> 5     0.003       0.000        0.003       NA    NA          NA          NA
#> 6     2.119       0.030        2.150       NA    NA          NA          NA
#> 7     0.148       0.004        0.151     0.18 -0.53        0.12        0.28
#> 8     0.002       0.000        0.002     0.81  0.58        0.87        0.70
#> 9     0.359       0.007        0.368     0.82  0.60        0.92        0.66
#>   roc_auc r_squared rmse
#> 1      NA      0.33 2.57
#> 2      NA      0.32 2.59
#> 3      NA      0.33 2.58
#> 4      NA      0.43 0.14
#> 5      NA      0.56 0.12
#> 6      NA      0.67 0.11
#> 7    0.86        NA   NA
#> 8    0.14        NA   NA
#> 9    0.12        NA   NA
```

## Available Datasets

-   [abalone](inst/datasets/abalone.dcf) - Predicting the age of abalone
    from physical measurements.
-   [ames](inst/datasets/ames.dcf) - Ames Housing Data.
-   [titanic](inst/datasets/titanic.dcf) - The original Titanic dataset,
    describing the survival status of individual passengers on the
    Titanic.

## Available Models

-   [bag_mars_classification](inst/models/bag_mars_classification.dcf) -
    Ensemble of generalized linear models that use artificial features
    for some predictors.
-   [bag_mars_regression](inst/models/bag_mars_regression.dcf) -
    Ensemble of generalized linear models that use artificial features
    for some predictors.
-   [lm](inst/models/lm.dcf) - Linear regression using the stats::lm
    function.
-   [logistic](inst/models/logistic.dcf) - Logistic regression using the
    stats::glm function.
-   [randomForest_classification](inst/models/randomForest_classification.dcf) -
    Random forest prediction model usign the randomForest R package.
-   [randomForest_regression](inst/models/randomForest_regression.dcf) -
    Random forest prediction model usign the randomForest R package.

## Creating Datasets

``` r
adult_data <- mldash::new_dataset(
    name = 'adult',
    type = 'classification',
    description = 'Prediction task is to determine whether a person makes over 50K a year.',
    source = 'https://archive.ics.uci.edu/ml/datasets/Adult',
    url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.data',
    dir = 'inst/datasets',
    data = function() {
        destfile <- tempfile()
        download.file("https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.data", destfile)
        df <- read.csv(destfile, header = FALSE)
        names(df) <- c('age', 'workclass', 'fnlwgt', 'education', 'education_num', 'marital_status',
                       'occupation', 'relationship', 'race', 'sex', 'capital_gain', 'captial_loss',
                       'hours_per_week', 'native_country', 'greater_than_50k')
        df$greater_than_50k <- df$greater_than_50k == ' >50K'
        return(df)
    },
    model = greater_than_50k ~ .,
    overwrite = TRUE
)
```

Results in creating the following file:

    name: adult
    type: classification
    description: Prediction task is to determine whether a person makes over 50K a year.
    source: https://archive.ics.uci.edu/ml/datasets/Adult
    reference: APA reference for the dataset.
    url: https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.data
    data: function () 
        {
            destfile <- tempfile()
            download.file("https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.data", 
                destfile)
            df <- read.csv(destfile, header = FALSE)
            names(df) <- c("age", "workclass", "fnlwgt", "education", 
                "education-num", "marital-status", "occupation", "relationship", 
                "race", "sex", "capital-gain", "captial-loss", "hours-per-week", 
                "native-country", "greater_than_50k")
            df$greater_than_50k <- df$greater_than_50k == " >50K"
            return(df)
        }
    model: greater_than_50k ~ .
    note:

## Creating Models

``` r
rf_model <- mldash::new_model(
    name = 'randomForest',
    type = 'classification',
    description = 'Random forest prediction model usign the randomForest R package.',
    train_fun = function(formula, data) {
        y_var <- all.vars(formula)[1]
        if(!is.factor(data[,y_var])) {
            data[,y_var] <- as.factor(data[,y_var])
        }
        randomForest::randomForest(formula = formula, data = data, ntree = 1000)
    },
    predict_fun = function(model, newdata) {
        y_var <- all.vars(model$terms)[1]
        if(!is.factor(newdata[,y_var])) {
            newdata[,y_var] <- as.factor(newdata[,y_var])
        }
        randomForest:::predict.randomForest(model, newdata = newdata, type = "prob")[,2,drop=TRUE]
    },
    packages = "randomForest",
    overwrite = TRUE
)
```

Results in the following file:

    name: randomForest
    type: classification
    description: Random forest prediction model usign the randomForest R package.
    train: function (formula, data) 
        {
            y_var <- all.vars(formula)[1]]
            if(!is.factor(data[,y_var)) {
                data[,y_var] <- as.factor(data[,y_var])
            }
            randomForest::randomForest(formula = formula, data = data, 
                ntree = 1000)
        }
    predict: function (model, newdata) 
        {
            y_var <- all.vars(formula)[1]]
            if(!is.factor(newdata[,y_var)) {
                newdata[,y_var] <- as.factor(newdata[,y_var])
            }
            randomForest:::predict.randomForest(model, newdata = newdata, type = "prob")[,2,drop=TRUE]
        }
    note:

## Code of Conduct

Please note that the mldash project is released with a [Contributor Code
of
Conduct](https://contributor-covenant.org/version/2/0/CODE_OF_CONDUCT.html).
By contributing to this project, you agree to abide by its terms.
