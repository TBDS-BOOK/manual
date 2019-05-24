### 隐式参数

#### 1. 时间隐式参数
a. 支持如下格式： 
```
${YYYYMMDD}   
或者 
${yyyyMMddHHmm} / ${yyyyMMddHHmmss}    
又或者 
 ${yyyy-MM-dd HH:mm}  
跟
 ${yyyy-MM-dd HH:mm:ss}  
```
b. 更多复杂的格式   
```
 可以使用 ${YYYY},${MM},${dd},${HH},${mm},${ss},${SSSSSS}"  
来组合,比如：  
'${YYYY}'-'${MM}'-'${dd}' '${HH}':'${mm}':'${ss}':${SSSSSS}
又或者  
${yyyy}${MM}${dd}/${yyyy}${MM}${dd}${HH}
```
c. 当然也向下兼容   
```
${YYYYMMDD} 以及 ${YYYYMMDDHH} ，${YYYYMMDDHHmm} 等格式
```
d. 时间加减表达式  
基于V5.0+版本   
必须使用如下表达式（小时只支持24小时）  
```
$TimeCompute(${yyyy-1}${MM+13}${dd+31})
$TimeCompute(${yyyy}${MM}${dd-1}) 
$TimeCompute(${yyyy}${MM-1}${dd})
$TimeCompute(${yyyy}${MM-1}${dd-1})
$TimeCompute(${yyyy-1}${MM}${dd})
$TimeCompute(${yyyy-1}${MM}${dd-1})
$TimeCompute(${yyyy-1}${MM-1}${dd-1})
$TimeCompute(${yyyy-1}${MM-1}${dd-1}${HH-2})
$TimeCompute(${yyyy-1}${MM-1}${dd-1}${HH+1})
```
