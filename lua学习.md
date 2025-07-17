## lua

#### 生成：

项目中如果带有lua插件那么在哪里使用的话在对应的蓝图界面点击蓝图模板，然后就会自动在content中生成lua相关的文件

在construct中可以编写一些构造函数相关数据，比如print('hello lua');的时候并不能直接在控制台打印，还需要在对应蓝图的class Setting 中创建接口unluainterface，添加接口之后，其中填上对应的lua文件名称即可在控制台打印hello lua  

能访问和拓展哪些内容：

lua访问权限和蓝图访问权限是一样的。

c++端声明为uproperty中声明为blueprinteditable或者是ufunction中blueprintcallable的函数。同时也可以访问蓝图中的内容。他能拓展的内容和蓝图能够拓展的内容是一样的。





Lua闭包概念：从外界捕获的指能够持久保持在闭包对象中，即使已经离开闭包的声明周期，依然能对其中的变量进行调用和处理



## 任务：

- [ ] 任务1：了解lua通用语法
- [ ] 任务2：unlua框架怎么和虚幻交互的
- [ ] 任务3：了解unlua的元表等特殊机制
- [ ] 任务4：学习UI中的lua的操作语法以及项目组自制的事件

### 数据类型

nil number(浮点和整数都有) bool  string  table userdata thread

陌生的：

`userdata` 可以保存任意C数据（对应一块原始内存）

#### userdata:

​	1. 定义：本质是一个储存着C++对象的指针（通常是void*) 通过这个指针能够在运行时访问和操作对应的C++对象。

2. Metatable 绑定：每一个userdata都关联一个元表，元表中定义了：
   1. 方法映射：将C++类中的UFunction暴露为Lua可调用的参数。
   2. 属性访问：通过`__index`和`__newindex`元方法实现对C++`Uproperty`的读写
   3. 生命周期：通过`__gc`元方法在Lua对象被垃圾回收的时候释放资源

疑问： 所以C++反射到lua也是通过宏定义进行处理的？是通过UHT嘛？还是Lua自己的反射处理系统？

- lua也是借助UE的UHT生成的反射代码（.genrated.h）然后借助反射信息，将C++类型注册到Lua环境中。
- 具体的流程就是，Lua环境初始化的时候unlua会遍历所有标记为可反射的C++类，并为每个类生成对应的Lua元表（应该是存储到对应的同名的`.lua`文件中？） 当调用C++方法的时候unlua会通过反射系统动态调用对应的函数。

----------------------------------------------------

`thread` 用于实现协同程序的独立执行线程

`table` 数组，可以保存除了nil之外的任意类型的数组



### 循环语句三个

for while repeat

- for循环 for i=1,5,2 do  

​		就是for循环中i为下表，初始值为1， 第三个参数为步长，步长为2 ，后面的步长可以省略，省略之后步长默认为1. 如果需要**递减**，那步长就是负数

```lua
   myTable ={'heroes',4,3,4,5}
    print("LymTable:",myTable[1])
    table.insert(myTable,5)
    for i=1,5,2 do
        print("LymTable:",myTable[i])
    end

打印信息：
LogUnLua: LymTable:   heroes
LogUnLua: LymTable:   heroes   --第一个元素
LogUnLua: LymTable:   3    --第3个元素
LogUnLua: LymTable:   5  	--第5个元素
```

- while

  ```lua
      myTable ={'heroes',4,3,4,5}
      table.insert(myTable,5)
      local i=0;
      while i~=6 do
          i=i+1 --先进行加1操作
          print("LymTable:",myTable[i])
      end
  输出结果：
  LogUnLua: LymTable:   heroes
  LogUnLua: LymTable:   4
  LogUnLua: LymTable:   3
  LogUnLua: LymTable:   4
  LogUnLua: LymTable:   5
  LogUnLua: LymTable:   5
  ```

  

- repeat:

```lua
    myTable ={'heroes',4,3,4,5}
    table.insert(myTable,5)
local i=0;
    repeat
        i=i+1
        print("Repeat LymTable:",myTable[i])
    until i==6

输出结果：
LogUnLua: Repeat LymTable:    heroes
LogUnLua: Repeat LymTable:    4
LogUnLua: Repeat LymTable:    3
LogUnLua: Repeat LymTable:    4
LogUnLua: Repeat LymTable:    5
LogUnLua: Repeat LymTable:    5
```





