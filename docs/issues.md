# 相关问题

## 部署平台没有写权限怎么办?

1.打开/src/config/[prod/dev/test]/index.js 文件,将 log 功能置为 false

2.打开/src/crawl/proxy.js 文件,按注释操作,修改文件返回函数

## 没有安装mongodb数据库可以运行吗?

没有安装数据库只能运行src/crawl中爬虫模块。具体调用参阅/test/crawl.test.js

## 没有安装redis可以运行吗?

可以。应用中 redis 主要用在了限制API请求频率上。修改/src/config/[prod/dev/test]/index.js 文件,将API_FREQUENCY 功能置 0 即可。此时测试用例将不会全部通过

## 应用安全性怎么样?

程序为了满足爬取各种各样的网页,开放了较大的权限给用户。程序关键是使用 eval 函数,将用户输入整合到上下文中。由此带来了较大的安全性问题,不良用户可能会利用这一点,对应用部署的服务器进行渗透等操作。但是,我认为只要能够规范全面的过滤掉用户输入的不安全内容，做好相应的防范措施，服务器被黑，程序被破坏的可能性还是很小的。

应用中，我在使用 eval 函数整合用户输入的模块中开启了严格模式，该模式下，有着不允许使用未声明的变量，不允许删除变量或对象，不允许变量重名，不允许使用转义字符，不允许使用八进制，在作用域 eval 中创建的变量将不能被调用等众多特点。此外，使用正则表达式对用户输入进行了过滤，用户输入中，只要包含环境关键字，程序即返回给用户错误信息。

## 应用中内置代理是从哪获取的?

代理来源于国外的免费的代理网站[FreeProxyList](https://free-proxy-list.net/).由我另外一个小项目[HttpProxy](https://proxys.herokuapp.com)进行可用代理检测，并且提供了相关API供用户使用。本项目中的内置代理即请求API由此可得

## 每次调用API都会抓取一次数据吗?那样岂不是响应很慢?

为了解决API响应问题,程序会将爬虫抓取结果保存到数据库中,请求API时，直接响应数据库中保存的数据。但也由此带来了一个问题，比如抓取新闻网站，抓取到的数据具有一定的时效性，每次请求API响应相同的数据肯定是不合适的。先前版本的WebSpider采用主动更新数据的机制,程序会自动定时更新一波数据。这种更新方式逻辑复杂,效率不高。目前这一版本采用了被动更新机制，由用户保存爬虫配置时提交的 interval 参数来决定API响应数据库还是爬虫新抓到的数据。如果两次API请求间隔时间小于 interval 参数，则直接返回数据库中的数据，否则调用爬虫抓取数据进行返回，并将数据更新到数据库中。这样由用户需要时更新一波数据，不需要时则不会更新数据。

## 为什么根目录下应用入口 app.js 只进行了端口开启操作?

运行测试用例(`npm test`)时,需要测试路由,测试路由则必须开启Web服务，所以需要将koa应用作为模块导出,测试时导入模块,启动端口打开服务。