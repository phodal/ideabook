文本转Logo
===

概况
---

### 背景

在设计技能树的时候需要做一些简单的Logo，方便我们来识别，这时候就想到了PIL。加上一些简单的圆角，以及特殊的字体，就可以构成一个简单的Logo。做成的图标看上去还不错:

### ShowCase

![Node](./images/node.png) ![Refactor](./images/refactor.png) ![TDD](./images/tdd.png) ![Clean Code](./images/clean_code.png)

代码见:[https://github.com/phodal/text2logo](https://github.com/phodal/text2logo)
  
### 需求说明

简单的说一些我们的附加需求

 - 圆角
 - 色彩(自动) 

步骤
---

###Step 1: Python 文字转Logo实战

一个简单的PIL生成图片的代码:

```python
# -*- coding: utf-8 -*-
from PIL import Image, ImageDraw, ImageFont

img = Image.new('L', (128, 128), 255)
draw = ImageDraw.Draw(img)
text_to_draw = unicode('xxs', 'utf-8')
font = ImageFont.truetype('fonts/NotoSansCJKsc-Regular.otf', 12)
draw.text((2, 2), text_to_draw, font=font)
del draw

img.save('build/image.png')
```

#### 圆角代码

我们需要的是在上面的代码上加上一个圆角的功能，于是Google到了这个函数

```javascript
def add_corners(im, rad):
    circle = Image.new('L', (rad * 2, rad * 2), 0)
    image = ImageDraw.Draw(circle)
    image.ellipse((0, 0, rad * 2, rad * 2), fill=255)
    alpha = Image.new('L', im.size, 255)
    w, h = im.size
    alpha.paste(circle.crop((0, 0, rad, rad)), (0, 0))
    alpha.paste(circle.crop((0, rad, rad, rad * 2)), (0, h - rad))
    alpha.paste(circle.crop((rad, 0, rad * 2, rad)), (w - rad, 0))
    alpha.paste(circle.crop((rad, rad, rad * 2, rad * 2)), (w - rad, h - rad))
    im.putalpha(alpha)
    return im
```

#### 颜色配置

在Github上找到了一个配色还不错的CSS，将之改为color.Ini，在里面配置了不同色彩与文字、前景的有关系等等，如:

	[Color]
	turqoise: #1abc9c,#ecf0f1
	greenSea: #16a085,#ecf0f1

	[Text]
	自动化测试: auto_test
	前端开发: web_front_develop

读取配置则用的是``ConfigParser``:

```python
import ConfigParser

ConfigColor = ConfigParser.ConfigParser()
ConfigColor.read("./color.ini")

bg_colors = []
font_colors = []

for color_name, color in ConfigColor.items('Color'):
    bg_colors.append(color.replace('#', '').split(',')[0])
    font_colors.append(color.replace('#', '').split(',')[1])

colors_length = ConfigColor.items('Color').__len__()
```

最后我们就可以得到我们想要的图片了~~
