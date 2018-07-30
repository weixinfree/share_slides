title: python实现简单的模板引擎
speaker: 王伟


[slide]

# python实现简单的模板引擎
----

1. 模板引擎？？what? 
2. 用python实现一个

[slide]

# 什么是模板引擎?

[slide]

## Vue

```html
<div id="app">
  {{ message }}
</div>

var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

[slide]

## Angular

```html
<h2>My Heroes</h2>
<ul class="heroes">
  <li>
    <span class="badge">{{hero.id}}</span> {{hero.name}}
  </li>
</ul>
```

[slide]

## React

```js
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[slide]

## Android DataBinding

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

[slide]

# 题外话: UI构建方式，从命令式向声明式的转变的趋势

[slide]

## Sublime Snippets

```html
<snippet>
     <content><![CDATA[
Hello, ${1:this} is a ${2:snippet}.
]]></content>
     <!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
     <!-- <tabTrigger>hello</tabTrigger> -->
     <!-- Optional: Set a scope to limit where the snippet will trigger -->
     <!-- <scope>source.python</scope> -->
</snippet>
```

[slide]


## AndroidStudio/IDEA Live Template

```java
for ($i$ : $data$) { 
    $cursor$ 
}
```

[slide]

## web服务器
```html
<!DOCTYPE html>
<html lang="zh" xml:lang="zh" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>打包服务器文件列表</title>
    <meta content="text/html; charset=utf-8" http-equiv="content-type">
</head>
<body>
<div>
    <ol>
    % for item in apk_files:
        <li>
            <div>
                <a href="{{item.download_url}}"><big>{{item.name}}</big></a> <sub>{{item.create_time}}, {{item.size}}</sub>
            </div>
        </li>
    % end
    </ol>
</div>
</body>

</html>
```

[slide]

## Python jinjia2 模板引擎语法
```html
<title>{% block title %}{% endblock %}</title>
<ul>
{% for user in users %}
  <li><a href="{{ user.url }}">{{ user.username }}</a></li>
{% endfor %}
</ul>
```

[slide]

## Java Volecity 模版引擎
```
#set( $iAmVariable = "good!" )
Welcome $name to velocity.com
today is $date.
#foreach ($i in $list)
$i
#end
$iAmVariable
```

[slide]

## Java FreeMaker 模板引擎
```xml
<<#list menu as food> 
${food.name} ${food.price?string.currency} 
</#list> 

<#if x == 1> 
 x is 1 
<#elseif x == 2> 
 x is 2 
<#elseif x == 3> 
 x is 3 
<#elseif x == 4> 
 x is 4 
<#else> 
 x is not 1 nor 2 nor 3 nor 4 
</#if> 
```


[slide]

## Java Thymeleaf 模板引擎

```html
<table>
  <thead>
    <tr>
      <th th:text="#{msgs.headers.name}">Name</th>
      <th th:text="#{msgs.headers.price}">Price</th>
    </tr>
  </thead>
  <tbody>
    <tr th:each="prod: ${allProducts}">
      <td th:text="${prod.name}">Oranges</td>
      <td th:text="${#numbers.formatDecimal(prod.price, 1, 2)}">0.99</td>
    </tr>
  </tbody>
</table>
```

[slide]

# 模板引擎的特性

1. **基本的控制结构: if for 赋值**
1. 过滤器
2. **内置全局函数列表**
4. html转义，安全
5. 模板继承
6. 模板宏

[slide]

# so...

[slide]


```java
%{fields = jclass.fields}%

private {{jclass.name}}(Builder builder) {
    %{for field in fields:}%
    this.{{field.name}} = builder.{{field.name}};
    %{end}%
}

public static class Builder {

    %{for field in fields:}%
    private {{field.jtype}} {{field.name}} = {{init_field(field)}};
    %{end}%

    %{for field in fields:}%
    %{if field.comment:}%
    /**
     * {{field.comment}}
     */
     %{end}%
    public Builder set{{field.name.title()}}({{field.jtype}} {{field.name}}) {
        this.{{field.name}} = {{field.name}};
        return this;
    }

    %{end}%
    public {{jclass.name}} build() {
        return new {{jclass.name}}(this);
    }
}
```


[slide]

`%{}%` => 代码块