### 连接语法：

..用于连接字符串。



### 语法规则：

允许多重赋值

```lua
myTable ={'heroes',4,3,4,5}
a,b,v=myTable[1],myTable[4]
print(a)
print(b)
print(v)

输出：
LogUnLua: heroes
LogUnLua: 4
LogUnLua: nil
```

可以看到可以多重的赋值，并且当没有对v赋值的时候，其为nil

#为长度运算符，可以看表的长度

```lua
myTable ={'heroes',4,3,4,5}
    print("myTable的长度： "..#myTable)
输出：
LogUnLua: myTable的长度： 5
```

其他运算符均与C++相同+ - * / %



```lua
 m=3.0 
 n=2
 print("m/n： "..m/n.."  m%n:"..m%n)
LogUnLua: m/n： 1.5  m%n:1.0

    m=4
    n=2.0
    print("m/n： "..m/n.."  m%n:"..m%n)
LogUnLua: m/n： 2.0  m%n:0.0

    m=4
    n=2
    print("m/n： "..m/n.."  m%n:"..m%n)
LogUnLua: m/n： 2.0  m%n:0
```

..的用法

```lua
    function return_sth(sss)
        local ss=666+sss
        return ss
    end
    function return_sth2(sss)
        local ff=tostring(888)..tostring(sss)
        return ff
    end
    PrintScreen(return_sth(999))
    PrintScreen(return_sth2(999))

-- ..只能用在两边都是string类型的情况下，不然会报错
```





#### 覆盖蓝图中的事件、函数：

只需要在蓝图事件前面添加Receive即可.例如：`function M: ReceiveBeginPlay() end `

覆盖蓝图中的函数的话是这样：

```lua
function M:SayHi(name)
	self.Overridden.SayHi(self,name)
end

---------------------具体示例如下：
local M = Unlua.Class()

function M:ReceiveBeginPlay()
    local msg = self:SayHi("陌生人")
    print(msg)
end


```



self对函数和属性的访问：

 ✅ `self:` 用于**调用方法（带 self）**
 ✅ `self.` 用于**访问字段或函数对象（不自动传 self）**



#### pairs迭代器

```lua
function Base:test1()
    -- 初始化嵌套表
    self.name[1] = {}
    self.name[1][2] = "lym"
    self.name[1][3] = "zhao"
    
    -- 使用pairs迭代器
    for i, value in pairs(self.name) do
        -- 使用print替代未定义的g_ScreenPrint
        print("name[" .. i .. "]: " .. table.concat(value, ", "))
    end
end
```

看不懂value是什么，原来是 for循环中能够循环键值对，所以key value都出来了。

```lua
   for i, value in pairs(self.name) do
            g_ScreenPrint("name[" .. i .. "]: " .." value: "..self.name[value])
        end
```

这种写法是错误的，如果要输出value的话应该直接输出value,而不是self.name[value],这不是数组，，，

那像根据键输出值呢？这能够像哈希一样输出嘛？

是的，table也可以视作哈希结构，查找数据的时间复杂度是O（1）

```lua

 Base.name={}
 Base.message={}
 Base.shuzu={{1,"dd"},{2,"asd"},{3,"sda"}}
    function Base:test1()
        for i = 1,#Base.shuzu do
            g_ScreenPrint("第"..i.."个元素：")
            for k,value in  pairs(self.shuzu) do
                g_ScreenPrint(k ,value)
            end
        end
    end
```

这个为什么打印不出来信息？

嵌套循环记得输出嵌套结构，这样才是对的

用两遍pairs

```lua
Base.shuzu={{1,"dd"},{2,"asd"},{3,"sda"}}
function Base:test1()
	for i,v in ipairs(self.shuzu) do
            g_ScreenPrint("第"..i.."个元素：")
            for k,value in  ipairs(v) do
                g_ScreenPrint(" value: "..value)
            end
        end
      
    end
```

##### ipairs和pairs有什么区别？

ipairs是遍历数组，pairs是遍历键值对。

- ipairs是顺序遍历，pairs是随机遍历

