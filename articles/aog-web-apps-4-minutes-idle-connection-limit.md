<properties
    pageTitle="关于 Azure Web 应用 4 分钟空闲连接的限制"
    description="关于 Azure Web 应用 4 分钟空闲连接的限制"
    service=""
    resource="webapps"
    authors="Zhang Hongmei"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Web Apps, Idle"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="app-service-web-aog"
    ms.date=""
    wacn.date="04/29/2017" />

# 关于 Azure Web 应用 4 分钟空闲连接的限制

Azure Web 应用后台在处理耗时较长的请求时，并且在此期间，客户端和 Azure Web 应用没有数据交互，即 TCP 连接一直处于空闲状态，此种情况超过 4 分钟后，Azure Web 应用会强制断开于客户端的 TCP 连接，并且客户端如浏览器会收到请求错误。

Azure Web 应用的工作流如下：

![work-flow](./media/aog-web-apps-4-minutes-idle-connection-limit/work-flow.png)

一个请求的完成是由客户端到 Web 应用服务器的 HTTP 连接和 Web 应用服务器到数据库的 TCP 连接组合完成的。

Azure 服务有个 Idle 的时间是 4 分钟，即客户端与服务之间如果在4分钟之内没有数据交互，连接将会被断开。如果持续有数据传输则不受此限制。

这个 Idle 的时间目前是没有办法修改的。

对于需要长于 4 分钟才能处理完毕的一些请求，建议采用异步处理，这样同时也提高了用户的体验。
