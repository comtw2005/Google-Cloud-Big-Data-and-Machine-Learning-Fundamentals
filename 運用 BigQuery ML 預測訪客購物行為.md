運用 BigQuery ML 預測訪客購物行為

### 總覽
BigQuery 機器學習 (BigQuery ML) 可讓使用者運用 SQL 查詢，在 BigQuery 建立並執行機器學習模型，目標是讓 SQL 使用者能透過現有的工具建立模型，而且不必移動資料，可加快開發速度，使所有人都能輕鬆使用機器學習技術。

BigQuery 已載入電子商務資料集，其中包含數百萬筆 Google Analytics 的 Google 商品網路商店記錄。在本實驗室中，您將使用這些資料建立模型，預測訪客是否會完成交易。

## 課程內容
如何在 BigQuery 建立、評估與使用機器學習模型

## 事前準備
Chrome 或 Firefox 瀏覽器
SQL 或 BigQuery 的基本知識

## 開啟 BigQuery 控制台
1. 在 Google Cloud 控制台中，依序選取「導覽選單」>「BigQuery」。

接著，畫面中會顯示「歡迎使用 Cloud 控制台中的 BigQuery」訊息方塊，當中會列出快速入門指南的連結和使用者介面更新內容。

2. 按一下「完成」。

## 工作 1：建立資料集

