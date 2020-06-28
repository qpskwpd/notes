##### yml/yaml基本格式

```yaml
#普通数据配置
name: zhangsan
#对象配置
person:
  name: zhangsan
  age: 18
  addr: beijing

#行内对象配置
#person: {name: zhangsan, age: 18, addr: beijing}

#数组/集合配置
#字符串
city:
  - beijing
  - tianjin
  - shanghai
#city: [beijing, tianjin, shanghai]

#对象数据
student:
  - name: tom
    age: 18
    addr: beijing
  - name: lucy
    age: 17
    addr: tianjin
#student: [{name: tom, age: 18, addr: beijing}, {name: lucy, age: 17, addr: tianjin}]

#map数据(就是对象)
map:
  key1: value1
  key2: value2
```