## OpenWrt的使用
### lua语言
Lua作为一门方便嵌入(其他应用程序)并可扩展的轻量级脚本语言来设计的，因此它一直遵从着简单、小巧、可移植、快速的原则，官方实现完全采用ANSI C编写，能以C程序库的形式嵌入到宿主程序中。Lua的每个版本都保持着开放源码的传统，不过各版采用的许可协议并不相同，自5.0版(最新版是5.1)开始它采用了著名的MIT许可协议。正由于上述特点，所以Lua在游戏开发、机器人控制、分布式应用、图像处理、生物信息学等各种各样的领域中得到越来越广泛的应用。
**Luci** 是Lua COnfigurationInterface的简称，意在OpenWrt整个系统的配置集中化。

### uHttpd
uhttpd是一个简单的web服务器程序，主要就是cgi的处理，openwrt是利用uhttpd作为web服务器，实现客户端web页面配置功能。对于request处理方式，采用的是cgi，而所有的cgi程序就是luci.


### luci cbi模板
/etc/config/ipsec配置文件如下:
```
config policy tunnel
    option name 'test'
    option enable '1'
```
cbi模板代码如下:
```
-- 1.创建uci配置文件ipsec对应的map对象
m = Map("ipsec", translate('IPSec'))
-- 2.创建uci type对象
s = m:section(TypedSection, 'policy', translate('Policy'))
s.template = 'cbi/tblsection'  --使用列表模板
s.anonymous = true     --不显示section名称
-- 3.创建uci name对象
s = m:section(NamedSection, 'tunnel', 'policy', 'translate('Policy')')
-- 4.创建uci option对象
name = s:option(DummyValue,'name',translate('Name'))
enable = s:option(Flag, 'enable', translate('Enable'))
enable.rmempty = false   --值为空时不删除
```
<h4>说明</h4>
#### 1.Map类
```
class Map(config, title, description)
```
参数说明：<br/>
config:/etc/config/目录下的uci文件名<br/>
title:页面显示名称<br/>
description:页面显示详细描述<br/>

#### 2.   :section(sessioncalss, ...)方法
<h5>2.1 class NameSection(name, type, title, description)</h5>
参数说明<br/>
name:uci Section名字 ---config type ```section```<br/>
type:uci section类型  ---config ```type``` section <br/>
title：页面显示名称<br/>
description:页面显示详情描述<br/>

对象属性<br/>
```
.addremove = false //此section是否允许删除或创建，为true时，页面会显示删除和创建按钮
.anonymous = true  //页面不显示此section名字
```

方法说明<br/>
```
:option(optionclass, ...)<br/>
```

<h5>2.2 class TypedSection(type, title, description)</h5>
参数说明<br/>
type:uci section类型 ---config ```type``` section<br/>
title:页面显示名称<br/>
description:页面显示详细描述<br/>

对象属性<br/>
```
.addremove = false //此section是否允许删除或创建，为true时，页面会显示删除和创建按钮
.anonymous = true  //页面不显示此secion名字
.extedit = luci.dspatcher.build_url("url") //设置此section编辑页面URL, 页面显示编辑按钮<br/>
.template = "cbi/template"  //设置此section页面模板
```

方法说明
```
:option(optionclass, ...)
```
#### 3.class Value(option,title,description)输入框
参数说明<br/>
option:uci option名称， --option enable '1'<br/>
title:页面显示名称<br/>
description:页面显示详情描述<br/>

属性说明<br/>
```
.defalut = nil  //缺省值
.maxlength = nil  //option值最大长度
.rmempty = true  //option值为空时，不写入uci文件
.size = nil  //页面表单对应位置大小
.optional = false  //是否为可选，为true可强制页面输入
.datatype = nil  //指定option值类型，用于输入合法性检查
.template = nil   //页面模板
.password = false  //密码输入框
```

