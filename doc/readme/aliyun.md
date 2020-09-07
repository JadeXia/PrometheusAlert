## 阿里云短信和电话告警配置

开启阿里云短信告警和语音告警需要提前注册好阿里云平台的帐号。并开通阿里云短信和语音相关服务。

开通服务可参考阿里云官方文档：

* 短信：https://help.aliyun.com/document_detail/55288.html?spm=5176.8195934.1283918.11.73af30c917Jfti&aly_as=m1ouSihk

* 语音（电话）：https://help.aliyun.com/document_detail/55070.html?spm=a2c4g.11186623.6.547.77e271b9SAbG6p

* 注意事项：开通阿里云短信和语音需要配置的模版请使用如下，`${code}`为变量名称，请勿替换：`prometheus告警:${code}`


配置阿里云短信和语音参数：

```
#---------------------↓阿里云接口-----------------------
#是否开启阿里云短信告警通道,可同时开始多个通道0为关闭,1为开启
open-alydx=1
#阿里云短信主账号AccessKey的ID
ALY_DX_AccessKeyId=xxxxxxxxxxxxxxxxxxxxxx
#阿里云短信接口密钥
ALY_DX_AccessSecret=xxxxxxxxxxxxxxxxxxxxxx
#阿里云短信签名名称
ALY_DX_SignName=阿里云
#阿里云短信模板ID
ALY_DX_Template=xxxxxxxxxxxxxxxxxxxxxx

#是否开启阿里云电话告警通道,可同时开始多个通道0为关闭,1为开启
open-alydx=1
#阿里云电话主账号AccessKey的ID
ALY_DH_AccessKeyId=xxxxxxxxxxxxxxxxxxxxxx
#阿里云电话接口密钥
ALY_DH_AccessSecret=xxxxxxxxxxxxxxxxxxxxxx
#阿里云电话被叫显号，必须是已购买的号码（为空则为使用阿里云公共号码池号码）
ALY_DX_CalledShowNumber=xxxxxxxxx
#阿里云电话文本转语音（TTS）模板ID
ALY_DH_TtsCode=xxxxxxxx
```

开启阿里云语音服务，并进行资质管理的认证以及语音通知模板的添加


${code}内容默认为rules规则中description字段内容

在AlertTemplate中添加阿里云电话模板



```
{{ range $k,$v:=.alerts }}故障主机{{$v.labels.appname}}{{$v.annotations.description}}{{ end }}
```

配置alertmanager的配置文件，指定路由规则，进行语音告警,使用prometheusalert接口，如下（部分配置内容）
```
  - receiver: 'prometheusalert-phone-db'
    group_wait: 10s
    match:
      level: '4'
      type: db

receivers:
- name: 'prometheusalert-phone-db'
  webhook_configs:
  - url: 'http://localhost:8080/prometheusalert?type=alydh&tpl=ali-phone&phone=136xxxx'
    send_resolved: false
```

注意：

1、阿里云已不支持专有号码进行拨打电话
2、阿里云屏蔽带有IP信息的语音
3、阿里云语音服务有流控 https://help.aliyun.com/document_detail/149826.html?spm=a2c4g.11186623.6.683.6cf54c07wo2zTO
