## Overview

The connection assessment algorithm is designed to continuously monitor and optimise network performance by dynamically adjusting key parameters. It is organised into five main phases:

1. Parameter Boundaries & Initialisation
2. Dynamic Parameter Optimisation
3. Connection Assessment Loop
4. Quality Change Detection
5. Optimisation Feedback Loop

## Phase 1: Parameter Boundaries & Initialisation

During the initialisation phase, the algorithm sets up essential parameters:

**Parameter Ranges:**
- **Packets:** Defined with a minimum (`PACKETS_MIN = 2`) and maximum (`PACKETS_MAX = 50`).
- **Interval:** Set between `INTERVAL_MIN = 0.1ms` and `INTERVAL_MAX = 10ms`.
- **Test Frequency:** The initial value is set to `TESTS_PER_10MIN = 60` (i.e., 60 tests per 10 minutes).

**Assessment Variables:**
- **Tolerance Threshold:** Defines the tolerance for metric changes (set to 10%).
- **Quality Window:** Defines the number of tests used to determine the initial network characteristics (set to 5 tests).
- **Adaptation Period:** Parameter changes are adjusted every 10 tests.

## Phase 2: Dynamic Parameter Optimisation

The dynamic parameter optimisation phase is divided into two main parts: initial calibration and adaptive parameter selection.

### Initial Calibration Phase
1. **Median Value Testing:** The algorithm begins with median values for the packet count (26 packets) and interval (5ms). For each test within the quality window, latency, jitter, and success rate are recorded.
2. **Baseline Metrics:** After recording the metrics, median values for latency (`med_latency`), jitter (`med_jitter`), and success rate (`med_success_rate`) are calculated to form the baseline.
3. **Tolerance Bands:** Tolerance bands for latency and jitter are established based on the calculated baseline values (`latency_tolerance = med_latency * tolerance_threshold`, `jitter_tolerance = med_jitter * tolerance_threshold`).

### Adaptive Parameter Selection
1. **Stability Analysis:** Recent tests are analysed for stability by examining variations in latency, jitter, and success rate. The `stability_score` is calculated as:

   ```
   stability_score = (0.5 * (1 - (std_latency / med_latency))) +
                     (0.3 * (1 - (std_jitter / med_jitter))) +
                     (0.2 * med_success_rate)
   ```

2. **Parameter Adjustment:**
   - **Packet Count:** Increased if stability is low, decreased if stability is high.
   - **Interval:** Adjusted based on success rate. Increased when success rate is low to avoid network overload and decreased when success rate is high to increase test frequency.
   - **Test Frequency:** Modified based on variance analysis; increased when variance is high and decreased when variance is low.

## Phase 3: Connection Assessment Loop

In the connection assessment loop, the network is continuously monitored, and parameters are optimised as needed.

1. **Execute Test:** Perform a test using the current packet count and interval.
2. **Process Results:** If the test is successful, the metrics for latency, jitter, and success rate are updated. If the results show significant deviations from the baseline, a quality change is reported.
3. **Optimise Parameters:** Periodically update parameters after each adaptation_period.
4. **Time Management:** Calculate and sleep for the time remaining until the next test.

## Phase 4: Quality Change Detection

To detect significant changes in network quality, the algorithm compares recent results to baseline values.

1. **Calculate Differences:** Compute normalised differences for latency and jitter compared to the baseline.
2. **Weighted Significance:** Apply weights to determine an overall significance score (60% latency, 40% jitter).
3. **Dynamic Tolerance Adjustment:** Adjust the tolerance threshold based on recent stability trends.

## Phase 5: Optimisation Feedback Loop

The optimisation feedback loop phase allows the algorithm to improve over time based on its effectiveness.

1. **Track Metrics:**
   - **Success Rate:** Monitors the overall success rate of connection tests.
   - **Variance:** Tracks the variance in latency and jitter measurements compared to the baseline.
   - **Detection Accuracy:** Measures how accurately the algorithm detects changes in network quality.

2. **Calculate Optimisation Score:**
   The Optimisation Score is a weighted formula that assesses the effectiveness of the current parameter settings:

   ```
   optimisation_score = (0.4 * success_rate_score) +
                        (0.3 * variance_score) +
                        (0.3 * detection_accuracy_score)
   ```

   - **Success Rate Score:** Calculated as the average success rate over the last adaptation period.
   - **Variance Score:** Derived from the stability_score formula, but with a higher weight on variance:

     ```
     variance_score = (0.7 * (1 - (std_latency / med_latency))) +
                      (0.3 * (1 - (std_jitter / med_jitter)))
     ```

   - **Detection Accuracy Score:** Measures how well the algorithm detected changes in network quality compared to the ground truth. This is calculated as the ratio of correctly identified quality changes to the total number of changes.

