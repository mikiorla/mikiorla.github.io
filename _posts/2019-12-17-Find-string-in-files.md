---
layout: post
#title: Get-VIPermissionsOnEntity
tags: [powershell,find]
comments: true
show-avatar: false
#image: /img/ITCS-s.png
---

<!-- ### Find 'my website' string in files -->
Simple reminder on how to search for a string in some files. <br/>
Find _my website_ string in files

~~~powershell
Get-ChildItem -Recurse | % { $file=$_; if (get-content $_ -ea SilentlyContinue| Select-String "my website") {$file.fullname}}
~~~
