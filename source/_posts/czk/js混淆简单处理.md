---
title: js混淆简单处理
author: czk
categories:
- js反混淆
tags: 
- js反混淆
date: 2021-04-16

---

​	工作中经常碰到一些被混淆的js代码，前面碰到的样本多多少少还有点可读性，知道大概的逻辑，最近得到了一个高混淆的js代码，不解一下完全无法分析；也没有找到特别好的接混淆的工具，也就有了本篇；

​	本篇是为了减小查看被混淆的js代码时的分析难度；简单记录一下中间使用的技术以及思路；



<!-- more -->



# 样本信息:

MD5: 53186932B9CAE96D7068F0F4069EB3FE
SHA1: 77502B806988CE2DFC2649766238C8FF24E77899



# 关键技术：

​	1.pyton调用js方法；





# 样本代码示例：

```
(function() {
	var _0x19f8 = ['dlZGV2Q=', 'V2ViVmk=', 'W25hdGk=', 'bG9hZEM=', 'c0d5THc=', 'T2JqYms=', 'RW51bWU=', 'YXNzTmE=', 'S2ZHWlQ=', 'a2V5MlA=', 'YXJjaEE=', 'aHRtYXI=', 'VnFrTU8=', 'bF9BcnI=', 'bXNXcmk=', 'U051QW4=', 'fDF8MA==', 'eHpYREc=', 'aWNlY2E=', 'VG91Y2g=', 'aW5uZXI=', 'Y29tL3M=', 'aW5jbHU=', 'VXJPSkg=', 'ZGVmZXI=', 'ZjE3', 'aXNTZWE=', 'REhBbU0=', 'cVhuQWs=', 'Zmxvb3I=', 'Y0pLa1k=', 'dVBHc2s=', 'U0ZwUlQ=', 'V2luMzI=', 'VHZkZmk=', 'REZiVXc=', 'OyBwYXQ=', 'dWZMWVQ=', 'YWRJZA==', 'IGFybQ==', 'dXp5Z2E=', 'fDJ8NHw=', 'UmVzaXo=', 'ZUV2ZW4=', 'anNFcnI=', 'bml1bV8=', 'SlBKb3g=', 'cm91bmQ=', 'TU9ybk4=', 'bXVCRnc=', 'T0VwRFo=', 'amlyRVE=', 'U3RyaW4=', 'a3l2UUc=', 'cHRxeEs=', 'QUxmZE8=', 'ZjE2', 'cXZOUlk=', 'TUl6Vks=', 'WEJ2eFk=', 'cmVy', 'aW9u', 'M3w0fDE=', 'c1RwS0g=', 'c3BpZGU=', 'dG9yc3Q=', 'Y29sbGU=', 'ZURhck0=', 'SHZ4eUI=', 'ZGRf', 'ZjEz', 'fDN8NHw=', 'dXRpbA==', 'aW5kZXg=', 'cGNTZWE=', 'Qk1YbVc=', 'RndSRnc=', 'b3Jt', 'OyBkb20=', 'UW5TR3g=', 'dFBGdmI=', 'U3pRSGk=', 'U09aRmI=', 'b2xsZWM=', 'V1pqZ3g=', 'LmJhaWQ=', 'aVBob24=', 'd2JfanM=', 'VXpUQUI=', 'VUlsY3k=', 'ZWN0RGE=', 'Q2pNZEY=', 'dWNoUG8=', 'dHVidHA=', 'UFlUcHQ=', 'aXZlcg==', 'ZGVy', 'fDEwfDE=', 'dW5kZWY=', 'aENFeVg=', 'Y2NIaU4=', 'ZGVd', 'YWFiZmY=', 'cEJzZnk=', 'cmVmZXI=', 'X19sb28=', 'NWYxYmE=', 'QUFxY2w=', 'UFpFR1A=', 'clh0Wno=', 'aW5n', 'ZjIy', 'Ly9oZWM=', 'Z2V0VGk=', 'TVdPTEY=', 'eW1iQ2c=', 'fDl8Mnw=', 'dG9HTVQ=', 'TmR4Yk8=', 'ZGZ5bk8=', 'X19fX3I=', 'WU5VbVA=', 'ZjMz', 'VXBCVnk=', 'QWdlbnQ=', 'SlR3VkM=', 'X3NlbGU=', 'ZjI5', 'bmRDbGk=', 'SlNCZWg=', 'cmVTZWc=', 'MTBfdm0=', 'a1haVXE=', 'Z2V0UGE=', 'aEJKZkk=', 'fDJ8MA==', 'dmVyc2k=', 'anNob3M=', 'aGFyZHc=', 'RmtUYk4=', 'YWNjZXM=', 'YUFpYXA=', 'cGx1Z2k=', 'ZjM1', 'V1hXZkE=', 'cGFyc2U=', 'YXJlQ28=', 'bWF4VG8=', 'Z2lmeQ==', 'UGZ3aUk=', 'NzZwZmM=', 'QVhIVmU=', 'c2V0SXQ=', 'ZjI4', 'QXJyYXk=', 'c3BsaXQ=', 'QnVmZmU=', 'YWdlbnQ=', 'dG9y', 'dE1zZw==', 'ZjIz', 'c3A5ODU=', 'dG9TdHI=', 'bmV3IFM=', 'ZFNZSGo=', 'dGhlbl8=', 'fDB8MXw=', 'ckl4emc=', 'V1ppS08=', 'V29yU20=', 'R1ZoaGI=', 'aFFURGo=', 'ZjI1', 'Qm11R3I=', 'bkdYeGc=', 'bnVtUGU=', 'VExPZWM=', 'cm9y', 'YWdlSW4=', 'RGtyQ3c=', 'RXpHWWY=', 'ZW50', 'b0lEbkM=', 'U1dDUm8=', 'c3VsdA==', 'cGdNRUI=', 'VkVZQnA=', 'cUl6TkY=', 'cnR5SXM=', 'YXJw', 'dmVmTlA=', 'bmN1cnI=', 'SVpDRWU=', 'YXNRZGk=', 'R0lpV1c=', 'M3wxfDA=', 'dG9t', 'Y29uY2E=', 'VHNwcE0=', 'ZXJ0eQ==', 'aGVpZ2g=', 'YXl1REE=', 'ZjI2', 'RmlyZWY=', 'RndQaVg=', 'TW9iaWw=', 'dWF0ZQ==', 'd2lkdGg=', 'aW5lZA==', 'UFFIVmE=', 'VFBpSG0=', 'UVBjT1Y=', 'bmJBRGI=', 'Y2FsbFM=', 'ZFdkZmk=', 'dXJFR3Q=', 'bWVudE4=', 'c3lyRkY=', 'bml1bQ==', 'YXBp', 'TGlzdEY=', 'Z2V0RW4=', 'bk16YVU=', 'dG9fXw==', 'X19mbnI=', 'ZXhlU2U=', 'Z2V0SXQ=', 'YnRqdWM=', 'dmUgY28=', 'YXBwZW4=', 'YWlkdS4=', 'ZERYRU0=', 'bXNNYXg=', 'Qk90a20=', 'ZjEy', 'WURxSVk=', 'ZjM4', 'aFFPSkw=', 'eWpOeU4=', 'dHRlcl8=', 'WkxtY2Y=', 'Y29va2k=', 'fDJ8NA==', 'ZjMy', 'SEhKSkw=', 'WUREUXo=', 'SFJ1bFE=', 'aFJlc3U=', 'c2V0VGk=', 'SldYVng=', 'dXNlckE=', 'VExKeXQ=', 'YWhrS2M=', 'TUkgUEE=', 'bWVTZWc=', 'c3RyaW4=', 'dGNfb24=', 'blByb3A=', 'ZjM5', 'SVdEZUM=', 'cig4KTs=', 'cm9w', 'ZW1lbnQ=', 'clNlZ20=', 'ZGVz', 'c2xpY2U=', 'VERXUVc=', 'c2VuZA==', 'Mzg2NzE=', 'b3RESE8=', 'R05VL0w=', 'aUZtS2Y=', 'am9pbg==', 'dFN0b3I=', 'dHlwZQ==', 'U2VnbWU=', 'ZjM3', 'X3Vud3I=', 'cmNoUmU=', 'ZmVhdHU=', 'WGZqRGE=', 'UHVIS0Y=', 'aGFyZWQ=', 'LmNvbQ==', 'X1NlbGU=', 'SHltZno=', 'ZENvbGw=', 'S0lTak4=', 'ZjIx', 'c0J5Q2w=', 'aW50cw==', 'fDExfDg=', 'NXwy', 'V3hRQk4=', 'bHpzTXY=', 'cUVudWc=', 'S2pVTWQ=', 'YWVz', 'cW93Ynk=', 'X19wcm8=', 'Ni5qcw==', 'WXNveGk=', 'QUR1cHU=', 'QkxpVGw=', 'aVdndXE=', 'ZW5pdW0=', 'YkVHdVo=', 'Z29Zdko=', 'YWFyY2g=', 'TGludXg=', 'cHJvcGU=', 'c2NyaXA=', 'ZjEw', 'RHFvZEg=', 'Q2VmU2g=', 'dS5jb20=', 'ZENoaWw=', 'Z2V0RWw=', 'Zk9xQkc=', 'Q2tZRmU=', 'ZjIw', 'UXJHalY=', 'd091blM=', 'cU1GVGI=', 'QWxs', 'ZW5jcnk=', 'cmFuZG8=', 'dHpRSFU=', 'ZjE5', 'YXlUWU0=', 'bF9Qcm8=', 'X2V2YWw=', 'Y2RjX2E=', 'ci1yZW4=', 'Y3JlYXQ=', 'UXdobnk=', 'bWlzZQ==', 'dGF0aWM=', 'bnZOR2o=', 'Rml4UFg=', 'WE1CZUI=', 'ZmFhSlQ=', 'aXNNb2I=', 'RnlVVnc=', 'RFdzTWo=', 'X3BoYW4=', 'bWF0Y2g=', 'cHVzaA==', 'YXRpYy4=', 'WUl2S3Q=', 'ZHpQdGk=', 'QWxHQ2w=', 'VEhVdmc=', 'cVNGT2o=', 'ZGF0YUw=', 'ZXJyTXM=', 'Q3RqRXA=', 'MnwwfDE=', 'Z2V0', 'SFRNTA==', 'RXZlbnQ=', 'Y2hyb20=', 'T2ttQlY=', 'cml0eVM=', 'bWVudA==', 'Y1BDcEE=', 'ZjM2', 'U3RhdGU=', 'YmNw', 'RVpqTFo=', 'ZnB5akM=', 'R2VyalY=', 'Y29kZWQ=', 'c2po', 'cHJvZHU=', 'QlNTaUk=', 'a3VwR2U=', 'UHRxSXE=', 'cG1sUVo=', 'VmpSdW8=', 'cGxhdGY=', 'TFZVY0o=', 'aUdsc2o=', 'c2NyZWU=', 'IGFybXY=', 'QW5kcm8=', 'Z2VudA==', 'SkpmVEQ=', 'ZW5jeQ==', 'dGVQcm8=', 'ZjMx', 'Z2V0RmU=', 'YXJt', 'ZGVmaW4=', 'Y2xpY2s=', 'ZjEx', 'KCkgPT4=', 'LmNvbS8=', 'dXNpbmc=', 'VXltcWg=', 'TWFyaw==', 'M2lfMWg=', 'SHNWZGI=', 'ZjMw', 'Ym9s', 'bG9JZXU=', 'Z0JJVk0=', 'eUN5dG4=', 'Q291bnQ=', 'V2Vi', 'X29iamU=', 'aURUQ3A=', 'd2luZG8=', 'c3RhY2s=', 'Q0hnVUs=', 'd2ViZHI=', 'UW93ZVU=', 'YXNuZmE=', 'YXRvcg==', 'bWVzc2E=', 'SWRxa1M=', 'NXw2fDE=', 'ZUVsZW0=', 'ZlVsbEk=', 'aXNOZWU=', 'X19zZWw=', 'ZHdWSmQ=', 'ZVFNb1U=', 'L2guZ2k=', 'dERhdGE=', 'ZWNvcmQ=', 'NHw1', 'aD0v', 'TUpJb0U=', 'Y2FsbFA=', 'SlBWbmo=', 'b25lcnI=', 'T3V0U2Q=', 'eHFQY2Y=', 'ak9iVE4=', 'cG5CYWg=', 'M3w0fDI=', 'd2Via2k=', 'UlpwS3Y=', 'TWFuYWc=', 'aHZ0RWo=', 'aXNXaW4=', 'aWxl', 'dG9yLmI=', 'QmFpZHU=', 'SFdHVE4=', 'YXZpb3I=', 'ZW5ndGg=', 'bVNZS2g=', 'cnNpb24=', 'Y3REYXQ=', 'bXFYTkE=', 'eE12SHE=', 'eEl0VG4=', 'ZG1MWUQ=', 'ZG9jX28=', 'RldCUUI=', 'b213S3o=', 'ZjE0', 'bmFtZQ==', 'ZG9RcG8=', 'bGhZbWo=', 'ZWNlVGQ=', 'b25Xa2U=', 'Z2V0VmU=', 'bF9TeW0=', 'ZjI3', 'VkFPWk4=', 'cGVybWk=', 'aGFzT3c=', 'MnwzfDY=', 'bmRpZGE=', 'MTg2NjE=', 'ZXNSbVU=', 'QkFfSEU=', 'ZWZQSlY=', 'YXBwZWQ=', 'YUpz', 'ZWxlbmk=', 'bmF2aWc=', 'JnQ9', 'aFNGa1I=', 'c2VhcmM=', 'NHwzfDY=', 'c3Npb24=', 'aW51eCA=', 'b1N5TU8=', 'ZnVuY3Q=', 'ZjQw', 'RVVHdFI=', 'U3hBbng=', 'MHwz', 'aW1nRXI=', 'UG9pbnQ=', 'akJYdlM=', 'V2luZG8=', 'Z2V0RGE=', 'ZWVxZG4=', 'bXFmSVI=', 'REpQaGY=', 'WHdpRkQ=', 'SURFX1I=', 'S1d0bG0=', 'T0lhZVA=', 'ZjE1', 'YmFpZHU=', 'RkROU3g=', 'T3Z3aXA=', 'U1ZVbGQ=', 'OyBleHA=', 'b3V0cHU=', 'bGVuZ3Q=', 'QlFqano=', 'Oi8vbS4=', 'QmFIZWM=', 'RCAy', 'aGFudG8=', 'c3Jj', 'VHlwZQ==', 'aGRTcVg=', 'b3NjcHU=', 'IGFhcmM=', 'Ym9keQ==', 'VGFyZ2U=', 'X19uaWc=', 'TldCYUw=', 'bGFMRko=', 'V2ViU2Q=', 'YVNlZ20=', 'eWRzcQ==', 'ZWdtZW4=', 'Z1ZJU2g=', 'S1RNams=', 'YWluPQ==', 'T1hYVGQ=', 'V1ZhZGs=', 'YWQtaW4=', 'bXNJc1M=', 'ZjE4', 'UkJCT2E=', 'cmFibGU=', 'amRpSkY=', 'a3lzeWs=', 'bG9naWM=', 'ZjM0', 'ZmlsZXI=', 'cG56b1c=', 'bG9nVXI=', 'ZjI0', 'N3wwfDE=', 'Q1RPUg==', 'b3JtYXQ=', 'aXJlcz0=', 'cHJvdG8=', 'V2xtTEY='];
	(function(_0x79f90c, _0x19f8a1) {
		var _0x1c20d7 = function(_0x3f1e36) {
			while (--_0x3f1e36) {
				_0x79f90c['push'](_0x79f90c['shift']());
			}
		};
		_0x1c20d7(++_0x19f8a1);
	}(_0x19f8, 0x84));
	var _0x1c20 = function(_0x79f90c, _0x19f8a1) {
		_0x79f90c = _0x79f90c - 0x0;
		var _0x1c20d7 = _0x19f8[_0x79f90c];
		if (_0x1c20['TsIdVr'] === undefined) {
			(function() {
				var _0x38ca73;
				try {
					var _0x3a69f4 = Function('return\x20(function()\x20' + '{}.constructor(\x22return\x20this\x22)(\x20)' + ');');
					_0x38ca73 = _0x3a69f4();
				} catch (_0x11a39b) {
					_0x38ca73 = window;
				}
				var _0x30cff8 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
				_0x38ca73['atob'] || (_0x38ca73['atob'] = function(_0x208603) {
					var _0x358d10 = String(_0x208603)['replace'](/=+$/, '');
					var _0x1ac552 = '';
					for (var _0x2019a2 = 0x0, _0x563110, _0x41c739, _0x3e0c39 = 0x0; _0x41c739 = _0x358d10['charAt'](_0x3e0c39++); ~_0x41c739 && (_0x563110 = _0x2019a2 % 0x4 ? _0x563110 * 0x40 + _0x41c739 : _0x41c739, _0x2019a2++ % 0x4) ? _0x1ac552 += String['fromCharCode'](0xff & _0x563110 >> (-0x2 * _0x2019a2 & 0x6)) : 0x0) {
						_0x41c739 = _0x30cff8['indexOf'](_0x41c739);
					}
					return _0x1ac552;
				});
			}());
			_0x1c20['wlqaGj'] = function(_0x1a857b) {
				var _0x859756 = atob(_0x1a857b);
				var _0x51066a = [];
				for (var _0x894af0 = 0x0, _0x317a97 = _0x859756['length']; _0x894af0 < _0x317a97; _0x894af0++) {
					_0x51066a += '%' + ('00' + _0x859756['charCodeAt'](_0x894af0)['toString'](0x10))['slice'](-0x2);
				}
				return decodeURIComponent(_0x51066a);
			};
			_0x1c20['MUJepi'] = {};
			_0x1c20['TsIdVr'] = !![];
		}
		var _0x3f1e36 = _0x1c20['MUJepi'][_0x79f90c];
		if (_0x3f1e36 === undefined) {
			_0x1c20d7 = _0x1c20['wlqaGj'](_0x1c20d7);
			_0x1c20['MUJepi'][_0x79f90c] = _0x1c20d7;
		} else {
			_0x1c20d7 = _0x3f1e36;
		}
		return _0x1c20d7;
	};
	var _0x513e8c = {};
	_0x513e8c[_0x1c20('0x105') + 'e'] = function(_0x41a471) {
		this[_0x41a471[_0x1c20('0x14c')]] = _0x41a471;
	};
	_0x513e8c[_0x1c20('0x10a')] = function(_0xc3447e) {
		return this[_0xc3447e];
	};
	var oo = _0x513e8c;
	...
				case '4':
					var util = oo[_0x1c20('0x10a')](_0x1c20('0x1f4'));
					continue;
			}
			break;
		}
	} catch (_0x3dd864) {
		if (typeof util[_0x1c20('0x87')] === _0x1c20('0x168') + _0x1c20('0x1e9')) {
			var _0x29dd62 = {};
			_0x29dd62[_0x1c20('0x8e')] = _0x1c20('0x1d8') + 'or';
			_0x29dd62[_0x1c20('0xf2') + 'ct'] = api[_0x1c20('0xf2') + 'ct'];
			_0x29dd62[_0x1c20('0x119')] = _0x3dd864[_0x1c20('0x119')] || '';
			_0x29dd62[_0x1c20('0xdf') + 'g'] = _0x3dd864[_0x1c20('0x11f') + 'ge'] || '';
			_0x29dd62[_0x1c20('0x4') + 'on'] = api[_0x1c20('0x4') + 'on'];
			util[_0x1c20('0x87')](_0x29dd62);
		}
	}
})();	
```

