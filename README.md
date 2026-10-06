# econ3916-lab04-anomaly-detection
# Robust Statistics — Automated Anomaly Detection

## Objective

I compared outlier detection methods on California Housing data to see which summary statistics survive contaminated data and whether a one-variable rule and a multivariate model flag the same rows.

## Methodology

- Loaded the California Housing dataset, 20,640 observations, and looked at the distribution of median house value per census block group.
- Computed six summary measures of center and spread: the mean, median, 10% trimmed mean, standard deviation, interquartile range, and median absolute deviation. Three of these resist extreme values and three do not.
- Implemented Tukey Fences by hand rather than calling a library: took the first and third quartiles, took their difference, and set the fences at 1.5 interquartile ranges below and above. Flagged every observation outside them.
- Plotted the fences on a histogram and alongside a boxplot, which draws the same fences automatically.
- Fitted an Isolation Forest on the eight feature columns, excluding the price, so that the two methods were looking at different information.
- Cross-tabulated the two sets of flags to count how many rows each method caught alone and how many both caught.
- Compared the average feature values of the rows only the forest flagged against the dataset as a whole, to work out what the forest was reacting to.
- Ran a contamination experiment: replaced 5% of prices at random with values between $1M and $5M, then recomputed all six measures and compared them against the clean figures.

## Key Findings

**The mean broke and the median did not.** After I corrupted 5% of the prices, the mean moved by 67.13%. The median moved by 3.56%. The median only cares about which value sits in the middle position, so changing a price from $500,000 to $5,000,000 does not move it — that observation was already above the middle and stays there. The mean adds every value up, so the size of each one matters.

**The mean and median already disagreed before any corruption.** On the clean data the mean sits above the median, which is what a long right tail does. Reporting one or the other changes the story: the mean describes a market pulled upward by expensive coastal property, the median describes what an ordinary buyer encounters.

**The two detection methods mostly disagree.** The overall agreement rate looks high, but that is because the great majority of rows are flagged by neither method. Among the rows that were flagged at all, only a small fraction were caught by both. Tukey Fences looked only at price, so it flagged expensive houses. The Isolation Forest looked at the eight other features, so it flagged rows whose combination was unusual even when no single value was extreme.

**The forest was reacting to crowding, not price.** The rows only the forest flagged had average occupancy and population well above the dataset average. These are census blocks with unusual household sizes — plausibly institutions recorded as single households. A price-based rule would never find them, because their prices are ordinary.

**The lower fence was unreachable.** Because house prices cannot be negative and the interquartile range is wide, the lower fence came out below zero. No observation could possibly fall outside it. Tukey Fences cannot detect suspiciously low values on this variable, and anything flagged as too cheap would have to come from knowing something about housing rather than from the rule.

**Many of the high flags are an artifact.** A large share of the flagged prices sit at exactly the same maximum value, which is a cap applied when the data was collected rather than a real price. These rows are not errors and should not be deleted — deleting them would remove the most expensive neighborhoods and bias any later model downward. The censoring needs to be handled or stated.

## What I would do with a flagged row

I would not delete it. A flag is a question, not a verdict. First I would check whether the value is possible at all, then whether it is an artifact of how the data was collected, then whether other columns in the same row corroborate it. Only a value that is impossible or contradicted elsewhere is a candidate for removal, and I would record the decision either way.
