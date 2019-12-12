# AWS_Serverless_with_zappa

AWS Lambda 可以讓你將各種演算法進行部屬，並且讓他人透過API進行操作。該服務支援PYTHON C# ...等語法，本文將紀錄如何將Django 環境的網站部屬至AWS Lambda 。我們將採用ZAPPA 工具，該工具可以幫助我們快速部屬。

以下為我們的環境與版本:

請特別注意，目前zappa 只支援到python 3.6 ，所以請以python 3.6安裝環境。

Package             Version
------------------- ----------
argcomplete         1.9.3
boto3               1.10.36
botocore            1.13.36
certifi             2019.11.28
cfn-flip            1.2.2
chardet             3.0.4
click               7.0
Django              2.1
django-s3-storage   0.13.0
docutils            0.15.2
durationpy          0.5
future              0.16.0
hjson               3.0.1
idna                2.8
jmespath            0.9.3
kappa               0.6.0
lambda-packages     0.20.0
pip                 19.3.1
placebo             0.9.0
python-dateutil     2.6.1
python-slugify      1.2.4
pytz                2019.3
pyyaml              5.2
requests            2.22.0
s3transfer          0.2.1
setuptools          42.0.2
six                 1.13.0
toml                0.10.0
tqdm                4.32.2
troposphere         2.5.3
Unidecode           1.1.1
urllib3             1.25.7
Werkzeug            0.16.0
wheel               0.33.6
wsgi-request-logger 0.4.6
zappa               0.48.2


## 設定 AWS Credientials

首先你必須要有一個AWS 帳號，並且找到你的aws_access_key_id 與 aws_secret_access_key。

並將你的aws_access_key_id 與 aws_secret_access_key。放入 /.aws/credemtials中

如果你是用windows，通常你的位置會在 C:\Users\YOURUSERID\ .aws

並在文件內寫入

```
[default]
aws_access_key_id=[...]
aws_secret_access_key=[...]
```

## 建立Django 與 Zappa 環境

首先我們要建立環境
先安裝 virtualenv, 這是一個pyhton 處理虛擬環境的套件。

```
pip install virtualenv
```

接下來建立一個資料夾，我們將以MyProject 當作資料夾名稱

```
mkdir MyProject
```
移動至該資料夾

```
cd MyProject
```

建立虛擬環境，注意，要安裝python 3.6。
我們將建立一個名叫 .env 的環境資料夾

```
virtualenv .env --python=python3.6
```

或找到你python3.6 版本的python.exe 位置。 假如你是用Anaconda，你python3.6 位置會是在你Anaconda python3.6版本的虛擬環境中。

例如今天我在Anaconda 中，有一個test_1 的虛擬環境，那他python.exe的位置會在
C:\Users\YOURUSERID\Anaconda3\envs\test_1

那麼你的安裝指令便是

```
virtualenv -p C:\Users\YOURUSERID\Anaconda3\envs\test_1\python.exe .env
```

安裝後我們可以看到我們資料夾內有一個.env 的環境資料夾

進入.env/Scripts/

```
cd .env/Scripts/
```

然後激活環境

```
activate
```
這時候檢查你的pip list 可以發現 你已經切換到新環境。

接下來，安裝django 我們使用django2.1，目前2019/12/12，django2.1對zappa環境較穩

```
pip install django==2.1
```

接下來安裝zappa ，安裝zappa 是筆者在處裡時遇到的第一個大大阻礙，大概花了一個晚上處理。

首先輸入

```
pip install zappa 
```

我們看到錯誤

```
 UnicodeDecodeError: 'cp950' codec can't decode byte 0xe2 in position 2339: illegal multibyte sequence
```

往上追蹤

```
 Collecting kappa==0.6.0
  Using cached https://files.pythonhosted.org/packages/ee/fa/1b8328d2199520ef5a257f8a2e9315ed0b0194e353a152ca1959490dfbc8/kappa-0.6.0.tar.gz
    ERROR: Command errored out with exit status 1:
```

是在安裝kappa 0.6.0 時出問題，筆者的解法是直接去手動安裝kappa 0.6.0
https://github.com/garnaat/kappa/releases/tag/0.6.0

到上面這個網址，點選左上角的 <>Code , 然後點右邊綠色的 Clone or download，選Download ZIP

下載解壓縮後進到該資料夾

```
cd kappa-0.6.0
```

進行安裝

```
python setup.py install
```

問題還是出現了，原來是setup.py 裡的解碼出問題，不能用cp950 讀，那我們就把它改成utf-8
如下圖所示
將第14行  

```
return open(os.path.join(os.path.dirname(__file__), fname))
```

改成

```
return open(os.path.join(os.path.dirname(__file__), fname), encoding='utf-8')
```

再進行一次

```
python setup.py install
```

成功安裝

這時候按一下你的 pip list 會發現 kappa 0.6.0 已經安裝完成

再次安裝zappa 

```
pip install zappa
```

成功安裝