```lua
local t = {
    name = "Alice",
    age = 25,
    [1] = "one"
}
for key, value in pairs(t) do
    print(key, value)
end
--实际输出顺序真的是随机的
```

- ipairs 遇到nil的时候就会停止。而pairs`pairs`会遍历所有的键值对

   对于混合表  pairs会先打印数组，然后打印键值对

```lua
print("-------------------")
local mixtable={ss="feiji",hh="火箭",12,"ry",34,"hero",name="lym"}

for i, v in ipairs(mixtable) do
    print("i:"..i.." v:"..v)
end   
print("-------------------")

for k, v in pairs(mixtable) do
    print("k:"..k.." v:"..v)
end   
```

![image-20250715172436166](C:\Users\yinming.li\AppData\Roaming\Typora\typora-user-images\image-20250715172436166.png)

Lua 的`pairs`遍历顺序是由表的内部结构决定的：先遍历连续整数索引的数组部分（按索引升序），再遍历哈希部分（顺序不确定）。这种设计是为了平衡性能和实现复杂度。

数组部分的连续内存访问比哈希表更快，优先遍历数组部分可以提高常见场景（如纯数组）的遍历效率。



### 元表：

特殊的表，作用：改变其他表的默认行为，比如增加规则，比较规则，索引查找规则等。

实现面向对象的特性。继承、多态 以及自定义操作语法（运算符重载？）

元表本质也是普通表

```lua
local myTable={"hero",1,2,3}
local meta={
    __tostring =function(t) --自定义打印行为
        local str="table content:"
        for _,v in ipairs(t) do
            str=str..tostring(v).." "
            end
        return str
        end
}
```

使用setmetatable之后，返回的表就是元表，之前的myTable是被元表影响行为的”目标表“

### 那元表在虚幻中有什么应用场景呢

在ue绑定蓝图之后申城的模板文件中 使用unlua.Class()创建类时，unlua会自动生成元表，用于模拟UE蓝图类的继承，事件重写

![元表与引擎关联](C:\Users\yinming.li\Desktop\MD\Snipaste\元表与引擎关联.png)

就是说元表中可以有属性，可以有函数操控ue蓝图或者C++中的数据或者对象

还有一点，能够响应引擎事件，AI给的例子是initialize的时候，**unlua底层是通过元表拦截实现的。什么意思？拦截是怎么实现的呢？**

**那元表有面向对象的功能吗可以继承和实现多态？**

#### **unlua.Class()做了什么呢？**

给的答案是做了以下行为：

```lua
简化模拟unlua.Class()的核心逻辑
local function Unlua_Class(baseClass){
        local classTable={}
        local metatable={}
        setmetatable(classTable,metatable)
        metatable.__index = baseClass
        metatable.__ue_events = { }
        return ClassTable
    }
    
    local M= Unlua_Class(UE4.AActor)
    function M:ReceiveBeginPlay()
        UE4.Log("actor开始播放了")
        end
    
    这代码能对吗？UE4.Log()有这个函数和对象吗？
```

#### **疑问：下划线这又是什么东西？**

总结：元表功能：配置文件+行为拦截器

unlua.Class()：相当于自动化工厂，自动关联UE专有元表+继承引擎能力

一句话就是，unlua.Class（）通过元表机制，让lua类接管UE事件，实现自定义逻辑



### 依次解决上面的问题：

#### 1.双下划线开头的属性或者函数是干什么的？

特殊元方法，定义表的默认行为。当表中执行特定操作的时候lua会自动查找元表中原方法并执行。常见的元方法

| 元方法       | 触发场景                       | 用途示例                         |
| ------------ | ------------------------------ | -------------------------------- |
| `__index`    | 访问表中不存在的键时           | 实现继承（找不到键时去父表查找） |
| `__newindex` | 修改表中不存在的键时           | 拦截写操作（如只读表、数据验证） |
| `__add`      | 表作为加法操作的操作数时       | 自定义表相加逻辑（如向量加法）   |
| `__tostring` | 表被转为字符串时（如 `print`） | 自定义表的打印格式               |
| `__call`     | 表作为函数调用时（如 `t()`）   | 让表可调用（如实现工厂模式）     |

