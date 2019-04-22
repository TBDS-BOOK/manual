### 隐式参数

#### 1. 时间隐式参数
支持如下格式： ${YYYYMMDD}   
或者 ${yyyyMMddHHmm} / ${yyyyMMddHHmmss}    
又或者 ${{yyyy-MM-dd HH:mm}  
跟 ${{yyyy-MM-dd HH:mm:ss}格式  

更多复杂的格式 可以使用   
${YYYY},${MM},${dd},${HH},${mm},${ss},${SSSSSS}" 来组合,比如：  
```
'${YYYY}'-'${MM}'-'${dd}' '${HH}':'${mm}':'${ss}':${SSSSSS}
又或者  
${yyyy}${MM}${dd}/${yyyy}${MM}${dd}${HH}
```

当然也向下兼容  
${YYYYMMDD} 以及 ${YYYYMMDDHH} ，${YYYYMMDDHHmm} 等格式
