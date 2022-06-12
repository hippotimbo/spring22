# Timetable Optimization of Multiple Bus Routes Along a Shared BRT Corridor
## I. Introduction
### 1. Why I Want to Solve this Problem.
Bus rapid transit (BRT) systems attempt to improve the quality of bus services by separating their right of way from other traffic, making them immune to congestion. They are widely regarded as a significantly cheaper alternative to metros or other rail systems. While higher-level systems may implement features such as grade-separation and priority signals at intersections, passing lanes at stations, off-board fare collection, and platform-level boarding, rudimentary ones consist simply of a bus-only lane.When multiple lines operate on this one lane at high frequencies (e.g., during peak hours), the bus lane itself can become congested such that it becomes even slower than "normal" lanes. This is especially so when multiple operators compete along the same corridor since while operating at a higher frequency will yield more revenue for that line at the expense of the travel time of all other passengers.

![brisbane brt congestion](https://user-images.githubusercontent.com/59413070/173245828-2efeed81-5038-4d96-b32b-af4401be9e1c.png)|![Brisbane brt map](https://user-images.githubusercontent.com/59413070/173246380-b65b4810-b62b-4df0-8c8f-ea6defc16314.png)
:--:|:--:
<b>Brisbane Metro at rush hour</b>|<b>Line map</b>

![Goyang_BRT_Congestion](https://user-images.githubusercontent.com/59413070/173245460-abe099f8-ed78-4700-a2d7-56e76a321744.jpg)
:--:
<b>Goyang BRT at rush hour</b> <i>(and why this problem is personal)</i>|

Ultimately this results in a Nash equilibrium where each operator chooses to employ more buses than is optimal for the sake of passenger utility. Because each operator seeks to maximize its revenue and avoid the worst-case scenarios, the rational choice for both is to increase frequency to the point where travel speed along the BRT corridor is slow and the revenue low.

![Nash equilibrium](https://user-images.githubusercontent.com/59413070/173248133-ff3e712d-1602-4c30-b02d-0110ae25160f.png)




### 2. Contribution
To the best of my knowledge, research on the bus scheduling problem with regards to variable travel times assumed "mixed" traffic conditions. In other words, travel times were considered to be dependent on the time of day and non-bus traffic conditions, independent to the operational pattern of buses. With respect to bus-only lanes, such an assumption cannot hold. The link travel time along a BRT corridor is dependent only on the volume (frequency) of buses on it. In other words, the schedulling of buses is the direct and only determinant of its travel time.
As such, by adjusting the timetables of multiple lines along a shared BRT corridor while taking changes in travel times due to the adjustments into account, a more desirable operation of a BRT system may be achieved. Because buses in South Korea are run in a (semi) public structure where operators do not have to compete with each other and overall passenger utility is pursued instead, this study focuses on minimizing the average travel time among *all* passengers involved.


## II. Background
### 1. Existing Studies
#### Header 4 test


### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/hippotimbo/spring22/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
