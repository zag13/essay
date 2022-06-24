# 使用 MongoDB 和 Go 分析数据

> [Analyze Data With MongoDB and Go](https://www.ardanlabs.com/blog/2013/07/analyze-data-with-mongodb-and-go.html)


_本文由 Safari Books Online 编写并发布_

我的公司正在构建一个名为 Outcast 的移动应用程序。Outcast 背后的想法是让热爱户外活动的人能够早早了解天气。通过分析实时浮标、潮汐、月球、太阳、用户偏好、用户体验等数据，该应用程序可以提供相关信息和预测。
用户通过在户外活动结束后提供体验评论来帮助预测。随着时间的推移，应用程序会了解每个用户的最佳状态、他们的活动和最喜欢的位置。

预测引擎是使用 MongoDB 和 Go 构建的。我创建了一个名为 MongoRules 的小程序，它展示了如何使用 MongoDB 和 Go 来快速挖掘和分析数据。