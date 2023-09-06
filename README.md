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
| long | helo_time     |  接听方进入通话say helo | 
| long | ehlo_time     |  播打放进入通话 回应 | 
| long | hang_time     |  挂断时间  ing表始终为0 | 
| long | hang_user_id  |  挂断用户id | 
| long | guest_hb_time |  用户心跳时间 | 
| long | host_hb_time  |   主播心跳时间 |
 
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


JAVA实体类信息
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



API provider
### CallService
播打
1. Call dialWithRtcValue(long hostUserId, long guestUserId, boolean direct, Map<Integer, Long> cmap);
direct true dialUserId = hostUserId,false guestUserId
心跳
2. Call heartbeat(long callId, long userId);
挂断
3. Call hang(long callId, long userId);
更新rtc-value 常用于1v1直播礼物收益/1v1 通话收益更新
4. void updateCounts(long callId, Map<Integer, Long> indexAndCount); 更新app_rtc_value数据
5. void incrCounts(long callId, Map<Integer, Long> indexAndCount); 更新app_rtc_value数据
心跳扣费
6. long ticketByResourceId(long userId, long provideUserId, String resourceId, boolean free); 


