title: 更新Xcode后Git报错
author: Yuanzi
tags:
  - Git 报错
categories:
  - Git
date: 2020-02-15 10:56:00
---
#### 问题
Accept Xcode/iOS license to Run Git

#### 解决办法
终端输入sudo xcodebuild -license
输入密码后同意条款,用空格键跳到最后

{% codeblock lang:bash %}
...
By typing 'agree' you are agreeing to the terms of the software license agreements. Type 'print' to print them or anything else to cancel, [agree, print, cancel] agree

You can view the license agreements in Xcode's About Box, or at /Applications/Xcode.app/Contents/Resources/English.lproj/License.rtf

{% endcodeblock %}