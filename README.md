#### 1v1通话改版
> 表新增

| 表名 | 备注
| ---------- | :-----------: |
| app_call_ing | 通话进行中表           
| app_rtc_value | 通话记录相关附加参数（下面会详细解释）
| app_call_record | 通话记录表
| app_trade_server | 单场通话信息表
| app_trade_ticket | 通话扣费表


## app_call_ing  
| 类型      | 字段     |    描述     |   
| ---------- | :-----------: | :-----------: |   
| long | call_id       | 自增id | 
| long | host_user_id  | 主播id | 
| long | guest_user_id | 用户id | 
| long | dial_time     |  播打时间 | 
| long | dial_user_id  |   播打用户id | 
| long | idal_time     |   接听时间 | 
| long | helo_time     |  接听方进入通话say helo 为0一般做正在连接中等状态显示 | 
| long | ehlo_time     |  播打放进入通话 回应 | 
| long | hang_time     |  挂断时间  ing表始终为0 | 
| long | hang_user_id  |  挂断用户id | 
| long | guest_hb_time |  用户心跳时间 | 
| long | host_hb_time  |   主播心跳时间 |
| int | cdn_id  |   cdn_id /声网-腾讯 |
 
###### host_user_id unique
###### guest_user_id unique 
###### call_id     pk


## app_call_record 
| 类型      | 字段     |    描述     |                
| ---------- | :-----------: | :-----------: |  
| long | call_id       | 自增id                 |
| long | host_user_id  | 主播id                 |
| long | guest_user_id | 用户id                 |
| long | dial_time     |  播打时间                |
| long | dial_user_id  |   播打用户id             |
| long | idal_time     |   接听时间               |
| long | helo_time     |  接听方进入通话say helo     |
| long | ehlo_time     |  播打放进入通话 回应          |
| long | hang_time     |  挂断时间                |
| long | hang_user_id  |  挂断用户id              |
| long | guest_hb_time |  用户心跳时间              |
| long | host_hb_time  |   主播心跳时间             |
| int | cdn_id  |   cdn_id /声网-腾讯 |

###### host_user_id unique
###### guest_user_id unique 
###### call_id     pk


## app_rtc_value
| 类型      | 字段     |    描述     |                
| ---------- | :-----------: | :-----------: |  
| long | rtc_unit_id     | 主键id                                                      |
| long | user_id |  callid                                                  |
| String | module | /live/call/room  目前暂时仅用call改版                             |
| long | numb1     | 代码做定义，如numb1记录本次通话收益，2记录本次通话是否匹配等                        |
| long | numb2.。。。。numb9 |     

###### rtc_unit_id pk
###### user_id index 



## app_trade_server
| 类型      | 字段     |    描述     |                
| ---------- | :-----------: | :-----------: |  
| long | server_id    | 自增                                    |
| long | user_id      | 用户id                                  |
| String | resource_id | 来源id   call-#{callId}                 |
| long | mode_id | call-id              |
| String | module_enum| chat/call 聊天/通话 价格                    |
| String | mode       | ONCE/EACH    单次付费/周期付费                |
| long | period_mills | 计费间隔                                  |
| long | price        | 价格                                    |
| long | create_time  | 创建时间                                  |

###### server_id pk
###### user_id,resource_id unique
###### module_enum, mode_id index

## app_trade_ticket
| 类型      | 字段     |    描述     |                
| ---------- | :-----------: | :-----------: |  
| long | ticket_id | 主键id |
| long | server_id | trade_server_id |
| long | user_id | 用户id |
| long | ticket_time | 付费时间 |
| long | done_time | 下次付费时间 mode为EACH 时生效 | 

###### ticket_id pk
###### user_id index
###### server_id index


## JAVA实体类信息

## Call 
#### (app_call_ing/app_call_record/app_rtc_value)
1. long callId
2. long hostUserId
3. long guestUserId
4. long dialTime
5. long dialUserId
6. long idalTime
7. long heloTime
8. long ehloTime
9. long hangTime
10. long hangUserId
11. long guestHbTime
12. long hostHbTime
13. List<Long> value  --app_rtc_value numb1-9会放这里

## TradeServer
#### (app_trade_server)
1. long serverId
2. long userId
3. String resourceId
4. Enum moduleEnum --CALL
5. Enum mode       --ONCE/EACH
6. long periodMills
7. long price
8. createTime

## TradeTicket
#### (app_trade_ticket)
1. long ticketId
2. long serverId
3. String userId
4. String ticketTime
5. String doneTime



