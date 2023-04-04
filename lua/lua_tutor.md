# Lua Note

## 注释方案

```
-- 单行注释

--[[
	多行注释
--]]
```

## 数据类型

- nil
- boolean
- number
- string
- function
- userdata
- thread
- table

`nil` 可以删除table里面的变量

```
tab1 = { key1 = "val1", key2 = "val2", "val3" }
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end

tab1.key1 = nil
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end
```

结果：
```
1 - val3
key1 - val1
key2 - val2
1 - val3
key2 - val2
```

`string` 表示方案
```
string1 = "this is string1"

string2 = 'this is string2'

html = [[
<html>
<head></head>
<body>
    <a href="//www.w3cschool.cn/">w3cschoolW3Cschool教程</a>
</body>
</html>
]]
```

字符串连接使用 ..

`function`的使用类似python

通过`anonymous function`传参
```
function anonymous(tab, fun)
    for k, v in pairs(tab) do
        print(fun(k, v))
    end
end
tab = { key1 = "val1", key2 = "val2" }
anonymous(tab, function(key, val)
    return key .. " = " .. val
end)
```

`thread`

`userdata`


## lua循环处理
- while 循环处理
- for 
- Lua repeat..until
- 嵌套

```
while(condition)
do
	statements
end
```

**for循环**

- 数值for
- 泛型for

```
for var=exp1,exp2,exp3 do
    <执行体>
end
```

泛型for
```
--打印数组a的所有值
for i,v in ipairs(a)
    do print(v)
end
```

**repeat until**
```
repeat
   statements
until( condition )
```

## Lua流程控制
- if
- if..else
  
**if**
```
if(布尔表达式)
then
   --[ 在布尔表达式为 true 时执行的语句 --]
end
```

**if..else**
```
if(布尔表达式)
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end
```

**if..else if**
```
if( 布尔表达式 1)
then
   --[ 在布尔表达式 1 为 true 时执行该语句块 --]

else if( 布尔表达式 2)
   --[ 在布尔表达式 2 为 true 时执行该语句块 --]

else if( 布尔表达式 3)
   --[ 在布尔表达式 3 为 true 时执行该语句块 --]
else
   --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
end
```

## function

```
string.find("www.w3cschool.cn", "w3cschool")
```
返回匹配串”开始和结束的下标”（如果不存在匹配串返回 nil）

### 可变参数
Lua 函数可以接受可变数目的参数，和 C 语言类似在函数参数列表中使用三点（…) 表示函数有可变的参数。

Lua 将函数的参数放在一个叫 arg 的表中，#arg 表示传入参数的个数。

```
function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```



## Lua迭代器
能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址

在 Lua 中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素

**泛型for迭代**
```
--for 在内部保存迭代函数，实际保存了
--三个值：迭代函数、状态常量、控制变量
for k, v in pairs(t) do
    print(k, v)
end
```
- 首先，初始化，计算 in 后面表达式的值，表达式应该返回范性 for 需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用 nil 补足，多出部分会被忽略。
- 第二，将状态常量和控制变量作为参数调用迭代函数（注意：对于 for 结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
- 第三，将迭代函数返回的值赋给变量列表。
- 第四，如果返回的第一个值为 nil 循环结束，否则执行循环体。
- 第五，回到第二步再次调用迭代函数

### Lua迭代器分类
1. 无状态
2. 多状态

```
function square(iteratorMaxCount,currentNumber)
   if currentNumber<iteratorMaxCount
   then
      currentNumber = currentNumber+1
      print("currentNumber is "..tostring(currentNumber).."\n")
      print("iteratorMaxCount is "..tostring(iteratorMaxCount) .."\n")
      return currentNumber, currentNumber*currentNumber
   end
end

for i,n in square,3,0
do
   print(i,n)
end
```
```
currentNumber is 1

iteratorMaxCount is 3

1       1
currentNumber is 2

iteratorMaxCount is 3

2       4
currentNumber is 3

iteratorMaxCount is 3

3       9
```

比较经典的ipairs实现
```
function iter (a, i)
    i = i + 1
    local v = a[i]
    if v then
       return i, v
    end
end

function ipairs (a)
    return iter, a, 0
end
```
迭代函数 iter、状态常量 a、控制变量初始值 0；
```
array = {"Lua", "Tutorial"}

function elementIterator (collection)
   local index = 0
   local count = #collection
   -- 闭包函数
   return function ()
      index = index + 1
      if index <= count
      then
         --  返回迭代器的当前元素
         return collection[index]
      end
   end
end

for element in elementIterator(array)
do
   print(element)
end
```

## Lua tables
table 常用的操作

