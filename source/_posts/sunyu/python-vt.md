---
title: 快速辨黑白脚本-python
author: 此生汝梦 
categories:
- 脚本工具
- python
tags: 
- python 
- 脚本 
- virustotal
date: 2020-05-22 15:13:00
excerpt: 使用python编写，能够批量解压缩，并将解压后的文件自动上传VT获取查杀结果 
---


# 整体描述
因为工作原因，我经常需要通过VirusTotal简单确认样本的黑白，但大多数样本获取时到时是一个加密的压缩包。解压缩，上传样本，获取vt扫描信息，这是一个重复且费时间的一个事情，如果写一个脚本就会简单快速很多。
整个脚本，可以分为两大部分，解压部分，上传扫描部分，这两部分又都涉及文件路径获取。所以整个脚本可以分成四个部分，文件操作部分，解压部分，VT扫描部分，汇总部分。

# 文件操作部分
*  需要导入的包
```
    import os
```
* 下属方法
1. 获取文件夹路径下的所有文件和文件夹路径并打印(会进行迭代遍历所有的) 
```
def traversalDirectory(directoyrPath, print_Str = False):
    # 获取当前文件夹路径
    directoyrPath_ = directoyrPath
    if os.path.isfile(directoyrPath):
        directoyrPath_ = os.path.dirname(directoyrPath)
    # 获取当前文件夹下的所有文件和目录
    fileAndDirectoyrList = os.listdir(directoyrPath_)
    # 将当前文件夹下的文件路径加入到文件路径列表
    # 将当前文件夹下的文件夹路径加入到文件夹路径列表
    fileList = []
    dirList = []
    for fileOrDirectoyr in fileAndDirectoyrList:
        if os.path.isfile(os.path.join(directoyrPath_, fileOrDirectoyr)):
            fileList.append(os.path.join(directoyrPath_, fileOrDirectoyr))
        else:
            dirList.append(os.path.join(directoyrPath_, fileOrDirectoyr))
            dir_fileOrDirectoyr_ = traversalDirectory(os.path.join(directoyrPath_,  fileOrDirectoyr))
            fileList = fileList + dir_fileOrDirectoyr_["fileLsit"]
            dirList = dirList + dir_fileOrDirectoyr_["dirLsit"]
    if print_Str:
        for filePath in fileList:
            print("文件：", filePath)
        for dirPath in dirList:
            print("文件夹：", dirPath)
    fileOrDirectoyr_ = {"fileLsit": fileList, "dirLsit": dirList}
    return fileOrDirectoyr_
```
2.获取文件夹路径下的所有文件路径并打印
```
def traversalDirectory_file(directoyrPath, print_Str = False):
    # 获取文件夹路径
    directoyrPath_ = directoyrPath
    if os.path.isfile(directoyrPath):
        directoyrPath_ = os.path.dirname(directoyrPath)
    # 获取当前文件夹下的所有文件和目录
    fileAndDirectoyrList = os.listdir(directoyrPath_)
    # 将当前文件夹下的文件路径加入到文件路径列表
    fileList = []
    for fileOrDirectoyr in fileAndDirectoyrList:
        if os.path.isfile(os.path.join(directoyrPath_, fileOrDirectoyr)):
            fileList.append(os.path.join(directoyrPath_, fileOrDirectoyr))
    if print_Str:
        for filePath in fileList:
            print("文件：", filePath)
    return fileList
```
3.获取文件名(extensionName-是否带拓展名，默认是带)
```
def getFileName(filePath, extensionName = True):
    if os.path.exists(filePath):
        if os.path.isfile(filePath):
            if extensionName:
                return os.path.basename(filePath)
            else:
                return os.path.splitext(os.path.basename(filePath))[0]
        else:
            return ""
    else:
        print(filePath, "不存在")
        return ""
```
4.获取类型(其实就是获取文件拓展名)
```
def getFileType(filePath):
    if os.path.exists(filePath):
        if os.path.isfile(filePath):
            return os.path.splitext(filePath)[1].split(".")[1]
        else:
            return ""
    else:
        print(filePath, "不存在")
        return ""
```
5.检查文件夹是否存在(make-当文件夹不存在时，是否创建，默认是不创建的)
```
def checkDir(path, make = False, print_Str = False):
    if not os.path.exists(path) and make:
        os.makedirs(path)
        if print_Str:
            print(path, "不存在，但已创建")
        return True
    if os.path.exists(path) and os.path.isdir(path):
        return True
    else:
        if print_Str:
            print(path, "不存在或不是文件夹")
        return False
```
# 解压部分
* 需要导入的包

