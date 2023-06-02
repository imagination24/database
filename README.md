# database
database of imagination24 Project

## Portfolio 结构

```
{
///作品实体列表
"portfolio_entity":[
    {
            //名字,不可为null
            "title":"FamiShield",
            //头图，不可为null
            "header_img":"https://i.imgur.com/eLVaJjW.png",
            //图标，不可为null
            "icon":"https://i.imgur.com/xlHnenB.png",
            //官网图标，大小为646*250，可以为null
            "web_link_img":null,
            //GooglePlay图标，大小为646*250，可以为null
            "googlePlay_link_img":null,
            //官网链接，可以为null
            "website_link":null,
            //googleplay链接，可以为null
            "googlePlay_link":null,
            //介绍（关于），不可以为null
            "intro":{
                //中文
                "zh_cn":"FamiShield是一款家庭安全应用，由孩子端和父母端组成，父母端可以控制孩子端的手机，包括使用时间，软件使用记录，电话短信网络记录，禁用应用，立即锁屏，网站拦截等功能。孩子端的信息以分钟级的水平上传，当父母端的配置更新时，孩子端以不超过一秒的时间同步配置，可以说父母端的操作是即时生效的。",
                //英文
                "en_us":"FamiShield is a family security application, which consists of a child terminal and a parent terminal. The parent terminal can control the child's mobile phone, including usage time, software usage records, phone and SMS network records, disabled applications, immediate lock screen, website blocking and other functions. The information on the child side is uploaded at the level of minutes. When the configuration of the parent side is updated, the child side will synchronize the configuration in no more than one second. It can be said that the operation of the parent side takes effect immediately."
            },
            //（技术要点），不可以为null
            "special":{
                //中文
                "zh_cn":[
                    "数据收集",
                    "数据上传",
                    "复杂权限申请",
                    "http",
                    "firebase",
                    "后台运行",
                    "软件更新",
                    "多语言",
                    "网站拦截",
                    "锁屏",
                    "地图和定位",
                    "Flutter插件实现"
                ],
                //英文
                "en_us":[
                    "data collection",
                     "Data Upload",
                     "Complex permission application",
                     "http",
                     "firebase",
                     "Background process",
                     "Software update",
                     "multi-language",
                     "Website blocking",
                     "Lock screen",
                     "Maps and Location",
                     "Flutter plugin implementation"
                ]
            },
            //截图，不可以为null，可以为空列表
            "screen":[
            ]
        }
    ]
}
```

## Blog结构
```
{
    "blog":[
        {
            //标题
            "title":"2022年总结",
            //副标题
            "subtitle":"随风而逝的2022会给未来带来什么影响？我不知道朋友们，活在当下吧",
            //标题大图
            "header_img":"https://i.imgur.com/VHU9lGY.jpg",
            //blogID,blogID与图片ID必须一致
            "detail":"VHU9lGY"
        },
    ]
}
```

## 如何获取blog内容
```
https://raw.githubusercontent.com/imagination24/database/main/blogDetails/$blogId.md
```
