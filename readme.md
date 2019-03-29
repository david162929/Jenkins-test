AWS EC2上的Node.js應用程序的Jenkins第1部分：在EC2上安裝Jenkins：https://medium.com/konvergen/jenkins-for-node-js-app-on-aws-ec2-part-1-installing-jenkins-on-ec2-24675cc08998

在EC2上的Appworksschool分組專案2019.3.15 instance裝。

安裝Jenkins：
https://jenkins.io/doc/tutorials/build-a-node-js-and-react-app-with-npm/
https://jenkins.io/doc/book/installing/#downloading-and-running-jenkins-in-docker

使用 Docker 裝：
docker run \
  --rm \
  -u root \
  -p 8080:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean

優先用這個
docker run \
  -u root \
  -d \
  -p 8080:8080 \
  jenkinsci/blueocean






出現訊息：
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

b67c4dfe476d4f24a31536f8a659b96c

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword


需要造訪 http://localhost:8080，但由於是在EC2上，改成 http://18.221.56.89:8080，且instance 要開放 8080 port 的連接。
Administrator password貼上c9a50fd08c454b0198eb188c3963e7e5。

Customizing Jenkins with plugins
選擇 Install suggested plugins

Creating the first administrator user

使用者名稱:David
密碼:
全名:David
電子郵件信箱:en162929@gmail.com

Jenkins URL:	http://18.221.56.89:8080/
The Jenkins URL is used to provide the root URL for absolute links to various Jenkins resources. That means this value is required for proper operation of many Jenkins features including email notifications, PR status updates, and the BUILD_URL environment variable provided to build steps.

注意!!!現在的instance 沒有綁彈性IP，所以重開機之後會變。
目前先綁。

Ctrl-C 離開後，docker的container就會停止，再run一次就重跑。
所以run的指令再加上 -d


https://medium.com/konvergen/jenkins-for-node-js-app-on-aws-ec2-part-2-creating-a-node-js-app-3a0fb6b63bc7

GitHub 新增 Create a new repository：david162929/Jenkins-test
本地端創建一個測試用的 Node.js：

npm init -y
npm install express --save
npm install -g nodemon

package.json 改為 "main": "app2.js"

app2.js：
/* ---------------Module--------------- */
const express = require("express");

const app = express();


app.get("/", (req, res) => {
	res.send("Wellcome David's Jenkins-test!");
});


/* ---------------Port--------------- */
app.listen(8081, () => {
	//console.log("this app is running on port 3000.");
});


mocha、supertest測試的部分先跳過。


EC2 要開新port 8081



git init

github新增一個new repository，stylish-web-v2

git remote add origin https://github.com/david162929/Jenkins-test.git
git push origin master
上傳成功。


GitHub SSH配置：
產生SSH金鑰：
ssh-keygen -t rsa -b 4096 -C "david.adm@gmail.com"
Enter file in which to save the key (C:\Users\David/.ssh/id_rsa):直接enter
Enter passphrase (empty for no passphrase):直接enter

進入 C:\Users\David\.ssh 可以看到新增了兩個檔案：id_rsa.pub 與 id_rsa
開啟 id_rsa.pub，複製內容

到 Github > Settings > SSH and GPG keys
Title：EC2 ssh
貼上剛剛的：


Clone the repository：
git clone https://github.com/david162929/Jenkins-test.git

目錄下多一個 Jenkins-test

使用 node app.js 試跑，可以正常造訪 http://18.221.56.89:8081/




https://medium.com/konvergen/jenkins-for-node-js-app-on-aws-ec2-part-3-jenkins-node-js-app-integration-1fa9d1306d25

前往管理頁：http://18.221.56.89:8080/
管理 Jenkins > 管理外掛程式 > 可用的
搜尋 NodeJS > 下載並於重新啟動後安裝
勾選當下載完成時且沒有工作正在執行時重新啟動
Jenkins完成重啟

管理 Jenkins > Global Tool Configuration > NodeJS 安裝 > 新增NodeJS > NodeJS 11.13.0

選擇 新增作業 > Enter an item name ： node-app > 建置 Free-Style 軟體專案
勾選 GitHub project > 填入 https://github.com/david162929/Jenkins-test

原始碼管理 > 勾選 git > 填入 https://github.com/david162929/Jenkins-test.git

建置觸發程序 > 勾選 GitHub hook trigger for GITScm polling

建置環境 > Provide Node & npm bin/ folder to PATH > NodeJS 11.13.0

建置 > 新增建置步驟 > 執行 shell
指令：npm install



到 https://github.com/david162929/Jenkins-test/settings > Webhooks > add webhook
Payload URL 填入：http://18.221.56.89:8080/github-webhook/
勾選 Just the Push Event
勾選 Active


由於用Docker建，要特別進去container使用者的目錄
https://stackoverflow.com/questions/20932357/how-to-enter-in-a-docker-container-already-running-with-a-new-tty
https://stackoverflow.com/questions/30172605/how-do-i-get-into-a-docker-containers-shell/30173220#30173220

docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                               NAMES
e7f9ca615ed4        jenkinsci/blueocean   "/sbin/tini -- /usr/…"   9 minutes ago       Up 9 minutes        0.0.0.0:8080->8080/tcp, 50000/tcp   wizardly_volhard


docker exec -it e7f9ca615ed4 bash	//進到 container

su - jenkins				//切換使用者Switch to Jenkins user


ssh-keygen -t rsa
Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa):直接enter
Enter passphrase (empty for no passphrase):直接enter
Enter same passphrase again:直接enter

cat ~/.ssh/id_rsa.pub
複製顯示出來的：

一直exit回到最外層，到專案目錄的地方。

vim ~/.ssh/authorized_keys		//進入金鑰的地方

在尾端按enter開新行，加上剛剛的一串ssh-rsa

chmod 700 ~/.ssh			//更改讀寫權限
chmod 600 ~/.ssh/*

重新進到container中：
docker exec -it e7f9ca615ed4 bash
su - jenkins
ssh ec2-user@18.221.56.89		//可以正常接上，代表剛剛設置有成功


回到專案 D:\David\阿成專用\Appworks School 2019 spring\駐點集訓\2019.3.29-期中考\Jenkins-test
新增script資料夾，裡面新增 deploy 檔案：

#!/bin/sh

ssh ec2-user@18.221.56.89 <<EOF
    cd ~/Jenkins-test
    git pull origin master
    curl -o-   https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh    | bash
    . ~/.nvm/nvm.sh
    nvm install v10.11.0
    npm install
    npm install -g nodemon pm2
    pm2 restart app2.js
    exit
EOF


回到 Jenkins > node-app專案 > 組態
http://18.221.56.89:8080/job/node-app/configure

執行Shell 加上 ./script/deploy

完整為：
npm install
./script/deploy




測試push之後會出現失敗，該次建構的終端機輸出會顯示：
+ npm install
env: ‘node’: No such file or directory


暫時拿掉 npm install


測試push之後會出現失敗，該次建構的終端機輸出會顯示：
+ ./script/deploy
/tmp/jenkins5465213239762472672.sh: line 2: ./script/deploy: Permission denied

解法：
https://stackoverflow.com/questions/21691202/how-to-create-file-execute-mode-permissions-in-git-on-windows
在windows內無法直接從644改成755，但是可以透過git來做。

在上傳前，進到 Jenkins-test\script，執行
git ls-files --stage
100644 c0db792e35d3ea4772b197fa6015b26662fde1fb 0       deploy		//確認是644

git update-index --chmod=+x deploy

git ls-files --stage
100755 c0db792e35d3ea4772b197fa6015b26662fde1fb 0       deploy		//確認改為755

之後再重新commit一次即可：
git commit -m "executable"



Push一次後，錯誤訊息變為：
+ ./script/deploy
Pseudo-terminal will not be allocated because stdin is not a terminal.
Host key verification failed.

解法：
https://stackoverflow.com/questions/7114990/pseudo-terminal-will-not-be-allocated-because-stdin-is-not-a-terminal
在deploy改為 ssh -tt ec2-user@18.221.56.89


Push一次後，錯誤訊息變為：
+ ./script/deploy
Host key verification failed.

因為我的 Jenkins 在安裝的時候是用 root user，剛剛的設置都是用 Jenkins user

ssh-keygen -t rsa
Enter file in which to save the key (/root/.ssh/id_rsa):

cat /root/.ssh/id_rsa.pub

回到專案目錄的地方。
vim ~/.ssh/authorized_keys 加上上述


再Push一次，成功!!!
[PM2] Applying action restartProcessId on app [app2.js](ids: 1)
[PM2] [app2](1) ✓
┌──────────┬────┬─────────┬──────┬───────┬─────────┬─────────┬────────┬─────┬────────────┬──────────┬──────────┐
│ App name │ id │ version │ mode │ pid   │ status  │ restart │ uptime │ cpu │ mem        │ user     │ watching │
├──────────┼────┼─────────┼──────┼───────┼─────────┼─────────┼────────┼─────┼────────────┼──────────┼──────────┤
│ app      │ 0  │ 1.0.0   │ fork │ 0     │ stopped │ 5       │ 0      │ 0%  │ 0 B        │ ec2-user │ disabled │
│ app2     │ 1  │ 1.0.0   │ fork │ 17591 │ online  │ 2       │ 0s     │ 0%  │ 788.0 KB   │ ec2-user │ disabled │
└──────────┴────┴─────────┴──────┴───────┴─────────┴─────────┴────────┴─────┴────────────┴──────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app
[ec2-user@ip-172-31-45-96 Jenkins-test]$     exit
logout
Connection to 18.221.56.89 closed.
Finished: SUCCESS