---
title: mac上的自动翻书
date: 2021-02-20 09:37:39
description: 懒癌重度患者的自救
tags: [懒癌, 过去]
category:
    - 100 学习类
    - 140 生产力工具
    - 143 mac
---

## 背景

工作可以不懒，为了生活我可以忍，但让我凭白翻书就不行！在mac上用预览和图书应用看书时，翻页需要滑动触控板or按动空格键，实在是太累了，何不自动按呢？故使用AppleScript自动按一下键盘的向下翻页。

## 解决方案

参考 StackOverflow 上一个好心人的[答案](https://stackoverflow.com/questions/60268384/macos-send-keystroke-to-the-active-app-periodically)，改改应用名称和[按键值](https://eastmanreference.com/complete-list-of-applescript-key-codes)，写出了第一版。

```apple script
tell application "System Events"
    repeat while (exists of application process "Preview")
        set activeApp to name of first application process whose frontmost is true
        if "Preview" is in activeApp then
            tell its application process "Preview"
                repeat while frontmost
                    key code 125
                    delay 30
                end repeat
            end tell
        end if
    end repeat
end tell
```

这样就能在预览窗口活动的情况下，每半分钟自动往下翻翻。

我就这么用了一小会之后，感觉不太方便，预览不活动的时候，它就不翻书了，很不人性化，我需要多任务，在使用其他 App 的同时，预览也在自动翻书。

研究了一会，实现了多个 App 窗口先后激活，先激活预览，再翻书，最后激活之前的窗口。

除了切换窗口那一瞬间的闪烁，堪称完美。

```apple script
repeat
	tell application "System Events"
		set appRunning to exists of application process "Preview"
	end tell
	if not appRunning then
		exit repeat
	end if
	delay 30
	tell application "Preview"
		if frontmost then
			set activeApp to "Preview"
		else
			tell application "System Events"
				set activeApp to name of first application process whose frontmost is true
			end tell
		end if
		activate
		tell application "System Events" to key code 125
		log "Next Page"
	end tell
	tell application activeApp
		activate
	end tell
end repeat
```



举一而反三，mac的图书应用亦可如此

```apple script
repeat
	tell application "System Events"
		set appRunning to exists of application process "Books"
	end tell
	if not appRunning then
		exit repeat
	end if
	delay 30
	tell application "Books"
		if frontmost then
			set activeApp to "Books"
		else
			tell application "System Events"
				set activeApp to name of first application process whose frontmost is true
			end tell
		end if
		activate
		tell application "System Events" to key code 125
		log "Next Page"
	end tell
	tell application activeApp
		activate
	end tell
end repeat

```



## 使用

打开应用，复制AppleScript脚本代码，命名为bookread.scpt文件，打开终端，执行`osascript bookread.scpt`命令即可。