上面只展示了部分样本代码，开头部分定义了一个匿名函数；

其中 _0x1c20 是方法名，有一个参数；其中大量的调用了此方法；

现在不知道这个方法的用途，但结合多次出现的方法后面加上一个字符串，大胆猜测方法返回的应该是个串；

我们尝试在浏览器中调试一下；



# 浏览器中调试：

首先浏览器中打开一个空白页，按F12,再切换到Console视图，把 _0x1c20 方法代码以及此方法前面到文件头部分都复制到console中，然后再在下面的调用方法中挑一个，这里选择  _0x1c20('0x105')，输入到console中，最后回车；

此时console中如下图：

![chromedebugjs](/pic/czk/jsConfusion/chromedebugjs.png)

现在可以看到执行  _0x1c20('0x105')  输出的是 “defin”，上面猜测对了，_0x1c20方法就是一个解密字符串的；



# python代码解密：

混淆后的代码中大量使用了这种方法加密字符串，可读性太差，如果能把这样的加密解密出来就好了，手动在浏览器中动态调试工作量有点大，此时我想到了用python脚本实现，让python调用js函数；

网上找一找还真支持这样的功能；



下面把实现代码贴出来：

```
#!/usr/bin/python3
# coding:utf-8

import os, sys
import execjs

def get_js():
 f = open("func_1c20.js", 'r', encoding='utf-8')
 line = f.readline()
 htmlstr = ''
 while line:
  htmlstr = htmlstr + line
  line = f.readline()
 return htmlstr
 
def get_test(funcname, funcarg):
 jsstr = get_js()
 ctx = execjs.compile(jsstr)
 # 调用js方法 第一个参数是JS的方法名，后面的为js方法的参数
 return ctx.call(funcname, funcarg)
 
#print( get_test('_0x1c20','0x105') )

def main():	
	ResultFile	=	open('Sample_Result.js','w')
	JSFile	=	open('Sample.js','r',encoding='utf-8')
	LineDatas	=	JSFile.readlines()

	for linedata in LineDatas:
		while( (linedata.find('_0x1c20(')) != -1):
			param = linedata.split('_0x1c20(')[1].split(')')[0]
			jsResult = get_test('_0x1c20',param.strip('\''))			
			replaceSrc	=	'_0x1c20(' + param + ')'
			replaceResult = linedata.replace(replaceSrc, '\'' + jsResult + '\'')
			linedata	=	replaceResult
		#处理字符串'+'拼接
		if '\' + \'' in linedata:
			linedata	=	linedata.replace('\' + \'','')		
		ResultFile.write( linedata )

if __name__ == '__main__':
    main()
```