- table.concat (table [, step [, start [, end]]]): concat 是 concatenate(连锁, 连接)的缩写. table.concat()函数列出参数中指定 table 的数组部分从 start 位置到 end 位置的所有元素, 元素间以指定的分隔符(sep)隔开。
- table.insert (table, [pos,] value): 在 table 的数组部分指定位置(pos)插入值为 value 的一个元素. pos 参数可选, 默认为数组部分末尾.
- table.maxn (table): 指定 table 中所有正数 key 值中最大的 key 值. 如果不存在 key 值为正数的元素, 则返回 0。(Lua5.2 之后该方法已经不存在了,本文使用了自定义函数实现)
- table.remove (table [, pos]): 返回 table 数组部分位于 pos 位置的元素. 其后的元素会被前移. pos 参数可选, 默认为 table 长度, 即从最后一个元素删起。
- table.sort (table [, comp]): 对给定的 table 进行升序排序。

## lua 模块和包 & 加载机制
````
module = {}

-- constant
module.constant = "常量"

function moudle.func1()
   io.wirte("public function \n")
end

local function func2()
   io.write("private func \n")
end

function module.func3()
   func2()
end

return module
````

调用机制：
```
require("<模块名>")
require "<模块名>"

```

自定义加载路径：
```
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"


source ~/.profile
```

加载C库

```
local path = "/usr/local/lua/lib/libluasocket.so"
-- 或者 path = "C:\\windows\\luasocket.dll"，这是 Window 平台下
local f = assert(loadlib(path, "luaopen_socket"))
f()  -- 真正打开
```

## Metatable
- 多个table操作

**元表处理方案**
- setmetatable(table,metatable): 对指定 table 设置元表(metatable)，如果元表(metatable)中存在__metatable键值，setmetatable 会失败 。
- getmetatable(table): 返回对象的元表(metatable)

```
mytable = setmetatable({},{})
```

**__index 元方法**

对表访问

```
mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)
```

**__newindex 元方法**

对表更新

```
-- 计算表中最大值，table.maxn在Lua5.2以上版本中已无法使用
-- 自定义计算表中最大值函数 table_maxn
function table_maxn(t)
    local mn = 0
    for k, v in pairs(t) do
        if mn < k then
            mn = k
        end
    end
    return mn
end

-- 两表相加操作
mytable = setmetatable({ 1, 2, 3 }, {
  __add = function(mytable, newtable)
    for i = 1, table_maxn(newtable) do
      table.insert(mytable, table_maxn(mytable)+1,newtable[i])
    end
    return mytable
  end
})

secondtable = {4,5,6}

mytable = mytable + secondtable
	for k,v in ipairs(mytable) do
print(k,v)
end
```

__call 自定义元方法

__tostring 元方法 --修改表的输出行为

## 协同程序（coroutine）

类比线程

拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其它协同程序共享全局变量和其它大部分东西

**线程和协同程序区别**

一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。

在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

```

coroutine.create()

coroutine.resume()

coroutine.yield()

coroutine.status()

coroutine.wrap()

coroutine.running()
```

应用实例
```

function foo (a)
    print("foo 函数输出", a)
    return coroutine.yield(2 * a) -- 返回  2*a 的值
end

co = coroutine.create(function (a , b)
    print("第一次协同程序执行输出", a, b) -- co-body 1 10
    local r = foo(a + 1)

    print("第二次协同程序执行输出", r)
    local r, s = coroutine.yield(a + b, a - b)  -- a，b的值为第一次调用协同程序时传入

    print("第三次协同程序执行输出", r, s)
    return b, "结束协同程序"                   -- b的值为第二次调用协同程序时传入
end)

print("main", coroutine.resume(co, 1, 10)) -- true, 4
print("--分割线----")
print("main", coroutine.resume(co, "r")) -- true 11 -9
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- true 10 end
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- cannot resume dead coroutine
print("---分割线---")
```

结果：
```
第一次协同程序执行输出 1   10
foo 函数输出    2
main true    4
--分割线----
第二次协同程序执行输出   r
main true    11  -9
---分割线---
第三次协同程序执行输出  x   y
main true    10  结束协同程序
---分割线---
main false   cannot resume dead coroutine
---分割线---
```

### 生产者-消费者 问题
```
local newProductor

function productor()
     local i = 0
     while true do
          i = i + 1
          send(i)     -- 将生产的物品发送给消费者
     end
end

function consumer()
     while true do
          local i = receive()     -- 从生产者那里得到物品
          print(i)
     end
end

function receive()
     local status, value = coroutine.resume(newProductor)
     return value
end

function send(x)
     coroutine.yield(x)     -- x表示需要发送的值，值返回以后，就挂起该协同程序
end

-- 启动程序
newProductor = coroutine.create(productor)
consumer()
```

## Lua 文件 IO
- simple model
- complete model

**simple model**
```
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 设置默认输入文件为 test.lua
io.input(file)

-- 输出文件第一行
print(io.read())

-- 关闭打开的文件
io.close(file)

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 设置默认输出文件为 test.lua
io.output(file)

-- 在文件最后一行添加 Lua 注释
io.write("--  test.lua 文件末尾注释")

-- 关闭打开的文件
io.close(file)

io.tmpfile():返回一个临时文件句柄，该文件以更新模式打开，程序结束时自动删除

io.type(file): 检测obj是否一个可用的文件句柄

io.flush(): 向文件写入缓冲中的所有数据

io.lines(optional file name): 返回一个迭代函数,每次调用将获得文件中的一行内容,当到文件尾时，将返回nil,但不关闭文件
```