看不懂这些元方法的地位是什么，是相当于封装的函数吗？

这些特殊的元方法在Unlua中定义的，我在项目中看到的比较少，可以先不用关心

#### 2.initialize怎么进行拦截的？

unlua重写initialize的时候底层大致流程如下：

注册事件：在创建类的时候会在元表中注册_ue_event_bind元方法

引擎调用：引擎触发initialize事件

元表拦截：通过元表中的_call或者 _index元方法捕获调用

转发到你的函数：查找并执行Lua中定义的initalize方法

#### 3.元表如何实现继承和多态的

##### 继承： 通过_index链实现多层继承 

```lua
local Base = { x= 10 }
local Sub = ({},__index=Base)
local Child = ({},__index=Sub)
print(Child.x)
```

通过`__index`变相实现继承。因为`__index`意味着如果找不到就去指向的元表中寻找

```lua

--基类：
Base={}
Base.__index= Base
function Base:Hello()
print("call  hello fuinction")
end
--以上写法的意思：Base.__index=Base，找不到的时候在自身找

--子类
Derived = setmetatable({},{__index = Base})-- 继承Base
Derived.__index = Derived
--以上含义： 创建了空表 ，表示我们的类Derived
-- 设置了他的元表，其中 __index = Base 意味着：问Derived.somekey的时候，如果在Derived中找不到，则会去Base中寻找


function Derived:Greet()
    print("Hello from Derived")
end
```



##### 多态：

```lua
不用声明为virtual,直接重写就行

local Animal={
    speak = function()return "..." end
}
local Dog= {
    Dog.speak = function() return "woof woof" end
}
local Cat ={
    Cat.speak = function()return "meow~~" end 
}
print(Dog.speak)
print(Cat.speak)
print(Animal.speak)
```

##### 封装

创建函数不需要声明返回值类型，直接在函数中创建局部变量继续计算，计算之后如果是想要的结果直接返回就行。如下：

```lua
local M=Unlua.Class()
function M:receiveBeginPlay(){
        --我想封装一个函数ScreenPrint直接在屏幕上输出
    function ScreenPrint(msg)
       local UE.FLinearColor(0,1,0,1)
       UE.UKismetSysteamLibrary.PrintString(self,msg,true,true,color,5.0)
    end
        
    function return_sth(sss)
     xx=xx+sss
     return xx
     end
        
    ScreenPrint(return_sth)
    }
```

> 可以看到这里有点问题的，Function 创建函数的时候最好是带上M:这样做的是该类的函数，而不加的话则是全局函数，。
>
> 通常是不推荐全局函数的，最重要的就是会**<u>在导入其他lua文件的时候可能会导致命名冲突</u>**  ，不易维护。
>
> 所以最推荐的做法就是将通用类写在一个GlobalFunctions.lua ,当其他lua文件需要某些函数的时候可以导入使用，

使用方式:

```lua
local Global =require("GlobalFunctions")
function M:ReceiveBeginPlay()
    local msg = Global.SayHi("陌生人")
    print(msg)
 end
```

使用require是最推荐的维护方式。





#### 5.屏幕打印字符的时候导致引擎崩溃

```lua
function M:Initialize(Initializer)
    local msg = "Hello from UnLua!"

    -- 输出到输出窗口
    print(msg)

    -- 打印到屏幕左上角
    local color = UE.FLinearColor(0, 1, 0, 1)  -- 绿色
    UE.UKismetSystemLibrary.PrintString(self, msg, true, true, color, 5.0)
 end
    
----原因：initialize的时候Actor并不是完全初始化的时候，所以此时访问引擎函数会造成崩溃，也就是可以理解为我还没有学会说话呢，就去联合国演讲，很崩溃


所以以上代码放在EventBeginplay里面没有任何问题：

function M:ReceiveBeginPlay()
    local msg = "打印成功"

    -- 输出到输出窗口
    print(msg)

    -- 打印到屏幕左上角
    local color = UE.FLinearColor(0, 1, 0, 1)  -- 绿色
    UE.UKismetSystemLibrary.PrintString(self, msg, true, true, color, 5.0)

end
```



#### 问题6：rider智能显示配额不足。多次修改配置不生效，解决方案：换IDE使用VSCode.

