# singularity_recipe_jekyll
Jekyllのsingularity containerを作成できるrecipeファイルを置いています。

# ファイル名が示す意味
例えば、`jekyll390_CentOS7` の場合

    jekyll + 390 + CentOS + 7

というパーツでファイル名が構成されます。これは、 このrecipeファイルを使うことで jekyll version 3.90 を CentOS ver. 7 にインストールしたsingularity containerが作成できることを示しています。 もしjekyll 3.9 Ubuntu版のコンテナを作りたいのであれば

    jekyll390_Ubuntu2004

を使えばよいことになります。

# Singularity コンテナを作成する
Singularityは各自のマシンで用意してください。https://sylabs.io/docs/ の各バージョン横にある User Guide 内にインストール方法が載っています。なお、コンテナ作成ではsudo権が必要です。

    コマンド
    sudo singularity build コンテナ名.sif recipeファイル名

カレントディレクトリにあるrecipeファイルを使って、カレントディレクトリにコンテナを作成する場合。

    sudo singularity build jekyll390CentOS.sif jekyll390_CentOS7

ホームのrecipeディレクトリにダウンロードしてきたrecipeファイルを使って、~/sifディレクトリにコンテナを作成する例

    sudo singularity build ~/sif/jekyll390CentOS.sif ~/recipe/jekyll390_CentOS7

# 作成したコンテナでJekyllを実行する
作成したコンテナを指定してJekyllを実行できます。コンテナの実行にはsudoは不要です。~/sifディレクトリにコンテナを作成したのであれば、次の書式でコマンドを実行します。

    singularity exec ~/sif/jekyll390CentOS.sif 実行命令

## 編集後のページを見たい・見せたい
Jekyll build とは

Jekyllにおいて、各自が編集するのはMarkdownファイル(.md)であり、編集したmdファイルを各自がGitHubにpushします。実際のWebサーバー上には、GitHubから最新のmdファイルが取得され、mdファイルのコンテンツにヘッダー・フッター、js、cssなどが適用されたhtmlが置かれます。Jekyllでは、mdファイルを基にhtmlを作成することをbuildと呼んでいます。もし、各自の環境で実際のWebサーバーで見るのと同じhtmlページを作り、どのように見えるかを確認したい場合は、各自の環境でbuildを行うことで可能になります。

### htmlだけ作成したい場合

    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build
    　↓
    _siteディレクトリに、htmlが作成される

### htmlを作成し、各自のマシンにあるブラウザから確認したい場合

    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve
    　↓
    md編集環境のあるマシンでブラウザを開いて、http://localhost:4000 にアクセスして確認
    かつ_siteディレクトリに、htmlが作成される

デフォルトではport 4000が使われます。別のポート番号を指定したいときには `--port=ポート番号`を指定します。

    ボート53000を使う場合
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --port=53000
    　↓
    http://localhost:53000 にアクセス

### htmlを作成し、ほかのマシンのブラウザで閲覧させたい場合

    firewallで公開ポートを開く、必要に応じてその公開ポートでのアクセス制限を行う
    　↓
    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --port=公開ポート番号 --host=0.0.0.0
    　↓
    ほかのマシンでブラウザを開いて、http://上記命令を実行したマシンのIPアドレス:公開ポート番号 にアクセスして確認
    かつ_siteディレクトリには、htmlが作成される

Singularityコンテナの実行詳細については、https://sylabs.io/docs/ の各バージョン横にある User Guideを参照して下さい。
