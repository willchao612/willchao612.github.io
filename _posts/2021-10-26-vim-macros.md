---
layout: post
title: '(Neo)Vim Trick | Using Macros'
subtitle: 'Vim 编辑器中「宏」的使用'
author: 'Will'
header-style: text
lang: en
tags:
  - (Neo)Vim
  - Tricks
  - 笔记
---

Macro is probably one of those hidden gems of (Neo)Vim, but most beginners don't
know the use cases although they know it's powerful.

> Recording a macro is a great way to perform a one-time task, or to get things
> done quickly when you don't want to mess with Vim script or mappings, or if
> you do not yet know how to do it more elegantly.

<iframe width="560" height="315" src="https://www.youtube.com/embed/wRFEBw02aT8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Say we need to turn this

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque
pellentesque ornare suscipit. Nam diam neque, viverra non dictum at, tempor ut
neque. Mauris feugiat convallis lacus, ac tincidunt magna consequat eu.
Vestibulum eget nibh odio. Aliquam tristique sapien id metus scelerisque,
a finibus mauris iaculis. Vestibulum ac tempus arcu. Cras.
```

... into this.

```
<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque</p>
<p>pellentesque ornare suscipit. Nam diam neque, viverra non dictum at, tempor ut</p>
<p>neque. Mauris feugiat convallis lacus, ac tincidunt magna consequat eu.</p>
<p>Vestibulum eget nibh odio. Aliquam tristique sapien id metus scelerisque,</p>
<p>a finibus mauris iaculis. Vestibulum ac tempus arcu. Cras.</p>
```

### Steps

1. Put cursor on the first letter of the first line
2. Hit `qi` to start recording into register `i`
3. Make changes to the line (`vLS<p>j` in this case)
4. Hit `q` to stop recording
5. Hit `@i` to use the macro

### Tips

1. Macros are persisted across Vim sessions
2. Hit `@@` to use last macro
3. Prepend a number to repeat multiple times
4. Append `@i` to recursively apply until the end of buffer
5. You can manually edit the content of a macro like with any register

Happy Vimming!