在VSCode中将intelliSence添加到工作区就能够启动智能提示了。

![添加到工作区](C:\Users\yinming.li\Desktop\MD\Snipaste\添加到工作区.png)





实现成功在场景中控制Actor中的组件旋转：

```lua
function M:PrintScreen(msg)
    local color=UE.FLinearColor(0,1,0,1)
    UE.UKismetSystemLibrary.PrintString(self,msg,true,true,color,5.0)
end

function M:ReceiveBeginPlay()
    self:PrintScreen("成功")
        -- 初始化Cube组件引用
        if self.Cube then
            self:PrintScreen("成功获取Cube组件"..tostring(self.Cube:K2_GetComponentRotation()))
        else
            self:PrintScreen("错误: 未找到Cube组件")
        end
end
function M:ReceiveTick(DeltaSeconds)
    if not self.Cube then return end
    local RotateSpeed = 90
    local currentRotation = self.Cube:K2_GetComponentRotation()
    if currentRotation then
     
       local newPitch = currentRotation.Pitch+RotateSpeed*DeltaSeconds
       local newRotation = UE.FRotator(newPitch, currentRotation.Yaw, currentRotation.Roll)
       self.Cube:K2_SetRelativeRotation(newRotation,false,nil,true)
    end
end
```

重点：很多情况下都是 函数 is 农田callable 注意，如果不是C++中的函数的话，就要检查是不是KismitSystemlibrary中的函数，如果是的，**调用节点函数记得加上`K2_`**





#### 问题7：unlua框架如何与UE交互的

**Lua 可以看作是蓝图类的子类，或者控制脚本的“外接主脑”**。





#### 问题8：蓝图中的一个函数，lua中重写

```lua
function M:changeSomething()
    self.Overridden.changeSomething(self) -- 重写，有这一行的时候，会继承蓝图中的逻辑如果没有则是完全重写方法
    self:PrintScreen("重写成功")
end
```

发现lua中的逻辑和蓝图中的逻辑都执行了.  -- 对的。 如注释所言



### unlua重写机制

#### 基本原理：

- 函数绑定 ：允许通过unlua脚本覆盖蓝图或者C++中的函数， 仅需lua中函数名相同。
- **动态调度**：当蓝图中调用该函数的时候，如果lua中重写了这个函数，则会执行lua中的逻辑。而不是蓝图函数中的逻辑
- Overriden成员： unlua提供的特殊成员，允许访问被重写的原始逻辑

#### 常用作用：

1.调试和打日志，在不修改蓝图的情况下，可以Overriden之后打印需要的字符串或log.

2.拓展功能，在不更改原有蓝图的逻辑基础上进行其他功能扩写，增强蓝图函数工程

```lua
function M:BeginPlay()
    self.Overridden.BeginPlay(self)
    -- 添加额外初始化逻辑
    self:InitLuaComponents()
end
```

3.条件执行，可以灵活运行某些条件来判断是否要Overriden原始的逻辑，还是重新写该函数的逻辑：

```lua
function M:fireWeapon()
    if self.CanFire then 
        return self.Overridden.FireWeapon(self)
    end
    print("武器冷却中，无法使用")
end

```



#### 注意：

- 在重写的时候要使用Overridden必须显示传递self,否则会报错

- 函数签名匹配，lua参数和返回值必须和蓝图或者C艹一致。



函数相互调用：

Unlua调用蓝图函数：就是同名函数直接自动绑定了，如果调用的是静态函数库中的函数，也可以调用比如`K2_`和引擎中的函数

蓝图中调用Unlua函数：有同名的话就是一样的，直接执行unlua中的逻辑，如果蓝图中调用unlua独特的函数呢？一般不会这样吧，，，lua都是脚本语言了。  但是呢也简单，蓝图中创建一个helua中相同的函数不就行了哈哈哈哈哈哈

unlua调用C++函数：

​	暂时不知道



2025年7月11日总结：

元表实现 封装继承多态，unlua和UE的交互：调用相关函数的方式以及和蓝图的相互交互，和蓝图交互时的继承关系

### 





项目中lua的加载消息的模块：

