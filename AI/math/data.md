# Data

## Types of statistical data

The data type describes the semantic content of the variable, and controls which sorts of probability distributions can logically be used.

| Category             | Type     | Description                             | Example                      |
| -------------------- | -------- | --------------------------------------- | ---------------------------- |
| Numerical (numbers)  | Interval | Numeric scale with meaningful inetrvals | Temp. in C                   |
|                      | Ratio    | Interval, but also with meaninful zero  | Height in cm                 |
|                      | Discrete | No arbitrary precision (integers)       | Population                   |
| Categorical (labels) | Ordinal  | Sortable, discrete                      | Education (school, uni, PhD) |
|                      | Nominal  | Non-sortable, discrete                  | Movie genre (Sci-fi, comedy) |

## Sample vs. population data

Most statistical procedures are designed either for sample or for population. Applying a procedure to the wrong data type can lead to incorrect results and incorrect interpretations.

| Category        | Description                                                     | Examples                                                 | Notation                               |
| --------------- | --------------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------- |
| Population data | Data of ALL members of a group                                  | Salaries in a department, population census              | $\mu$, $\beta$, $\sigma^2$             |
| Sample data     | Data from SOME members of a group (hopefully randomly selected) | average height of Spaniards (take just a sample of data) | $\hat\mu$, $\hat\beta$, $\hat\sigma^2$ |

## Sample sizes of data

- **N=1**: Only one patient. Likely to be noise or non-represntative data. Statical inference is difficult or impossible.
- **N>1**: Small or large scale study, actual research with potentially important and generalizable findings. Can suffer from sampling variability, noise and outliers.

## Links

- [Statistical data type](https://en.wikipedia.org/wiki/Statistical_data_type)