```
import os
import zipfile
import rarfile
import 文件操作
```
* 下属方法
1.解压RAR
```
def unRar( filePath, unFilePath, filePassword = None):
    if not rarfile.is_rarfile(filePath):
        print(文件操作.getFileName(filePath), "不是RAR文件")
        return False
    rarfile.UNRAR_TOOL = 'C:\\Program Files\\WinRAR\\UnRAR.exe'
    rarfileObj = rarfile.RarFile(filePath)
    if rarfileObj.needs_password() and filePassword != None:
        rarfileObj.setpassword(filePassword)
    elif rarfileObj.needs_password() and filePassword == None:
        print(filePath, "需要密码")
        rarfileObj.close()
        return False
    try:
        rarfileObj.extractall(unFilePath)
        return True
    except rarfile.Error as e:
        print(文件操作.getFileName(filePath), "解压失败")
        print("错误提示：", e)
    finally:
        rarfileObj.close()
    return False
```
在这个函数中需要注意**rarfile.UNRAR_TOOL**这个属性的值，因为rar算法并不公开所以，需要先下载**安装winRAR然后将WinRAR中的UnRAR.exe路径填写到此处**。不然一定会报错。

2.解压zip
```
def unZip( filePath, unFilePath, filePassword = None):
    if not zipfile.is_zipfile(filePath):
        print(文件操作.getFileName(filePath), "不是ZIP文件")
        return False
    zipFileObj = zipfile.ZipFile(filePath)
    zipFileObj.setpassword(filePassword)
    try:
        zipFileObj.extractall(unFilePath)
        return True
    except zipfile.BadZipFile as e:
        print(文件操作.getFileName(filePath), "解压失败")
        print("错误提示：", e)
    finally:
        zipFileObj.close()
    return False
```
3.解压7z
```
def un7z( filePath, unFilePath, filePassword = None):
    unState = False
    cmd = "7z x " + os.path.abspath(filePath) + " -y " + " -o" +  os.path.abspath(unFilePath)
    if filePassword != None:
        cmd = "7z x "+os.path.abspath(filePath)+" -p"+filePassword+" -y "+"  -o"+os.path.abspath(unFilePath)
    if os.system(cmd) == 0:
        unState = True
    if unState:
        return True
    else:
        print(文件操作.getFileName(filePath), "解压失败，可能是密码错误")
        return False
```
 这个函数需要注意，需要**提前安装7z**，并在**用户环境变量中添加7z安装路径**。因为是通过命令行调用7z所以需要提前设置环境变量，经过尝试仅在系统环境变量中添加7z安装路径时，os.system这个函数并不能正确调用7z，只有在用户环境变量中也添加7z安装路径才能解决这个问题。
![ALt text](/pic/sunyu/python-vt/7z环境变量.png)

4.根据不同的类型进行解压
```
def switch_Unzip( fileType, filePath, unFilePath, filePassword=None):
    # 把后缀名全部转成小写然后进行对比
    if fileType.lower() == "zip":
        unZip(filePath, unFilePath, filePassword.encode())
    elif fileType.lower() == "rar":
        unRar(filePath, unFilePath, filePassword)
    elif fileType.lower() == "7z":
        un7z(filePath, unFilePath, filePassword)
```
5.整合-如果文件名称中带密码则尝试自动填入
```
def unDirectory(zipPath, uZipPath, password="infected"):
    filePathList = 文件操作.traversalDirectory_file(zipPath)
    文件操作.checkDir(uZipPath, True, True)
    for filePath in filePathList:
        print("正在解压", 文件操作.getFileName(filePath))
        fileType = 文件操作.getFileType(filePath)
        uZipFilePath = os.path.join(uZipPath, 文件操作.getFileName(filePath, False))
        try:
            fileNameStr = 文件操作.getFileName(filePath, False)
            passwordStr_index = fileNameStr.index("密码")+2
            passwordStr = fileNameStr[passwordStr_index:]
            if passwordStr[0:1] == ":":
                passwordStr_index = passwordStr_index+1
                passwordStr = fileNameStr[passwordStr_index:]
            switch_Unzip(fileType, filePath, uZipFilePath, passwordStr)
        except ValueError as e:
            switch_Unzip(fileType, filePath, uZipFilePath, password)
        if not 文件操作.checkDir(uZipFilePath):
            print(文件操作.getFileName(filePath), "解压失败")
```
# VT扫描部分
* 需要导入的包
```
from virustotal_python import Virustotal
import hashlib
import os.path
import time
import 文件操作
```
* 初始化
星号部分为你的VT账号的API_Key
```
# 初始化对象，填入API_Key
vtotal = Virustotal("***************************************************************")
```
登录VT后，点击API key
![ALt text](/pic/sunyu/python-vt/ApiKey-1.png)
在新转到的页面中，红框部分即为你自己的API key，黄框部分为通过该API key进行请求的速率，如图所示普通免费用户为**每分钟4次请求，每天1000次请求，每个与30000次请求**。
![ALt text](/pic/sunyu/python-vt/ApiKey-2.png)
这里需要注意一点的是，这是按照**网络请求来消耗次数**的，那怕提交的是重复的样本哈希值也会消耗次数。
* 下属方法
1.扫描单个文件
```
def scanfile(filePath):
    file_hash = None
    if os.path.isfile(filePath):
        f = open(filePath, "rb")
        f_buffer = f.read()
        f.close()
        file_hash = hashlib.md5(f_buffer).hexdigest()
    file_hash_list = []
    file_hash_list.append(file_hash)
    try:
        resp = vtotal.file_report(file_hash_list)
    except Exception as e:
        print(e)
        time.sleep(5)
        try:
            resp = vtotal.file_report(file_hash_list)
        except Exception as err:
            print("重试失败")
            print(err)
            return None
    file_report_num = 0
    while resp["status_code"] == 204:
        print("超过请求速率",filePath,"将在60S后重试")
        time.sleep(60)
        resp = vtotal.file_report(file_hash_list)
        file_report_num = file_report_num + 1
        if file_report_num == 4:
            return resp
    # pprint(resp)
    if resp["json_resp"]["response_code"] == 0:
        resp = vtotal.file_scan(filePath)
        if resp["json_resp"]["response_code"] == 0:
            print("上传失败")
            return None
    print(filePath, resp["json_resp"]["positives"])
    return resp
```
先计算文件哈希值，然后使用该文件哈希值进行请求，如果文件哈希在VT上未记录则上传文件，因为上传文件进行检测，其扫描优先级比较低，会比较耗时间，所以需要等一段时候后重新通过哈希值获取检测结果。
在实际测试中发现file_report该API可能会因为网络原因出错，所以此处做了错误处理，等待5s后重新尝试获取检测结果。(当我写好此处错误处理后，再进行测试时再未发现会抛出网络错误，不知道是什么原因，所以暂时这样处理)


