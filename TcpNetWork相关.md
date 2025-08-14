### TCPNetWork_Base.lua  核心作用：

网络通信基类，提供跨模块通用的消息处理框架

 • 定义基础网络功能：消息注册（ RegisterMsgHandle ）、消息发送（ SendMsg ）、消息分发（ OnServerMsg ）

 • 实现RPC机制：通过 _rpcRequestKey 和 _rpcResponseKey 处理请求-响应模式

 • 维护消息句柄映射表： totalMsgHandles 存储注册的消息回调函数 

 消息处理范围：

 • 不直接处理具体业务消息，而是提供基础通信能力给子类

 • 通过 SendServerMsg 接口与底层C++网络模块交互 

• 所有网络模块（Client/DS）的公共逻辑抽象（如心跳机制、连接状态管理） 



### TCPNetWork_DS.lua  核心作用：

数据服务器（DS）网络模块，处理与GameServer的通信

 • 继承自Base类并扩展DS特有逻辑

 • 注册DS专属消息处理：如 OnServerGameServerOptItemReq （道具操作请求）

  消息流向：    GameServer → DS：

- - MsgID_ServerGameServerOptItemReq（道具操作请求）
  - MsgID_ServerDSBasicCraftReq（基础合成请求）
    DS → GameServer：
  - MsgID_ServerGameServerOptItemResp（道具操作响应）
  - MsgID_ServerDSConditionResp（条件检查响应）
       • 在 OnServerGameServerOptItemReq 中完成道具操作的核心事务（数据一致性校验、持久化存储）  

### TCPNetWork_Client.lua  核心作用：

客户端网络模块，处理与服务器的通信 

继承自Base类并专注客户端场景 

注册客户端专属消息处理：如 OnClientRoleItemsChangePush （道具变更推送）

消息流向： 服务器 → 客户端（推送）：

- MsgID_ClientRoleItemsChangePush（道具变更）
- MsgID_ClientRoleEquipsChangePush（装备变更）
- MsgID_ClientRoleCDGroupChangePush（CD组变更）
  客户端 → 服务器（请求）：
- MsgID_ClientHeartBeatReq（心跳包）
     • 通过 GetClientCharacter 获取角色对象，将网络消息转换为游戏逻辑更新 
-  模块协作关系 ：    
- TCPNetWork_Base.lua（基类）
      ↑
      ├─ TCPNetWork_DS.lua（服务端-数据层） ←→ GameServer ←→ 客户端
      └─ TCPNetWork_Client.lua（客户端-表现层）       ↑
                                                共享Base提供的网络基础设施

###  关键设计特点：

- 继承复用：DS/Client模块通过 UnLua.Class 继承Base类，复用80%基础网络代码
- 职责分离：Base处理通信框架，子类专注业务逻辑 
- 消息闭环：所有跨进程消息均通过这三个模块完成双向流转