**complete model**
```
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 输出文件第一行
print(file:read())

-- 关闭打开的文件
file:close()

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 在文件最后一行添加 Lua 注释
file:write("--test")

-- 关闭打开的文件
file:close()
```

```
file:seek(optional whence, optional offset): 设置和获取当前文件位置,成功则返回最终的文件位置(按字节),失败则返回nil加错误信息。参数 whence 值可以是:
    "set": 从文件头开始
    "cur": 从当前位置开始[默认]
    "end": 从文件尾开始
    offset:默认为0
不带参数file:seek()则返回当前位置,file:seek("set")则定位到文件头,file:seek("end")则定位到文件尾并返回文件大小

file:flush(): 向文件写入缓冲中的所有数据

io.lines(optional file name): 打开指定的文件filename为读模式并返回一个迭代函数,每次调用将获得文件中的一行内容,当到文件尾时，将返回nil,并自动关闭文件。
若不带参数时io.lines() io.input():lines(); 读取默认输入设备的内容，但结束时不关闭文件,如

for line in io.lines("main.lua") do

　　print(line)

　　end
```

|  模式     | 描述       |
|---------  |--------    |      
| *n        | 取一个数字并返回它。例：file.read('*n')
  *a        |从当前位置读取整个文件。例：file.read("*a")
  *l(默认)  | 读取下一行，在文件尾 (EOF) 处返回 nil。例：file.read("*l")
  number    | 返回一个指定字符个数的字符串，或在 EOF 时返回 nil。例：file.read(5)


## Lua 异常处理

- assert
- error

```
error (message [, level])

Level=1[默认]：为调用error位置(文件+行号)
Level=2：指出哪个调用error的函数的函数
Level=0:不添加错误位置信息
```

pcall、xpcall、debug

pcall接收一个函数和要传递给后者的参数，并执行，执行结果：有错误、无错
误；返回值true或者或false, errorinfo。
```
if pcall(function_name, ….) then
-- 没有错误
else
-- 一些错误
end

```
pcall以一种”保护模式”来调用第一个参数，因此pcall可以捕获函数执行中的任何错误。

通常在错误发生时，希望落得更多的调试信息，而不只是发生错误的位置。但pcall返回时，它已经销毁了调用桟的部分内容。

Lua提供了xpcall函数，xpcall接收第二个参数——一个错误处理函数，当错误发生时，Lua会在调用桟展看（unwind）前调用错误处理函数，于是就可以在这个函数中使用debug库来获取关于错误的额外信息了。 debug库提供了两个通用的错误处理函数:

- debug.debug    : 提供 Lua 提示符，让用户来检查错误原因
- debug.traceback: 根据调用栈来扩建一个扩展的错误消息

```
function myfunction ()
   n = n/nil
end

function myerrorhandler( err )
   print( "ERROR:", err )
end

status = xpcall( myfunction, myerrorhandler )
print( status)
```
报错如下；
```
ERROR: test2.lua:2: attempt to perform arithmetic on global 'n' (a nil value)
false
```

## Lua垃圾回收

- collectgarbage("collect"): 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：
- collectgarbage("count"): 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。
- collectgarbage("restart"): 重启垃圾收集器的自动运行。
- collectgarbage("setpause"): 将 arg 设为收集器的 间歇率 （参见 §2.5）。 返回 间歇率 的前一个值。
- collectgarbage("setstepmul"): 返回 步进倍率 的前一个值。
- collectgarbage("step"): 单步运行垃圾收集器。 步长”大小”由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。
- collectgarbage("stop"): 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。


## Lua oop

```
-- Meta class
Rectangle = {area = 0, len = 0, breadth = 0}

-- 派生类方法 new
function Rectangle:new (o, len, breadth)
   o = o or {}
   setmetatable(o, self)
   self.__index = self
   self.len = len or 0
   self.breadth = breadth or 0
   self.area = len * breadth
end

function Rectangle:printArea()

   print("Area is ", self.area)
end


r = Rectangle:new(nil,10,20)
```

访问属性：
`print(r.len)`

访问方法：
`r:printArea()`

#### Lua 继承
```
 -- Meta class
Shape = {area = 0}
-- 基础类方法 new
function Shape:new (o,side)
  o = o or {}
  setmetatable(o, self)
  self.__index = self
  side = side or 0
  self.area = side*side;
  return o
end
-- 基础类方法 printArea
function Shape:printArea ()
  print("面积为 ",self.area)
end

接下来的实例，Square 对象继承了 Shape 类:

Square = Shape:new()
-- Derived class method new
function Square:new (o,side)
  o = o or Shape:new(o,side)
  setmetatable(o, self)
  self.__index = self
  return o
end
```
