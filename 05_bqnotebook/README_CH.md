# 5. 交互式数据探索

### 如有需要，请跟上前几章的内容
如果你没有阅读第2-4章，最简单的赶上方法是从我的存储桶复制数据：
* 进入GCP网页控制台的存储部分并创建一个新的存储桶
* 打开CloudShell并克隆此代码库：
    ```
    git clone https://github.com/GoogleCloudPlatform/data-science-on-gcp
    ```
* 然后运行：
    ```
    cd data-science-on-gcp/02_ingest
    ./ingest_from_crsbucket bucketname
    ./bqload.sh  (csv-bucket-name) YEAR 
    ```
* 运行：
    ```
    cd ../03_sqlstudio
    ./create_views.sh
    ```
* 运行：
    ```
    cd ../04_streaming
    ./ingest_from_crsbucket.sh
    ```

## 尝试查询
* 在BigQuery中，查询第4章创建的时间校正文件：
    ```
    SELECT
       ORIGIN,
       AVG(DEP_DELAY) AS dep_delay,
       AVG(ARR_DELAY) AS arr_delay,
       COUNT(ARR_DELAY) AS num_flights
     FROM
       dsongcp.flights_tzcorr
     GROUP BY
       ORIGIN
    ```
* 尝试此目录下queries.txt中的其他查询。

* 导航到GCP控制台的Vertex AI Workbench部分。

* 启动一个新的托管笔记本。然后，复制并粘贴<a href="exploration.ipynb">exploration.ipynb</a>中的单元格并点击运行来执行代码。

* 创建trainday表BigQuery表和CSV文件，因为稍后你会需要它们：
    ```
    ./create_trainday.sh
    ```