# 2. 将数据导入云端

### 创建一个存储桶
* 进入GCP网页控制台的存储部分并创建一个新的存储桶

### 将你需要的书籍数据填充到你的存储桶中

* 打开CloudShell并克隆此仓库：
    ```
    git clone https://github.com/GoogleCloudPlatform/data-science-on-gcp
    ```
* 进入仓库的02_ingest文件夹
* 编辑./ingest.sh文件以反映你想处理的年份（至少需要2015年）
* 执行 ./ingest.sh bucketname

### [可选] 计划每月下载
* 进入仓库的02_ingest/monthlyupdate文件夹。
* 运行命令 `pip3 install google-cloud-storage google-cloud-bigquery`
* 运行命令 `gcloud auth application-default login`
* 尝试使用Python脚本导入一个月的数据：`./ingest_flights.py --debug --bucket your-bucket-name --year 2015 --month 02`
* 通过运行`./01_setup_svc_acct.sh`设置一个名为svc-monthly-ingest的服务账户
* 现在，尝试以服务账户身份运行导入脚本：
  * 访问GCP控制台的服务账户部分：https://console.cloud.google.com/iam-admin/serviceaccounts
  * 选择新创建的服务账户svc-monthly-ingest并点击管理密钥
  * 添加密钥（创建一个新的JSON密钥）并下载到名为tempkey.json的文件
  * 运行 `gcloud auth activate-service-account --key-file tempkey.json`
  * 尝试导入一个月的数据 `./ingest_flights.py --bucket $BUCKET --year 2015 --month 03 --debug`
  * 使用 `gcloud auth login` 回到以你自己的身份运行命令
* 部署到Cloud Run：`./02_deploy_cr.sh`
* 测试你是否可以使用Cloud Run调用该函数：`./03_call_cr.sh`
* 测试获取下一个月的功能是否有效：`./04_next_month.sh`
* 设置一个Cloud Scheduler任务以每月调用Cloud Run：`./05_setup_cron.sh`
* 访问GCP控制台的Cloud Run和Cloud Scheduler部分，删除Cloud Run实例和计划任务——你不再需要它们。