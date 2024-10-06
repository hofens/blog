


``` dataview
table without id
	file.link as "需求",
	备注,
	分支,
	dateformat(publishTime,"yyyy-MM-dd") as publishTime
from "CC/业务需求"
sort file.ctime desc
```

>  https://zhuanlan.zhihu.com/p/614881764



```contributionGraph
title: Contributions
graphType: default
dateRangeValue: 366
dateRangeType: LATEST_DAYS
startOfWeek: 1
showCellRuleIndicators: true
titleStyle:
  textAlign: left
  fontSize: 15px
  fontWeight: normal
dataSource:
  type: PAGE
  value: ""
  dateField:
    type: FILE_CTIME
  filters: []
fillTheScreen: true
enableMainContainerShadow: false
cellStyle:
  minHeight: 4px
cellStyleRules:
  - id: Lovely_a
    color: "#fedcdc"
    min: 1
    max: 2
  - id: Lovely_b
    color: "#fdb8bf"
    min: 2
    max: 3
  - id: Lovely_c
    color: "#f892a9"
    min: 3
    max: 5
  - id: Lovely_d
    color: "#ec6a97"
    min: 5
    max: 9999

```



