# Timetable Optimization of Multiple Bus Routes Along a Shared BRT Corridor

## I. Introduction
### 1. Why I Want to Solve this Problem.
Bus rapid transit (BRT) systems attempt to improve the quality of bus services by separating their right of way from other traffic, making them immune to congestion. They are widely regarded as a significantly cheaper alternative to metros or other rail systems. While higher-level systems may implement features such as grade-separation and priority signals at intersections, passing lanes at stations, off-board fare collection, and platform-level boarding, rudimentary ones consist simply of a bus-only lane.When multiple lines operate on this one lane at high frequencies (e.g., during peak hours), the bus lane itself can become congested such that it becomes even slower than "normal" lanes. This is especially so when multiple operators compete along the same corridor since while operating at a higher frequency will yield more revenue for that line at the expense of the travel time of all other passengers.

|![brisbane brt congestion](https://user-images.githubusercontent.com/59413070/173245828-2efeed81-5038-4d96-b32b-af4401be9e1c.png)|![Brisbane brt map](https://user-images.githubusercontent.com/59413070/173246380-b65b4810-b62b-4df0-8c8f-ea6defc16314.png)|
|:---:|:---:|
|<b>Brisbane Metro at rush hour</b>|<b>Line map</b>|

|![Goyang_BRT_Congestion](https://user-images.githubusercontent.com/59413070/173245460-abe099f8-ed78-4700-a2d7-56e76a321744.jpg)|
|:---:|
|<b>Goyang BRT at rush hour</b> <i>(and why this problem is personal)</i>|

Ultimately this results in a Nash equilibrium where each operator chooses to employ more buses than is optimal for the sake of passenger utility. Because each operator seeks to maximize its revenue and avoid the worst-case scenarios, the rational choice for both is to increase frequency to the point where travel speed along the BRT corridor is slow and the revenue low.

![Nash equilibrium](https://user-images.githubusercontent.com/59413070/173248133-ff3e712d-1602-4c30-b02d-0110ae25160f.png)




### 2. Contribution
To the best of my knowledge, research on the bus scheduling problem with regards to variable travel times assumed "mixed" traffic conditions. In other words, travel times were considered to be dependent on the time of day and non-bus traffic conditions, independent to the operational pattern of buses. With respect to bus-only lanes, such an assumption cannot hold. The link travel time along a BRT corridor is dependent only on the volume (frequency) of buses on it. In other words, the schedulling of buses is the direct and only determinant of its travel time.
As such, by adjusting the timetables of multiple lines along a shared BRT corridor while taking changes in travel times due to these adjustments into account, a more desirable operation of a BRT system may be achieved. Because buses in South Korea are run in a (semi)public structure where operators do not have to compete with each other and overall passenger utility is pursued instead, this study focuses on minimizing the average travel time among *all* passengers involved.


## II. Background
### 1. A Data-Driven and Optimal Bus Scheduling Model With Time-Dependent Traffic and Demand (Wang et al., 2017)
#### Summary
* Extrapolates varying travel times between consecutive bus stops by time of day from smart card data.
* Divides the day into 1,440 intervals (1 minute) and calculates the average travel time for each link at each interval.
* Makes use of multiple-stop trip entries by dividing them into several consecutive segments.
* Solved using CPLEX.

#### Challenges & Shortcomings
* Travel times are dependent on time of day only.
* Cannot distinguish between time spent on boarding/alighting, waiting in queue, and actual travelling.

|![image](https://user-images.githubusercontent.com/59413070/173249479-8af7993a-39d7-4d99-95da-e974fc716294.png)|![image](https://user-images.githubusercontent.com/59413070/173249493-b12dd606-cef4-443d-9daf-f60327ffc4d6.png)|
|:-----:|:-----:|
|<b> Average travel time from A to B at t<sub>0</sub> (dotted line).</b>|<b>Travel time from B to C extrapolated using trip from A to C (pink).</b>|

### 2. A bi-level programming for bus lane network design (Yu et al., 2015)
#### Summary
* Algorithm for identifying best locations for bus-only lanes.
* Employs BPR function to calculate dwell times, and travel times for bus-only lanes and mixed traffic lanes.
* User equilibrium among modes and routes depending on travel time.
* Frequency optimization for given layout of bus-only lanes.

![Dalian](https://user-images.githubusercontent.com/59413070/173250487-79014f5b-bbec-4c49-b881-9845b477a7c7.PNG)

#### Challenges & Shortcomings
* Travel times are dependent on time of day only.
* BPR function parameters ($\alpha$, $\beta$) fixed at the generic values.
* No calibration and comfirmation of model design with actual figures.


## III. Data Wrangling
Smart card data for a four-day period (16 Sep ~ 19 Sep 2017) was used. Data for each day contains 20+ million rows, where each row represents one unlinked trip. In cases where the passenger transferred from one bus to another, it is recorded as two separate trips. For each trip, the time at which the passenger tapped on and off the bus, as well as the location in terms of the nearest stop was recorded. Additionally, the license plate and departure time at origin, as well as the line number, for each bus is included. If multiple passnegers used the same smart card to board, the number of passengers is also specified.


### Glimpse

```
glimpse(sample_df)
```

|![image](https://user-images.githubusercontent.com/59413070/173251346-601802a5-514a-4714-aacf-43f4cd325901.png)|
|:---:|
|<b> sample structure of data used </b>|

### OD and operation records
<details>
<summary>Trips where either the origin or the destination is part of Goyang BRT were filtered, and those that shared the same bus were aggregated. The earliest and latest boarding and alighting times for each bus at each stop were also recorded, along with the number of passengers for each.</summary>

```r
today_brt[today_brt == "~"] <- NA
today_brt$운행출발일시  <- ymd_hms(today_brt$운행출발일시)
today_brt$승차일시  <- ymd_hms(today_brt$승차일시)
today_brt$하차일시  <- ymd_hms(today_brt$하차일시)
today_brt$하차정류장ID_정산사업자  <- as.numeric(today_brt$하차정류장ID_정산사업자)
today_brt$승차정류장ID_교통사업자 <- mapvalues(
  today_brt$승차정류장ID_교통사업자,
  from = c(9004357,9034780,9034778,9034804,9034774,9034799,9036576,7000088,9034805,9034770),
  to = c(4106837,4106840,4106851,4106852,4110807,4106859,4106821,4196166,4196168,4106818)
)
today_brt$하차정류장ID_정산사업자  <- mapvalues(
  today_brt$하차정류장ID_정산사업자,
  from = c(9004357,9034780,9034778,9034804,9034774,9034799,9036576,7000088,9034805,9034770),
  to = c(4106837,4106840,4106851,4106852,4110807,4106859,4106821,4196166,4196168,4106818)
)
#### dw and tt sep ####
this_on <- today_brt %>%
  group_by(노선ID_정산사업자,운행출발일시,승차정류장ID_교통사업자,차량등록번호) %>%
  dplyr::summarise(boardings=sum(이용객수_다인승),firston=min(승차일시),laston=max(승차일시))
this_off <- today_brt %>%
  group_by(노선ID_정산사업자,운행출발일시,하차정류장ID_정산사업자,차량등록번호) %>%
  dplyr::summarise(alightings=sum(이용객수_다인승),firstoff=min(하차일시),lastoff=max(하차일시))
this_onoff <-
  full_join(this_on,
            this_off,
            by = c('노선ID_정산사업자', '차량등록번호','운행출발일시', '승차정류장ID_교통사업자' = '하차정류장ID_정산사업자')) |>
  filter(is.na(boardings)|is.na(alightings)|(boardings < 60 & alightings < 60))
this_onoff$boardings[is.na(this_onoff$boardings)] <- 0
```                                                     
</details>

|![arregated](https://user-images.githubusercontent.com/59413070/173253531-6cb942b5-ff9c-4e72-a2b6-2e2e56942d16.PNG)|
|:---:|
|Note the significant number of NA's indicating no boarding or alighting passengers at that stop|

  
## IV. Preliminary results
### Dwell time
  
It was found that the overwhelming proportions of stops only involved a handful of boardings and alightings, if at all.
 
|<b> Distributions of boardings and alightings|
|:---:|
|![Rplot01](https://user-images.githubusercontent.com/59413070/173252061-f8ae6eca-add8-48b3-ad3b-88ab4e81c0c2.png)|

As explored above by Yu et al. (2015), the BPR function on dwell time assumes that dwell time is dependent on the dominant action at that stop, i.e. the greater number between boardings and alightings. Applying the same principle to this data however suggested that not only is that not the case, the utilization of the BPR function is impossible. The <i>maxdwell</i> time is calculated as the difference between the latest boarding or alighting time and the earliest boarding time. This represents the longest feasible dwell time, as the first boarding must have happened after the bus came to a full stop and all alightings may have happened before the bus departed. In cases where only one tag was made at a stop, the dwell time is assumed to have been zero and excluded from regression.
Other values calculated by combinations of tag-in and tag-out times were also explored, but none yielded acceptable values.

|<b> Dwell time shows no meaningful correlation with number of boarding/alighting passengers|
|:---:|
|![Rplot](https://user-images.githubusercontent.com/59413070/173252225-2dad85e6-53f7-42a3-89cd-727158c7fb53.png)|
  
 
### Travel time
Because the chosen method of estimating dwell time was not viable, meaningful attempts at deducing travel time could not be made. Instead, the more simplified algorithm established by Wang et al. (2017) was used to extrapolate a **link** time, which conceptually is the sum of dwell time and travel time for each link.

An exploration of the travel time from one end of the BRT corridor to another clearly showed a temporal variance throughout the day, with the morning and evening rush hours requiring double the time in comparison to the earliest and latest buses of the day. This is also in line with the number of bus departures by time of day. In other words, the frequency of buses appears to have a positive correlation with travel time on the bus-only lane.
  
|<b>Travel Time along BRT Corridor (Busiest Line)</b>|<b>Travel Time along BRT Corridor (All Lines)</b>|
|:---:|:---:|
|![image](https://user-images.githubusercontent.com/59413070/173252808-1d53c8f8-bbe8-4e4f-a08e-44200422cfc7.png)|![image](https://user-images.githubusercontent.com/59413070/173252812-054488b5-2a1b-44ef-8175-38e8bba0ec8e.png)|
|<b># of Buses by Hour (Busiest Line)</b>|<b># of Buses by Hour (All Lines)</b>|
|![deptime hist](https://user-images.githubusercontent.com/59413070/173253309-ec48dd1c-ba09-4dda-9003-228deb7d59f7.png)|![all lines hist](https://user-images.githubusercontent.com/59413070/173253311-b149c5e5-d16f-443b-8e93-2f18d024ec64.png)|

It need be noted however that when looking at each individual link between two consecutive stations, no such correlation could be observed. This is probably due to the fact that while travel time dominates dwell time on a larger scale such is not the case at the segment level.
  
|<b>Travel Time on a single link (Busiest Link)</b>|<b>Travel Time on a single link (Terminus Link)</b>|
|:---:|:---:|
|![image](https://user-images.githubusercontent.com/59413070/173253409-07546470-9494-41a9-8e3b-fa1fb815b549.png)|![image](https://user-images.githubusercontent.com/59413070/173253413-0082ad61-9160-41be-bf9d-9c23743e86d3.png)|

## V. Concluding Remarks
### Next steps for this study
- [x] Simplify network by truncating branches.
> <img src="https://user-images.githubusercontent.com/59413070/173253829-7eb6fcc3-9d51-4b8c-a3d2-7c1a1216cb97.png" height="250"/>
  
- [ ] Elect more fit model for estimating dwell time and travel time. (separately)
- [ ] Add capacity and maximum travel time constraints.

### Further studies
* Introduce comptetition between operators.
* Differentiate waiting time value and travelling time value.
