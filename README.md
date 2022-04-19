# 在ubuntu/kubuntu中安裝MySQL Server

本篇會介紹:
1. 安裝與初始化MySQL Server
2. 管理資料庫與用戶
3. 自動備份資料庫與還原
4. 遠端登入MySQL Server

***

## 前言

最近買了新筆電灌了ubuntu，想說那就順便來架看看MySQL的資料庫好了。以前對SQL了解不深，基本上就只會幾個簡單的指令從資料庫中拿出資料和存入資料而已，正好趁這次機會補全一下這方面的知識。
本篇使用的是VirtualBox安裝的kubuntu-20.04作為例子，kubuntu跟ubuntu的差別只有桌面環境不一樣而已，所以本篇的內容可以直接套在ubuntu上也是可以的。

***

## 安裝與初始化MySQL Server

在一切開始前請先更新`apt`:

        $ sudo apt update

### 安裝MySQL Server

安裝 MySQL Server:
        
        $ sudo apt install mysql-server

他會問你是否要安裝？輸入y安裝。
![alt tag](https://imgur.com/8ZsAcuO.png)

### 啟動MySQL服務

安裝好資料庫後，我們就要對資料庫進行初始化的工作了，但在開始初始化之前請先確認你的資料庫是否已經啟動了:

        $ systemctl status mysql.service

![alt tag](https://imgur.com/y5olsL1.png)

看到圖中綠色的`active(running)`就說明服務已經啟動了。

如果服務沒有啟動，可以用`systemctl`起動MySQL的服務:

        $ systemctl start mysqld


結束後可以使用`netstat`查看MySQL使用的是什麼端口(默認是3306)：

        $ sudo netstat -tnlp

### 初始化MySQL

下面進行資料庫的初始化:

        $ sudo mysql_secure_installation

輸入初始化指令後會看到下面這個畫面:
![alt tag](https://imgur.com/NkR9l9X.png)
這邊會問你要不要設置驗證密碼插件(VALIDATE PASSWORD PLUGIN)，沒有特別的需要的話，這邊選擇不要就好了。

然後會叫你設定資料庫的root密碼:
![alt tag](https://imgur.com/mRRIXiJ.png)

系統會詢問你是否移除匿名用戶，這邊我們為了安全選則移除(y):
![alt tag](https://imgur.com/9gifh49.png)

系統會詢問你是否禁止root遠程登入，這邊我們為了安全選則禁止(y):
![alt tag](https://imgur.com/x8Y1tz8.png)

系統會詢問你是否刪除測試用的資料庫:
![alt tag](https://imgur.com/D79tCTS.png)

系統會詢問你是否重新載入權限表，這邊建議重新載入(y):
![alt tag](https://imgur.com/bZVrx2I.png)

### 測試是否安裝成功

這邊我們直接登入看看來測試是否安裝成功，我們使用root身份登入，後面我們再建立其他用戶。
登入的指令如下:

        $ sudo mysql -u用戶名 -p密碼
        
或是使用下面的方式登入，使用下面的方式登入則會需要使用交互的方式輸入密碼。

        $ sudo mysql -u 用戶名 -p

我們是使用root登入，所以是:

        $ sudo mysql -u root -p

當畫面最底下出現`mysql>`就說明已經成功登入MySQL Server了。

最後我們可以輸入`exit;`離開。

***

## 管理資料庫與用戶

前面我們架好了MySQL Server，下面我們要來管理MySQL Server了。具體來說，我們主要要做的事情就是：
1. 建立資料庫和資料表。
2. 新增用戶，並設定用戶權限。

### 建立資料庫和資料表

首先，我們在本地使用root用戶登入:

        $ sudo mysql -u root -p

登入後會出現`mysql>`符號，然後我們可以使用下面的指令來建立資料庫`stock_db`:

        mysql> CREATE DATABASE stock_db;

然後可以我們進入資料庫`stock_db`，並在資料庫中建立資料表`test_stock`:

```
mysql> USE stock_db;

mysql> CREATE TABLE test_stock (        # 新增資料表stock
    `ID` INT NOT NULL AUTO_INCREMENT,   # 新增id
    `TradeDate` DATE NULL,              # 新增交易日期
    `StockID` VARCHAR(32) NULL,         # 新增股票代號
    `ClosePrice` DECIMAL(9,3) NULL,     # 新增收盤價
    `Volumn` INT NULL,                  # 新增成交量
    PRIMARY KEY (`ID`));                # 主要索引
```

建立成功後可以插入測試資料看看：

        mysql> INSERT INTO test_stock (TradeDate, StockID, ClosePrice, Volumn)
            VALUES ('2022-4-15', '0050', 132.25, 20628);
            
        mysql> INSERT INTO test_stock (TradeDate, StockID, ClosePrice, Volumn)
            VALUES ('2022-4-18', '0050', 131.55, 17915);

查詢資料表`test_stock`中的所有資料：

        mysql> SELECT * FROM test_stock;


### 新增用戶，並設定用戶權限

下面是建立用戶的指令格式:

        CREATE USER 用戶名@HOST IDENTIFIED BY 密碼;

其中`HOST`是指定可以登錄的主機，如果填的是`localhost`表示本機，`%`則表示所有主機。
比如我們要創立用戶`testuser`，密碼是`123`:

        CREATE USER 'testuser'@'%' IDENTIFIED BY '123';        

創立一個用戶後，user表中會插入一行新的數據，但是這時候該用戶是沒有任何權限的。所以我們接下來要設定用戶權限，命令格式如下:

        GRANT 操作權限 ON 資料庫.資料表 TO 用戶名@HOST;

其中`操作權限`是用戶在指定的`資料庫.資料表`中所可以擁有的權限，比如:`SELECT`, `INSERT`, `UPDATE`。如果要授予所有權限用`all privileges`。而`資料庫`和`資料表`可以用`*`來表示所有的資料庫或資料表，比如我要賦予用戶`testuser`對資料庫`stock_db`中的所有資料表擁有全部的權限，則可以表示為:

        GRANT ALL privileges ON stock_db.* TO 'testuser'@'%';
        
建立好新用戶並設定好權限後，我們可以登出目前的`root`用戶並使用`testuser`用戶登入:
        
        $ sudo mysql -u testuser -p

***

## 自動備份資料庫與還原

要讓資料庫能夠自動備份，我們需要:
1. 編寫備份資料庫的腳本
2. 使用cron定期執行腳本

### 編寫備份資料庫的腳本

備份資料庫中特定表的指令如下:

        $ mysqldump -u用戶名 -p密碼 資料庫名 [資料表1 ...] > 備份檔案名.sql

備份多個資料庫指令如下:

        $ mysqldump -u用戶名 -p密碼 --databases 資料庫名1 資料庫名2 ... > 備份檔案名.sql

備份所有資料庫指令如下:
        
        $ mysqldump -u用戶名 -p密碼 --all-databases > 備份檔案名.sql

使用上面的指令可以將資料庫備份到`備份檔案名.sql`檔案中，一般備份檔案的名稱會加入備份時的時間方便辨識。下面我們來編寫自動備份的腳本(`sql_dump.sh`)並將其放到家目錄(`~`)下:
        
        #!/bin/bash
        
        file_name="sqldump_"$(date +'%Y%m%d')".sql"
        
        mysqldump -utestuser -p123 --databases stock_db > ~/$file_name

別忘了加上執行權限:
        
        $ chmod a+x ~/sql_dump.sh
        
**`常見問題:`**
當你使用上面的語法進行備份時可能會發現系統會報錯:

        mysqldump: [Warning] Using a password on the command line interface can be insecure.
        mysqldump: Error: 'Access denied; you need (at least one of) the PROCESS privilege(s) for this operation' when trying to dump tablespaces
這時候請在備份指令中加上`--no-tablespaces`選項，來解決。

### 使用cron定期執行腳本

輸入指令編輯`crontab`:

        $ crontab -e

加入每天執行自動備份的腳本的設定:

        # 每天早上 3 點 30 分執行自動備份的腳本
        30 03 * * * ~/sql_dump.sh

### 還原資料庫

還原方法有兩種，一種是登入MySQL後輸入指令:
        
        mysql> source 備份檔案路徑

另一種是使用命令行執行:

        $ mysql -u用戶名 -p密碼 < 備份檔案路徑

一般是使用第一種方式。

***

## 遠端登入MySQL Server

如果MySQL Server是安裝在虛擬機上面的話，那首先請將虛擬機的網路配置設置為`橋接介面卡`而不是`NAT`，使用`橋接介面卡`可以使得虛擬機使用一組你本機所在網段的ip。

![alt tag](https://imgur.com/kSf1bL5.png)

使用MySQL遠端登入MySQL Server可以用下面的指令:

        $ mysql -h遠端地址 -u用戶名 -p

然而即便配置與用戶權限都設定正確，可能依然無法遠端登入Server。當你嘗試遠端登入Server時，可能會出現下面的錯誤提示:

        2003, "Can't connect to MySQL server..."

這時候要在server上使用下面的步驟來解除這個錯誤:
1. 運行命令: `$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`。
2. 找到`bind-address = 127.0.0.1`並將該行註釋掉。
3. 重新啟動MySQL Server。

重新啟動後你就可以正常使用遠端登入了。

***

## 後記

原本只以為只要安裝好就萬事大吉了，沒想到原來管理一個MySQLServer背後居然有這麼多的學問。

在學習的過程中遇到的問題也不少，但好在現在網路上很多大神都會提供解答，問題最後也都能解決。印象比較深刻的是遠端登入遇到的一個問題，當時是無論怎麼設定用戶權限跟修改配置文件都還是無法遠端登入，到處去問人也都無解，最後是我重新啟動虛擬機就一切正常了。

如果你也遇到類似的問題，不妨也試試重啟電腦看看吧，搞不好可以幫你省下許多時間。

***

## 參考資料

[How to fix the mysqldump access denied process privilege error.](https://anothercoffee.net/how-to-fix-the-mysqldump-access-denied-process-privilege-error/)

[ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1'(111)](https://stackoverflow.com/questions/1673530/error-2003-hy000-cant-connect-to-mysql-server-on-127-0-0-1-111)