2.批量上传文件，返回需要重新检查的文件的路径
```
def scanfiles(filepaths):
    testagain = []
    for file_path in filepaths:
        resp = scanfile(file_path)
        if resp == None:
            print(file_path, "上传失败")
        elif resp["status_code"] == 204:
            print(file_path, "扫描失败")
        elif not ("positives" in resp["json_resp"]):
            print(file_path, "排队等待重新上传")
            testagain.append(file_path)
        time.sleep(10)
    return testagain
```
3.扫描指定文件夹下的文件
```
def scanfiles(dirPath):
    filesAndDirectoyrs = 文件操作.traversalDirectory(dirPath)
    filesPath = filesAndDirectoyrs["fileLsit"]
    testagain = virustotal.scanfiles(filesPath)
    if len(testagain) != 0:
        for filePath in testagain:
            print(filePath, "请稍后重新查询")
```
# 汇总部分
* 需要导入的包
```
import 解压
import os
import getopt
import sys
import virustotal
```
* 下属方法
主函数-根据命令行参数进行操作
```
if __name__ == '__main__':
    try:
        opts, args = getopt.getopt(sys.argv[1:], "-z:-u:-p:-s-S:")
    except getopt.GetoptError as err:
        print(err.opt, "参数填写错误")
        sys.exit(2)
    passwordStr = ""
    zipPath = ""
    unzipPath = ""
    scan = ""
    scan_bool = False
    for opt_name, opt_value in opts:
        if opt_name == "-z":
            zipPath = opt_value
            print(zipPath)
        if opt_name == "-u":
            unzipPath = opt_value
            print(unzipPath)
        if opt_name == "-p":
            passwordStr = opt_value
            print(passwordStr)
        if opt_name == "-S":
            scan = opt_value
            print(scan)
        if opt_name == "-s":
            scan_bool = True
    if zipPath != "":
        if os.path.isdir(zipPath):
            解压.unDirectory(zipPath, unzipPath)
        if scan_bool:
            virustotal.scanfiles(unzipPath)
    if scan != "":
        if os.path.isdir(scan):
           virustotal.scanfiles(scan)
```
# 总结
* 总结
    1.大体完成了批量解压文件，然后将解压的文件上传VT获取报毒的引擎数量这个功能
    2.文件操作部分，主要用于遍历文件夹路径下的文件，并将文件路径返回，以及一些获取文件类型（后缀名）等操作
    3.解压部分，因为主要面对的三种类型的压缩文件并没有一个统一的库可以使用，所以这里使用了3种方法来应对这三种压缩文件。
    4.vt扫描部分，主要是调用现有API来获取扫描结果，并对获取的结果进行一定程度上的筛选，比如现在只打印报毒引擎数，就是筛选后的结果。
    5.把不同的功能分割成了不同的模块，在一定程度上降低了代码的耦合度，使得修改更加容易
* 不足
    1.使用起来还是有一些繁琐，需要在代码中写死路径，APIkey，在灵活性上来说还是差了一点。
    2.查询结果目前只显示了报毒引擎的数量，并没对查询结果做进一步的筛查，没有生成相对应的持久化文件，并不方便于后续的样本处理。
    3.没有做过大量测试，可能还存在一些未知的设计错误。
* 下一步改进
    1.自动生成配置文件，代码读取配置文件中的rar组件路径，APIkey等信息，提高灵活性
    2.对查询结果进行一次筛选，生成需要的相对应查询结果文件。
##### 项目地址：
    https://github.com/770807993/batch_Unzip_VT_python