我把放在浏览器console中的代码单独保存在了 func_1c20.js 文件中，

Sample.js就是需要处理的被混淆文件；

Sample_Result.js  就是新生成的结果文件；

这段代码的逻辑就是 逐行解析被混淆的js文件，把每处调用_0x1c20方法的地方都调用方法得到解密后的字符串，再替换回原处；但是 源代码还有大量的  _0xb877e1[_0x1c20('0x130') + 'or'] 类似这样的字符串拼接代码；所以上面代码中处理完每行的替换后，再次处理一下字符串拼接，得到一个完成的串，这样看着更直观一些；



# 处理结果对比：

下图就是处理前后的对比图，右边的是处理后的；可读性强了不少；

![image-20210415155405067](/pic/czk/jsConfusion/calljsresults.png)





观察右侧的代码还有需要解密的地方；

65行处定义了一个  数组 _0x19ea70，紧接着每个成员都被赋值了一个方法，两个参数，后面都是使用的方法成员；

这里后面就不继续进行了，大体解密逻辑也按使用上面的思路，把定义的方法摘出来都放到func_1c20.js中，把每个使用方法的地方都替换成解密后的内容，我看有的地方还有方法作为参数的，嵌套调用，这些处理的时候需要特殊注意下；

好了本文到此结束；





