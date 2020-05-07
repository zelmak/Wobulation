# Rapid Prototype for Percentile based Cluster Wobulation
## Overview
This is a basic research effort focused on the examination of a percentile-based cluster wobulation strategy. 
Focus is strictly on the proposed algorithm’s ability to “follow the data” toward a sufficiently accurate steady state solution.
The initial effort will focus on wobulation of a single cluster.
## Concept
Recursively update cluster descriptions based on a preserved frequency distribution of the relevant underlying data.  
The underlying frequency distribution is also recursively updated in conjunction with the cluster definition, maintaining a predefined, fixed width buffer above and below the current cluster boundaries.  
The use of percentiles about the frequency distribution mean to establish cluster boundaries enables the formation of asymmetric cluster ranges and does not require a normality assumption.

![alt text](https://github.com/dpxt2o9az/Wobulation/blob/master/img/number-line.png "Logo Title Text 1")

## Assumptions
1. The underlying cluster data is not normally distributed.
2. The underlying data is uniformly distributed within a given bin (small bin assumption.)
## Recursive Update Process
1. Increment frequency count by 1 for the bin containing the new data point. If the data point does not correspond to an existing bin \[inside the cluster\], discard the data point without update.
2. Update the cluster mean \[(see formula, below.)\]
3. Update the cluster min to be the lower boundary of bin containing the (1-p)th percentile of the data below the new mean.
4. Update the cluster max to be the upper boundary of the bin containing the pth percentile of the data above the new mean.
5. Add or delete bins \[from the cluster\] so that the lowest boundary of the lowest bin is cluster_min-H and the upper boundary of the largest bin is the cluster_max +H.
### Input Parameters
p – percentile to be used to form cluster boundaries, range: (0,1), default = 0.9  
H – horizon to be monitored beyond cluster boundaries  (could be integer used as incr multiplier)  
Incr - Processing increment (default = 0.1)  
### Output Values
![alt text](https://github.com/dpxt2o9az/Wobulation/blob/master/img/mu.png "Logo Title Text 1") – cluster mean  
C_min – lower cluster boundary  
C_max – upper cluster boundary  
### Computational
#### Precision
Cluster boundaries and bin boundaries are in tenths, bin midpoints are of the form xxx.x5
#### Calculating the Mean
![alt text](https://github.com/dpxt2o9az/Wobulation/blob/master/img/general-mean-formula.png "Logo Title Text 1") or recursively ![alt text](https://github.com/dpxt2o9az/Wobulation/blob/master/img/recurrent-mean-formula.png "Logo Title Text 2")
where  
f = (bin) frequency  
m = (bin) midpoint (in the latter, m_i is the midpoint matched by the current data value)  
N = sum of all bin counts  
\[Therefore, if m_i is the current value's midpoint, that implies that i is the index which implies that N_k+1 is equivalent to i.\]
#### Calculating the percentiles
Assume uniform distribution within any given bin.  Interpolate to determine how many values within the mean bin are above and below the mean value.  Interpolation on the other end is not required since it is only necessary to determine which bin the specified percentile falls in.
#### Initialization
Assumption is that the process is used to refine an existing cluster as new data is processed, and defective initialization could potentially cause the algorithm to diverge from a meaningful steady state solution.  Hence, a reasonable starting point should be defined prior to processing streaming data.  Another alternative may be to ensure the initial data stream is relatively “well behaved”, in which case the algorithm may be able to converge on a solution starting from a single data point.
## Simple Validation Case
Initialize 9 bins having the midpoints shown in the top row with the initial frequencies shown in the bottom row.
|0.65 |0.75 |0.85 |0.95 |1.05 |1.15 |1.25 |1.35 |1.45 |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|
|1    |2    | 3   |8    |8    |5    |5    |3    |1    |

Process 125 new data points all having a value of 0.85.

*Expected results:*  
Cluster limits start at 0.7 -1.4, eventually converging to 0.8-1.0
## Other Test/Experimental Cases
### Test Data Set 2
Process the attached data (Wob_Valid_case2.csv) in the order provided.

There are no pre-existing bin values, i.e., start with the first value and establish bins appropriately. Since the first value is ~8.36, the initial mean bin will be [8.3,8.4).

#### Case 1:
Use a "horizon" of 0.3 (i.e., 3 bins above and 3 bins below the "mean bin"  
Note: on first iteration there will be 7 bins:

[8, 8.1), [8.1, 8.2), [8.2, 8.3), [8.3, 8.4). [8.4, 8.5), [8.5, 8.6), [8.6, 8.7) with bin counts: 0,0,0,1,0,0,0

*Expected Result*
Initial Cluster Range: 8.3-8.4
Final Cluster Range: 9.3-10.9

#### Case 2:
Use a "horizon of 1.0 (i.e., monitor 10 bins below and 10 bins above the "mean bin")  

*Expected Result*
Initial Cluster Range: 8.3 - 8.4
Final Cluster Range: 9.3-10.9

Detailed results/transitions are in attached spreadsheet.
