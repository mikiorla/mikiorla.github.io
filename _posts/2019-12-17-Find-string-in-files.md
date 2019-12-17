---
layout: post
#title: Get-VIPermissionsOnEntity
tags: powershell,find
comments: true
show-avatar: false
#image: /img/ITCS-s.png
---

### Find 'my website' string in files
~~~
Get-ChildItem -Recurse | % { $file=$_; if (get-content $_ -ea SilentlyContinue| Select-String "my website") {$file.fullname}}
~~~
