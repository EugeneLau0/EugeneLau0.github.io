---
title: '使用rich库中的progress bar实现个性化进度条'
categories:
  - Blog
tags:
  - Python
---
使用rich库的progress包，实现个性化的进度条

<!--more-->

在许多耗时的场景中，为了解决用户等待的体验，我们常常会加上进度条，以使得使用者知道程序还在执行中。

# 自定义进度条

著名的rich库中就有可自定义的进度条，对此感兴趣的，可以阅读[官方文档](https://rich.readthedocs.io/en/stable/progress.html)就可以非常快速的做出一个自定义的进度条。

代码img：

![](../assets/images/attachments/20240913-image-of-rich-progress-code.png)

完整的代码

```python
import time

from rich.progress import Progress

'''
使用rich库的progress包，实现个性化的进度条

在命令终端执行，可以看到动态的效果
'''

with Progress() as progress:
    task1 = progress.add_task("[red]Downloading...", total=1000)
    task2 = progress.add_task("[green]Processing...", total=1000)
    task3 = progress.add_task("[cyan]Cooking...", total=1000)

    while not progress.finished:
        progress.update(task1, advance=0.5)
        progress.update(task2, advance=0.3)
        progress.update(task3, advance=0.9)
        time.sleep(0.02)

```

实现的效果

![](../assets/images/attachments/20240913-image-of-rich-progress-result.png)

# 探索更多

rich库里除了可以实现进度条以外，它还有需要有意思的能力，比如富文本、控制台美化、文本高亮、提示语、渲染markdown等等。