- 赋值
- 函数调用
- if 控制流
- for 控制流

`{{}}` => 数据块

`{{init_filed(field)}}` => 数据块内支持函数调用


[slide]

## 模板引擎实现方式

1. 解释
2. **编译**
  

[slide]

## 编译方式

1. 生成新的源代码文件 + 动态语言的动态import （非常干净利落的实现）(埋点代码生成)

2. **直接使用python的compile 和 exec, 运行时生成代码和编译代码 (潜在的全局命名空间污染问题)**

[slide]

## python代码动态编译

### Compile

```java
Help on built-in function compile in module builtins:

compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)
    Compile source into a code object that can be executed by exec() or eval().

    The source code may represent a Python module, statement or expression.
    The filename will be used for run-time error messages.
    The mode must be 'exec' to compile a module, 'single' to compile a
    single (interactive) statement, or 'eval' to compile an expression.
    The flags argument, if present, controls which future statements influence
    the compilation of the code.
    The dont_inherit argument, if true, stops the compilation inheriting
    the effects of any future statements in effect in the code calling
    compile; if absent or false these statements do influence the compilation,
    in addition to any features explicitly specified.
```

### exec
```java
Help on built-in function exec in module builtins:

exec(source, globals=None, locals=None, /)
    Execute the given source in the context of globals and locals.

    The source may be a string representing one or more Python statements
    or a code object as returned by compile().
    The globals must be a dictionary and locals can be any mapping,
    defaulting to the current globals and locals.
    If only globals is given, locals defaults to it.
```

[slide]

# Fist Step
----
模板分割

[slide]

### 模板分割 - 识别不同的组成部分

1. **正则表达式**
2. 字符流  L1 识别符号 + 栈成对的匹配符号
3. 字符流  双字符流 + 栈成对的匹配符号

[slide]

```python
@staticmethod
def _split_template(tmpl: str) -> Sequence[Tuple[str, str]]:

    segments = re.finditer(
        r'(?<=\n).*?%\{(?P<code>.*?)\}%.*?\n|\{\{(?P<data>.*?)\}\}', tmpl)

    last_end = 0
    for seg in segments:

        start, end = seg.span()

        if start > last_end:
            text = tmpl[last_end:start]
            yield 'text', text

        code, data = seg.groups()

        if code:
            yield 'code', code
        if data:
            yield 'data', data

        last_end = end

    yield 'text', tmpl[last_end:]
```

[slide]

## 正则表达式解析

```
(?<=\n).*?%\{(?P<code>.*?)\}%.*?\n|\{\{(?P<data>.*?)\}\}
```
### 匹配 `%{code}%`
```
%\{(?P<code>.*?)\}%
```
### 匹配 `{{data}}`
```
\{\{(?P<data>.*?)\}\}
```
### code对模板样式的影响

- code所在的行对模板结果不产生影响
- 后向匹配: `.*?\n`
- 正则表达式的前向匹配: `(?<=\n).*?`
```
(?<=\n).*?%\{(?P<code>.*?)\}%.*?\n
```

[slide]

# Second Step
----
代码生成

[slide]

## 代码generator
```python
class _CodeGenerator:

    def __init__(self):

        self.indent = 0
        self.code = []

        self.code.append('_result = []')
        self.code.append('_append = _result.append')

    def add_code(self, _code):
        if _code.strip() == 'end':
            self.indent -= 4
        else:
            self.code.append('\n')
            self.code.append(' ' * self.indent + _code.strip())

            if _CodeGenerator._is_control_statement(_code):
                self.indent += 4

    def add_text(self, text):
        self.code.append(' ' * self.indent + f'_append("""{text}""")')

    def add_data(self, data):
        self.code.append(' ' * self.indent + f'_append({data})')

```

[slide]

```python
@staticmethod
def _is_control_statement(code):
    code = code.strip()
    control_flags = [
        lambda: code.startswith('if '),
        lambda: code.startswith('elif '),
        lambda: code.startswith('else '),
        lambda: code.startswith('try '),
        lambda: code.startswith('except '),
        lambda: code.startswith('finnaly '),
        lambda: code.startswith('with '),
        lambda: code.startswith('for '),
        lambda: code.startswith('while ')
    ]
    return any([is_control() for is_control in control_flags])
```

