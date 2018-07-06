# im
golang IM server

该项目只处理IM方面，不与主业务冲突，可与使用者现有产品分离

支持socket与websocket互通。
支持单用户多端登录。

数据库：mongodb，你可以换成你熟悉的。
消息转发采用了kafka。

使用说明：
先注册用户。用户以房间形式存在。消息发送为单个用户向某个房间发送。
房间可以只存在一个用户（单聊），或者多个用户（群聊）。
业务处理详见logic，可以根据使用者自己的业务修改。

comet：
主要负责连接。
数据格式：protobuf,当然你也可以换成其他的。
数据协议：包头+包体，详细见libs/proto。

logic：
主要负责业务处理。

适合使用：
单台服务器，中小用户量。
你可以根据用户量分发多台服务器，达到大用户量的目的。

目前生产环境状态稳定良好。

如果你有好的改进方法，请让他更强大


其他说明：（详见logic/rpc.go）
假如：用户A(用户房间号为a)，用户B(用户B的房间号为b)
A向b发送一条消息，系统会先查找A是否存在。若A不存在，则丢弃该消息。
之后查询b是否存在。若不存在或者b->B离线，将开启api推送到使用者其他业务处理模块（详见logic/http.go）
B接收到的数据为：来之某一个房间a(to)的消息，发送人是A(from)

为了保证消息到达率和可靠性采用了消息回执机制
比如：A <-> server <-> B
A -> server：server收到消息后会回执A，并且携带唯一消息id和当前时间 A <- server
server -> B：server会多次向B发送(最多3次，间隔 n * 10s)，直到收到B的回执 server <- B
