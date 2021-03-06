---
title: 只是为了把 Steam 游戏放到开始屏幕上
date: 2022-07-10T02:23:58+08:00
draft: false
---
很久以前, 我有一个梦想, 我的开始菜单上能直接固定一些 steam 游戏的磁贴.

然后我苦苦地寻找, 发现了[这篇](https://zhuanlan.zhihu.com/p/28416752)知乎文章,
然后我忍痛剁手, 花了我六块大洋, 买了这个叫 Steam Tiles 的 UWP 应用, 而这一切,
只是噩梦的开始.

-----

初次使用这个 UWP 应用, 它就刷新了我的三观. 一个功能如此简单的应用居然能这么卡.

它需要读你安装 steam 游戏的目录, 然后从里面寻找合适大小的图片来制作磁贴, 也允许
用户自定义图片, 可以从 url 下载图片, 可以从本地选取图片文件, 允许用户改变图片的
位置, 大小, 以及磁贴的背景色.

听起来很美好, 该自定义的都能自定义. 但是事实上.

我是想破脑袋也想不出来, 为什么他读文件找图片能这么慢, 5个游戏, 他头一次找个图要
将近半个小时.

![image-20200504214634198](https://i.loli.net/2020/05/04/gFqwyaEukvbALjJ.png)

不仅慢, 还有可能会读不出图片来, 比如像下面这样:

![image-20200504213156290](https://i.loli.net/2020/05/04/xCsnIKF9TBfRwUt.png)

而这个时候我唯一找到的解决方法, 就是 TMD 清除掉他对于图片的缓存, 然后再让他找一
次.

为什么这么慢呢? 我也不知道. 任务管理器开起来一看:

![image-20200504213307798](https://i.loli.net/2020/05/04/MKvWVlPTcQGzHOr.png)

不占用 CPU, 内存也不大, 那是磁盘读写么?

![image-20200504213452252](https://i.loli.net/2020/05/04/eKTNusvSVZ3Co9m.png)

看起来也不是磁盘读写.那咋回事呢? 天知道.

好在这种找图的过程在你使用这个软件的过程中只会出现一次 (当然如果你装了个新游戏,
然后想把它钉在开始菜单里面, 你又会经历一遍这个神奇的事情.) 经过几次的尝试, 我终
于可以正常的加载出图片了, 这个时候我发现了一个特别操蛋的事情.

![image-20200504213638142](https://i.loli.net/2020/05/04/3tEiTaeurcUo8lC.png)

看到那个输入 url 的框了么? 它怎么了呢? 事情是这样的. 你只要往里面打一个字, 这个
框就会实去焦点, 然后你得把你的鼠标再点击一次这个框, 才能再次打一个字母. 虽然我
知道, 一般人确实不会手打 url, 这个 bug 在你粘贴 url 进去的时候并不会出现, 但是
我也很难想象一个开发者往自己的应用里面放了一个输入框, 而他连在这个输入框里面打
字来做一下测试这种事情都没有做过.

哦不, 也许他真的做过, 不信你接着看.

-----

有玩过桌面开发框架的应该都知道这个 bug 是怎么出现的, 类似输入框这种控件, 它在里
面的文字改变的时候, 会触发一个叫 Text Changed 的事件, 开发者可以配置程序, 然后
让程序在这个事件发生的时候运行一段代码. 而由于不知道什么原因, 这个 UWP 应用的这
段代码会使得输入框失去焦点, 所以这个魔性的 bug 就产生了.

解决的办法其实也很明显, 我们其实只要把这段代码从 Text Changed 解绑, 然后绑定到比
如 Lose Focus 这种事件上去就行了.

但是这是一个在微软应用商店上架的, 收费的, 应用软件. 我应该是看不到它的源码的. 所
以我没有办法修正这个傻逼至极的错误.

嗯? 真的是这样么?

![image-20200504213724438](https://i.loli.net/2020/05/04/hLrUG1EKqBIjus3.png)

???

-----

这个老哥的心非常的大, 他居然把自己放在应用商店贩卖的应用的源码公布了\! 虽然这种
操作是挺正常的. 也有一些很棒的软件在做类似的事情, 开源和收费的确不矛盾, 但是还
算是比较少见的.

但是打开了他的 github, 我就发现这个事情非常的诡异:

![image-20200504213838350](https://i.loli.net/2020/05/04/reYa3s52xMAi8fl.png)

嗯?

![image-20200504213915629](https://i.loli.net/2020/05/04/5cRrqLIBm4n9ugA.png)

真的是头一回见啊.

更有意思的是:

![image-20200504213944643](https://i.loli.net/2020/05/04/Y9M78IkXGVO41eD.png)

就是说这哥们开发这玩意整整四年了, 然后这个输入框的 bug 仍然存在????

我本着好奇的心态看了一下他的 commit, 然后我震惊了: 除了下面这八个 commit 以外,

![image-20200504214039085](https://i.loli.net/2020/05/04/FYkbGqUANRahHwZ.png)

![image-20200504214057873](https://i.loli.net/2020/05/04/wz7ycOiBAlQkmeo.png)

![image-20200504214111285](https://i.loli.net/2020/05/04/8r6fYhjtcZb4p2U.png)

他的整整 146 个 commit, 都是这个画风:

![image-20200504214125833](https://i.loli.net/2020/05/04/FhQIHmpbuXrCU9l.png)

哇, nb, 我已经可以预见到这哥们不会写任何的注释和文档了.

-----

我下载代码, 把他给代码打的证书删除, 稍微调整了一下配置文件. 我就可以编译这个软件
了\!

哇哦, 那我一开始花了六块钱买这玩意不是血亏?

算了算了.

我们赶紧来看代码, 先把这个傻逼的 bug 修了吧.

![image-20200504214246118](https://i.loli.net/2020/05/04/RkWowNbLzZOjGuh.png)

但是我眉头一皱, 发现事情并不简单, 就在上面这个图力, 输入框下面的这个下拉菜单应该
对应的是程序中的这个:

![image-20200504214302928](https://i.loli.net/2020/05/04/9VpnmC2JEjS5gTK.png)

但是为什么这玩意的两个选项, 他们并没有自己的名字, 而是用 1 和 2 编号呢???

回想一下刚刚那整整 $$146-8=132$$ 个 commit, 我释怀了, 修 bug 吧.

果不其然, 我上面的猜测是对的, 在对应的程序里我看到了这么[一
行]([https://github.com/pepeizq/Steam-Tiles/blob/master/Steam%20Tiles/Modulos/Tiles/Personalizacion.vb#L125]):

![image-20200504214320965](https://i.loli.net/2020/05/04/Oknv6D8GuQrSHKm.png)

可以看到开发者将一个叫 `CambioImagenInternet` 的函数绑定到了这个输入框的
`TextChanged` 上.

但是, 等一下, Cambio 这个词我不认识可能确实是因为我英语垃圾, 但是 Imagen 是个什
么鬼. 不是 Image 么?

为了防止我需要重新回去读小学英语, 我决定查一下 Cambio 这个词:

![image-20200504214341048](https://i.loli.net/2020/05/04/1R3J9aSqXPuNhbc.png)

哦, 那看来不是我的问题, 而是这个老哥是个西班牙人, 看了一眼 github 主页, 果然是这
样:

![image-20200504214410625](https://i.loli.net/2020/05/04/KmuysZUVv7YH1Gj.png)

好吧好吧, 但是你为什么给这个框绑一个函数得先解绑呢?

以及你这个解绑外面为什么会有一个 `If Not tbImagenInternet Is Nothing` 呢?

然后我发现了一个令我感到迷惑的事情:

这个老哥他在同一个类下面, 多次需要找到同一个控件的时候, 他每次都会写这几行代码:

``` vb
Dim frame As Frame = Window.Current.Content
Dim pagina As Page = frame.Content
```

然后再用 `pagina.FindName` 去直接通过这个控件的 id 找到它.

讲个鬼故事:

![image-20200504214441345](https://i.loli.net/2020/05/04/IW8QvDCZfqNhjPw.png)

同一份代码里面, 上面这两句话出现了十六次.

我的人生阅历增加了\! 这个奇怪的软件里出现什么样的事情, 我都不会奇怪了吧\!

大概.

-----

为了搞清楚他到底为什么要这么来绑定函数, 我打了断点看了看这个程序的逻辑,

然后我发现有的时候, 一个明明存在的控件, id 也对, 我也可以在窗口中看到这个控件的
情况下, `pagine.FindName` 也有可能会找不到它. 具体原理我并不明白.

这个时候碰巧我发现了另一个 bug, 真是一波未平一波又起.

![image-20200504214905779](https://i.loli.net/2020/05/04/E4fS1lcsz7FCvAu.png)

上面这个是调整图片大小和位置的窗口, 如果我们选中一个游戏, 编辑大磁贴, 在这里改变
Margin 和 X 还有 Y, 然后保存并返回, 再编辑中磁贴的时候, 这里的 Margin 和 X Y 的
滑块并不会归位, 而是会保持刚刚的状态.

? 这是初始化写错了么?

不对啊, [这
里](https://github.com/pepeizq/Steam-Tiles/blob/master/Steam%20Tiles/Modulos/Tiles/Personalizacion.vb#L168)
是有初始化的啊?

``` vb
Dim sliderImagenMargen As Slider = pagina.FindName("sliderPersonalizacionImagenMargen")
If Not sliderImagenMargen Is Nothing Then
    sliderImagenMargen.Minimum = 0

    RemoveHandler sliderImagenMargen.ValueChanged, AddressOf CambiarImagenMargen
    AddHandler sliderImagenMargen.ValueChanged, AddressOf CambiarImagenMargen
End If
```

难道? 第二次打开这个窗口的时候, `FindName` 找不到这个控件, 所以就没有清0 ?

我大概知道这个老哥写出这段代码的心路历程了:

> 我们现在来写初始化吧, 那么写初始化肯定得先用 `FindName` 找到对应的控件吧? 哎
> 呀? `FindName` 有的时候找不到? 找不到咋办啊? 算了算了, 找不到就不初始化了.

关于先解绑再绑定估计是这样的:

> 我现在要给这些个控件触发的事件绑定函数了\! 在哪里绑呢? 在窗口初始化里面写吧,
> 就跟在控件初始化后面\! emmm, 但是我写的这个所谓的控件初始化是每次打开窗口都会
> 运行的, 所以写在这里的代码会被运行多次, 但是一个事件又不能绑上多个函数, 那样
> 会出错的. 啊\! 我只要每次都先解绑再绑定不就可以了么\! 我真是个小天才.

我真是惊了.

-----

经过一些调试我发现这个函数第一次运行的时候, `FindName` 是真的能找到所有的控件的.
所以我稍微整理了一下, 在类里面加了几个成员, 把这些控件都存了一个引用在类里面, 这
样要用的时候就不用重新 `FindName` 了. 同时把这个先解绑然后再绑定的神代码也给整
成了只会运行一次.

现在这个软件已经基本能用了.

然后我决定再来看看这里面还有什么奇葩东西.

然后我就在[这
里](https://github.com/pepeizq/Steam-Tiles/blob/master/Steam%20Tiles/Modulos/Tiles/Personalizacion.vb#L467)
看到了一个奇葩函数. 看不懂没关系, 我一开始也看不懂. 但是我花了很大力气看懂了以
后, 我被雷到了, 这是一种很神奇的快乐, 现在我来解释一下这个东西有多奇葩:

这个函数是绑给刚才看到的那个下拉框选 inner box 还是 outer box 的. 我们可以先不管
它为什么要这么设计这个程序, 我们边看这个程序边给你大概描述一下这个函数要做的工
作是什么: 现在有两个 grid 控件, 它们一个是 `gridPersonalizacionExterior`, 是那
个 Outer box, 另一个是下面找的那个 `gridPersonalizacionInterior`, 是那个 Inner
box, 看 xaml 文件可以知道.这两个东西的关系是这样的:

    Outer box
    |- Inner box

如果图片在 Inner box 里面, 那么应该是这样的:

    Outer box
    |- Inner box
       |- Image

而如果图片在 Outer box 里面, 作者它希望他们之间的结构变成这样:

    Outer box
    |- Inner box
    |- Image

我们一起来看看作者是怎么处理这个功能的吧.

``` vb
Private Sub CambiarImagenUbicacion(sender As Object, e As SelectionChangedEventArgs)
   
   Dim frame As Frame = Window.Current.Content
   Dim pagina As Page = frame.Content
   ' 上面这两句就是那重复的16遍里的其中一遍.

   Dim cb As ComboBox = sender

   Dim imagen As New ImageEx

   Dim gridExterior As Grid = pagina.FindName("gridPersonalizacionExterior")
   ' 这代码是修改前的, 所以我们在这里还能看到作者用FindName来找控件.
   
   For Each hijo In gridExterior.Children
       ' 遍历 Outer box 的所有孩子: [ 等一下, 这是图啥, Outer box要不然是一个孩子要不然是两个孩子你快住手啊!
       Dim imagen2 As ImageEx = Nothing
       ' 创建一个 imagen2
       ' 看看 Outer box 的孩子能不能赋值给 imagen2, 如果成功了说明这个孩子是一张图, 说明图片现在在 Outer box 里面
       Try
           imagen2 = hijo
       
       Catch ex As Exception

       End Try

       If Not imagen2 Is Nothing Then 
           imagen.Source = imagen2.Source
           imagen.IsCacheEnabled = imagen2.IsCacheEnabled
           imagen.Stretch = imagen2.Stretch
   	' 把试出来的imagen2里面的图片数据,赋值给imagen[ 等一下, 那你为什么不一开始就用imagen来试呢?
           gridExterior.Children.Remove(hijo)
       End If
   Next
   ' 以上同样的事情对 Inner box 再做一次, 这两个过程中总会有一个能找到那张图.
   Dim gridInterior As Grid = pagina.FindName("gridPersonalizacionInterior")

   For Each hijo In gridInterior.Children
       Dim imagen2 As ImageEx = Nothing

       Try
           imagen2 = hijo
       Catch ex As Exception

       End Try

       If Not imagen2 Is Nothing Then
           imagen.Source = imagen2.Source
           imagen.IsCacheEnabled = imagen2.IsCacheEnabled
           imagen.Stretch = imagen2.Stretch

           gridInterior.Children.Remove(hijo)
       End If
   Next
   ' 到这里为止,我们拿到了图, 我们只要把图放到对应的框里面就写完啦!
   If cb.SelectedIndex = 0 Then
       gridInterior.Children.Add(imagen)
   ElseIf cb.SelectedIndex = 1 Then
       gridExterior.Children.Add(imagen)
   End If

End Sub
```

我一时间不知道说什么好.

-----

我 fork 了一份, 大概修了修, 修到我能用的程度就停手了, 因为我实在是, 感觉自己的智
商受到了侮辱, 甚至折损了阳寿.项目地址在[这
里](https://github.com/black-desk/Steam-Tiles), 还有很多 bug 我没管, 一开始说的
那个性能问题我也没管. 我自己编译了一份扔上去了. 要是有人想用的话可以自己查一下
怎么安装.

是的, 为了这个开始屏幕, 我刷新了我的三观.

![image-20200504214511318](https://i.loli.net/2020/05/04/AmuqEhfIndWBOgi.png)

而这一切只是为了, 把steam游戏, 放在开始屏幕上.