[slide]

```python
class _CodeGenerator:
    ...

    def handle(self, _type, _data):
        getattr(self, f'add_{_type}')(_data)

    def end(self):
        self.code.append('TMPL_EVAL_RESULT = "".join(_result)')

    def compile(self):
        raw_code = '\n'.join(self.code)

        if DEBUG_TEMPLATE:
            print('>>' * 10, 'start generated code', '<<' * 10)
            print(raw_code)
            print('>>' * 10, 'end generated code', '<<' * 10)

        return compile(raw_code, f'tmpl_{abs(id(self))}.py', 'exec')
```

[slide]

## 根据模板编译代码

```
@lru_cache(maxsize=128)
def _gen_code(self):
    code_gen = _CodeGenerator()

    [code_gen.handle(_type, _data)
     for _type, _data in Template._split_template(self.tmpl)]

    code_gen.end()

    return code_gen.compile()
``` 

[slide]

# Third Step
----
模板渲染

[slide]

## 模板渲染

```python
def render(self, jclass, eval_env={}):
    compiled_code = self._gen_code()

    env = {'jclass': jclass}
    env.update(BUILTIN_TMPL_EVAL_ENV)
    env.update(eval_env)

    exec(compiled_code, env)

    return env['TMPL_EVAL_RESULT']
```

[slide]

## 模板引擎支持的功能

- 代码块中支持任意的代码，包括多种控制结构
- 数据块支持函数调用
- 支持eval 环境，命名空间


[slide]

# check reuslt
----
## 模板编译结果

[slide]

# 模板的使用

```python
def test_template():
    class Obj:
        pass

    def jfield(jtype: str, name: str, *, modifier: str = '', init_value: str = '', comment: str = ''):
        field = Obj()
        field.modifier = modifier
        field.jtype = jtype
        field.name = name
        field.initial_value = init_value
        field.comment = comment

        return field

    data = Obj()
    data.name = 'ShareConfig'
    data.fields = []
    data.fields.append(jfield('Tencent', 'tencent', modifier='private final'))
    data.fields.append(
        jfield('IWXApi', 'wxApi', modifier='private final', comment='wechat share'))

    print(Template(TMPL_BUILDER).render(data))
```

[slide]

# 生成的代码

[slide]

```python
_result = []
_append = _result.append
_append("""
""")


fields = jclass.fields
_append("""
private """)
_append(jclass.name)
_append("""(Builder builder) {
""")


for field in fields:
    _append("""    this.""")
    _append(field.name)
    _append(""" = builder.""")
    _append(field.name)
    _append(""";
""")
_append("""}

public static class Builder {

""")
```

[slide]

```python

for field in fields:
    _append("""    private """)
    _append(field.jtype)
    _append(""" """)
    _append(field.name)
    _append(""" = """)
    _append(init_field(field))
    _append(""";
""")
_append("""
""")

```

[slide]

```

for field in fields:


    if field.comment:
        _append("""    /**
     * """)
        _append(field.comment)
        _append("""
     */
""")
    _append("""    public Builder set""")
    _append(field.name.title())
    _append("""(""")
    _append(field.jtype)
    _append(""" """)
    _append(field.name)
    _append(""") {
        this.""")
    _append(field.name)
    _append(""" = """)
    _append(field.name)
    _append(""";
        return this;
    }

""")
```

[slide]

```
_append("""    public """)
_append(jclass.name)
_append(""" build() {
        return new """)
_append(jclass.name)
_append("""(this);
    }
}
""")
TMPL_EVAL_RESULT = "".join(_result)
```


[slide]

# 模板生成的代码

[slide]

```
private ShareConfig(Builder builder) {
    this.tencent = builder.tencent;
    this.wxApi = builder.wxApi;
}

public static class Builder {

    private Tencent tencent = null;
    private IWXApi wxApi = null;

    public Builder setTencent(Tencent tencent) {
        this.tencent = tencent;
        return this;
    }

    /**
     * wechat share
     */
    public Builder setWxapi(IWXApi wxApi) {
        this.wxApi = wxApi;
        return this;
    }

    public ShareConfig build() {
        return new ShareConfig(this);
    }
}
```

[slide]

# Thanks
