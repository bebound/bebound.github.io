+++
title = "Using Chinese Characters in Matplotlib"
date = 2018-10-04T15:53:00+08:00
lastmod = 2025-08-10T18:44:05+08:00
tags = ["Matplotlib"]
categories = ["Programming"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

After searching from Google, here is easiest solution. This should also works on other languages:

```python
import matplotlib.pyplot as plt
%matplotlib inline
%config InlineBackend.figure_format = 'retina'

import matplotlib.font_manager as fm
f = "/System/Library/Fonts/PingFang.ttc"
prop = fm.FontProperties(fname=f)

plt.title("你好",fontproperties=prop)
plt.show()
```

Output:

{{< figure src="/images/matplot_chinese.png" class="image-size-s" >}}
