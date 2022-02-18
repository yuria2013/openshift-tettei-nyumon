# 正誤表

|ページ|節|概要|
|---|---|---|
|162|8.2|[サンプルアプリのデプロイ時、Serivce作成コマンドが必要](https://github.com/team-ohc-jp-place/openshift-tettei-nyumon/blob/main/Errata.md#82%E3%81%A7%E3%81%AE%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AE%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4-p162)|
|173|8.2.5|[図8.5のノードのIPアドレス](https://github.com/team-ohc-jp-place/openshift-tettei-nyumon/blob/main/Errata.md#%E5%9B%B385%E3%81%AE%E3%83%8E%E3%83%BC%E3%83%89%E3%81%AEip%E3%82%A2%E3%83%89%E3%83%AC%E3%82%B9)|

## 8.2でのサンプルアプリのデプロイ (p.162)

8.2節(p.162)で

```
$ oc new-project proj1
$ oc new-app https://github.com/orimanabu/openshift-sample-go --name hello
```

とありますが、この次に必要なコマンドがひとつ抜けていました。
下記コマンドを実行し、Serviceを作成する必要があります。

```
$ oc expose deploy/hello --port=80 --target-port=8080
```

## 図8.5のノードのIPアドレス

8.2.5の図8.5(p.173)で、worker-2のenp1s0のIPアドレスが `172.16.11.104` と記載されていますが、正しくは `172.16.11.106` です。
