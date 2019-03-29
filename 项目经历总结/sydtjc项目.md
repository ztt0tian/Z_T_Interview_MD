项目三：sydtjc

#### 1.整体架构

spring boot

mybaits

mysql

thymeleaf

layui

echart

maven

#### 2.流程

##### 1.powerdesigner 设计数据库,生成sql文件

##### 2.navicat 导入 生成数据库表 可视化

##### 3.使用mybatis逆向工程生成实体类、实体Example、dao Interface及对应Mapper文件

##### 4.业务逻辑开发

#### 3.遇到的问题及解决

###### 1.前端页面代码杂乱且繁重（3000多行）

​	问题简述：糅杂太多重复性的代码，有很多不必要的div js代码以及一些样式控制代码，且一些ID命名随意可读性较差，后面整合实在太过复杂。

​	解决：因为主要用于显示表格信息和一些统计图表显示,加之使用的layui的table组件和date组件，可以通过js来render，所以重组界面，大致格局不变，尽可能将表格的渲染工作通过引入js文件，通过加载函数来实现页面一些组件的渲染。命名尽可能规范。

示例：导航栏页面内“跳转”

原：

```javascript
function opens(obj) {
    for (var i = 1; i <= 30; i++) {
        if (i === obj) {
            document.getElementById("con" + i).style.display = "block";
        } else
            document.getElementById("con" + i).style.display = "none";
    }
}
```

后：

```javascript
function display(table_div_id) {
    $('.display-table').css('display', 'none');
    var _table_div_id = '#' + table_div_id;
    $(_table_div_id).css('display', 'block');
}
```

###### 2.前端传递数组数据,后台接收出错

解决：

ajax中：

```javascript
type: "post",
url: "/getCustomYearInfo",
dataType:"json",
contentType:"application/json",
data:JSON.stringify(itemArray),
```

Controller

```java
@RequestMapping(value = "/getCustomYearInfo",method = RequestMethod.POST)
public CustomizedYearFenxiTableJson getSomeYearInfo(@RequestBody int[] years) 
```

起初方法为‘get’.报错

`Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986`

经查
RFC 3986文档规定，Url中只允许包含英文字母（a-z，A-Z）、数字（0-9）、- _ . ~ 4个特殊字符以及所有保留字符。
RFC3986中指定了以下字符为保留字符：! * ’ ( ) ; : @ & = + $ , / ? # [ ]

就将method改成‘post’，没有问题。

###### 3.sql查询及自定义查询对象封装

`INNER JOIN ( SELECT '规模2' AS scale ) s` 添加自定义描述

`union all`  合并结果

`left join`连接查询

`group by` 要配合聚合函数使用 否则达不到想要的效果

自定义实体对象封装查询结果。

`<resultMap></resultMap>标签完成实体到mysql查询结果表列的映射`

`<select></select>`标签设置sql语句 mapper文件中的比较符号（<,>,<=,>=）要使用转义符

###### 4.自动生成Mapper文件<resultMap>ID重复问题

1.  可能原因generator多次执行且上次文件未清除，直接在原文件附加内容
2. 当前连接中其他数据库含有相同结构和名称的表

###### 5.ajax异步添加select内容，但select下拉内容不显示

在layui中需要对表单重新渲染

```
layui.use('form', function(){
        var form = layui.form;
        form.render();
    });
```

###### 6.跨域问题

加@CrossOrigin(allowCredentials="true",maxAge = 3600)

```java
@RestController
@CrossOrigin(allowCredentials = "true", maxAge = 3600)
public class CustomizedController {...}
```

#### 4.项目总结

​	项目本身复杂度不是很大,主要是存储一些数据，然后查找和统计数据。主要涉及多表连接查询。从这个项目大致懂得了如何去使用Spring Boot开发，切身体会到了其便利，不用像原来那般为环境配置而花费过多时间成本。对AJAX的使用也比较熟练了，了解到了Layui，另外还有thymeleaf，它对前后端的测试兼容性很好，可同时用于静态和动态情景。

​	评价：这个项目还很不完善，缺乏可视化的后台管理系统、一些必要的输入规范验证、缺乏完善的权限控制（这里仅仅通过session取值来验证权限）和安全策略、日志记录等，这些对于系统的健壮性和安全性都有很大帮助。

​	后期希望可以将shiro框架整合到这个项目中提高安全性。

