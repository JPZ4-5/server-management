---
title: "使用Tailscale，ssh连接服务器指南"  
date: 2025-01-13  
---
# 0. 熟悉网络的你为什么要用tailscale（给有困惑的人提供的）
1. 当然，我们把三层交换机直接接入校园网，可以让我们在校园网内直接通过`ssh user@ip:port`的形式访问服务器。
2. 但是，由于校园网并没有对校园网内的网络进行严格的限制，导致如果没有路由器的自带防火墙保护，很容易被入侵并植入挖矿病毒（已知的是老胡、成像等组的服务器都被入侵并植入挖矿病毒过）。
3. 所以，为了便捷起见，暂时采用了使用Tailscale进行异地组网的方式让大家连接服务器。在校园网内，Tailscale局域网内的设备是通过VPN直接连接的（成功的情况下。目前仅出现一例安装后失败的情况。通过关闭Tailscale客户端，并在任务管理器关闭Tailscale服务后，重启Tailscale解决了。）
4. Tailscale提供了一个异地的局域网，并提供了十分方便的跨局域网访问方式：在你创建一个Tailscale网络之后，只要在你的其他设备上也下载Tailscale并登录此账号，都可以通过Tailscale访问到局域网内的任何设备。
5. 服务器之间建立了一个局域网。在这里，我们使用[share machine](https://tailscale.com/kb/1084/sharing)的模式让大家访问到服务器，在这种share模式下，大家可以主动访问服务器，但是服务器不能主动访问大家的设备。share user的方式，存在安全性问题以及免费Plan用户数上限的问题。
6. 此外，不让大家把自己的设备直接加入服务器网络的原因主要在于：大家可以把自己的所有个人设备放在一个单独的局域网中，而不会让服务器能够直接访问到大家的设备，提高私密性。
7. 当然，配置接入校园网的三层交换机也是可以的。~~但是因为不会且没时间学，暂时搁置了。~~ 如果看到这里的你恰好会配置交换机的防火墙，且十分热心，请你去配置一下。接入使用的Console线我已经买好，放在桌子边了。提前谢谢你！
我这里还有厂家提供的三层交换机使用手册。

# 00. 写在前面
1. 本文档写给大家供参考，感谢大家的支持与理解。
2. 本文档主要面对Windows用户，兼容Mac用户。如果你是Linux用户，相信你一定已经有足够的知识、技能与耐心来灵活处理问题。

---
# 1. 创建Tailscale账号
1. 打开[Tailscale欢迎界面](https://login.tailscale.com/start)，准备创建你的账号。
2. Tailscale提供了几种认证方式。推荐Microsoft和Apple，过程中不需要翻墙。当然，如果你有稳定的梯子，且你完全知道你在干什么，你可以随便选择任何一种认证方式。
3. 注册完毕。

# 2. 下载Tailscale并安装
没什么好说的，[下载](https://tailscale.com/download)并安装。

# 3. 登录Tailscale
1. 安装好后，应该会直接在你的默认浏览器中弹出登录网页。如果没有，在你的界面右下角（Win10）小图标或者隐藏的图标中找到在后台运行的Tailscale的图标，单击图标，找到admin console或者login等栏并单击，应当可以打开登录网页。如果还没有跳出，直接输入`https://login.tailscale.com/admin`。
2. 你应当看到的是一个有`Welcome to Tailscale!`的界面。选择`Personal use`，点击`Next`，并在下一个界面选择下方的`Skip`样。
3. 完成后，你应当看到的是一个大字写着`Machines`的界面。你已经完成了Tailscale的下载与登录！现在MACHINE栏里面唯一的一栏，就是你现在使用的这台机器。（悄悄告诉你，这个页面可以调成中文的）
4. （扩展使用。如果不太理解，可以放心的忘掉这一栏）如果你还想在自己其他设备上使用tailscale，可以在其他设备下载tailscale客户端并登录。只要在校园网内，是一定可以直连的。校园网外，tailscale根据IPV4和IPV6的具体情况尝试进行直连（截止目前，在川渝能够较稳定直连，在高峰时段仍然不稳定，这是一个玄学问题），如果NAT层数过多将严重影响直连的成功率！如果不能进行直连，Tailscale会尝试通过公司自己提供的Derp服务器进行连接。但是由于没有在大陆使用的服务器，且服务器公用，导致延迟完全无法接受。（肉身在HKG，TKY等Derp服务器所在地区，则速度很快）

# 4. ssh连接服务器
1. 找现在的服务器管理员询问账号、网址、服务器的share link。
2. 在你的浏览器内输入share link，一直点ok。点完后，你应当会在`Machines`界面看到标有`shared in`的机器，这个就是服务器。
3. 打开一个shell（cmd/powershell），输入`tailscale status`。
4. 你应该会看到有一行或多行类似`100.X.X.X XXXXXX.tailXXXXXXXX.ts.net XXXX linux -`的输出，每一行代表一个服务器。如果没有ts.net，说明是你自己的设备。
5. 对于你想要访问的服务器，输入`tailscale ping XXXXXXX.tailXXXXXXXX.ts.net`（这段直接复制你之前获得的输出，可以用IPV4或者域名）。
6. 你应当会看到类似以下的输出：
   ```shell
   pong from XXXXX via DERP(XXX) in XXXms
   pong from XXXXX via DERP(XXX) in XXXms
   pong from XXXXX via DERP(XXX) in XXXms
   pong from XXXXX via DERP(XXX) in XXXms
   pong from XXXXX via XXX.XXX.XXX.XXX:XXXXX in XXms  
   ```
   如果出现了10ms以下的延迟，则说明我们已经建立了到服务器的直连（教研室内。如果在宿舍，或者校外，只要via的是一段ipv4或者ipv6，说明已经建立了直连）（如果没有，有类似无法建立连接的输出，请看0.3括号中的内容）
7. 输入`ssh 账号@XXXXXXX.tailXXXXX.ts.net`或者`ssh 账号@100.XXX.XXX.XXX`（账号由4.1得，XXXX部分由4.4得）
8. 输入4.1得到的密码。注意，linux中输入密码不会有任何视觉反馈。这是正常的。
9. 在接下来的命令行中根据你自己的需求输入`yes`或者其他命令。
10. 你应当已经登入了服务器。有类似以下行：
    ```shell  
    user@device:~$
    ```
11. 好耶
