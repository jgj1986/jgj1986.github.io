---
title:  emqtt添加pubsub权限控制
date: 2015-11-19 18:12:28 +0800
tags: iot
- a
- b
---

* toc
{:toc}

## 背景

物联网中规划的应用，用户和对应的某个物（此处称为设备）之间建立起了绑定关系，而只有有绑定关系的用户和设备的topic才能相互的pub/sub。为此在emqttd已有的权限控制基础上添加针对此功能的权限验证。

## 设计

用户和设备的绑定关系，在Redis中存放，emqttd端在收到用户pub/sub消息时，根据UserName和Topic来做判断。Topic的命名方式为 xxx/xxx/Dvid 或 xx/UsrId，为了描述方便，将Topic的最后字符串称为Name，也就是这里的Dvid或UsrId。如果请求时上传的UserName和TopicName一致，或 UserName绑定的设备中包含TopicName，则验证通过。

注意，在实际的业务逻辑中，没有设备pub/sub用户名命名的Topic，所以上述过程不需要反向考虑。

实际的应用场景中，需要记录物联网中中的操作日志，用于以后的数据分析，提高用户体验，所以我们预定一个特殊的用户名可以pub/sub所有Topic。

## 实现

emqttd_access_control.erl:
    
    ...
    topic_name(Topic) ->
        TopicL = lists:reverse(Topic),
        sub_topic_name([],TopicL,[]).
    sub_topic_name([],[],Result) -> Result;
    sub_topic_name(R1,[],Result) -> R1;
    sub_topic_name(R1,[H|T],Result) ->
        case H of
        47 ->
            case R1 of
                -> sub_topic_name([],[],R1)
            end;
        _-> sub_topic_name([H|R1],T,Result)
    end.

    check_bind(UserId, DeviceId) ->
        {ok,C}  = eredis:start_link(),
        {ok, Devs} = eredis:q(C,["SMEMBERS", UserId]),
        eredis:stop(C),
        case lists:member(DeviceId, Devs) of
            true -> allow;
            false->deny
        end.

    check_nametopic(#mqtt_client{username = Username}, Topic) ->
        case Username of
            undefined -> deny;
            <<"SpecName">> -> allow;
            _ ->
                TopicName = list_to_binary(topic_name(binary_to_list(Topic))),
                if Username == TopicName ->
                    allow;
                true->
                    check_bind(Username, TopicName)
                end
        end.
    ...

    check_acl(Client, PubSub, Topic) when ?IS_PUBSUB(PubSub) ->
        case check_nametopic(Client, Topic) of
        ...

说明，eredis使用服务器本机的redis和默认端口，所以没有添加host与port。关于eredis的更多说明，参考其[官网](https://github.com/wooga/eredis){:target="_blank"}。

是`emqttd_protocol.erl`中执行publish、subscribe之前会调用 check_acl做权限控制。而此处是 emqttd_client.erl中监听到`inet_async`消息做进一步处理，执行到的。

