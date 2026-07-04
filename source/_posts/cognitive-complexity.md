---
title: cognitive_complexity
date: 2023-11-30 15:20:52
tags: ["前端"]
---
Recently, I focused on the code improvement, to fix the issues found out by SonarQube.
The SonarQube has a rules that one function cognitive complexity should not over 15, otherwise, it will be reported as a critical issue.

#### Cognitive complexity definition
Cognitive complexity is a measure of how difficult a method's control flow is to understand. Methods with high cognitive complexity will be difficult to maintain

#### How to calculate cognitive complexity
The method to calculate cognitive complexity is to count the control flow key words and the levels they placed.

(1) &&, || Condition judgment symbol +1
(2) if, else if, else, switch branch statement +1
(3) for, while, do while loop statements +1
(4) catch catch exception statement +1
(5) break, continue interrupt statement +1
(6) if there is a nest of if, for, while, do while, catch, the inner statement is +1 relative to the outer statement

#### How to reduce the cognitive complexity
The target is to reduce the times of the usage of if/else/for and so on. 
1.Replace if/else with switch
2.Replace if/else with LUT(lookup table)
3.Early return to reduce the nesting use
4.Split function
5.... 