```lua
pb = require("pb")
serpent = require("serpent")

-- 解码时构造默认嵌套消息
pb.option("decode_default_message", true)
```

### pb和seroent是什么呢？

protocol buffers 库（协议缓冲区），用于处理protobuf格式的消息编解码 。其是一种和语言平台无关可拓展的结构化数据序列化机制。可类比与XML但是比起更小更快更简单。只需定义一次数据的结构化方式，就能够使用特定生成的源代码将结构化数据写入各种数据流，以及从这些数据流中读取数据。而且还支持不同语言的操作

在和Unreal Engine中的应用场景：

- 网络通信：客户端和服务器通过protobuf消息互通
- 数据存储：序列化游戏状态、配置等数据
- 跨模块交流：C++和lua之间通过protobuf消息传递复杂数据结构

示例：

```lua
一， 准备阶段
1.编写.proto文件定义消息结构
syntax = "proto3";  -- 这是啥？
package Game ;
message PlayerInfo {
	string name = 1 ;
	int32 level =2 ;
	repeated int32 items =3 ;
}
2. 使用protobuf编译器生成Lua代码
二，Lua中使用
local pb = require ("Game_pb")  -- 项目中在 GameInstance.lua中已经全局处理了。
local player = pb.PlayerInfo() -- 这就是加载.protobuf中的结构了吧
player.name = "张三"   -- 定义结构信息
player.level = 10
player.items = {1, 2, 30}

编码为二进制数据

local buffer = player.PlayerInfo:SerializeToString()

--解码（自动生成默认字段）
pb.option("decode_default_message",true)
local newPlayer = pb.PlayerInfo():ParseFromString(buffer)


```

在项目中的protobuf buffer的操作：

```lua
-- 先加载消息
pb = require("pb")
serpent = require("serpent")

-- 解码时构造默认嵌套消息，设置解码选项，当收到的消息缺少某些字段时，自动创建默认的嵌套消息结构，避免出现nil错误。
pb.option("decode_default_message", true)
-- serpent打印时的参数设置
local g_serpentoptions = { comment = false, compact = true, numformat = "%.0f" }

-- 加载消息pb，pb.loadfile函数内没有做打包后的路径转换
local pbdata = UE.FFileHelper.LoadFileToArray(UE.UBlueprintPathsLibrary.ProjectContentDir() .. "Script/MODS/MsgPr.oto.desc")
if pbdata then
    local success, err = pb.load(pbdata)
    if success then
        UnLua.Log("消息desc文件加载成功")
    else
        UnLua.LogError("消息desc文件加载失败:", err)
    end
else
    UnLua.LogError("消息desc文件读取失败")
end
```

解释：

疑问：pb.option是不用定义消息结构嘛? 直接创建对应的消息结构，避免出现nil错误? 

- 解释：不是的，他不会自动生成消息结构，他仅仅处理已经定义，但是没有在消息中出现的嵌套消息字段。比如：

- ```lua
  message Player
  {
  string name = 1; -- 这个是基本消息类型
  Position pos = 2; --这个是嵌套消息类型
  }
  --所以用代码解释上面的话就是 name 缺失的时候会报错 nil pos字段缺失的时候，会自动创建为{}，也就是自动创建空表，而不是包含默认值的表，如果需要默认值需要在.proto文件中定义
  
  
  --所以起初理解是错误的，其不能代替消息结构定义，消息结构仍需在.desc文件或者其他方式注册
  
  ```

  pb.option总结：

  - 自动补全缺失的嵌套结构
  - 不能够代替消息结构定义



`local pbdata.... `这个是读取并加载Protobuf描述文件（.desc)并将其加载到Protobuf库中

desc文件是什么？ ：protobuf编译器生成的二进制文件，包括消息结构的元数据，用于运行时动态解析消息



#### 为什么这么做？

1. 动态解析Protobuf消息 。虽然我们开发中经常使用Protobuf格式，但是lua并不支持Protobuf格式。需要通过库实现编解码，通过加载.desc文件Lua可以在运行时动态解析Protobuf消息
2. 在Gameintance中写，是为了一次加载，整个生命周期都能够使用消息定义



### serpent 

serpent : serpent库，用于将Lua表转换为字符串。类似JSON序列化，但是更灵活



