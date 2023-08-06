# Web Notifications

## blink::mojom::NotificationService

blink::mojom::NotificationService是一个mojo接口，是Browser端和Renderer端之间Notificaitons功能的一个入口点。需要跨进程通信的服务都需要经过这个类。

1. 先生成NotificationID
2. 注册EventHandler
3. browser_context_->GetPlatformNotificationService()->DisplayNotification(...)

PlatformNotificationServiceImpl中的逻辑

1. 将blink::PlatformNotificationData转换成message_center::Notification
2. 将展示Notifications的功能转发到NotificationDisplayServiceFactory::GetForProfile()

NotificationDisplayServiceFactory
NotificationDisplayService

Notification可以直接复用。
