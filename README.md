# singularity_recipe_jekyll
DDBJ homepageは、静的サイトジェネレータ[Jekyll](https://jekyllrb.com/)で作られています。ここに設置した[Singularity](https://sylabs.io/) recipeファイルを使うことで、DDBJ homepage用jekyll環境がインストールされたsingularityコンテナを各自のマシン上で作成できます。

# ファイル名が示す意味
例えば、`jekyll434_Ubuntu2204` の場合

    jekyll + 434 + Ubuntu + 2204

というパーツでファイル名が構成されます。これは、 このrecipeファイルを使うことで jekyll version 4.3.4 を Ubuntu ver. 22.04 にインストールしたsingularity containerが作成できることを示します。 formerフォルダーにあるのは、現在は使用していない古いrecipeです。

# Jekyllのバージョンについて
Jekyll 4.3.4  
本物のDDBJ Webサーバーでは、このバージョンを使用しています(2025年2月3日～)。

Jekyll 4.2.0  
2021年1月ぐらいから2025年2月3日まで、本物のDDBJ Webサーバーで使用していたバージョンです。このバージョン以降、過去の3.#.# 系バージョンと比較して、実行速度の大幅な向上、バックグラウンド実行モードがデフォルトになるなど、使い勝手がかなり向上しました。当時GitHub Pages用のプラグインには対応してなかったです。

Jekyll 3.9.0  
GitHubには GitHub Pagesという機能が備わっており、リポジトリからJekyllでのウェブ公開ができます。そのGitHub Pageで使用されていたバージョンです(2021年1/21時点)。

Jekyll 3.8.5  
DDBJ homepage開発でhtml確認用に最初に使ったバージョンです。

# Singularity コンテナを作成する
Singularityは各自のマシンで用意してください。https://sylabs.io/docs/ の各バージョン横にある User Guide 内にインストール方法が載っています。なお、コンテナ作成ではsudo権が必要です。

    e.g. mkdir ~/sif
    cd ~/sif
    git clone https://github.com/ddbj/singularity_recipe_jekyll.git
    sudo singularity build jekyll434ubuntu.sif singularity_recipe_jekyll/jekyll434_Ubuntu2204

# Jekyll実行の前に行うこと
gitの使い方を覚えましょう。そして、GitHub https://github.com/ddbj/www.git にあるDDBJ homepageレポジトリをあなたのマシンにクローンしてください。そこがあなた自身の編集環境になります。

# 作成したコンテナでJekyllを実行する
コンテナには、DDBJ homepage作成のために必要な部品がすべてインストールされていますから、`bundle install`で ruby gemを追加インストールする必要はありません。コンテナ実行にsudoは不要です。~/sifディレクトリにコンテナを作成したのであれば、次の書式でコマンドを実行します。

    singularity exec ~/sif/jekyll434ubuntu.sif 命令

    例 Jekyllバージョン表示
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll --version
    例 Jekyllヘルプ表示
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll --help
    例 build オプションヘルプ表示
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --help
    例 html作成
    cd www/
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --no-watch

# htmlファイルをbuildする
Jekyll build とは

Jekyllにおいて、各自が編集するのはMarkdownファイル(.md)であり、編集したmdファイルを各自がGitHubにpushします。実際のWebサーバー上では、GitHubから最新のmdファイルを取得後、mdファイルのコンテンツを基にヘッダー・フッター、js、cssなどを適用したhtmlファイルが置かれます。この説明内では、Jekyllを使って mdファイルを基にhtmlを作成することをbuildと呼びます。もし、各自の環境で実際のWebサーバーで見るのと同じhtmlページを作り、どのように見えるかを確認したい場合は、各自の環境でbuildを行えば可能です。

## 全mdファイルを対象にhtmlを作成する場合

    編集対象のディレクトリにcdする
    　↓
    例 cd www
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --no-watch
    　↓
    _siteディレクトリに、htmlが作成される

Jekyll コマンドでオプション指定無しだと`--watch`オプションが適用されます。--watch指定時には、プロンプトを返さず、ファイルの変更を見張り続けます。jekyll 4.2以降において、次の2つのコマンドは同じ動作になります。

    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --watch
    停止するには
    Ctrl + c

一方で、build後ファイル変更を見張ることなく、命令を終了するには次のように`--no-watch`を指定します。

    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --no-watch


なお、Jekyll 3.8.5 と 3.9.0 では、デフォルトで`--no-watch`が適用されますので、次の2つのコマンドは同じ動作になります。

    singularity exec ~/sif/jekyll390CentOS.sif jekyll build
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --no-watch

Jekyll 3.8.5 と 3.9.0 を使用していたころは、ファイルの変更を見張り続けるために`--watch`を指定する必要がありました。

    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --watch
    停止するには
    Ctrl + c

**更新があったファイルのみのbuidに絞りたいとき**

jekyll buildをオプション指定なしで実行すると、すべてのmdファイルを対象にbuildが行われます。DDBJ homepage全体ではすべてのbuildに数十秒かかります(jekyll 4.2.0以降での目安、3.9.0では1分以上)。これだと煩わしいので、変更が生じたファイルだけを対象にしておきたいところです。これを可能にするには、jekyllの実行時に`--incremental`オプションを加えます。

Jekyll 4.2以降でオプション指定無しだと`--watch`オプションが適用されますので、`--incremental`だけを指定すれば` --incremental --watch`を指定したのと同じになります。

    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --incremental
    上のコマンドは singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --incremental --watch と同じ
    停止するには
    ctrl + c

Jekyll 4.2以降において、--watch を適用したくないときは、`--no-watch`を指定する必要があります。

    singularity exec ~/sif/jekyll434ubuntu.sif jekyll build --incremental --no-watch

一方で、Jekyll 3.8.5 と 3.9.0 では、--no-watchがデフォルトで適用されます。

    プロンプトを返さずにファイルの変更を見張り続けるとき
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental --watch
    停止は ctrl + c

    build後、命令を終了する場合    
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental
    上のコマンドは singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental --no-watch と同じ

## htmlを作成し、各自のブラウザから確認したい場合
Jekyllの簡易サーバー機能を利用して、buildされたhtmlページを各自のブラウザで閲覧することができます。`--incremental`と併用すると、jekyll server実行中に変更があったファイルだけを対象にして、buildが行われます。そのためファイル編集後、すぐにブラウザ上で編集結果を確認することができます(とはいっても、--incrementalの有無での違いは30秒から1分ぐらいか)。

    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll serve
    または、build対象を変更されたファイルだけにしぼる
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll serve --incremental
    　↓
    md編集環境のあるマシンでブラウザを開いて、http://localhost:4000 にアクセスして確認、_siteディレクトリに、htmlが作成される。停止はctrl+c。

デフォルトではport 4000が使われます。別のポート番号に変えたいときには `--port=ポート番号`で指定します。

    ボート53000を使う場合
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll serve --port=53000
    または
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll serve --port=53000 --incremental
    　↓
    http://localhost:53000 にアクセス、停止はctrl+c

## htmlを作成し、ほかのマシンのブラウザから閲覧させたい場合
`--host=0.0.0.0`オプションを加えると、buildされたhtmlを自分のマシンのIPから公開することができます。Firewallでポートを開く必要があるので、この方法はセキュリティの知識のある方限定でお願いします。

    firewallで公開ポートを開く、必要に応じてその公開ポートでのアクセス制限を行う
    　↓
    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll serve --port=公開ポート番号 --host=0.0.0.0
    または
    singularity exec ~/sif/jekyll434ubuntu.sif jekyll serve --port=公開ポート番号 --host=0.0.0.0 --incremental
    　↓
    ほかのマシンでブラウザを開いて、http://上記命令を実行したマシンのIPアドレス:公開ポート番号 にアクセスして確認、_siteディレクトリには、htmlが作成される、停止はctrl+c

*Singularityコンテナ実行詳細については、https://sylabs.io/docs/ の各バージョン横にある User Guideを参照して下さい。*

# 以降、過去の運用情報
## コンテナrecipeのOSでCentOS版かUbuntu版ではどちらがよいか
インストールされているJekyllの機能は一緒なので、好みで選択して構いません。もし迷ったらCentOS版を使ってください。Ubuntu版では、あなたの homeの環境変数LANGがen_US.UTF-8(おそらく、ja_JP.UTF-8でない場合)のとき、以降に書かれているJekyllコマンド実行時に特定のRubygemがエラーになります。Ubuntu版では各自の実行環境で次の設定が必要です。CentOS版では必要ありません。

    export LANG=ja_JP.UTF-8

## Gemfile.lockとGemfile
クローンしたファイル内には Gemfile、Gemfile.lock というファイルが含まれているはずです。このページからダウンロードできるsingularity recipeでコンテナを作成し、そのコンテナを使ってhtmlファイルの確認だけを行うのであれぱ、GemfileとGemfile.lockファイルはいずれも不要です。次のようにして削除します。

    rm Gemfile Gemfile.lock

## Gemfileを削除せずに残して編集を行う場合
もし、Gemfileを削除しないのであれば、jekyll versionに応じたGemfileの編集が必要になります。いずれの場合も、git clone直後のGemfile.lockは削除してください。

## git clone直後のGemfile.lockは不要
git clone直後のGemfile.lockは削除します。  
以降のjekyll buildやjekyll serverコマンド実行後、Gemfile.lockが再び作成されますがその時は削除不要です。ただし、途中でjekyll のversionを変えた場合には、Gemfile.lockをいったん削除してください。

    rm Gemfile.lock

## Gemfileの編集
Jekyll versionによって、編集内容が異なります。

### Jeyll 各versionで共通
    gem "jekyll", "~> 3.8.5"
    　↓　Jeyll versionによらず、~> を >= に変えます。
    gem "jekyll", ">= 3.8.5"

### Jekyll 4.2.0
Jekyll 4.2.0 では、github-pages に未対応(2021年1/21時点)のため以下のようにして github-pages が使われないようにします。

    gem "github-pages", group: :jekyll_plugins
    　↓　この行をコメントアウト
    # gem "github-pages", group: :jekyll_plugins

    加えて、group :jekyll_plugins do ～ end 行までのセクションを次のようにします。

    group :jekyll_plugins do
      gem "jekyll-feed", "~> 0.12"
      gem "jekyll-sitemap"
      gem "jekyll-seo-tag"
      gem "jekyll-haml"
    end
    　↓
    gem "jekyll-feed", "~> 0.12"
    gem "jekyll-sitemap"
    gem "jekyll-seo-tag"
    gem "jekyll-haml"

### Jekyll 3.9.0
    gem "github-pages", group: :jekyll_plugins
    　↓　この行の github-pages に "209" を指定します。
    gem "github-pages", "209", group: :jekyll_plugins

### Jekyll 3.8.5
    gem "github-pages", group: :jekyll_plugins
    　↓　この行の github-pages に "204" を指定します。
    gem "github-pages", "204", group: :jekyll_plugins
