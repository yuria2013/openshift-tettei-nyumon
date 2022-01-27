# 正誤表

|ページ|節|概要|
|---|---|---|
|162|8.2|サンプルアプリのデプロイ時、Serivce作成コマンドが必要|

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
