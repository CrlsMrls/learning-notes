# Types of data

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

### Sample sizes of data

- **N=1**: Only one patient. Likely to be noise or non-represntative data. Statical inference is difficult or impossible.
- **N>1**: Small or large scale study, actual research with potentially important and generalizable findings. Can suffer from sampling variability, noise and outliers.

## Types of data Visualization

### Lines

```python
import matplotlib.pyplot as plt

x = [1,2,3,4,5,6]
y = [2,4,8,16,32,64]

plt.plot(x,y)
plt.show()

plt.plot(x,y)
plt.yscale('log')
plt.show()

```

![Line](./imgs/data-line1.png 'Line')
![Line](./imgs/data-line2.png 'Line')

### Box and Whisker plot

A box and whisker plot (also called box plot) illustrates the overall distribution of the data and possible outliers or unnusually large/small data points.

It shows the five number summary of a set of data:

![Box plot](./imgs/data-box-plot.png 'Box plot')

- Minimum (Q0 or 0th percentile): the lowest data point in the data set excluding any outliers
- Maximum (Q4 or 100th percentile): the highest data point in the data set excluding any outliers
- Median (Q2 or 50th percentile): the middle value in the data set (50% of data)
- First quartile (Q1 or 25th percentile): the lower quartile qn(0.25) (25% of data).
- Third quartile (Q3 or 75th percentile): the upper quartile qn(0.75) (75% of data).

#### Python code example

```python
import matplotlib.pyplot as plt

data = [1,2,3,4,6,6,6,6,6,7,8,9,10,17]

plt.boxplot(data)
plt.show()
```

![Box plot](./imgs/data-box-plot-python.png 'Box plot')

### Histograms

To construct a histogram, the first step to divide the entire range of values into a series of intervals (this is to "bin" (or "bucket") the range of values), and then count how many values fall into each interval. The bins are usually consecutive and non-overlapping intervals.

Comparing them with Bar plots, these have categories on the x-axis,
on the other hand, histograms have binned c ontinuous data on the x-axis.

All of the bars must sum 100%.

Histograms are better for quantitive analysis.

#### Python code example

```python
import matplotlib.pyplot as plt
import numpy as np

# number of data points
n = 100

# generate data - log-normal distribution
data = np.exp( np.random.randn(n)/2 )

plt.plot(data, 's')
plt.show()
```

![Data without histogram](./imgs/data-histograms-plot.png 'Data without histogram')

```python
# show as a histogram

# number of histogram bins
k = 40

plt.hist(data,bins=k)
plt.show()
```

![Data in histogram](./imgs/data-histograms-hist.png 'Data in histogram')

### Pie charts

A pie chart is a circular statistical graphic divided into slices to illustrate numerical proportion.

Pie charts can show nominal, ordinal and discretes type of data.

The sum of all pieces must sum to 1 or 100%.

#### Python code example

```python
import matplotlib.pyplot as plt

data = [5, 15, 35, 45]

plt.pie(data, labels=data)
plt.show()
```

![Pie chart](./imgs/data-pie-chart.png 'Pie chart')

## Links

- [Statistical data type](https://en.wikipedia.org/wiki/Statistical_data_type)
- [Box plot - Wikipedia](https://en.wikipedia.org/wiki/Box_plot)