3. **Adjust Adaptation Strategy:**
   Based on the Optimisation Score, the algorithm adjusts its adaptation strategy:

   - If the Optimisation Score is high (e.g., above 0.8), the adaptation strategy becomes more conservative, making smaller, gradual changes to the parameters.
   - If the Optimisation Score is low (e.g., below 0.5), the adaptation strategy becomes more aggressive, making larger, more frequent changes to the parameters.

This allows the algorithm to continuously improve its performance by learning from past experiences and adjusting its approach accordingly.

## Summary of Parameter Adjustments Based on Analysis

| Metric     | Condition     | Parameter Adjustment |
|------------|---------------|----------------------|
| Stability  | Low stability | Increase packet count |
|            | High stability| Decrease packet count |
| Variance   | High variance | Increase test frequency|
|            | Low variance  | Decrease test frequency|
| Success Rate| Low success rate | Increase interval between packets |
|            | High success rate| Decrease interval between packets |

## Potential Flaws and Improvements

1. **Baseline Metric Calculation:** The current implementation calculates the baseline metrics (latency, jitter, success rate) by taking the median of the values in the quality window. This provides a more robust measure, less susceptible to outliers or extreme values compared to using the average.

2. **Tolerance Band Calculation:** The tolerance bands for latency and jitter are calculated as a fixed percentage (tolerance_threshold) of the baseline median values. Consider using a more dynamic approach, such as using a sliding window to recalculate the tolerance bands over time or basing them on statistical properties (e.g., standard deviations) of the recent measurements, to better adapt to changing network conditions.

3. **Variance Score Calculation:** The variance score formula gives a higher weight to latency variance (0.7) compared to jitter variance (0.3). This may not be appropriate in all scenarios, as the relative importance of latency and jitter can vary depending on the specific use case. Consider making the weights more configurable or using a more sophisticated approach to combine the variance metrics.

## Pseudocode

```
# Phase 1: Parameter Boundaries & Initialisation
Set PACKETS_MIN, PACKETS_MAX, INTERVAL_MIN, INTERVAL_MAX, TESTS_PER_10MIN
Set TOLERANCE_THRESHOLD, QUALITY_WINDOW, ADAPTATION_PERIOD

# Phase 2: Dynamic Parameter Optimisation
# Initial Calibration Phase
Perform median value testing and record latency, jitter, success_rate
Calculate med_latency, med_jitter, med_success_rate as baseline metrics
Calculate latency_tolerance, jitter_tolerance based on baseline

# Adaptive Parameter Selection
for adaptation_period in range(ADAPTATION_PERIOD):
    Analyse recent tests for stability
    Calculate stability_score
    
    # Parameter Adjustment
    if stability_score is low:
        Increase packet_count
    else:
        Decrease packet_count
    
    Adjust interval based on success_rate
    Adjust test_frequency based on variance analysis

# Phase 3: Connection Assessment Loop
end_time = current_time + 600  # 10 minutes
while current_time < end_time:
    Execute test with current packet_count and interval
    Update latency, jitter, success_rate metrics
    if results show significant deviation from baseline:
        Report quality change
    
    current_time += time_between_tests

# Periodically optimize parameters
Optimise parameters after ADAPTATION_PERIOD

# Phase 4: Quality Change Detection
Calculate normalized differences in latency and jitter from baseline
Apply weights to determine overall significance score
Adjust tolerance_threshold based on stability trends

# Phase 5: Optimisation Feedback Loop
track_success_rate = []
track_variance = []
track_detection_accuracy = []

for adaptation_period in range(ADAPTATION_PERIOD):
    track_success_rate.append(calculate_success_rate())
    track_variance.append(calculate_variance())
    track_detection_accuracy.append(calculate_detection_accuracy())

success_rate_score = calculate_average(track_success_rate)
variance_score = calculate_variance_score(track_variance)
detection_accuracy_score = calculate_average(track_detection_accuracy)

optimisation_score = (0.4 * success_rate_score) + \
                     (0.3 * variance_score) + \
                     (0.3 * detection_accuracy_score)

Adjust adaptation strategy based on optimisation_score
```