## unlua的垃圾回收机制

> unLua和UE相互传递的变量是如何防止被GC的？



### unlua和C++交互

通过`UE. `后面加上对应的类名然后就能够访问其中的变量和函数

### unlua闭包



### unlua构造顺序

initaialize preconstruct construct 以及。。。每个阶段的主要功能是什么？



```lua
function g_MsgEvent:PushEvent(OwnerActor,Id,Data, ...)
    if OwnerActor then
        if self.OwnerListeners[Id] and self.OwnerListeners[Id][OwnerActor] then
            local Listener = self.OwnerListeners[Id][OwnerActor]
            if Listener and Listener:Count() > 0 then
                for Ref,Func in pairs(Listener:Map()) do
                    Func(Ref,Data, ...)
                end
            end
        end
    else 
        local Listener = self.EventListeners[Id]
        if Listener and Listener:Count() > 0 then
            for Ref,Func in pairs(Listener:Map()) do
                Func(Ref,Data, ...)
            end
        end
    end
end

```

`...`是变长参数的意思.

```lua
g_MsgEvent = {}
g_MsgEvent.EventListeners = {}
g_MsgEvent.OwnerListeners = {}
g_MsgEvent.UEMsgHandle = {}
g_MsgEvent.DBMsgHandle = {}

--OwnerActor 是该事件Id的所属关联者
function g_MsgEvent:AddEventListener(Ref,OwnerActor,Id,Func)
    if OwnerActor then
        --起初不是很理解这段双中括号。因为并不是很理解table，其实是
        self.OwnerListeners[Id] = self.OwnerListeners[Id] or {}
        self.OwnerListeners[Id][OwnerActor] = self.OwnerListeners[Id][OwnerActor] or g_MapCount.New()
        self.OwnerListeners[Id][OwnerActor]:Add(Ref,Func)
    else 
        self.EventListeners[Id] = self.EventListeners[Id] or g_MapCount.New()
        self.EventListeners[Id]:Add(Ref,Func)
    end
end

```

这里是Lua中的委托的概念



PushEvent就是相当于触发了监听器呗，相当于一个开关，掉用之后，如果绑定了回调函数，就执行回调函数中的逻辑

###### ChatGPT 说：

完全正确！你对 `PushEvent` 的理解 **非常准确且清晰** 👏👏👏：

## ✅ 总结一句话：

> `PushEvent` 就是**触发事件广播的“总开关”**，当事件被推送时，它会遍历对应事件 ID 下绑定的监听器，如果有回调，就执行这些回调函数。





### Lua Debug之打印函数堆栈：

```
    -- 在需要打印堆栈的地方添加以下代码
   print(debug.traceback("堆栈跟踪信息:"))
```





对背包部分的UI学习：

confirm对应的就是选中背包中物品之后的操作。背包物品栏BagTileView

lua文件中：

​	Construct进行构造添加按钮绑定事件。声明并定义owner.获取palyer的背包组件

在C++文件中找到了委托和调用委托广播的地方，但是却没有看到委托的回调函数在那里注册的



- 现在是找到了点击confirm按钮，点击之后打印相关log,然后向服务器发送消息，这个func函数是向服务器发送消息的函数？这是哪里定义的全局函数？

  以下是深入传递服务器函数逻辑之后，层层返回到背包相关业务逻辑的思路：

不懂func是什么，最后还是找到了，这是_sendUEMsg函数的变量。去看看其他地方调用`_sendUEMsg`的都是传入的什么函数

```lua
function _sendUEMsg(controller, func, msgid, msg, OwnerActor, ObjOrObjArray)
    
     -- .........
   	 				UnLua.Log(logdesc,msgid,MsgName,protoinfo:Num(),serpent.line(msg,g_serpentoptions),objname)

    func(controller, msgid, protoinfo, OwnerActor, ObjOrObjArray)
end



UEMsgHandle.lua文件中：


function SendUEMsgToServer(msgid, msg, OwnerActor, ObjOrObjArray)
--  ..省略对其他参数的判定逻辑，只看后面调用_sendUEMsg时候的参数信息
    if g_IsTArray(ObjOrObjArray) then
        _sendUEMsg(controller, controller.SendMsgToServer_MoreTarget, msgid, msg, OwnerActor, ObjOrObjArray)
    else
        _sendUEMsg(controller, controller.SendMsgToServer, msgid, msg, OwnerActor, ObjOrObjArray)
    end
end
```

