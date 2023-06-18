# 第一章：使用者人數 -- 從零到百萬規模

此篇主要在講最簡單的單台主機設置，到百萬規模的系統，中間如何逐步加入各個元件，以及每個元件如何擴展。
經過第一章的介紹，便可以對一個可規模化 (scalable) 的系統有相對完整與綜觀的認識。

## 單台主機

關鍵元件：

1. Client (web, mobile)
2. DNS server
3. Web server

主要流程：

1. 客戶端發起請求
2. DNS 伺服器解析出網站伺服器 IP
3. 網站伺服器處理流量

## 資料庫

關鍵元件：

1. Client (web, mobile)
2. DNS server
3. Web server
4. Database

主要流程：

1. 客戶端發起請求
2. DNS 伺服器解析出網站伺服器 IP
3. 網站伺服器處理流量
4. 網站伺服器向資料庫請求資料，或送出資料修改請求

資料庫型態：

1. 關聯式資料庫 (RDBMS)
2. 非關聯式資料庫 (NoSQL)
   1. CouchDB
   2. Cassandra
   3. Hbase
   4. AWS DynamoDB

## 擴展

1. 垂直擴展：加大機器的 CPU, memory, disk 等硬體處理能力。但存在以下問題：
   1. 一台機器的運算與儲存擴展有硬體上的極限
   2. 會造成單點故障 (single point of failure), 即系統的單一節點故障便導致整個系統變得不可用
2. 水平擴展：增加更多台機器。能夠解決以上問題，也是在討論擴展時的主要對策。

## 負載平衡 (Load Balancer)

關鍵元件：

1. Client (web, mobile)
2. DNS server
3. *Load Balancer*
4. Web server
5. Database

主要流程：

1. 客戶端發起請求
2. DNS 伺服器解析出網站伺服器 IP
3. *負載平衡接收流量後往後面多台機器分配請求*
4. 多台網站伺服器處理流量
5. 網站伺服器向資料庫請求資料，或送出資料修改請求

解決的問題

1. 可透過增加網頁伺服器數量來處理更多的請求，解決了擴展性的問題 (scalability)
2. 當有一台機器故障時，負載平衡器可以將請求發送至其他運作中的機器，增加了可用性 (availability)

注意事項：

1. 由於資安因素，通常網頁伺服器會透過內網通訊，只有負載平衡器連接外網
2. 負載平衡器可能變成系統中單點故障所在，但可使用 DNS server 的循環解析 (Round-robin DNS resolution) 或其他方式來解決單點故障問題

## 資料庫複製 (Database Replication)

關鍵元件：

1. Client (web, mobile)
2. DNS server
3. Load Balancer
4. Web server
5. *Database with multiple replica*

主要流程：

1. 客戶端發起請求
2. DNS 伺服器解析出網站伺服器 IP
3. 負載平衡接收流量後往後面多台機器分配請求
4. 多台網站伺服器處理流量
5. 網站伺服器向資料庫請求資料，或送出資料修改請求
6. *資料庫主節點(master)負責處理寫入與讀取，從節點(slave)負責處理讀取請求*

解決的問題

1. 當讀取資料的請求很多時，從節點可分攤讀取請求，並且可透過增加從節點來達成擴展
2. 資料備份：即使有節點損毀，依然能保證可以從其他節點獲得完整的數據

## 快取 (Cache)

關鍵元件：

1. Client (web, mobile)
2. DNS server
3. Load Balancer
4. Web server
5. *Cache*
6. Database with multiple replica

主要流程：

1. 客戶端發起請求
2. DNS 伺服器解析出網站伺服器 IP
3. 負載平衡接收流量後往後面多台機器分配請求
4. 多台網站伺服器處理流量
5. 網站伺服器向*快取*請求資料，或送出資料修改請求
6. *快取負責暫時性儲存資料，回應伺服器需求*
7. 資料庫主節點(master)負責處理寫入與讀取，從節點(slave)負責處理讀取請求

注意事項

1. 使用快取最需要關注的問題是決定何時資料是失效的，可能的方法有
   1. 快取壽命 (Time to live, TTL)：快取資料經過特定時間長度後即視為無效
   2. Read-through：快取接收到請求時判別有沒有資料，有就回傳，沒有的話從資料庫拿取，回應請求，並貯存在快取中
   3. Write-through：服務接收到寫入請求時，同時往快取與資料庫寫入資料，以保證資料都是最新的。缺點是兩邊都要寫入可能導致回應時間變長
2. 快取滿了的時候的處理方式
   1. LRU (Least-recently-used)：上次被使用時間距今最久的先清掉
   2. LFU (Least-frequently-used)：最不常使用到的先清掉
   3. FIFO (First-in-first-out)：先進來的先清掉

## 內容傳遞網路 (Content Delivery Network, CDN)

關鍵元件：

1. Client (web, mobile)
2. *CDN*
3. DNS server
4. Load Balancer
5. Web server
6. Cache
7. Database with multiple replica

主要流程：

1. 客戶端發起請求
2. *對於靜態資料如圖片或指令碼，CDN 能夠直接回應請求，減少系統負擔*
3. DNS 伺服器解析出網站伺服器 IP
4. 負載平衡接收流量後往後面多台機器分配請求
5. 多台網站伺服器處理流量
6. 網站伺服器向快取請求資料，或送出資料修改請求
7. 快取負責暫時性儲存資料，回應伺服器需求
8. 資料庫主節點(master)負責處理寫入與讀取，從節點(slave)負責處理讀取請求

注意事項：