1. 在專案中建立新資料集：在「Explorer」部分點選專案 ID 旁的三點圖示，按一下「建立資料集」。
「建立資料集」對話方塊會隨即開啟。

  ![圖片](https://github.com/user-attachments/assets/7b393466-79c6-4e33-b806-11548ba03a0c)


3. 在「資料集 ID」輸入 bqml_lab，點選「建立資料集」(接受其他預設值)。
![圖片](https://github.com/user-attachments/assets/a9a6cc5e-6c6a-4d1d-ad6e-f16319c514e4)

![圖片](https://github.com/user-attachments/assets/74015dd9-4fce-449b-958d-5222ddfd1c1a)





## 工作 2：探索資料

本實驗室要使用的資料位於 bigquery-public-data 專案，所有人皆可存取。我們來看看這些資料範例。

1. 在「未命名的查詢」方塊中新增查詢，點選「執行」按鈕。

```
#standardSQL
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170631'
LIMIT 10000;
```

![圖片](https://github.com/user-attachments/assets/33befa72-2cb6-45d7-872f-55987070c3b5)

ref:
    Google-Cloud-Big-Data-and-Machine-Learning-Fundamentals/bigquery-public-data

資料表有許多欄，但我們只需其中幾欄來建立 ML 模型。這裡使用下列條件判斷訪客是否會完成交易：訪客裝置使用的作業系統、該裝置是否為行動裝置、訪客的國家/地區，以及網頁瀏覽次數。
在本例中，label 是您調整或預測的依據。

這些資料會成為您所建立 ML 模型的訓練資料，且僅限從 2016 年 8 月 1 日到 2017 年 6 月 31 日收集的資料，以保留最後一個月的資料來用於「預測」。為了節省時間，進一步將資料點限制為 10,000 個。

接著將資料儲存為訓練資料。點選「儲存」，從下拉式選單選取「儲存檢視表」，將這個查詢儲存為 view。在彈出式視窗中，「資料集」選取 bqml_lab，「資料表名稱」輸入 training_data，然後點選「儲存」。

![圖片](https://github.com/user-attachments/assets/c9972758-8866-4eba-9f8f-e56f29461e39)

![圖片](https://github.com/user-attachments/assets/95a1043e-def7-46b3-94f7-39e4d4e3505b)

![圖片](https://github.com/user-attachments/assets/a168df72-7cf5-4f86-b100-98a5ce6faeb5)

![圖片](https://github.com/user-attachments/assets/cb20fcea-295a-4533-bdc8-f869d438fd87)


### 工作 3：建立模型
現在將查詢換成下列內容，建立模型來預測訪客是否會完成交易：

```
#standardSQL
CREATE OR REPLACE MODEL `bqml_lab.sample_model`
OPTIONS(model_type='logistic_reg') AS
SELECT * from `bqml_lab.training_data`;
```

在本例中，bqml_lab 是資料集名稱，sample_model 是模型名稱，training_data 則是前一項工作查看的交易資料。指定的模型類型是二元邏輯迴歸。

執行 CREATE MODEL 指令會建立非同步執行的查詢工作，這樣就能關閉或重新整理 BigQuery UI 視窗等。


[非必要] 模型資訊與訓練統計資料
如果感興趣，您可以點選左選單中的 bqml_lab 資料集，按一下 UI 中的 sample_model 模型來取得模型資訊。在「詳細資料」下，您會找到一些基本的模型資訊，以及用於產生模型的訓練選項。
在「訓練」下，您會看見類似下方的資料表：

![圖片](https://github.com/user-attachments/assets/ef3f0cc5-db2a-469f-a23c-42f1b636c1fa)

![圖片](https://github.com/user-attachments/assets/9d71e662-4373-4ce1-b84b-5a516383ed84)

![圖片](https://github.com/user-attachments/assets/0d70de56-fb4f-4723-ae66-e959f7ed9db8)

![圖片](https://github.com/user-attachments/assets/a35934b1-94bf-40fa-9f19-1e4955a5dd92)

## 工作 4：評估模型
現在將查詢換成下列內容：

```#standardSQL
SELECT
  *
FROM
  ml.EVALUATE(MODEL `bqml_lab.sample_model`);
```


在本查詢中，您使用 ml.EVALUATE 函式，根據實際資料評估預測值，該函式提供一些與模型成效相關的指標。您會看見類似下方的資料表：

結果資料表

![圖片](https://github.com/user-attachments/assets/195b54d0-e4d9-4999-8301-28a5beb39432)

![圖片](https://github.com/user-attachments/assets/2bf30462-f09e-446e-9dab-443ed729c3c8)


### 工作 5：使用模型

現在點選「SQL 查詢」並執行下列查詢：

```
#standardSQL
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews,
  fullVisitorId
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801';
```
您會發現查詢中的 SELECT 和 FROM 部分，與用於產生訓練資料的部分類似。另外還有 fullVisitorId 欄，您可以用該欄預測個別使用者的交易。WHERE 部分則反映時間範圍 (2017 年 7 月 1 日至 8 月 1 日) 內的變化。

![圖片](https://github.com/user-attachments/assets/d20db4ab-9358-4c69-96a6-be0b9d0583b6)



2. 接著儲存 7 月的資料，在接下來的兩個步驟，使用該資料和我們的模型進行預測。點選「儲存」，從下拉式選單選取「儲存檢視表」，將這個查詢儲存為 view。在彈出式視窗中，「資料集」選取 bqml_lab，「資料表名稱」輸入 july_data，然後點選「儲存」。

3. 預測各國家/地區的購物行為

透過這項查詢，您將嘗試預測各國家/地區訪客的交易量、排序結果，並選出購買量前 10 多的國家/地區：

```
#standardSQL
SELECT
  country,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT * FROM `bqml_lab.july_data`))
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

本次查詢使用的是 ml.PREDICT，且查詢的 BigQuery ML 部分是用標準 SQL 指令包裹。在本實驗室中，您感興趣的是國家/地區與各國家/地區的總購買量，所以選擇使用 SELECT、GROUP BY 和 ORDER BY。LIMIT 則用於確保只會取得購買量前 10 多的結果。

您會看見類似下方的資料表：

![圖片](https://github.com/user-attachments/assets/08057991-a220-4e58-a83f-192793a297bd)


4. 預測各使用者的購買量
我們再看另一個例子。這次，您將嘗試預測各訪客的交易量、排序結果，並選出交易量前 10 多的訪客：

```
#standardSQL
SELECT
  fullVisitorId,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT * FROM `bqml_lab.july_data`))
GROUP BY fullVisitorId
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

您會看見類似下方的資料表：

![圖片](https://github.com/user-attachments/assets/2bd68ac5-ad17-495c-9d8b-ce2c37ae8776)















