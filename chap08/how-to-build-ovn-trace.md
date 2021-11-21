# ovn-traceにパッチを適用して再コンパイルする方法

原稿執筆時点(2020-11-21)、OCP v4.9.7 (のovnkube-masterのnorthdコンテナ) に付属のovn-traceに不具合があり、正しいトレースが取れない場合がありました。下記パッチを適用し、再コンパイルすることで、正しいトレースが取れます。

- https://github.com/ovn-org/ovn/commit/c117ce1cf32a868af50c25eadc790ea20cd08fe8

# パッチ適用方法

1. RHEL8が稼動する環境を用意する (仮想マシンがお手軽だと思います)

2. 下記2つのリポジトリを有効化する
  - fast-datapath-for-rhel-8-x86_64-rpms
  - codeready-builder-for-rhel-8-x86_64-rpms
  
```
sudo subscription-manager repos --enable fast-datapath-for-rhel-8-x86_64-rpms  --enable codeready-builder-for-rhel-8-x86_64-rpms
```

3. ovnのソースコード (Source RPM) を取得しインストールする

```
dnf download --source ovn-2021
rpm -ivh ovn-2021-21.06.0-29.el8fdp.src.rpm
cd ~/rpmbuild
```

4. ビルドに必要なrpmをインストールする

```
sudo dnf install rpm-build
sudo dnf builddep SPECS/ovn-2021.spec
```

5. パッチをダウンロードする

```
curl -s -o SOURCES/ovn-trace.patch https://github.com/ovn-org/ovn/commit/c117ce1cf32a868af50c25eadc790ea20cd08fe8.patch
```

6. SPECファイルを編集する

```
patch -p0 <<END
--- SPECS/ovn-2021.spec.orig    2021-11-21 22:41:55.044680285 +0900
+++ SPECS/ovn-2021.spec 2021-11-21 22:41:55.044680285 +0900
@@ -86,6 +86,7 @@
 Source506: x86_64-native-linuxapp-gcc-config

 Patch:     ovn-%{version}.patch
+Patch1:    ovn-trace.patch

 # FIXME Sphinx is used to generate some manpages, unfortunately, on RHEL, it's
 # in the -optional repository and so we can't require it directly since RHV
END
```

7. rpmパッケージをビルドする

```
rpmbuild -bb SPECS/ovn-2021.spec
```

8. テストスイート (make check-local) が流れ始めたら `^C` で止める

9. コンパイルできたバイナリ `BUILD/ovn-21.06.0/utilities/ovn-trace` をscp等でOCPのノードに転送する

# 実行方法

1. OVSDBアクセス用の証明書と鍵を取得する

```
oc -n openshift-ovn-kubernetes get cm/ovn-ca -o json | jq -r '.data."ca-bundle.crt"' > ca-bundle.crt
oc -n openshift-ovn-kubernetes get secret/ovn-cert -o json | jq -r '.data."tls.crt"' | base64 -d > tls.crt
oc -n openshift-ovn-kubernetes get secret/ovn-cert -o json | jq -r '.data."tls.key"' | base64 -d > tls.key
```

2. 1.で取得したca-bundle.crt, tls.crt, tls.keyをOCPのノードにscp等で転送する

3. コンパイルしたバイナリ、証明書、鍵を転送したノードで下記のように実行する (下記コマンド例では、転送したファイルが全てカレントディレクトリにあると想定)

```
./ovn-trace -p ./tls.key -c ./tls.crt -C ./ca-bundle.crt \
--db 'ssl:<RaftのleaderになっているmasterノードのIPアドレス>:9642' \
<datapath名> --ct new \
<トレースのパラメータ>
```

補足
- どのmasterノードのovnkube-master PodがRaftのどのRoleか (leaderかmasterか) は、ovnkube-master Podに対して下記のコマンドを実行することで確認できる

```
oc -n openshift-ovn-kubernetes exec -c sbdb pod <ovnkube-masterのPod名> -- ovn-appctl -t /var/run/ovn/ovnsb_db.ctl cluster/status OVN_Southbound | grep ^Role
```

- datapath名は、ロジカルスイッチやロジカルルーターの名前を指定する

