---
title: Tkinter相邻两行之间间隔太大
date: 2021-12-31 13:40
tags: 
 - tkinter
categories: 
 - Python
---

![Python](python.png) 本文主要介绍使用Tkinter时，如果没有正确配置调整大小的参数(grid_rowconfigure)，就会出现行与行之间距离过远的问题。

通过运行以下代码，可以在界面的左侧显示一个选择列表框，因为列表框内容设置为空(template_names = '')，所以列表框中没有任何可选项。在右侧显示两行文字，第一行文字内容为Incident Office1，第二行文字内容为Incident Office2。但是，第一行和第二行之间的文字距离很远。

```
from tkinter import *
from tkinter import ttk

root = Tk()
c=ttk.Frame(root, padding=(5, 5, 12, 12))
root.grid_columnconfigure(0, weight=1)
root.grid_rowconfigure(0,weight=1)
c.grid(column=0, row=0, sticky=(N,W,E,S))
c.grid_columnconfigure(0, weight=1)
c.grid_rowconfigure(0, weight=1)

template_names = ''
cnames = StringVar(value=template_names)
lbox = Listbox(c, listvariable=cnames,height=10, width=50)

lbox.grid(column=0, row=0, rowspan=12, sticky=(N,S,E,W))

for i in range(1,4):
    lbl_incident_office = ttk.Label(c, text="Incident Office%s" % i )
    lbl_incident_office.grid(column=1, row=i-1, sticky=NW)
    print(lbl_incident_office.grid_info())

root.mainloop()
```

<!-- more -->

导致出现以下问题的主要原因是没有正确配置grid中的设置项(grid_rowconfigure)。当前的设置如下，该设置第1行的元素会随着Frame的大小变化扩展空间，但是其它行的元素则保持不变。

```
c.grid_rowconfigure(0, weight=1)
```

![Tkinter](2021-12-31_132348.jpg)

通过修改配置，将其设置为第6行的元素随着Frame的大小变化扩展空间，这时候第1-6行之间就会连续排列，第6行和第7行之间间距就会变大。

```
c.grid_rowconfigure(5, weight=1)
```

![Tkinter](2021-12-31_133746.jpg)