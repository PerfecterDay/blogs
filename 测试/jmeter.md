 jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]

jmeter -n -t ./trade.jmx -l result.jtl3.10.1  -e -o ./report


Jmeter响应中的中文乱码：
添加 BeanShell 后置处理程序，添加如下脚本：
```
prev.setDataEncoding("utf-8")
```


jmeter -n -t ./0627.jmx -l ./user0627/20-1-2  -e -o ./user0627/web
jmeter -n -t ./0627.jmx -l ./search0627/20-1-2  -e -o ./search0627/web
jmeter -n -t ./0627.jmx -l ./msg0627/20-1-2  -e -o ./msg0627/web
jmeter -n -t ./0627.jmx -l ./info0627/20-1-2  -e -o ./info0627/web