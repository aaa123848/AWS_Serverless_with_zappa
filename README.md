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


## 建立Django 專案

這部分沒啥神奇的，就是建立django 專案，建立就是create 的意思

確認你在Myproject中

然後建立

django-admin startproject Myproject

這時候你就可以發現你的MyProject 資料夾中 又冒出一個 Myproject資料夾
這邊p分大小寫是故意的，讓各位大概分得清楚結構。

目前結構大概為

MyProject

-.env

-Myproject

--Myproject

--db.sqlite3

--manage.py

進入Myproject

```
cd Myproject
```

輸入
```
python manage.py runserver
```

就可以利用django 自己的袖珍迷你伺服器進行測試

將 http://127.0.0.1:8000/

放入你瀏覽器的網址中，enter進去就可以看到django 專安設置成功的畫面

接下來就是要把Django 部屬到AWS Lambda 上了。

## 用ZAPPA 部屬

在部屬前，確認你的 aws_access_key_id 與 aws_secret_access_key 已經放入/.aws/credemtials中，
要不然ZAPPA 會沒有權限進行AWS 佈署，很可憐。

接下來輸入
```
zappa init
```

然後ZAPPA 大招牌就跳出來歡迎你

```
███████╗ █████╗ ██████╗ ██████╗  █████╗
╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
  ███╔╝ ███████║██████╔╝██████╔╝███████║
 ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
███████╗██║  ██║██║     ██║     ██║  ██║
╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝
Welcome to Zappa!
Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!
Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'):
```

首先我們佈署的stages 選dev, 這樣在APIGateway 那邊，ZAPPA就會幫你建立DEVELOP 的 STAGE

```
AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
We found the following profiles: default, and hdx. Which would you like us to use? (default 'default'): 
```

這邊ENTER 直接default 下去

```
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want call your bucket? (default 'zappa-108wqhyn4'): django-zappa-sample-bucket
```

幫你的s3水桶取個名子，s3水桶是要全球獨一無二的，所以可以根據你所在的時間與專案名取名。

```
It looks like this is a Django application!
What is the module path to your projects's Django settings?
We discovered: django_zappa_sample.settings
Where are your project's settings? (default 'django_zappa_sample.settings'): 
```
ENTER
```
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: 
```
ENTER
```
{
    "dev": {
        "aws_region": "us-east-1", 
        "django_settings": "django_zappa_sample.settings", 
        "profile_name": "default", 
        "project_name": "django-zappa-sa", 
        "runtime": "python2.7", 
        "s3_bucket": "django-zappa-sample-bucket"
    }
}


Does this look okay? (default 'y') [y/n]: y
```
迴車(ENTER)

然後他就會幫你在Myproject中創立一個zappa_setting.json 的檔案。

zappa_setting.json這東西請確定一定要擺在 manage.py 同樣的地方，架構上來說如下

MyProject

-.env

-Myproject

--Myproject

--db.sqlite3

--manage.py

--zappa_setting.json

筆者因為這沒擺到對的位置，卡了一天半，卡到夢到爺爺，很可憐的。

接下來進行部署了

```
zappa deploy dev
```

```
Calling deploy for stage dev..
Creating zappatest-dev-ZappaLambdaExecutionRole IAM Role..
Creating zappa-permissions policy on zappatest-dev-ZappaLambdaExecutionRole IAM Role.
Downloading and installing dependencies..
Packaging project as zip..
Uploading zappatest-dev-1496245095.zip (11.0MiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 11.5M/11.5M [00:15&lt;00:00, 751KB/s]
Scheduling..
Scheduled zappatest-dev-zappa-keep-warm-handler.keep_warm_callback!
Uploading zappatest-dev-template-1496245132.json (1.6KiB)..
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1.61K/1.61K [00:01&lt;00:00, 1.23KB/s]
Waiting for stack zappatest-dev to create (this can take a bit)..
 75%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▎                                      | 3/4 [00:10&lt;00:05,  5.48s/res]
Deploying API Gateway..
Deployment complete!: https://hashrajwrlajlralhaojp.execute-api.us-west-2.amazonaws.com/dev
```

可以看看最下面，ZAPPA吐了一個網址給你，那個就是你網站的API了，你把它貼到瀏覽器上，
就會出問題。

你得把你的API 貼到 DJANGO 的 setting.py 的ALLOW_HOST中

setting.py 在 MyProject/Myproject/Myproject中

MyProject

-.env

-Myproject

--Myproject   <=這個資料夾立面

--- ...

---setting.py

--- ...

--db.sqlite3

--manage.py

--zappa_setting.json

打開setting.py
然後找到ALLOW_HOST
按以下輸入
```
ALLOWED_HOSTS = ['hashrajwrlajlralhaojp.execute-api.us-west-2.amazonaws.com', 
    '127.0.0.1',
    ]
```

這樣你的django 就允諾這個API了。

至於'127.0.0.1' 是為了方便你之後本機端測試用

不知道什麼意思就把'127.0.0.1'拿掉，然後

```
python manage.py runserver 
```

就知道了

然後接下來要把你更新的CODE 更新到LAMBDA上

```
zappa update dev
```

就佈署上去了