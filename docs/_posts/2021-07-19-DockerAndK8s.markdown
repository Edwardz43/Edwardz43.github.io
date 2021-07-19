---
layout: post
title:  "Docker & k8s 開發部署共筆"
date:   2021-07-19 23:25:49 +0800
categories: jekyll update
---

## Container

[容器簡介：什麼是容器？(google)](https://cloud.google.com/containers/?hl=zh-TW)

重點節錄
>容器提供一種邏輯封裝機制，能夠將應用程式從實際執行所在環境抽取出來。無論目標環境是私人資料中心、公用雲端還是開發人員的個人筆記型電腦，這種分離的方式都可以==輕鬆、一致地部署==容器型應用程式。容器化提供了一種俐落的==分工模式==，當==開發人員專注於應用程式邏輯與依附元件==時，==IT 營運團隊可將注意力集中到部署與管理==上，而不用擔心諸如特定軟體版本與應用程式特定設定之類的應用程式細節。
>容器不像虛擬機器會將==硬體堆疊虛擬化==，而是會在==作業系統層級進行虛擬化==，讓多個容器直接在 OS 核心之上執行。這表示==容器遠比虛擬機器輕量==，不但共用 OS 核心、啟動速度較快，而且使用的記憶體量也比啟動完整 OS 要少很多。
>容器讓開發人員能夠建立可以預測的環境，而且這個==環境是與其他應用程式隔離開的==。容器也可能包含應用程式需要的軟體依附元件，例如特定版本的程式設計語言執行階段與其他軟體程式庫。從開發人員的角度來看，無論最終應用程式將部署到何處，這==一切都能保證一致==。這一切全都代表生產力：開發人員與 IT 營運團隊可以==減少花費在偵錯與診斷環境差異上的時間，而將更多時間用在為使用者提供新功能==。而且開發人員現在可以在開發與測試環境中，做出同樣適用於實際工作環境的假設，因此錯誤的發生機率也會隨之降低。
>==容器幾乎能在任意位置執行，顯著降低了開發與部署的難度==：在 Linux、Windows 與 Mac 作業系統上；在虛擬機器或不含作業系統的機器上；在開發人員的機器或資料中心內部部署中；當然，還有公開雲端中。廣泛應用於容器的 Docker 映像檔格式更加強化了可攜性。每當您想要執行軟體時，就可以使用容器。
>元件化也可讓您以更快的速度及更可靠的方式開發；==程式碼集越小，維護起來越容易==，而且由於服務是分離的，要==針對輸出測試特定輸入會更簡單==。
>由於容器能夠抽離程式碼，因此您可以將個別服務當成黑箱來處理，進一步減少開發人員需要顧慮的空間。當開發人員處理需要彼此依賴的服務時，他們==可以針對特定服務輕鬆啟動容器，而不用浪費時間提前設定正確的環境及疑難排解==。

### 加入容器化的開發流程

```flow
st=>start: new freature
e=>end: release
op=>operation: commit source code
op2=>operation: build image
op3=>operation: deploy
op4=>operation: bug fix
cond=>condition: QA pass ?

st->op->op2->op3->cond
cond(yes)->e
cond(no)->op4->op
```

:::info
:bulb: 這一連串流程，可以透過 pipeline 的方式做自動化流程，例如：
[Jenkins](https://www.jenkins.io/zh/doc/)
[Google Cloud Build](https://medium.com/@blaze0207/google-cloud-build-%E6%95%99%E5%AD%B8-%E4%B8%80-ed7103d3fb62)
:::

另外，容器化在開發階段也是相當好用，例如：
前端，後端， server 將各自的專案打包成 image 供彼此使用，所以整個專案可以在本機上跑測試，不需要透過開發站主機或連接到他人的主機

#### 其他參考資料

* [了解 Linux 容器(RedHat)](https://www.redhat.com/zh/topics/containers)
* [What are Containers(IBM)](https://www.ibm.com/cloud/learn/containers)

## The most popular container runtime : **Docker**

### Docker 三元素

* **映像檔 Image** : 模板(template)，用來重複產生容器實體
* **容器 Container** : 用映像檔建立出來的執行實例
* **倉庫 Repository** : 集中存放映像檔檔案的場所

>最大的公倉 [Docker Hub](https://hub.docker.com/)

參考資料:

* [Docker 基礎教學與介紹 101](https://cwhu.medium.com/docker-tutorial-101-c3808b899ac6) <- 非常詳細
* [10個Q&A快速認識Docker](https://www.ithome.com.tw/news/91847)
* [什麼是 Docker？(microsoft)](https://docs.microsoft.com/zh-tw/dotnet/architecture/containerized-lifecycle/what-is-docker)
* [淺談 Container 實現原理, 以 Docker 為例](https://www.hwchiu.com/container-design-i.html) <- 這篇內容比較深，從 OS 底層來說明

### 安裝

* [Install Docker Desktop on Mac](https://docs.docker.com/docker-for-mac/install/)
* [CentOS Linux 7 安裝 Docker 步驟與使用教學](https://blog.gtwang.org/linux/centos-linux-7-install-docker-tutorial/)

### Dockerfile

docker file 可說是建立映象檔的腳本，docker engine 會根據腳本建立對應的 image

:::success

* [Dockerfile 基本結構](https://philipzheng.gitbook.io/docker_practice/dockerfile/basic_structure)
* [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) <- 最佳實踐，官方完整範例&解說
* [Node.js 成員映像檔的 Dockerfile 範例(IBM)](https://www.ibm.com/docs/zh-tw/was-liberty/nd?topic=apis-example-dockerfile-nodejs-member-image)
:::

---

#### 範例:以 Block Service 為例

先在專案根目錄下建立一個名為 `Dockerfile` 的檔案

```dockerfile=
#來源映像檔
FROM node:6.9.5-alpine

#先建立 pomeloapp 資料夾
RUN mkdir -p /usr/src/pomelo
WORKDIR /usr/src/pomelo

#複製專案內容
COPY . /usr/src/pomelo/

#安裝 pomelo 及相關需要的套件
RUN npm i pomelo@2.2.5 -g

#以我們特製版的 pomelo node_modules 取代核心功能
COPY pomelo-custom/pomelo /usr/local/lib/node_modules/pomelo

#切換到指定目錄執行命令
WORKDIR /usr/src/pomelo/game-server/
CMD ["pomelo", "start"]
```

接著執行指令

```shell=
docker build -t pomelo/block:latest .
```

就會建立名為 `pomelo/block` 的映像檔 版本為 `latest`

:::info
:mag_right: 若有多個不同名稱的 docker file，可以下 -f <Dockerfile路徑>指定，不然預設是指向 `Dockerfile`
:::

---

### Docker command

* [Container指令基礎](https://joshhu.gitbooks.io/dockercommands/content/Containers/ContainersBasic.html)
* [Docker 基本教學](https://ithelp.ithome.com.tw/articles/10199339)

---

#### 範例: 執行 Nginx

```shell=
docker run --name some-nginx -d -p 8080:80 nginx
```

##### 參數解說

```shell
--name # 容器名稱，沒有提供的話會隨機生成一個
-d     # 背景模式(daemon)
-p     # 容器對外暴露的 port，格式為：本機:容器，可以多個 => -p 80:80  -p 443:443 ...
```

執行指令後，用瀏覽器打開 `http://localhost:8080`
或用 cli 執行 `curl http://localhost:8080`
可以看到 nginx 的歡迎畫面

![nginx](https://i.imgur.com/XkvcxvL.png)

---

#### 範例: 執行 MySQL

```shell!
docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:5.7
```

---

* 參數解說

```shell
-v     # 掛載，將本機的特定資料夾對應到容器裡的指定路徑，格式為： 本機:容器
-e     # 環境變數，這邊要看映像檔的提供文件支援哪些變數，通常格式為全大寫加底線
tag    # 指定映像檔的版號，沒給的話會接直以 latest 取代
```

上面的指令將會執行一個 mysql 5.7 的 container，並且將資料掛載到本機的 `/my/custom/` 底下

---

#### 範例: 執行 Block Service

請先建立映像檔
在前一個範例中的 `Dockerfile` 同路徑下，執行：

```shell=
docker build -t pomelo/block .
```

>若沒有指定版本，預設會是 `latest`

* 參數解說

```shell
-t    # 指定的 tag name
```

接著執行 container

```shell
docker run --name pomelo-block -p 3001:3001 pomelo/block
```

這邊不用背景模式，方便觀察 log

接著在 terminal 執行 `createBoots` 指令

```shell
curl http://localhost:3001/createBoots?cardKind=1
```

成功的話將可以得到下圖的回應

![response](https://i.imgur.com/3DkD4oo.png)

---

## Container Orchestration : 容器編排

* [什么是容器编排?](https://www.redhat.com/zh/topics/containers/what-is-container-orchestration)
* [Container Orchestration Explained(簡中字幕)](https://www.youtube.com/watch?v=kBF6Bvth0zw)

### docker-compose

輕量的容器編排工具，很適合開發/測試階段使用

需要另外安裝，若是 windows/mac 安裝 docker desktop 版本的話，會包含在內

docker-compose 會透過 yaml 設定檔執行
我們可以拿上面範例的 image 來用:

首先建立一個名為 `docker-compose.yaml` 的檔案
>`.yml` 與 `.yaml` 同義

```yaml=
version: "3.8"

services:
  app:
    container_name: some-nginx
    image: nginx:latest
    ports:
      - 8080:80

  mysql:
    container_name: some-mysql
    image: mysql:5.7
    volumes:
      - /my/custom:/etc/mysql/conf.d
    environment: 
      MYSQL_ROOT_PASSWORD: my-secret-pw
      
```

`version` 參考本機的 docker [版本](https://docs.docker.com/compose/compose-file/)，新的版本會有新功能，盡量使用最新的

```yaml=3
    app:
        container_name: some-nginx
    ...
```

`app` 為自行定義的服務名稱， `container-name` 則是顯示在 `docker ps` 當中的容器名稱

在同目錄底下執行 `docker-compose up`，就會將兩個服務跑起來

比較常用的指令

* [up](https://docs.docker.com/compose/reference/up/) ： Builds, (re)creates, starts, and attaches to containers for a service
* [stop](https://docs.docker.com/compose/reference/stop/) ： Stops running containers ==without removing== them
* [down](https://docs.docker.com/compose/reference/down/)：Stops containers and ==removes== containers, networks, volumes, and images created by `up`

:::info
:bulb:[更多指令](https://docs.docker.com/compose/reference/)
:::

參考文件:

* [使用 Docker Compose(MicroSoft)](https://docs.microsoft.com/zh-tw/visualstudio/docker/tutorials/use-docker-compose)
* [使用 Docker-Compose 啟動多個 Docker Container](https://ithelp.ithome.com.tw/articles/10194183)
* [docker-compose cheatsheet](https://devhints.io/docker-compose)

### k8s

先來一段[影片](https://www.youtube.com/watch?v=aSrqRSk43lY)(簡中字幕)

![k8s](https://i.imgur.com/O6FWbN4.jpg)

`Node` 可以想像成實體機器或 VM， `Master Node` 透過 `kubectl` 與 `Worker Nodes` 溝通， `API Server` 是整個叢集的指揮中心，提供各項功能，並擔任 `Node` 之間的溝通橋樑

`Pod` 是 k8s 上運行的最小元件，一個 `Node` 上可以運行多個 `Pods`，每個 `Pod` 上又可以運行多個 `Containers` ，但是一般來說一個 `Pod` 上運行一個 `Container` 較好

>[參考資料](https://cwhu.medium.com/kubernetes-basic-concept-tutorial-e033e3504ec0)

---

#### 單機版 k8s : **minikube**

[github](https://github.com/kubernetes/minikube)

[安裝說明](https://ithelp.ithome.com.tw/articles/10192490)

>Minikube 是由 Google 發布的一個輕量級工具。讓開發者可以在本機上輕易架設一個 Kubernetes Cluster，快速上手 Kubernetes 的指令與環境。Minikube 會在本機上跑起一個 virtual machine，並且在這 VM 裡建立一個 signle-node Kubernetes Cluster，本身並不支援 HA (High availability)，也不推薦在實際應用上運行

通常 hardcore 一點會直接用 cli 透過 `kubectl` 操作 k8s，例如：

```shell=
kubectl get nodes # 查看節點
```

>`kubectl -h` 查看更多指令

#### Lens

若是不習慣 cli 的話，可以參考這款免費開源且簡單實用的 IDE：[**Lens**]
(<https://medium.com/@tasslin/%E7%B0%A1%E5%96%AE%E5%AF%A6%E7%94%A8%E7%9A%84kubernetes-ide-lens-d8bba1fc06ad>)

![lens](https://i.imgur.com/gdrQtqI.png)

安裝完成，開啟，點右下角的 +

![add context](https://i.imgur.com/jcygMHF.png)

Select contexts，這邊以 `minikube` 為例

![minikube](https://i.imgur.com/LjEk4h6.png)

成功後就可以看到圖示

![success](https://i.imgur.com/C1MQXq2.png)

若資源監控沒有顯示，要另外安裝 Prometheus 插件

![monitor](https://i.imgur.com/OiiKbal.png)

Features -> Install

![done](https://i.imgur.com/lDa5kl3.png)

---

#### 宣告式 vs 命令式

* [什麼是宣告式(Declarative)程式設計和命令式imperative)程式設計](https://www.itread01.com/content/1547101808.html)
* [Imperative vs. Declarative JavaScript](https://dzone.com/articles/imperative-vs-declarative-javascript)
* [Kubernetes宣告式API與程式設計正規化](https://iter01.com/593544.html)
* [攥寫設定能不學嗎：yaml](https://ithelp.ithome.com.tw/articles/10193509)

#### yaml : k8s 的設定檔

這邊先以 `ngnix.yaml` 為例:

```yaml=
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
   app: web
spec:
 containers:
   – name: web
     image: nginx
     ports:
       – containerPort: 80
```

這邊的 yaml 檔表示：我需要一個 pod，名稱是 `ngnix`，標籤是 `web`，這個 pod 裡面包含一個 container 名稱是 `web`，映像檔是 `ngnix`，會佔用到 80 `port`

透過 `kubectl` 指令可以向 `master node` 的 `API server` 請求資源:

```shell=
kubectl create -f ngnix.yaml
pod "web" created
```

看到訊息代表資源建立完成

---

### 更多補充

* [About Service & Ingress](https://hackmd.io/@rd7-edlo/rynjra6sd)

#### tags: `k8s` `docker` `container`
