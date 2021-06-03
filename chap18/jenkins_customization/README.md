# 18章 Jenkinsのカスタマイズ
S2Iを用いたJenkinsのカスタマイズのサンプルです。
以下2つのファイルを用いて、Jenkinsをカスタマイズします。

| ファイル | 説明 |
:-----------|:------------|
`jenkins_build.yaml` | JenkinsイメージをビルドするためのBuildConfigなどのマニフェストファイル|
`pluginx.txt` | Jenkinsにインストールするプラグインとそのバージョンを管理 |
`configuration/jobs/preset-job/config.xml` | Jenkinsに事前設定するジョブ設定|
