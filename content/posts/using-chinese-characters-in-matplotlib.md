+++
title = "Using Chinese Characters in Matplotlib"
author = ["KK"]
date = 2018-10-04T15:53:00+08:00
lastmod = 2023-12-18T21:38:37+08:00
tags = ["Python", "Matplotlib"]
draft = false
noauthor = true
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

{{< figure src="/images/matplot_chinese.png" width="400" >}}