1. CDN 通常在全球分佈，接收並處理地理距離上較近的請求，以減少回應時間
2. CDN 也要注意如何使資料無效 (invalidation)，常見的方法有
   1. TTL：經過一定時間後資料無效
   2. 版本：在 URL 攜帶版本號，因此當客戶端請求不同版本時，CDN 便會向伺服器詢問最新版的資料

## 伺服器狀態 (Stateful vs Stateless)

### Stateful

伺服器會將使用者資料貯存在同個伺服器上，例如將會話 (session) 存在機器上

問題：當水平擴展時，如果使用者的請求被負載平衡器分配到不同的機器上，則該機器沒有使用者的會話資料，將導致系統出現預期外的行為

### Stateless

伺服器將所有的資料貯存在第三方元件或系統中，而非自身的貯存空間

伺服器在處理請求時，可以把每個請求看成獨立的流程來處理，不需要考慮當下請求跟之前的請求有沒有關聯，也不需要考慮該請求所需要的資訊是否存在自己的貯存空間中

可用的會話資料處存方案：
1. Memcached
2. Redis
3. NoSQL

備注：
1. 在討論水平擴展時，通常假設伺服器是無狀態的，這樣在部署時就不需要考慮負載平衡器該如何解決前後請求之間的關係
2. 即使如此，一些負載平衡器還是可以提供所謂"親和性"的機制 (affinity)，例如將來自同一個 IP 的請求都分配到同一台機器上

## 資料中心 (Data Centers)

關鍵元件：

1. Client (web, mobile)
2. CDN
3. DNS server
4. Load Balancer
5. *Web server in multiple data centers*
6. Cache
7. Database with multiple replica

主要流程：

1. 客戶端發起請求
2. 對於靜態資料如圖片或指令碼，CDN 能夠直接回應請求，減少系統負擔
3. DNS 伺服器解析出網站伺服器 IP
4. 負載平衡接收流量後往後面多台機器分配請求
5. *多台網站伺服器分布在不同的資料中心中處理流量*
6. 網站伺服器向快取請求資料，或送出資料修改請求
7. 快取負責暫時性儲存資料，回應伺服器需求
8. 資料庫主節點(master)負責處理寫入與讀取，從節點(slave)負責處理讀取請求

注意事項：

1. DNS 通常會根據使用者的地理位置來解析請求，回應請求一個距離使用者較近的資料中心
2. 這裡雖然用的名稱是 Data Center, 但我認為用 AWS 的 Availability Zone 來說明會更好解釋。即不同的資料中心是在地理距離上分佈相距較遠的，如此一來可以在某區域發生災害時依舊保證有其他地理區域的資料中心依舊可用。例如當 ap-northeast-1 (東京) 的資料中心所在區域發生地震時而導致資料中心不可用時，us-east-2 依舊可用，不會因為地理位置相近而導致所有資料中心同時損毀。

## 訊息隊列 (Message Queue)

關鍵元件：

1. Client (web, mobile)
2. CDN
3. DNS server
4. Load Balancer
5. Web server in multiple data centers
6. *Message Queue*
7. *Workers*
8. Cache
9. Database with multiple replica

主要流程：

1. 客戶端發起請求
2. 對於靜態資料如圖片或指令碼，CDN 能夠直接回應請求，減少系統負擔
3. DNS 伺服器解析出網站伺服器 IP
4. 負載平衡接收流量後往後面多台機器分配請求
5. 多台網站伺服器分布在不同的資料中心中處理流量
6. 網站伺服器向快取請求資料，或送出資料修改請求。*長時間操作則將資訊送往訊息隊列*
7. *Worker 接受隊列內的消息並逐一處理*
8. 快取負責暫時性儲存資料，回應伺服器需求
9. 資料庫主節點(master)負責處理寫入與讀取，從節點(slave)負責處理讀取請求

注意事項

1. Queue 只是這邊提到的一種方式，其他的訊息溝通系統還包含生產-訂閱 (Pub/Sub) 等機制。
2. 費時較長且不需要使用者立即得知結果的操作，設計上通常不會由網站伺服器處理，這時就可以讓網站伺服器把這類工作打包成訊息，丟到隊列中，並且由特定的工作伺服器 (worker) 去消化隊列訊息，並異步處理。
3. 常見的應用情境包含：寄信、影像處理、資料分析。
4. Queue 的作用還包含服務之間的*去耦合*，即讓接收訊息與處理訊息者不需要了解彼此的部署與實作，並且兩者可以實現不同的擴展策略。

## 日誌、指標、自動化 (Logging, Metrics, Automation)

- 日誌：對於了解服務哪裡出錯以及獲取除錯所需的資訊來說至關重要
- 指標：了解服務的關鍵資訊，例如回應時間、系統負載、使用者數量、關鍵業務請求數量等
- 自動化：使用 CI/CD 等手段確保即使系統成長的更加複雜，大量事務能夠被自動化處理，服務能持續運行

## 資料庫水平擴展

擴展方法：

1. 垂直擴展：加大機器處理與貯存的運算能力
2. 水平擴展：分片 (Sharding)

注意事項

1. 分片需注意使用什麼欄位作為 Sharding key，要避免資料集中在一個分片上
2. 若要重新分片資料時會是個複雜的挑戰
3. 名人問題：大量讀取可能都是針對少部分資料，導致特定分片承受大量流量，而其他分片則閒置
4. 分片會使得要做 join 等聚合操作時變得困難，這些邏輯可能需要在應用層完成
5. 資料庫擴展方法有很多，這裡提到的主從式架構 Master-slave 只是其中一種，其他還有包含
   1. Multi-master
   2. Consistent Hashing
   3. Partitioning
   4. Distributed Transaction
   5. Command Query Responsibility Segregation