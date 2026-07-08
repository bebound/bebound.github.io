# Using Chinese Characters in Matplotlib


After searching from Google, here is the easiest solution. This should also works on other languages:

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


---

> Author: KK  
> URL: https://fromkk.com/posts/using-chinese-characters-in-matplotlib/  

