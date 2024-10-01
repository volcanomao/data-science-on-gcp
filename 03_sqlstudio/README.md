# 3. 创建引人注目的仪表板

### 跟上第2章
如果您尚未这样做，请将原始数据加载到BigQuery数据集中：
* 前往GCP网页控制台的存储部分并创建一个新的存储桶
* 打开CloudShell并克隆此仓库：
    ```
    git clone https://github.com/GoogleCloudPlatform/data-science-on-gcp
    ```
* 然后，运行：
    ```
    cd data-science-on-gcp/02_ingest
    ./ingest.sh bucketname
    ```


### 可选：将数据加载到PostgreSQL
* 访问 https://console.cloud.google.com/sql
* 选择 创建实例
* 选择 PostgreSQL，然后填写表单如下：
  * 实例名称为 flights
  * 点击 生成 生成一个强密码
  * 选择默认的 PostgreSQL 版本
  * 选择您的 CSV 数据存储桶所在的区域
  * 选择单区实例
  * 选择具有 2 个 vCPU 的标准机器类型
  * 点击 创建实例
* 输入（根据需要更改存储桶）：
  ```
   gsutil cp create_table.sql \
    gs://cloud-training-demos-ml/flights/ch3/create_table.sql
  ```

* 使用网页控制台创建空表：
  * 导航到Cloud SQL的数据库部分，创建一个名为bts的新数据库
  * 导航到flights实例并选择导入
  * 指定create_table.sql在您的存储桶中的位置
  * 指定您要在数据库bts中创建一个表
* 将CSV文件加载到此表中：
  * 浏览到您存储桶中的201501.csv
  * 指定CSV作为格式
  * bts作为数据库
  * flights作为表
* 在Cloud Shell中，连接到数据库并运行查询
  * 使用以下两个命令之一连接到数据库（如果您不需要SQL代理，则使用第一个，如果需要，则使用第二个——通常，如果您的组织设置了安全规则以仅允许授权网络访问，则需要SQL代理）：
    * ```gcloud sql connect flights --user=postgres```
    * 或者 ```gcloud beta sql connect flights --user=postgres```
  * 在提示符中，输入 ```\c bts;```
  * 输入以下查询：
  ``` 
  SELECT "Origin", COUNT(*) AS num_flights 
  FROM flights GROUP BY "Origin" 
  ORDER BY num_flights DESC 
  LIMIT 5;
  ```
* 添加更多月份的CSV数据，并注意性能下降。
完成后，删除Cloud SQL实例，因为您在本书的其余部分不再需要它。

### 在BigQuery中创建视图
* 运行脚本 
  ```./create_views.sh```
* 通过运行脚本计算各种阈值的列联表 
  ```
  ./contingency.sh
  ```

### 构建仪表板
按照本章主要文本中的步骤设置Data Studio仪表板并创建图表。
