## 一、购物车

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201224151003.png)

- 用户可以在登录状态下将商品加入[在线购物车/用户购物车]
  - 放入`MongoDB`；
  - 放入`MySQL`；
  - 放入`Redis`(采用)，登录以后，会将临时购物车中的数据合并过来。
- 用户可以在未登录状态下将商品加入[离线购物车/游客购物车]
  - 放入`localStorage`；
  - 放入`Cookie`；
  - 放入`WebSQL`；
  - 放入`Redis`(采用)，即使浏览器关闭，临时购物车数据都在。
- 用户可以使用购物车一起结算下单
- 用户可以添加商品到购物车
- 用户可以查询自己购物车
- 用户可以选中购物车中商品
- 用户可以在购物车中修改购买的商品数量
- 用户可以在购物车中删除商品
- 在购物车中展示优惠信息
- 提示购物车商品价格变化

> 京东给每个用户生成一个值类似于 UUID 的`user-key`，有效期一个月，存储在`Cookie`，浏览器保存以后，每次访问都会带上这个`cookie`。
>
> 登录后：`session`有用户信息
>
> 未登录：`cookie`中的`user-key`
>
> 第一次使用时，如果没有临时用户，帮忙创建一个临时用户。



## 二、ThreadLocal

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201224163537.png)

```java
// 创建
public static ThreadLocal<UserInfo> threadLocal = new ThreadLocal<>();
// 赋值
threadLocal.set(userInfo);
// 获取
UserInfo userInfo = threadLocal.get();
```

## 三、重复刷新问题

> 购物车页面重复刷新数量重复变化问题，可以使用重定向功能解决。
>
> - 加入购物车后台成功后，重定向至购物车页面。
> - 添加购物车一个接口，跳转成功页一个接口，重复刷新成功页不会重复添加。

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201224173817.png)



## 四、RedirectAttribute

> 重定向携带数据

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201228150231.png)