# remef

`remef` is an R package designed for the removal of statistical effects.

The aim of `remef` is to provide tools for extracting partial effects from
data. 

Mixed-effects models include both fixed and random effects. The dependent
variable is expressed as a combination of fixed and random effects and an
error term. `remef` is a practical tool for removing any combination
of fixed and random effects from the data. The removal of effects is very
helpful for understanding and visualizing certain effects in highly 
complex statistical models. Currently, the package works with objects 
generated by the package `lme4`.

Key features:

- removes fixed effects and random variance from a dependent variable.

- optionally, `remef` takes into account associated lower-order or 
  higher-order effects when removing partial effects (this is important
  for statistical models including interactions).
  
- finds model coefficients for a given term (e.g., for factors and
  polynomial contrasts) and vice versa.


## Installation

To get the current development version from github, you can use the package
`devtools`:

```R
# install.packages("devtools")
devtools::install_github("hohenstein/remef")
```

## Examples

For the examples, we create a simple linear mixed effects model. It is one of the examples of the `lmer` function. The model consists of an intercept (`1`), the fixed effect `Days` and the corresponding variance components (random  intercept and slope) for the random factor `Subject`.

```R
library(lme4)
fm1 <- lmer(Reaction ~ 1 + Days + (1 + Days | Subject), sleepstudy)
```

Of course, we have to load the `remef` package:

```R
library(remef)
```

### Remove a fixed effect

The following commands are a demonstration of how the `remef` package can be used to remove the influence of certain effects from a dependent variable (`Reaction`).

Remove the fixed effect of `Days`:

```R
p_fm1_1 <- remef(fm1, fix = "Days", keep.intercept = TRUE)
```

Since `keep.intercept = TRUE` is the default, we can shorten this command to

```R
p_fm1_2 <- remef(fm1, fix = "Days")
```

We can also used the index of the coefficient name `"Days"` instead of the name. The intercept corresponds to index 1, `Days` corresponds to index 2.

```R
p_fm1_3 <- remef(fm1, fix = 2)
```


### Remove a variance component (random effect)

In this example, the random slope of `Days` should be removed:

```R
p_fm1_4 <- remef(fm1, ran = list(Subject = "Days"))
```

Of course, we can replace the character string `"Days"` with its index. Note that the index corresponds to the order of variance components for the random factor `Subject` and is *not* related to the order of fixed-effect coefficients.

```R
p_fm1_5 <- remef(fm1, ran = list(Subject = 2))
```


### Remove the intercept

To remove the intercept, we can use `keep.intercept = FALSE`.

```R
p_fm1_6 <- remef(fm1, keep.intercept = FALSE)
```

The above command will remove the intercept and therefore center `Reaction` on the intercept.


### Remove multiple fixed an random effects

If we want to remove both fixed effects and both random effects, we can use

```R
p_fm1_7 <- remef(fm1, fix = "Days", ran = list(Subject = c(1, 2)), keep.intercept = FALSE)
```

Since this command will remove all fixed and random effects, the result is equal to the residuals of the model.

If we want to remove *all* random effects from a model, we can pass the character string `"all"` to the parameter `ran`. This is a simple alternative to a list including all random effects, particularly for complex models with multiple random factors.

```R
p_fm1_8 <- remef(fm1, ran = "all")
```


### Keeping effects: the function `keepef`

If we want to remove a large proportion of fixed or random effects, it might be easier to specify the effects that should be kept instead. Here, the function `keepef` can be used. In contrast to `remef`, `keepef` keeps the specified effects but removes all the remaining ones.

```R
# the following commands are euqivalent
p_fm1_9 <- remef(fm1, fix = "Days", ran = "all")
p_fm1_10 <- keepef(fm1)
```

The commands above remove all effects except the intercept. Note that `keep.intercept` defaults to `TRUE` for `keepef` too. If we want to remove all effects including the intercept with `keepef`, we have to use

```R
p_fm1_11 <- keepef(fm1, keep.intercept = FALSE)
```

The values obtained with the command above are equal to the model residuals.


### More examples

For more examples, have a look at the documentations of `remef` and `keepef`.
```R
?remef
?keepef
```

---

### Important information for users of the old standalone `remef` function

Before the launch of the `remef` package, the function `remef` was available from the [Potsdam Mind Research Repository](http://read.psych.uni-potsdam.de/index.php?option=com_content&view=article&id=134:hohenstein-2013-the-remef-function-for-r&catid=13:r-playground&Itemid=15) only. The latest version of the standalone function is 0.6.10. This function will not be developed further, but remain available. Please use its successor, the `remef` *package*.

If you have already been working with the old standalone function, note that there are some syntactic differences to the new version in the package:

- A mixture of coefficient names (character strings) and numeric indices (e.g., `c(1, "factorA")`)  for `fix` or `ran` is no longer valid. In the current version, you can use vectors of *either* coefficient names (character strings) *or* numeric indices corresponding to the model coefficients.

- The parameter `keep`, that was used to specify whether effect should be removed or kept, is no longer part of the `remef` function. In the package, there is a function `remef` for removing effects and a function `keepef` for keeping effects.

- The list `ran` must be a named list now. In the old `remef` function, the order of its list elements was the same as the order of random effects in the model. In the new `remef` package, the names of `ran` correspond to the names of the random factors. An example list `ran` is `list(Subject = c(1, 3), Item = c(2, 3, 4))`.

- In the old `remef` function, the elements of the list `ran` were numeric index vectors corresponding to variance components (random intercept/slopes). Now, you can use character vectors including names of the variance components too. You can use both numeric index vectors and character vectors in `ran`, e.g., `list(Subject = c("(Intercept)", "Factor2"), Item = c(2, 3, 4))`). For a model fit object `model`, the command `ranef_labels(model)` returns a list containing the names of all variance components of all random factors.

- There is a new (logical) parameter `inverse` that indicates whether the inverse of the link function is applied. It replaces the old `family` parameter that accepted two values, `gaussian` and `binomial`. In the new `remef` package, the family of the model is detected automatically. The parameter `inverse` is important for *generalised* linear mixed-effects models, created with `lme4::glmer`, only.