### RPC API provider
### CallService

播打
1. Call dialWithRtcValue(long hostUserId, long guestUserId, boolean direct, Map<Integer, Long> cmap);
direct true dialUserId = hostUserId,false guestUserId
2. Call heartbeat(long callId, long userId);心跳
3. Call hang(long callId, long userId);挂断
4. void updateCounts(long callId, Map<Integer, Long> indexAndCount); 更新app_rtc_value数据,常用于1v1直播礼物收益/1v1 通话收益更新
5. void incrCounts(long callId, Map<Integer, Long> indexAndCount); 更新app_rtc_value数据
6. long ticketByResourceId(long userId, long provideUserId, String resourceId, boolean free); 心跳扣费-计费心跳调用
7. List<Call> listAll(); 直播中所有列表

8. Call idal(long callId, long userId)接听通话
9. publishServer(long userId, String resourceId(call-${callId}), moduleEnum.CALL, long periodMills, long price) 发起通话后init当前通话收费情况
### 通话流程处理

1. heartbeat api  通话中需根据心跳循环调用，超时未调用会触发hang api
2. 起单线程循环 扫描 app_call_ing guest_hb_time/host_hb_time 超过15s的 List<Call> 做hang通话处理 --心跳超时
3. 起单线程循环 扫描 app_call_ing dial_time 超过30s的 List<Call> 做hang通话处理   -- 播打超时
4. 三方rtc，以声网举例 channelName 以CallId生成token
5. 关于心跳处理，需调用ticketByResourceId，将生成app_trade_ticket表记录，每次拉最后一条记录doneTime 对比 app_trade_server mode=EACH && > periodMills 则进行下次扣费，并生成新的app_trade_ticket记录


### 服务端客户端交互

> 用户客户端
>
>> 1发起通话 dialWithRtcValue 以callId下发 RTC—TOKEN
>>
> 主播客户端
>>
>> 2.接听电话 以CallId下发RTC—TOKEN
>
>  3.im下发通话建立
>
>  4.双方收到通话建立im-发起心跳（可以http/目前im心跳-需确认能否拿到在房间的字段）
>
>  异常流程 一方心跳超时 hang结束通话，app_call_ing写入app_call_record记录表，清楚ing记录
>
> 

关于单次通话存在两种计费模式
1.前x秒使用通话卡，计费模式  用户测不做扣费/主播recv收获对应的通话卡收益(分全额+非全额)

关于ticketByResourceId 封装，free=true，则不进行用户测扣费-对主播收益进行增加

```
ticketByResourceId(long userId, long provideUserId, String resourceId, boolean free){
    TradeServer server = get$app_trade_server(provideUserId, resourceId);
    if (server.mode == EACH){
        TradeTicket ticket = get$app_trade_ticket(server.id, userId) order by ticket_id desc limit 1;
        if(ticket == null) {
            if (free) { //使用通话卡
                //扣减通话卡
                //增加主播通话卡收益
                insert$app_trade_ticket(serve_id,user_id,ticket_time,done_time); //done_time=System.currentTimeMillis() + server.periodMillis()
                return;
            }
        }
        if(ticket.doneTime > System.currentTimeMillis()){
            return;//不扣费 
        }
        int multiple = (System.currentTimeMillis() - ticket.doneTime) / server.periodMillis()
        if(free) {
            //增加主播通话卡收益 * multiple
            insert$app_trade_ticket(serve_id,user_id,ticket_time,done_time); //done_time=System.currentTimeMillis() + server.periodMillis()
            return;
        } else {
            //增加主播收益 * multiple
            //扣减用户金币 * multiple
            insert$app_trade_ticket(serve_id,user_id,ticket_time,done_time); //done_time=System.currentTimeMillis() + server.periodMillis()
            return;
        }
    }
}
```

//每次insert$app_trade_ticket完做mq下发，做后续活动等相关捕获处理


### rest API
1. /call/dial
    -校验用户测通话卡|余额是否足够
``  call = CallService.dial(hostuid,guestuid,direct);
    publishServer(long userId, String resourceId(call-${callId}), moduleEnum.CALL, long periodMills, long price)

2. /call/idal
    CallService.idal();

3. /call/heartbeat 心跳，http/im都可

4. /call/hang    挂断


### im改动
原有通话流程相关 字段新增callId


### 版本兼容方案
TODO
服务端做开关。callId 客户端判断空-则走旧的逻辑流程，非空则走新的逻辑获取rtc-token，待客户端版本覆盖量上来后，服务端做整体切换


