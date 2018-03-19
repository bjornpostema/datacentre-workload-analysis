# Phase 1: Rough Distribution Estimation (Algorithm 1)
This section discusses the essential steps of the algorithm in more detail. The three steps of the first phase of the algorithm are elaborated. 

## Step I: Find Local Maxima and Minima.
The data is transformed into a histogram for fitting purposes (as can be seen in Figure 1). This histogram is used to find the local maxima and minima (as shown in Figure 2). The *x*-axis shows the equally sized bins (1 ms each) of the service time and the *y*-axis shows the number of occurrences of each of those bins. Service times with large values are omitted in these figures, because of the few occurrences of these service times. Each red dot indicates a local minimum, and each black dot a local maximum. 

Local maxima and mimina in functions are (often) obtained by finding the locations of the zero crossing of the first derivative. Since the data often has significant noise, the first derivate amplifies this noise. Therefore, the data requires some degree of smoothing; making the peaks more clear and data less erratic. 

For smoothing the histogram data, the *Savitzky-Golay filter* is used. This filter generates a polynomial of an adjustable degree (*polyorder*) over chunks of input data over an adjustable window length (*window_length*). For this purpose we imported a *Python* library from the *SciPy* package with a function called *signal.savgol_filter* that does exactly this.

In the case of histogram data, the extrema are found by comparison of a number (*x*) of values left and right to the value of the potential extrema. The difficulty of this approach is picking the correct value for *x*. Wrongly chosen values of *x* lead to an incorrect number of extrema. Although this simple approach already provides an accurate result, an improvement of this approach could be by making use of the easy to obtain first derivative to find zero crossings after smoothing the data with the Savitzky-Golay filter.

The relative local minima and maxima are determined by comparison of a number of data points. For the purpose of finding these local extrema in histogram data, we used the *Python* library from the *SciPy* package with a function called *signal.argrelextrema* with the value of *x* as the *order* argument.

## Step II: Compute Distribution Boundaries.
From the maxima and minima obtained in the first step, we extract the boundaries of the normal distributions (as can been seen in Figure 3. The vertical lines represent these boundaries.

Each pair of boundaries is characterized with a maximum surrounded by two minima. Ideally, the first step has exactly the right number of maxima and minima to determine each boundary pair, such that: |*Maxima*|=|*Minima*|+1.

However, in the first step not every maxima and minima is found. We correct this with a small procedure. First, we check if the lowest and highest values are indeed minima. Then pick for every set of successive minima, a maximum. A maximum is chosen from a set of maxima between the two minima. With an empty set, the value with the most occurrences in between the minima is chosen.

The toolkit *ProFiDo* is used to compute the normal distributions based on the data between these boundaries. Providing boundaries that result in normal distributions with the best fit is essential. Therefore, we added additional preprocessing before computing the normal distribution based on experimenting with \textsc{ProFiDo}. We found that boundaries that have an equal distance to the corresponding maximum, result in a better normal distribution fit. The reason for obtaining a better fit is that data is cut off at the boundaries. In order to attain an equal distance to the maximum, the distance of the farthest boundary is shortened to a distance equal to that of the boundary closest to the maximum.

## Step III: Compute Normal Distributions.
The toolkit *ProFiDo* allows us to compute one normal distribution at a time by using an input file with all the data entries between the boundaries (as computed in the previous step).

In order to speed up computation of the normal distribution in *ProFiDo*, samples are extracted from the input data with 1 sample after every 1000 data entries, by default. This slightly alters the distribution resulting in a different fit, however, it significantly improves the computation time spent by *ProFiDo*. Our implemented code allows to adjust this so-called *speedUp* parameter, which is the number of data entries of which one sample should be extracted from.

# Phase 2: Distribution Refinement (Algorithm 2)
## Goodness of Fit Metric.
In the second phase of the algorithm, the mixture of normal distributions is potentially improved. For distribution refinement, a goodness-of-fit metric is required to decide the best in a set of alternative fits of the distribution. Such a metric allows us to study the difference between observed and expected frequencies, hence, decide how well the mixture of normal distributions fits the actual data set. 

A well-known test to study the goodness of fit is the well-known \textit{Pearson chi-squared test}. The chi-square value  {\chi}^2=\sum_{i=1}^{N} \frac{(O_i -E_i)^2}{E_i} is used to calculate the goodness of fit, where O_i is the observed frequency, E_i the expected frequency and N is the number of service times bins.

The chi-squared test suffices in many situations. However, the number of samples in our data sets is too large to obtain a statistically significant fit, i.e., when the distribution is off by a relatively small margin, the goodness of fit will change significantly. This often leads to a goodness of fit of zero, unless the fit would be almost perfect. In order to still give an useful indication with our large number of samples over the entire range of service times, the alternative goodness of fit value sum_{i=1}^{N} (O_i -E_i)^2 is computed using the same parameters for frequencies and bins (O_i, E_i and N) as with the chi-squared test.

With some experiments, we found that our goodness of fit value result in a mean and variance slightly closer to the observed values and in significantly better estimates of the peaks. Our goodness of fit value is different in that it considers an accurate estimation of the mean and variance to be more important than accurately fitting the lower frequencies to the actual data.

## Distribution Improvement
The second phase of the algorithm alters the distribution in two very similar ways to improve the goodness of fit. First, the bounds of the data are shifted. Second, the mean and standard deviation values are altered. These two ways are both elaborated below. 

The *first way* to improve the goodness of fit slightly shifts the lower and upper bounds. The bounds are shifted in an amount equal to the employed bin size. Since the service times are rounded to the nearest millisecond, the smallest possible shift is one millisecond. 

The bounds are altered in four steps. First the left bound is shifted to the right, after which the distribution is recomputed using *ProFiDo*. Next, the recomputed distribution is checked whether the fit has improved using the goodness of fit. If the recomputed distribution has improved, it is set as the current optimal value and the bounds are again shifted using the same procedure. If the goodness of fit worsens, the procedure continues with the second step. 

The second step is equal to the first step with the exception that the lower bound is shifted to the left until the recomputed distribution has a worsened goodness of fit. If in the first step the lower bound is shifted more than once, the second step is, of course, skipped. The last two steps are identical to the first two steps, however, tailored to the upper bound instead of the lower bound.

The *second way* to improve the goodness of fit consists of changing the mean and standard deviation values in three sequential steps. First, the mean value is shifted towards the corresponding maximum (as found for the histogram). The second and third step, respectively, increase and decrease the standard deviation value. The computation procedure with *ProFiDo* is the same as with refining the boundaries; the computation procedure is only much faster and has more accurate results, because the shifting steps are *not* restricted to the bin size of the histogram. However, adjustment of one normal distribution affects the other normal distributions in such a way that it requires to again compute the optimal values. Ideally, this procedure should stop until the goodness of fit value does *not* improve. However, the execution time has been limited to the point improvements are negligible.
