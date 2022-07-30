## 关于币圈监控、推特监控、钱包监控的技术实现（1）

说到信息监控，特别是币圈信息监控，其实本质就是在这个信息爆炸的年代，尽可能快速的可靠的获取关键信息，才能帮助我们在的第一时间做出最好的决策建议。

下面就来简单的说下币圈各种监控需求的实现思路

 **1. 各大交易所的消息监控（币安、火币等）**
 
例如需求如下：

> **实时监控 币安交易所的新币上新、价格突变、带关键字筛选新闻 等信息**

其实像 币安、火币 这些中心化交易所，他们都有开放API提供给开发者调用，我们只需注册并申请使用这些API，然后代码进行调用即可。当然也有不需要注册申请就能使用的API。

例如：查询BNB对USDT的币价接口

```
https://api.binance.com/api/v3/ticker/price?symbol=BNBUSDT
```
直接浏览器访问接口后，接口会返回结果

> {"symbol":"BNBUSDT","price":"290.00000000"}    //price：即是返回的最新价格

复制链接到浏览器即可看得到接口返回的BNB价格。

然后新币上新等监控接口一般都需要注册申请后才能使用，但大多也是免费使用的。

在这里贴出获取币安交易所新币上新的非接口调用代码片段（用模拟浏览器浏览的方式去获取）

```typescript
// 获取币安新币上新信息
$platform = DataCapturePlatformModel::getCapturePlatform(2);
$scriptHtml = QueryList::get($platform['api_url'], [], [
    'timeout' => 10,
    'headers' => [
        'accept'             => 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-language'    => 'zh,zh-CN;q=0.9',
        'user-agent'         => 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.88 Safari/537.36',
        'sec-ch-ua'          => ' Not A;Brand";v="99", "Chromium";v="102", "Google Chrome";v="102',
        'sec-ch-ua-mobile'   => '?0',
        'sec-ch-ua-platform' => 'Windows',
        'sec-fetch-dest'     => 'empty',
        'sec-fetch-mode'     => 'cors',
        'sec-fetch-site'     => 'same-origin',
    ],
])->find('script#__APP_DATA')->html();
$scriptData = json_decode($scriptHtml, true, 512, JSON_THROW_ON_ERROR);
$catalogsData = $scriptData['routeProps']['b723']['catalogs'] ?? [];
$newList = array_merge([], ...array_column($catalogsData, 'articles'));
$isTranslate = (int)$platform['is_translate'];
$existNewIds = BinanceLatestNewsModel::whereIn('new_id', array_column($newList, 'id'))->column('new_id');
$insertList = [];
$time = time();
...
```

​

> **也可以参考下该（每刻数据[meikeshuju.com](https://meikeshuju.com)），它实现了实时从币安、火币等中心化交易所实时监控新币上新、价格监控等功能。支持了自动翻译英文内容，而且已经支持发送到个人微信/微信群、个人QQ/群等。**


本节的分享就到这里了。

之后的内容，再来聊聊推特监控。
