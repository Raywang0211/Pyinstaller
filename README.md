# Pyinstaller
Pyinstaller practice 


# System

ubuntu18.04

# Install

```
pip install pyinstaller
```

# Basic using

Pyinstaller 主要目的是將完成的專案程式碼打包成可執行檔(.exe)，藉此讓使用者可以直接執行同時保護程式碼不被其他人看到。

## 打包單個程式碼

範例程式：(single_code.py)

```
import numpy as np

a = np.array([[1,2,3],[4,5,6]])
print("Hi pyinstaller ... ")
print("single_code.py")
print("a = ",a)
print("a.shape = ",a.shape)

```

1. 打包
    1. `pyinstaller -F single_code.py`
    2. 打包完成後會產生2個新的資料夾（dist,build）而.exe會在dist中
2. 測試
    1. `cd dist
    ./single_code`
3. 移機測試
    1. 需要先透過指令讓exe變成可被執行的
    2. `sudo chmod +x ./single_code`

## 打包有相依程式碼的程式

範例程式主程式：(package_main.py)

```
from lib.package_module import Function

lib_fun = Function()
lib_fun.Fun_1("Ray")
lib_fun.Fun_2(18)
```

範例程式副程式：(/lib/package_module.py)

```

import numpy as np

class Function():
    def __init__(self) -> None:
        print("Using Funtion")

    def Fun_1(self,name):
        print("name : ",name)
    def Fun_2(self,age):
        print("age : ",age)

```

1. 打包
    1. `pyinstall -F ./package_main.py -p /lib/package_module.py`
    
    # Advanced using

如果需要將動態連結庫打包到exe中則需要針對.spec檔案進行設定，最後針對.spec檔案進行打包

## 打包步驟

大步驟：

1. 產生.spec檔案（決定要打包成exe或是file）
2. 修改.spec檔案（注意 .so或是檔案路徑以及隱藏套件）
3. 直接透過.spec打包
4. 完成

### 產生.spec檔案

1. 透過程式碼先產生.spec檔案
    1. `pyi-makespec -F tt.py #產生 .spec`
    2. 如果使用 -F 前綴完成.spec則之後就會產生.exe檔案，如果使用-D當成前綴則會產生一個資料夾裏面存放著.exe(可以看到相關的資料以及套件)
    3. 
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/afc86e4f-4591-4684-bd72-713770f33cad/Untitled.png)
    
2. 修改.spec檔案
    1. 將需要使用到的.so檔案指定出來

```
# -*- mode: python ; coding: utf-8 -*-

block_cipher = None

a = Analysis(['main.py'],
             pathex=['/home/ray/vs_project/pyinstaller_practice/advanced_using'],
             binaries=[],
             datas=[("/home/ray/vs_project/pyinstaller_practice/advanced_using/lib/package_module.so","/lib")],
             hiddenimports=["numpy"],
             hookspath=[],
             hooksconfig={},
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)

exe = EXE(pyz,
          a.scripts,
          a.binaries,
          a.zipfiles,
          a.datas,
          [],
          name='main',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          upx_exclude=[],
          runtime_tmpdir=None,
          console=True,
          disable_windowed_traceback=False,
          target_arch=None,
          codesign_identity=None,
          entitlements_file=None )

```

基本上大部份都不需要修改，主要需要修改的部份只有[binaries,datas,hiddenimports]

1. binaries：需要打包的.so檔案需要寫在這裡
    
    binaries = [("/home/ray/vs_project/pyinstaller_practice/advanced_using/lib/package_module.so","/lib")],
    
    （要打包的.so完整路徑以及檔名，打包後要存放的位置）
    
    可以把打包想像成要把東西全部複製貼上到另一個資料夾，接著將這個資料夾上鎖，
    
    因此套件之間的路徑位置就會影響導入套件是否成功，所以要指定套件或是檔案在這個資料夾中的確切位置在
    
    如果有多個需要用 ” ，” 進行分隔
    
2. datas： 需要打包的檔案或是資料，使用方式跟binaries相同，經過實驗之後發現只要將會用到的.so檔案透過datas存放到exe當中就可以正確的執行了，不一定要透過binaries
3. hiddenimports：如果是要打包.so檔案，則pyinstall無法抓到.so中import 的其他相依套件，因此需要將抓不到的套件寫在這個位置，讓pyinstaller在打包時可以知道

---

問題蒐集

---

1. exe 中有包到tensorflow 或是keras發生下面問題
    
    ```
    ModuleNotFoundError: No module named 'keras.engine.base_layer_v1'
    
    ```
    
    需要再.spec中加入kera.engine.base_layer_v1 就可以了
    
    [https://ithelp.ithome.com.tw/m/articles/10272337](https://ithelp.ithome.com.tw/m/articles/10272337)
    
2. 抓不到 .so檔案，需要確認再打包前.spec檔案中是否有加入binary, 確認套件之間的相對位置是否正確以及import的路徑是否正確

---

[https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/472824/](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/472824/)

[http://c.biancheng.net/view/2690.html](http://c.biancheng.net/view/2690.html)

[https://zhuanlan.zhihu.com/p/165431703](https://zhuanlan.zhihu.com/p/165431703)

問題蒐集

---

1. exe 中有包到tensorflow 或是keras發生下面問題
    
    ```bash
    ModuleNotFoundError: No module named 'keras.engine.base_layer_v1'
    ```
    
    需要再.spec中加入kera.engine.base_layer_v1 就可以了
    
    [https://ithelp.ithome.com.tw/m/articles/10272337](https://ithelp.ithome.com.tw/m/articles/10272337)
    
2. 抓不到 .so檔案，需要確認再打包前.spec檔案中是否有加入binary, 確認套件之間的相對位置是否正確以及import的路徑是否正確

---

[https://codertw.com/程式語言/472824/](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/472824/)

[http://c.biancheng.net/view/2690.html](http://c.biancheng.net/view/2690.html)

[https://zhuanlan.zhihu.com/p/165431703](https://zhuanlan.zhihu.com/p/165431703)

[https://blog.csdn.net/tangfreeze/article/details/112240342](https://blog.csdn.net/tangfreeze/article/details/112240342)
