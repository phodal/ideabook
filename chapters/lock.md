制作简易Mac OS上的伪锁屏工具
===

概况
---

### 背景

某天中午在锁屏的时候，想到一件有意思的事。平时我会有两种“锁屏方式”：

 - 传统的锁屏方式：command + alt + 电源，适用于长时间离开。
 - 将屏幕亮度调暗，适用于上个厕所什么的。

然后在我回来的时候，我在想如果别人都知道我的屏幕是变暗的话，那么就有意思了。。。然后我就想做个简单的工具，来Mock当前的屏幕：

1. 一键截取当前屏幕
2. 打开截图将设为最大化
3. 降低屏幕亮度

接着，我就开始编写了。

步骤
---

### Step 1：屏幕截图

在Mac OS上有一个工具``screencapture``，可以用于截取当前屏幕。如：

```
screencapture screen.png
```

将图片存储为screen.png，我们只需要调用Python调用系统的命令，即可：

```bash
import os

os.system("screencapture screen.png")
```

### Step 2：调节亮度

在Mac OS上有一个工具叫``brightness``，可以用百分比调节屏幕的亮度，如

```bash
brightness 0
```

可以将屏幕的亮度调到0，即最低。所以，先安装这个工具：

```bash
brew install brightness
```

然后用Python的os模块，即可调用 。

### Step 3：全屏图片

随后，用GTK简单的弄了个全屏图片的脚本，就完成了。

```python

import os
import pygtk
import gtk

os.system("screencapture screen.png")
os.system("brightness 0")

win = gtk.Window()

im = gtk.Image()
pixbuf = gtk.gdk.pixbuf_new_from_file("screen.png")
scaled_buf = pixbuf.scale_simple(1440,900,gtk.gdk.INTERP_BILINEAR)
im.set_from_pixbuf(scaled_buf)
im.show()

vbox = gtk.VBox()
vbox.pack_start (im)
win.add(vbox)

win.fullscreen()
win.show_all()

win.set_keep_above(True);
gtk.main()
```

最后，再写个简单的函数即可：

```bash
function ss() { 
    python "/Users/fdhuang/learing/mock-screen/main.py" 
}
```
