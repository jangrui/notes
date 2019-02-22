# shell 变量替换与测试

## 变量替换

|规则|说明|
|-|-|
|${变量名#匹配规则}	|从变量开头进行规则匹配，将符合最短的数据删除|
|${变量名##匹配规则}	|从变量开头进行规则匹配，将符合最短的数据删除|
|${变量名%匹配规则}	|从变量尾部进行规则匹配，将符合最短的数据删除|
|${变量名%%匹配规则}	|从变量尾部进行规则匹配，将符合最短的数据删除|
|${变量名/旧字符串/新字符串}	|变量内容符合旧字符串，则第一个旧字符串会被新字符串取代|
|${变量名//旧字符串/新字符串}	|变量内容符合旧字符串，则全部旧字符串会被新字符串取代|

## 变量测试

|变量配置方式|str没有配置|str为空字符串|str已配置且非空|
|-|-|-|-|
|var=${str-expr} |var=expr|var=    |var=$str|
|var=${str:-expr}|var=expr|var=expr|var=$str|
|var=${str+expr} |var=    |var=expr|var=expr|
|var=${str:+expr}|var=    |var=    |var=expr|
|var=${str=expr} |var=expr|var=    |var=$str|
|var=${str:=expr}|var=expr|var=expr|var=$str|

