---
title: rabbitmq
---
## 延迟队列实现
TTL+死信队列，组合实现延迟队列的效果

### TTL 消息过期时间设置
TTL，全称Time To Live，消息过期时间设置。消息的TTL就是消息的存活时间。RabbitMQ可以对队列和消息分别设置TTL。对队列设置就是队列没有消费者连着的保留时间，也可以对每一个单独的消息做单独的设置。超过了这个时间，我们认为这个消息就死了，称之为死信。
队列过期后，会将队列所有消息全部移除。 一个队列中某一个消息过期后，只有消息在队列顶端，才会判断其是否过期(移除掉)，如果不在队列顶端，那么是无效的，过期时间有队列的过期时间判定。
如果队列设置了，消息也设置了，那么会取时间短的。所以一个消息如果被路由到不同的队列中，这个消息死亡的时间有可能不一样（不同的队列设置）。
我门一般通过设置消息的x-message-ttl属性来设置时间

### 死信队列
死信队列，英文缩写：DLX 。Dead Letter Exchange（死信交换机），当消息成为Dead message后，可以被重新发送到另一个交换机，这个交换机就是DLX。


消息成为死信的三种情况：

1.  队列消息长度到达限制
2.  消费者拒接消费消息，basicNack/basicReject,并且不把消息重新放入原目标队 列,requeue=false
3.  原队列存在消息过期设置，消息到达超时时间未被消费

队列绑定死信交换机，给队列设置参数： x-dead-letter-exchange 和 x-dead-letter-routing-key，就能成功绑定了

Dead Letter Exchange其实就是一种普通的exchange，和创建其他exchange没有两样。只是在某一个设置Dead Letter Exchange的队列中有消息过期了，会自动触发消息的转发，发送到Dead Letter Exchange中去。

### 实现
延迟任务通过消息的TTL和Dead Letter Exchange来实现。
我们需要建立2个队列，一个用于发送消息，一个用于消息过期后的转发目标队列。

场景
- 订单超时取消
## 应用
- 应用解耦
- 流量削锋
- 异步消息