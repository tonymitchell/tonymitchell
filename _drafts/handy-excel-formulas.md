---
title: Handy Excel Formulas
---

## A1 vs. R1C1 reference style

See: [A1 or R1C1 Notation](https://bettersolutions.com/excel/formulas/cell-references-a1-r1c1-notation.htm)

Change:
1. File > Options
1. Open the Formulas section.
1. Under the **Working with formulas** heading, find the "R1C1 reference style" checkbox.
1. Uncheck to use A1 style, check to use R1C1 style.

## Formatting all days that are weekends

Create new Conditional Formatting rule.

### Example in R1C1 format

In this example we have a column of dates in the first column (A:A or C1)

|Field|Value (R1C1)|Value (A1)|
|-|-|-|
|Formula|`=AND(RC1<>"",OR(WEEKDAY(RC1)=1,WEEKDAY(RC1)=7))`|`=AND($A1<>"",OR(WEEKDAY($A1)=1,WEEKDAY($A1)=7))`|
|Format|{Yellow Highlight}|{Yellow Highlight}|
|Applies To|`=C1`|`=$A:$A`|


## Validate that a field in a named list 

Create a list of values
Select the list of values
On the ribbon bar, select Formulas > Defined Names (section) > Define Name

![New Name](/assets/img/2022-06-05-17-59-16.png)
