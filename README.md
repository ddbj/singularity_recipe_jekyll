# singularity_recipe_jekyll
DDBJ homepageは、静的サイトジェネレータ[Jekyll](https://jekyllrb.com/)で作られています。ここに設置した[Singularity](https://sylabs.io/) recipeファイルを使うことで、DDBJ homepage用jekyll環境がインストールされたsingularityコンテナを各自のマシン上で作成できます。

# ファイル名が示す意味
例えば、`jekyll420_CentOS7` の場合

    jekyll + 420 + CentOS + 7

というパーツでファイル名が構成されます。これは、 このrecipeファイルを使うことで jekyll version 4.2.0 を CentOS ver. 7 にインストールしたsingularity containerが作成できることを示します。 もしjekyll 3.9 Ubuntu版のコンテナを作りたいのであれば

    jekyll390_Ubuntu2004

を使えばよいことになります。

# Jekyllのバージョンについて
Jekyll 3.8.5、3.9.0、および 4.2.0 の3つのJekyllバージョン用recipeファイルを用意しています。各自の環境だけでページ確認をすることが目的であれば、速度の速い、一番新しいバージョンを使用するのが使いやすいでしょう。

Jekyll 4.2.0  
本物のDDBJ Webサーバーでは、このバージョンを使用しています。3.#.# の過去バージョンと比較して、実行速度の大幅な向上、バックグラウンド実行モードがデフォルトになるなど、使い勝手もよくなっています。当時GitHub Pages用のプラグインには対応してなかったです。

Jekyll 3.9.0  
また、GitHubには GitHub Pagesという機能が備わっており、リポジトリからJekyllでのウェブ公開ができます。そのGitHub Pageで使用されるバージョンでもあります(2021年1/21時点)。

Jekyll 3.8.5  
DDBJ homepage開発でhtml確認用に最初に使ったバージョンです。

# CentOSかUbuntuではどちらがよいか
インストールされているJekyllの機能は一緒なので、好みで選択して構いません。もし迷ったらCentOS版を使ってください。Ubuntu版では、あなたの homeの環境変数LANGがen_US.UTF-8(おそらく、ja_JP.UTF-8でない場合)のとき、以降に書かれているJekyllコマンド実行時に特定のRubygemがエラーになります。Ubuntu版では各自の実行環境で次の設定が必要です。CentOS版では必要ありません。

    export LANG=ja_JP.UTF-8

# Singularity コンテナを作成する
Singularityは各自のマシンで用意してください。https://sylabs.io/docs/ の各バージョン横にある User Guide 内にインストール方法が載っています。なお、コンテナ作成ではsudo権が必要です。

    sudo singularity build コンテナ名.sif recipeファイル名

カレントディレクトリにあるrecipeファイルを使って、カレントディレクトリにコンテナを作成する場合。

    sudo singularity build jekyll420CentOS.sif jekyll420_CentOS7

ホームのrecipeディレクトリにダウンロードしてきたrecipeファイルを使って、~/sifディレクトリにコンテナを作成する例

    sudo singularity build ~/sif/jekyll420CentOS.sif ~/recipe/jekyll420_CentOS7

# Jekyll実行の前に行うこと
gitの使い方を覚えましょう。そして、GitHub https://github.com/ddbj/www.git にあるDDBJ homepageレポジトリをあなたのマシンにクローンしてください。そこがあなた自身の編集環境になります。

# Gemfile.lockとGemfile
クローンしたファイル内には Gemfile、Gemfile.lock というファイルが含まれているはずです。このページからダウンロードできるsingularity recipeでコンテナを作成し、そのコンテナを使ってhtmlファイルの確認だけを行うのであれぱ、GemfileとGemfile.lockファイルはいずれも不要です。次のようにして削除します。

    rm Gemfile Gemfile.lock

# Gemfileを削除せずに残して編集を行う場合
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

# 作成したコンテナでJekyllを実行する
コンテナには、DDBJ homepage作成のために必要な部品がすべてインストールされていますから、`bundle install`で ruby gemを追加インストールする必要はありません。コンテナ実行にsudoは不要です。~/sifディレクトリにコンテナを作成したのであれば、次の書式でコマンドを実行します。

    singularity exec ~/sif/jekyll420CentOS.sif 実行命令

    例 Jekyllバージョン表示
    singularity exec ~/sif/jekyll420CentOS.sif jekyll --version
    例 Jekyllヘルプ表示
    singularity exec ~/sif/jekyll420CentOS.sif jekyll --help
    例 build オプションヘルプ表示
    singularity exec ~/sif/jekyll420CentOS.sif jekyll build --help

# 編集後のページをhtmlとして見たい・見せたい
Jekyll build とは

Jekyllにおいて、各自が編集するのはMarkdownファイル(.md)であり、編集したmdファイルを各自がGitHubにpushします。実際のWebサーバー上では、GitHubから最新のmdファイルを取得後、mdファイルのコンテンツを基にヘッダー・フッター、js、cssなどを適用したhtmlファイルが置かれます。この説明内では、Jekyllを使って mdファイルを基にhtmlを作成することをbuildと呼びます。もし、各自の環境で実際のWebサーバーで見るのと同じhtmlページを作り、どのように見えるかを確認したい場合は、各自の環境でbuildを行えば可能です。

## 全mdファイルを対象にhtmlを作成する場合

    編集対象のディレクトリにcdする
    　↓
    例 singularity exec ~/sif/jekyll420CentOS.sif jekyll build --no-watch
    　↓
    _siteディレクトリに、htmlが作成される

Jekyll 4.2では、オプション指定無しだと`--watch`オプションが適用されます。--watch指定時には、プロンプトを返さず、ファイルの変更を見張り続けます。jekyll 4.2において、次の2つのコマンドは同じ動作になります。

    singularity exec ~/sif/jekyll420CentOS.sif jekyll build
    singularity exec ~/sif/jekyll420CentOS.sif jekyll build --watch
    停止するには
    Ctrl + c

一方で、build後ファイル変更を見張ることなく、命令を終了するには次のように`--no-watch`を指定します。

    singularity exec ~/sif/jekyll420CentOS.sif jekyll build --no-watch


Jekyll 3.8.5 と 3.9.0 では、デフォルトで`--no-watch`が適用されますので、次の2つのコマンドは同じ動作になります。

    singularity exec ~/sif/jekyll390CentOS.sif jekyll build
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --no-watch

Jekyll 3.8.5 と 3.9.0 で、ファイルの変更を見張り続けるには`--watch`を指定します。

    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --watch
    停止するには
    Ctrl + c

**更新があったファイルのみのbuidに絞りたいとき**

jekyll buildをオプション指定なしで実行すると、すべてのmdファイルを対象にbuildが行われます。DDBJ homepage全体ではすべてのbuildに数十秒かかります(jekyll 4.2.0使用時の目安、3.9.0では1分以上)。これだと煩わしいので、変更が生じたファイルだけを対象にしておきたいところです。これを可能にするには、jekyllの実行時に`--incremental`オプションを加えます。

Jekyll 4.2でオプション指定無しだと`--watch`オプションが適用されますので、`--incremental`だけを指定すれば` --incremental --watch`を指定したのと同じになります。

    singularity exec ~/sif/jekyll420CentOS.sif jekyll build --incremental
    上のコマンドは singularity exec ~/sif/jekyll420CentOS.sif jekyll build --incremental --watch と同じ
    停止するには
    ctrl + c

Jekyll 4.2において、--watch を適用したくないときは、`--no-watch`を指定する必要があります。

    singularity exec ~/sif/jekyll420CentOS.sif jekyll build --incremental --no-watch

一方で、Jekyll 3.8.5 と 3.9.0 では、--no-watchがデフォルトで適用されます。

    プロンプトを返さずにファイルの変更を見張り続けるとき
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental --watch
    停止は ctrl + c

    build後、命令を終了する場合    
    singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental
    上のコマンドは singularity exec ~/sif/jekyll390CentOS.sif jekyll build --incremental --no-watch と同じ

## htmlを作成し、各自のマシンにあるブラウザから確認したい場合
Jekyllの簡易サーバー機能を利用して、buildされたhtmlページを各自のブラウザで閲覧することができます。`--incremental`と併用すると、jekyll server実行中に変更があったファイルだけを対象にして、buildが行われます。そのためファイル編集後、すぐにブラウザ上で編集結果を確認することができます(とはいっても、--incrementalの有無での違いは30秒から1分ぐらいか)。

    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll420CentOS.sif jekyll serve
    または、build対象を変更されたファイルだけにしぼる
    singularity exec ~/sif/jekyll420CentOS.sif jekyll serve --incremental
    　↓
    md編集環境のあるマシンでブラウザを開いて、http://localhost:4000 にアクセスして確認、_siteディレクトリに、htmlが作成される。停止はctrl+c。

デフォルトではport 4000が使われます。別のポート番号に変えたいときには `--port=ポート番号`で指定します。

    ボート53000を使う場合
    singularity exec ~/sif/jekyll420CentOS.sif jekyll serve --port=53000
    または
    singularity exec ~/sif/jekyll420CentOS.sif jekyll serve --port=53000 --incremental
    　↓
    http://localhost:53000 にアクセス、停止はctrl+c

## htmlを作成し、ほかのマシンのブラウザで閲覧させたい場合
`--host=0.0.0.0`オプションを加えると、buildされたhtmlを自分のマシンのIPから公開することができます。Firewallでポートを開く必要があるので、この方法はセキュリティの知識のある方限定でお願いします。

    firewallで公開ポートを開く、必要に応じてその公開ポートでのアクセス制限を行う
    　↓
    編集対象のディレクトリにcdする
    　↓
    singularity exec ~/sif/jekyll420CentOS.sif jekyll serve --port=公開ポート番号 --host=0.0.0.0
    または
    singularity exec ~/sif/jekyll420CentOS.sif jekyll serve --port=公開ポート番号 --host=0.0.0.0 --incremental
    　↓
    ほかのマシンでブラウザを開いて、http://上記命令を実行したマシンのIPアドレス:公開ポート番号 にアクセスして確認、_siteディレクトリには、htmlが作成される、停止はctrl+c

*Singularityコンテナ実行詳細については、https://sylabs.io/docs/ の各バージョン横にある User Guideを参照して下さい。*
