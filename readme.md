# Jenkins

- 可以單獨為一台 Server，也可以裝在原本 Server 裡作為一個服務。
- 用 Docker build 環境，要求環境是 JAVA（最低的要求是Java7，建議是使用Java8），不用 Docker 的話安裝的流程有點麻煩，還要做一堆環境設置。
- 優點：可以單獨作為一台 Server，方便維護、管理。

## 操作流程
### 1. 開一個新的 instance
- Amazon Linux 2 AMI (HVM)
- Elastic IP (Optional，這樣 instance 重啟 IP 不會任意亂跑)

### 2. 安裝 git、node、PM2、docker
- `sudo yum update -y`
- `sudo yum install git`
- `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash`
- `nvm install node`
- `npm install pm2 -g`
- `sudo yum install -y docker`
- `sudo service docker start`
- `sudo usermod -a -G docker ec2-user`

### 3. 使用 Docker 安裝 Jenckins
```
docker run \
-u root \
-d \
-p 8080:8080 \
```
`-u root` 表示用 root 來跑 jenkins，賦予 jenkins 足夠權限執行應用場景
`-d` detached mode，在背景 run container
`-p 8080:8080` 前面是開放出去 host 的 port 號，後面是對內的 container 的 port 號


感謝 Stone 大大補充：
```
docker image name/tag:version
```
### 4. Unlock Jenkins

![](https://i.imgur.com/FGZpC9H.jpg)
- 需要進到以下路徑：/var/jenkins_home/secrets/initialAdminPassword
但由於我們是用 Docker 安裝，所以該路徑會是在 container 之內。用以下指令進到 container 中：
`docker exec -it [container ID] bash`
取得 initialAdminPassword 回傳值：`cat initialAdminPassword`
貼回 Administrator password 欄位。

- 選擇 Install suggested plugins。
- Creating the first administrator user
- Jenkins URL: 依照預設即可。

### 5. 建構要套用 CI/CD 流程的專案
- Github 弄一個 repository，當中是可以跑的 Node.js 專案。
- 用 git clone 到 instance 內。

### 6. Jenkins 安裝 node.js

- 前往管理頁：http://[Jenkins IP]:8080/
- 管理 Jenkins > 管理外掛程式 > 可用的
- 搜尋 NodeJS > 下載並於重新啟動後安裝
- 勾選當下載完成時且沒有工作正在執行時重新啟動
- Jenkins 完成重啟
- 管理 Jenkins > Global Tool Configuration > NodeJS 安裝 > 新增NodeJS > NodeJS 12.4.0

### 7. 新增要套用的目標專案
- 選擇 新增作業 > Enter an item name ： node-test > 建置 Free-Style 軟體專案
- 勾選 GitHub project > 填入 https://github.com/[repository]

- 原始碼管理 > 勾選 git > 填入 https://github.com/[repository].git

- 建置觸發程序 > 勾選 GitHub hook trigger for GITScm polling

- 建置環境 > Provide Node & npm bin/ folder to PATH > NodeJS 12.4.0

- 建置 > 新增建置步驟 > 執行 shell
- 指令：等等在做

### 8. 設定 github web hook
- 到 https://github.com/[repository] > Webhooks > add webhook
- Payload URL 填入：http://[Jenkins IP]:8080/github-webhook/
- 勾選 Just the Push Event
- 勾選 Active

### 9. 賦予 Jenkins server 有 EC2 的進入權限
- 進到 container 中：`docker exec -it [container ID] bash`
- `ssh-keygen -t rsa` Enter 按到底，不設密碼。
Enter file in which to save the key (/root/.ssh/id_rsa):
- `cat /root/.ssh/id_rsa.pub` 取得回傳值。
- 離開 container 回到專案目錄。
- vim ~/.ssh/authorized_keys 加上上述回傳值。

### 10. 專案內新增指令檔案：
- 新增 deploy 檔案：
```
#!/bin/sh

ssh -tt ec2-user@[專案 server IP] <<EOF
    cd ~/[專案資料夾]
    git pull origin master
    npm install
    npm install -g pm2
    pm2 restart app2.js
    exit
EOF
```

- 回到步驟 7 的指令部分，執行Shell 加上 ./deploy

### 11. 測試 Push 能否自動佈署
可在 Jenkins 主頁追蹤專案情況，有錯誤可以看輸出的 output console log。
