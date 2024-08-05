# 4. 流数据：发布和摄取

### 如果有需要，先补充第1-3章的内容
* 前往 GCP Web 控制台的存储部分并创建一个新桶
* 打开 CloudShell 并克隆此仓库：
    ```
    git clone https://github.com/GoogleCloudPlatform/data-science-on-gcp
    ```
* 然后运行：
cd data-science-on-gcp/02_ingest
./ingest_from_crsbucket bucketname
* 运行：
cd ../03_sqlstudio
./create_views.sh
### DataFlow中的批处理转换
* 设置：
    ```
	cd transform; ./install_packages.sh
    ```
* 解析机场数据：
	```
	./df01.py
	head extracted_airports-00000*
	rm extracted_airports-*
	```
* 添加时区信息：
	```
	./df02.py
	head airports_with_tz-00000*
	rm airports_with_tz-*
	```
* 将时间转换为UTC：
	```
	./df03.py
	head -3 all_flights-00000*
	```
* 修正日期：
	```
	./df04.py
	head -3 all_flights-00000*
	rm all_flights-*
	```
* 创建事件：
	```
	./df05.py
	head -3 all_events-00000*
	rm all_events-*
	```  
* 读/写到云端：
	```
    ./stage_airports_file.sh BUCKETNAME
	./df06.py --project PROJECT --bucket BUCKETNAME
	``` 
    在 BigQuery 中查找新表（flights_simevents）
* 在云端运行：
	```
	./df07.py --project PROJECT --bucket BUCKETNAME --region us-central1
	``` 
* 前往 GCP Web 控制台并等待 Dataflow ch04timecorr 任务完成。这可能需要30分钟到2+小时，具体取决于项目的配额（可以通过 https://console.cloud.google.com/iam-admin/quotas 修改配额）。
* 然后，导航到 BigQuery 控制台并输入：
	```
        SELECT
          ORIGIN,
          DEP_TIME,
          DEST,
          ARR_TIME,
          ARR_DELAY,
          EVENT_TIME,
          EVENT_TYPE
        FROM
          dsongcp.flights_simevents
        WHERE
          (DEP_DELAY > 15 and ORIGIN = 'SEA') or
          (ARR_DELAY > 15 and DEST = 'SEA')
        ORDER BY EVENT_TIME ASC
        LIMIT
          5

	```
### 模拟事件流
* 在 CloudShell 中运行：
	```
    cd simulate
	python3 ./simulate.py --startTime '2015-05-01 00:00:00 UTC' --endTime '2015-05-04 00:00:00 UTC' --speedFactor=30 --project $DEVSHELL_PROJECT_ID
    ```
 
### 实时流处理
* 在另一个 CloudShell 标签中运行 avg01.py：
	```
	cd realtime
	./avg01.py --project PROJECT --bucket BUCKETNAME --region us-central1
	```
* 大约一分钟后，您可以在 BigQuery 控制台查询事件：
	```
	SELECT * FROM dsongcp.streaming_events
	ORDER BY EVENT_TIME DESC
    LIMIT 5
	```
* 通过按 Ctrl+C 停止 avg01.py
* 运行 avg02.py：
	```
	./avg02.py --project PROJECT --bucket BUCKETNAME --region us-central1
	```
* 大约5分钟后，您可以从 BigQuery 控制台查询：
	```
	SELECT * FROM dsongcp.streaming_delays
	ORDER BY END_TIME DESC
    LIMIT 5
	``` 
* 查看数据输入的频率：
	```
    SELECT END_TIME, num_flights
    FROM dsongcp.streaming_delays
    ORDER BY END_TIME DESC
    LIMIT 5
	``` 
* 可能管道会卡住，您需要在 Dataflow 上运行此操作。
* 按 Ctrl+C 停止 avg02.py
* 在 BigQuery 中截断表：
	```
	TRUNCATE TABLE dsongcp.streaming_delays
	``` 
* 运行 avg03.py：
	```
	./avg03.py --project PROJECT --bucket BUCKETNAME --region us-central1
	```
* 前往 GCP Web 控制台的 Dataflow 部分并监控作业。
* 作业开始写入 BigQuery 后，运行此查询并保存为视图：
	```
	SELECT * FROM dsongcp.streaming_delays
    WHERE AIRPORT = 'ATL'
    ORDER BY END_TIME DESC
	```
* 创建机场最新到达延迟视图：
	```
    CREATE OR REPLACE VIEW dsongcp.airport_delays AS
    WITH delays AS (
        SELECT d.*, a.LATITUDE, a.LONGITUDE
        FROM dsongcp.streaming_delays d
        JOIN dsongcp.airports a USING(AIRPORT) 
        WHERE a.AIRPORT_IS_LATEST = 1
    )
     
    SELECT 
        AIRPORT,
        CONCAT(LATITUDE, ',', LONGITUDE) AS LOCATION,
        ARRAY_AGG(
            STRUCT(AVG_ARR_DELAY, AVG_DEP_DELAY, NUM_FLIGHTS, END_TIME)
            ORDER BY END_TIME DESC LIMIT 1) AS a
    FROM delays
    GROUP BY AIRPORT, LONGITUDE, LATITUDE

	```   
* 按照本章的步骤连接到 Data Studio 并创建 GeoMap。
* 在 CloudShell 中停止模拟程序。
* 从 GCP Web 控制台停止 Dataflow 流管道。