
<!-- README.md is generated from README.Rmd. Please edit that file -->

[![Actions
Status](https://github.com/difuture-lmu/ds.roc.glm/workflows/R-CMD-check/badge.svg)](https://github.com/difuture-lmu/ds.roc.glm/actions)
[![License: LGPL
v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
[![codecov](https://codecov.io/gh/difuture-lmu/ds.roc.glm/branch/master/graph/badge.svg?token=E8AZRM6XJX)](https://codecov.io/gh/difuture-lmu/ds.roc.glm)

# ROC-GLM for DataSHIELD

## Overview

## Installation

At the moment, there is no CRAN version available. Install the
development version from GitHub:

``` r
remotes::install_github("difuture-lmu/ds.roc.glm")
```

#### Register methods

It is necessary to register the aggregate and assign methods in the OPAL
administration. The assign methods are:

**Assign methods:**

  - `rocGLMFrame`

**Aggregate methods:**

  - `getPositiveScores`
  - `getNegativeScores`
  - `calculateDistrGLMParts`

## Usage

``` r
library(DSI)
library(DSOpal)
library(DSLite)
library(dsBaseClient)


## DataSHIELD login:
## ========================================

builder = DSI::newDSLoginBuilder()

builder$append(
  server   = "ibe",
  url      = "*****''",
  user     = "***",
  password = "******",
  table    = "ProVal.KUM"
)

logindata = builder$build()
connections = DSI::datashield.login(logins = logindata, assign = TRUE, symbol = "D",
  opts = list(ssl_verifyhost = 0, ssl_verifypeer=0))

### Get available tables:
DSI::datashield.symbols(connections)

## Read test data (same as on server)
## ========================================

# We use this data to calculate a model which we want to evaluate. Here a simple logistic regression:
dat = read.csv("data/test-kum.csv")
mod = glm(gender ~ age + height, family = "binomial", data = dat)



## Preperation for ROC-GLM
## ========================================

### The ROC-GLM requires scores and true values. The function `predictModel` calculates the
### scores based on a model. In our case the logistic regression form above.

### Upload model to DataSHIELD server:
pushModel(connections, mod)

### Predict uploaded model on server data. Scores are stored in an object called `pred`:
predictModel(connections, mod, "pred", "D", predict_fun = "predict(mod, newdata = D, type = 'response')")

### The `pred` object is later used for the ROC-GLM.

### Get object on server:
DSI::datashield.symbols(connections)



## Calculate and visualize ROC-GLM
## ========================================

### Now, calculate ROC-GLM:
roc_glm = dsROCGLM(connections, "D$gender", "pred")
roc_glm

### And plot it:
plot(roc_glm)



## Logout from DataSHIELD server
## ========================================

DSI::datashield.logout(conns = connections, save = FALSE)
```
