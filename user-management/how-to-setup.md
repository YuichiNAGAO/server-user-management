# Ansible + Self-Hosted Github Actionsの環境構築


![picture](https://github.com/Hyper-Vision-Research-Laboratory/server-management/blob/images/images/picture1.png)

### 前提

2021年12月時点研究室で使われている共同サーバーは以下のようである。

- 10.241.189.8 (CPU Server)
- 10.241.172.213 (DL-Box)
- 10.241.219.135 (DL-Box II)
- 10.241.27.59 (DL-Box III)
- 10.241.35.204 (DL-Box IV)
- 10.241.45.5 (DL-Box V)
- 10.241.145.140 (crest-super-01)*
- 10.241.2.196 (crest-super-03)*

*あまり稼働していない古いサーバーである。


### 1. 全ての共同サーバでのsudo権限をもつAnsibleユーザーの発行

1. ansibleユーザーの発行。`-m`はホームディレクトリを設定する。
```
sudo useradd -s /bin/bash -m ansible
```
2. パスワードの設定(サーバー係が知っています)
```
sudo passwd ansible
```
3. ansibleのsudoグループへの追加
```
sudo usermod -aG sudo ansible
```
4. ansibleがsudoに追加されているか確認
```
cat /etc/group | grep sudo
```

- ロールバック
```
sudo userdel -r ansible
```

### 2. Self-Hosted Github Runnerの設置

Github Actionをホストする環境として、10.241.189.8 (CPU Server)を選ぶこととする。

https://qiita.com/hanaokatomoki/items/af47da39a61948fb123f

1. [Settings]→[Actions]→[New self-hosted runner]

2. 10.241.189.8 (CPU Server)で以下のコマンドを実行

    2.1 ansibleユーザーでssh

    ```
    ssh ansible@10.241.189.8
    ```
    2.2 1での指示通りに行う(コマンドは一例)
    ```
    mkdir actions-runner && cd actions-runner
    curl -o actions-runner-linux-x64-2.284.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.284.0/actions-runner-linux-x64-2.284.0.tar.gz
    echo "1ddfd7bbd3f2b8f5684a7d88d6ecb6de3cb2281a2a359543a018cc6e177067fc  actions-runner-linux-x64-2.284.0.tar.gz" | shasum -a 256 -c
    tar xzf ./actions-runner-linux-x64-2.284.0.tar.gz
    ./config.sh --url https://github.com/Hyper-Vision-Research-Laboratory/server-management --token AMFU5LIVXFTEHL52HRR4HM3BUELV4
    ```
    2.3 サービス化
https://docs.github.com/en/actions/hosting-your-own-runners/configuring-the-self-hosted-runner-application-as-a-service
    ```
    cd actions-runner
    sudo ./svc.sh install
    sudo ./svc.sh start
    ```
- ロールバック
```
sudo ./svc.sh stop
sudo ./svc.sh uninstall
```
### 3. Ansibleの設定。
1. CPU-Server(10.241.189.8)においてAnsibleのインストール(Self-Hosted Runnerにansibleはインストールされている必要がある)
```
ssh ansible@10.241.189.8
sudo update 
sudo apt install ansible
```

2. 秘密鍵でSlef-Hosted(10.241.189.8)から他のサーバーに接続できるようにする

```
ssh ansible@10.241.189.8
cat .ssh/id_rsa.pub | pbcopy
```

```
ssh ansible@<self-hosted以外のサーバー>
cd .ssh
vim authorized_keys
chmod 600 authorized_keys
```


3. 環境変数の設定
Self-Hosted Github Runner(10.241.189.8)から他のサーバーに接続したのち、sudoになるためのパスワードをGithubのSecretsを使って環境変数に設定する。

    [Settings]→[Secrets]→[New Repository Secret]
    
    Key: SUDO_PASSWD
    Value: <Ansibleユーザーのパスワード>