接着往前回到了WBP_DebugItem.lua文件中

```lua
function M:SendDebugCmd(cmd)
    local pawn = self:GetOwningPlayerPawn()
    local controller = pawn:GetController()
    if controller:IsOfflineDebugMode() then
        SendUEMsgToServer(g_Msg.MsgID_UEServerDebugCmd, cmd, nil)      
    else
        local serverNet = UE.UServerNetworkSubsystem.GetServerNetwork(controller)
        if serverNet then
            serverNet:SendMsg(g_Msg.MsgID_ClientToGameDebugCmdReq,cmd)
        end
    end
end
```



当某个函数不是局部函数也不是全局函数的时候，看看其是不是作为参数传进来的函数。。。。无语了



emmm也就是说，点击confirm的时候向服务器传输信息是通过调用controller发送的，但是你并不能找到controller在哪，因为这是加入游戏的时候服务器分配的controller。 所以具体关于controller的函数应该在服务器分配的controller中寻找

去c++的controller中寻找 BP_ProjectAirPlayerController

发现lua中调用的playercontroller中的函数SendMsgToServer_MoreTarget，但是该函数又使用了unlua中的g_MsgEvent.HandleUEMsg函数..?

```lua
  _sendUEMsg(controller, controller.SendMsgToServer_MoreTarget, msgid, msg, OwnerActor, ObjOrObjArray)
```

总之最后调用了`g_MsgEvent:AddEventListener(Ref,OwnerActor,Id,Func)`上的回调函数Func

unlua中的自定义系统：

MsgEvent.lua文件，实际上实现的是自定义的事件系统，其中主要包含事件ID、事件监听、事件移除和事件触发。

就是AddEventListener:添加事件监听器，支持关联特定的OwnerActor

RemoveEventListener:移除事件监听器，同样支持关联特定的OwnerActor

PushEvent:触发事件，会调用所有注册该事件的回调函数。





## 联网Server：

------联网流程基本就是在controller进入游戏的时候通过UServerNetworkSubsystem这个自定义的子系统进行网络连接

```C++
void AProjectAirPlayerController::BeginPlay()
{
	Super::BeginPlay();
	
	PlayerCameraManager->SetViewTarget(GetCharacter());

	if ( GetNetMode() == NM_Client && !IsOfflineDebugMode())
	{
         //先获取网络子系统实例，接着验证其有效性。

		auto pNet= UServerNetworkSubsystem::GetServerNetwork(this);
		if (IsValid(pNet))
		{
            //构建ClientGameRoleInfoReq消息，此消息用于请求游戏角色信息。
			protocol::ClientGameRoleInfoReq msg;
            //最后通过网络子系统把消息发送到服务器。
			pNet->SendMsg(protocol::MsgID_ClientGameRoleInfoReq,msg);	

		}
	}
}
```

## C++调用Lua函数

被封装在了ProjectAirUtilities文件中

```c++
void UProjectAirUtilities::CallLua_PushEvent(AActor* OwnerActor, int32 ID)
{
    lua_State* L = UnLua::GetState();
    UnLua::CallTableFunc(L, "g_MsgEvent", "PushEvent",ID);
}这种就是封装的函数吧，用于访问lua中的函数，但是什么情况下需要访问lua中的函数呢？调用的lua函数有没有返回值呢？
```

- 调用 Lua 函数一般是为了实现热更新、加快迭代速度或者分离业务逻辑。
- 在你的代码示例中，`PushEvent(ID)` 的核心作用是：
  1. **传递事件 ID**：从 C++ 层传递事件标识（如 `ID=1001` 代表 "玩家登录" 事件）。
  2. **触发 Lua 监听器**：Lua 层预先注册了对该事件 ID 的回调函数，当 C++ 调用 `PushEvent` 时，Lua 层执行对应的回调逻辑。

## 心跳

在player controller中，如果是客户端的controller的话是要发送心跳的

心跳的作用是什么？确保客户端一直在线?