方法说明:
```
:depends(key, value)  当key值等于value时，页面才显示
```
**其他**<br/>
function o.validate(self, value)<br/>
end<br/>
重构option合法性检查方法，可自己控制option值<br/>
<br/>
function o.formvalue(self, key)<br/>
end<br/>
重构option从表单获取值方法
```
当表单值为空时返回-，在合法性检验时判断如果value等于-即表示页面没有输入值
function o.formvalue(...)
    return Value.formvalue(...) or "-"
end
function o.validate(self, value)
    if value == "-" then
        return nil, translate("required fields have no value!")
    end
    return value
end
```
functin o.write(self, section, value)<br/>
end<br/>
重构option写入UCI文件方法
```
在用户输入的值后追加内容s:
function o.write(self, section, value)
    Value.write(self, section, value .. "s")
end
```
function o.cfgvalue(self, section)<br/>
end<br/>
重构option值输出到页面方法
```
页面只显示UCI option值开始为数字部分内容:
function o.cfgvalue(self, section)
    local v = Value.cfgvalue(self, section)
    if v then
        return string.sub(v, string.find(v, "%d+"))
    else
        return nil
    end
end
```

#### 4.class ListValue(option, title, description)列表框
参数说明:<br/>
option: UCI option名称, option action 'drop'<br/>
title: 页面显示名称<br/>
description: 页面显示详细描述<br/>

属性说明:<br/>
同class Value<br/>

方法说明:<br/>
同class Value<br/>

**:value(ucivalue, showkey)**<br/>
设置下拉列表显示与值对应关系<br/>
```
o = s:option(ListValue, "enable", translate("Enable"))
o:value("1", translate("Enable"))
o:value("0", translate("Disable"))
```

##### 5.class Flag(option, title, description)单选框
参数说明:<br/>
option: UCI option名称, option enable '0'<br/>
title: 页面显示名称<br/>
description: 页面显示详细描述<br/>

属性说明:<br/>
同class Value<br/>

方法说明:<br/>
同class Value<br/>

##### 6.class Button(option, title, description)按钮
参数说明:<br/>
option: UCI option名称, option enable '0'<br/>
title: 页面显示名称<br/>
description: 页面显示详细描述<br/>

属性说明:<br/>
同class Value<br/>
```
.inputstyle = nil
按钮样式, apply, reset,
```
方法说明:<br/>
同class Value<br/>

### luci htm语法
##### 1.<% %>引用lua语言
1. 第一种是<% %>，这个一般在开头，里面完全使用lua语句，在在这里一般是用来获取系统的基本信息之类的。
2. 第二种是<%=ssid%>，这个是获取lua变量的ssid值。
3. 第三种是<%- %><% -%>这种拆分使用lua语言的，也就是在两个lua语句之间直接嵌入html
4. 第四种是<%:%>这个用来显示国际化

##### 2.luci.http.formvalue
获取post过来的表单

### lua 基本函数方法
**1. ipairs (t)**<br/>
功能：返回三个值 next函数、表、0<br/>
多用于穷举表的键名和键值对,如：

```
for n,v in pairs(t) do

end
每次循环将索引赋给n，键值赋给v
```
注：本函数从下标1开始，遍历表格式如：t={"1","cash"}

**2. pairs (t)** <br/>
多用于穷举表的键名和键值对

```
for i,v in pairs(t) do

end
每次循环将索引赋给i，键值赋给v
```
注： 本函数遍历表格式可以包含键值对如：t={"h",id="1",name="cash"}

**3. tonumber (e [, base])**<br/>
功能：尝试将参数e转换为数字，当不能转换时返回nil<br/>
base(2~36)指出参数e当前使用的进制，默认为10进制，如tonumber(11,2)=3

**4. unpack (list [, i [, j]])**<br/>
功能：返回指定表的索引的值,i为起始索引，j为结束索引

**5. setmetatable (table, metatable)**<br/>
功能：为指定的table设置元表metatable，如果metatable为nil则取消table的元表，当metatable有__metatable字段时，将触发错误<br/>
注：只能为LUA_TTABLE 表类型指定元表

**6. setfenv (f, table)**<br/>
功能：设置函数f的环境表为table<br/>
参数：f可以为函数或调用栈的级别，级别1[默认]为当前的函数,级别0将设置当前线程的环境表

**7. getfenv(f)**<br/>
功能：返回函数f的当前环境表<br/>
参数：f可以为函数或调用栈的级别，级别1[默认]为当前的函数,级别0或其它值将返回全局环境_G
