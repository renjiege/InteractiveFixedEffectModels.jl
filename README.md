
This package estimates factor models on datasets where each row represents an observation,  as opposed to a matrix N x T.

I'll use the term "panels" to refer to these long datasets , and id x time to refer to the two dimensions of the factor structure - they correspond to (variable x observation) in PCA and (user x movie) in recommandation problems.

### PanelFactorModel
Starting from a a dataframe,  an object of type `PanelFactorModel` is constructed by specifying the id variable, the time variable, and the factor dimension. Both the id and time variable must be of type `PooledDataVector`.

```julia
using RDatasets, DataFrames, PanelFactorModels
df = dataset("plm", "Cigar")
# create PooledDataVector
df[:pState] =  pool(df[:State])
df[:pYear] =  pool(df[:Year])
# create PanelFactorModel in state, year, and rank 2
pfm = PanelFactorModel(:pState, :pYear, 2)
```

#### Estimate factor models by incremental SVD
Estimate a factor model for the variable `Sales`

```julia
fit(pfm, :Sales, df)
fit(pfm, :Sales, df, weight = :Pop)
```

The factor model is estimated by incremental SVD, i.e. by minimizing the sum of the squared residuals incrementally for each dimension. By default, the minimization uses a gradient descent. This yields three importants benefits compared to an eigenvalue decomposition:

1. estimate unbalanced panels, i.e. with missing (id x time) observations. 

2. estimate weighted factor models, where weights are not constant within id or time

3. avoid the creation of a matrix N x T

Another way to solve 1. would be to use a version of the EM algorithm, replacing missing values by the predicted values from the factor model until convergence. However, the EM algorithm is generally slower to converge.

#### Interactive Fixed Effect Models
Estimate models with interactive fixed effects (Bai 2009) 

```julia
fit(pfm, Sales ~ Price, df)
fit(pfm, Sales ~ Price |> pState + pYear, df)
```


## Install

```julia
Pkg.clone("https://github.com/matthieugomez/PanelFactorModels.jl")
```