### 简单表达式

```
<input type="text" name="userName" value="James Carrot" th:value="${user.name}" />
```
#### --选择/星号表达式 *{……}

```
<div th:object="${session.user}">                                                                       
     <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>    
</div>
```
#### --文字国际化表达式  #{……}

```
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
```

### 常用的th标签
#### --简单数据转换（数字，日期）

```
<dt>价格</dt>
  　 <dd th:text="${#numbers.formatDecimal(product.price, 1, 2)}">180</dd>
　　 <dt>进货日期</dt>
　　 <dd th:text="${#dates.format(product.availableFrom, 'yyyy-MM-dd')}">2014-12-01</dd>
```
#### --字符串拼接

```
<dd th:text="${'$'+product.price}">235</dd>
```
#### 转义和非转义文本

```
<div th:text="${html}">
　　This is an &lt;em&gt;HTML&lt;/em&gt; text. &lt;b&gt;Enjoy yourself!&lt;/b&gt;
</div> 
<div th:utext="${html}">
　　This is an <em>HTML</em> text. <b>Enjoy yourself!</b>
</div>
```

#### 表单中

```
<form th:action="@{/bb}" th:object="${user}" method="post" th:method="post">

    <input type="text" th:field="*{name}"/>
    <input type="text" th:field="*{msg}"/>

    <input type="submit"/>
</form>
```
#### 显示页面的数据迭代

```
<tbody th:remove="all-but-first">
　　　　　　　　　　//将后台传出的 productList 的集合进行迭代，用product参数接收，通过product访问属性值
                <tr th:each="product:${productList}">
　　　　　　　　　　　　//用count进行统计，有顺序的显示
　　　　　　　　　　　　<td th:text="${productStat.count}">1</td>
                    <td th:text="${product.description}">Red Chair</td>
                    <td th:text="${'$' + #numbers.formatDecimal(product.price, 1, 2)}">$123</td>
                    <td th:text="${#dates.format(product.availableFrom, 'yyyy-MM-dd')}">2014-12-01</td>
                </tr>
                <tr>
                    <td>White table</td>
                    <td>$200</td>
                    <td>15-Jul-2013</td>
                </tr>
                <tr>
                    <td>Reb table</td>
                    <td>$200</td>
                    <td>15-Jul-2013</td>
                </tr>
                <tr>
                    <td>Blue table</td>
                    <td>$200</td>
                    <td>15-Jul-2013</td>
                </tr>
      </tbody>
```






### --条件判断

```
<span th:if="${product.price lt 100}" class="offer">Special offer!</span>
```

```
<!-- 当gender存在时，选择对应的选项；若gender不存在或为null时，取得customer对象的name-->
<td th:switch="${customer.gender?.name()}">
    <img th:case="'MALE'" src="../../../images/male.png" th:src="@{/images/male.png}" alt="Male" /> <!-- Use "/images/male.png" image -->
    <img th:case="'FEMALE'" src="../../../images/female.png" th:src="@{/images/female.png}" alt="Female" /> <!-- Use "/images/female.png" image -->
    <span th:case="*">Unknown</span>
</td>
```


#### th:unless

```
//除非period_name等于null,不然显示，并且加入th:placeholder
<input type="text"  th:placeholder="${period_name}" th:unless="${period_name} eq null"/>
								
<label for="period_name" class="col-sm-4 control-label" th:unless="${name} ne null">纯销期间</label>
```


```
<!-- 增加class="enhanced"当balance大雨10000 -->
<td th:class="${customer.balance gt 10000} ? 'enhanced'" th:text="${customer.balance}">350</td>
```

#### --根据后台数据选中select的选项

```
<div class="form-group col-lg-6">
      <label >性别<span>&nbsp;Sex:</span></label>
      <select th:field="${resume.gender}" class="form-control" th:switch="${resumes.gender.toString()}"
            data-required="true">
              <option value="男" th:case="'男'" th:selected="selected" >男</option>
              <option value="女" th:case="'女'" th:selected="selected" >女</option>
              <option value="">请选择</option>
      </select>
 </div>
```
因为gender是定义的Enum（枚举）类型，所以要用toString方法。用th:switch指定传出的变量,用th:case对变量的值进行匹配。！"请选择"放在第一项会出现永远选择的是这个选项。或者用th:if

```
<div class='form-group col-lg-4'>
          <select class='form-control' name="skill[4].proficiency">
                <option >掌握程度</option>
                <option th:if="${skill.level eq '一般'}" th:selected="selected">一般</option>
                 <option th:if="${skill.level eq '熟练'}" th:selected="selected">熟练</option>
                 <option th:if="${skill.level eq '精通'}" th:selected="selected">精通</option>
           </select>
</div>
```

#### --spring表达式语言

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Thymeleaf tutorial: exercise 10</title>
        <link rel="stylesheet" href="../../../css/main-static.css" th:href="@{/css/main.css}" />
        <meta charset="utf-8" />
    </head>
    <body>
        <h1>Thymeleaf tutorial - Solution for exercise 10: Spring Expression language</h1>
  
        <h2>Arithmetic expressions</h2>
        <p class="label">Four multiplied by minus six multiplied by minus two module seven:</p>
        <p class="answer" th:text="${4 * -6 * -2 % 7}">123</p>
 
        <h2>Object navigation</h2>
        <p class="label">Description field of paymentMethod field of the third element of customerList bean:</p>
        <p class="answer" th:text="${customerList[2].paymentMethod.description}">Credit card</p>
 
        <h2>Object instantiation</h2>
        <p class="label">Current time milliseconds:</p>
        <p class="answer" th:text="${new java.util.Date().getTime()}">22-Jun-2013</p>
        
        <h2>T operator</h2>
        <p class="label">Random number:</p>
        <p class="answer" th:text="${T(java.lang.Math).random()}">123456</p>
    </body>
</html>
```

