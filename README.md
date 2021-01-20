# singularity_recipe_jekyll
DDBJ homepageは[Jekyll](https://jekyllrb.com/)で作られています。ここに設置した[Singularity](https://sylabs.io/) recipeファイルを使うことで、DDBJ homepage用のjekyllを実行環境がインストールされたsingularityコンテナを、各自のマシン上で作成できます。

# ファイル名が示す意味
例えば、`jekyll390_CentOS7` の場合

    jekyll + 390 + CentOS + 7

というパーツでファイル名が構成されます。これは、 このrecipeファイルを使うことで jekyll version 3.90 を CentOS ver. 7 にインストールしたsingularity containerが作成できることを示します。 もしjekyll 3.9 Ubuntu版のコンテナを作りたいのであれば

    jekyll390_Ubuntu2004

を使えばよいことになります。

# CentOSかUbuntuではどちらがよいか
インストールされているJekyllの機能は一緒なので、好みで選択して構いません。もし迷ったらCentOS版を使ってください。Ubuntu版では、あなたの homeの環境変数LANGがen_US.UTF-8(おそらく、ja_JP.UTF-8でない場合)のとき、以降に書かれているJekyllコマンド実行時に特定のRubygemがエラーになります。Ubuntu版では各自の実行環境で次の設定が必要です。CentOS版では必要ありません。

    export LANG=ja_JP.UTF-8

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

# Jekyll実行の前に行うこと
gitの使い方を覚えましょう。そして、GitHub https://github.com/ddbj/www.git にあるDDBJ homepageレポジトリをあなたのマシンにクローンしてください。そこがあなた自身の編集環境になります。

# Gemfileを編集、クローン直後のGemfile.lockを削除
クローンしたファイル内には Gemfile、Gemfile.lock というファイルが含まれているはずです。以降に記載されたコマンドを各自のマシンで実行するためには、まず Gemfile の編集とGemfile.lockファイルの削除が必要です。
まず、Gemfile.lockを削除して下さい。つづいてGemfileを編集します。変更箇所は以下の2か所です。

    - クローン直後のGemfile.lockを削除する
    rm Gemfile.lock
    
    - Gemfileを編集
    gem "jekyll", "~> 3.8.5"
    　↓　~> を >= に変える
    gem "jekyll", ">= 3.8.5"

    gem "github-pages", group: :jekyll_plugins
    　↓　この行をコメントアウトする
    # gem "github-pages", group: :jekyll_plugins

# 編集後のページを見たい・見せたい
Jekyll build とは

Jekyllにおいて、各自が編集するのはMarkdownファイル(.md)であり、編集したmdファイルを各自がGitHubにpushします。実際のWebサーバー上には、GitHubから最新のmdファイルが取得され、mdファイルのコンテンツにヘッダー・フッター、js、cssなどが適用されたhtmlが置かれます。Jekyllでは、mdファイルを基にhtmlを作成することをbuildと呼んでいます。もし、各自の環境で実際のWebサーバーで見るのと同じhtmlページを作り、どのように見えるかを確認したい場合は、各自の環境でbuildを行うことで可能になります。

## 全mdファイルを対象にhtmlを作成したい場合

    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build
    　↓
    _siteディレクトリに、htmlが作成される

**更新があったファイルのみのbuidに絞りたいとき... Jekyll 3.9と3.8.5バージョンが対象**

jekyll buildをオプション指定なしで実行すると、すべてのmdファイルを対象にbuildが行われます。DDBJ homepage全体ではすべてのbuildに1～2分程度かかります。これだと煩わしいので、変更が生じたファイルだけを対象にしておきたいところです。これを可能にするには、jekyllの実行時に`--incremental --watch`オプションを加えます。  

--incrementalは、更新のあった差分ファイルのみを対象にするというオプション、  
--watchは、編集があったことを見張り続けるためのオプションです。

    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental --watch

**Jekyll 4.2では、--incrementalがデフォルトで適用されるため、--watchのみでも良いようです**

## htmlを作成し、各自のマシンにあるブラウザから確認したい場合
Jekyllの簡易サーバー機能を利用して、自分のブラウザでbuildされたhtmlページを閲覧することができます。`--incremental --watch`オプションと併用すれば、mdファイルを編集しながら、同時にブラウザでhtmlの確認も行うことができます。

    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve
    または
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --incremental --watch
    　↓
    md編集環境のあるマシンでブラウザを開いて、http://localhost:4000 にアクセスして確認、停止はctrl+c
    かつ_siteディレクトリに、htmlが作成される

デフォルトではport 4000が使われます。別のポート番号に変えたいときには `--port=ポート番号`で指定します。

    ボート53000を使う場合
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --port=53000
    または
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --port=53000 --incremental --watch
    　↓
    http://localhost:53000 にアクセス
    停止はctrl+c

## htmlを作成し、ほかのマシンのブラウザで閲覧させたい場合
`--host=0.0.0.0`オプションを加えると、buildされたhtmlを自分のマシンのIPから公開することができます。Firewallでポートを開く必要があるので、この方法はセキュリティの知識のある方だけに限定します。

    firewallで公開ポートを開く、必要に応じてその公開ポートでのアクセス制限を行う
    　↓
    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --port=公開ポート番号 --host=0.0.0.0
    または
    singularity exec ~/sif/jekyll390CentOS.sif bundle exec jekyll serve --port=公開ポート番号 --host=0.0.0.0 --incremental --watch
    　↓
    ほかのマシンでブラウザを開いて、http://上記命令を実行したマシンのIPアドレス:公開ポート番号 にアクセスして確認
    停止はctrl+c
    _siteディレクトリには、htmlが作成される

*Singularityコンテナ実行詳細については、https://sylabs.io/docs/ の各バージョン横にある User Guideを参照して下さい。*
