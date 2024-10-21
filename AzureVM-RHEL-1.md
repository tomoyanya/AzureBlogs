こんばんは。

ここ数年、AWSに所属していたり、Microsoftの中の人になっていたりと変化が多いこの頃です。
いろいろAzure のTips が溜まってきていることや、**いろんな方に Microsoft Azure の理解を深めて頂きたい**ので、短い記事を連発していく予定です。

中の人になって、Microsoft はLinux に積極的に投資や活動をおこなっているな、と改めて感じたこの頃。
サティア・ナデラCEOになったときに　**Microsoft ♥ Linux**　と発信しはじめてから、以下のようなBlog が公開され、今でも閲覧可能になっています。
Blog がまだ公開されていますね。

https://www.microsoft.com/en-us/windows-server/blog/2015/05/06/microsoft-loves-linux/

では、Azure VM 上でRHEL をご利用頂く際に、考えていくことを順番に書いていきます。大きくはこれらがわかれば使っていけると思います。

**１．支払い方法**
**２．一般的にどのVMイメージを使うのか**
**３．Azure VM（オンデマンド） のRHELの更新プログラムはどうやって入手、適用するのか**

### １．支払い方法
支払い方法として、大きく２つあります。

#### オンデマンドの利用（Azure VM（RHEL）の従量課金の料金にライセンス費用を盛り込んで使う方法）

多くのユーザーがライセンスの管理が煩雑になるため、１のオンデマンドの利用をご選択いただいています。システム管理の観点で、ライセンスがXX年XX月に切れる、サブスクリプションがXX年XX月に切れる、保守がXX年XX月に切れる、のような管理を数百台、数千台のOS分管理するのはしんどいものです。
Excelで管理しているけど、最新かわからない、なんてよくある話。。

#### RHELのライセンスを持ち込んでAzure VM でお使い頂く方法 ####

Azure上のVM利用時にもともと持っているRHELのライセンスを持ち込んで、Azure VMとして利用する方法があります。
Azure の月額の利用料は減るものの、サポート窓口がOS部分がRHEL社になるなど、運用観点でワンストップサポートにならないことが多いのが難点です。

### ２．一般的にどのVMイメージを使うのか
Azure Virtual Machine のイメージ選択時に ↓　を選びます。![20240828000250.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3872919/1850c472-cd9a-a607-eab6-6a45b624cd23.png)
![20240828000146.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3872919/921928ff-db5e-af08-ceff-f157a3d2e3a4.png)
実は、RHEL のイメージは複数種類あって、選ぶときにややこしい！と思うことが多々あります。今回もパット見だけでわかりずらいものがありました。例えば「Red Hat Inc」で検索をかけた結果がこちら。わかりずらい。

![20240828000748.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3872919/3b6abd1b-b21c-c350-3a9e-e4c1c9ea86d0.png)

きちんとバージョンごとにOSイメージが用意されているのが本物なので、きちんと目的のイメージを選択するようにしてほしい。

![20240828001313.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3872919/77ad4c81-6b1f-3027-e86d-716fb14f0110.png)


### ３．Azure VM（オンデマンド） のRHELの更新プログラムはどうやって入手、適用するのか

オンプレミスやハウジング環境等でこれまでRHELを使っていた方は、オンライン上にあるRed Hatパッケージリポジトリを使用するために、Subscription Managerを使って登録したあと、Red Hat のレポジトリにアクセス可能か確認し、yumで更新するのが一般的でした。
もしくは、隔離された環境であれば、RHELのサイトへログインし必要なパッチを確認し手動アップデートを行っている人もいるかもしれません。

Azure VM としてRHELを稼働させる場合、RHELの更新プログラム適用については、すごく簡単になっています。

まず、Subscription Manager でRHELのサブスクリプションを登録などといった作業は不要です。

なら、どこからどうやって更新プログラムを入手するんや！ということになるわけですが、最新の更新プログラムを取得するには、sudo yum update の実行で更新可能です。

yum は大まかにいうとパッケージのインストールやアップデート、削除などを簡単に実行してくれるものですが、sudo yum update の実行で参照される先（レポジトリ）は、Azure RHUI になります。

（参考資料）

https://learn.microsoft.com/ja-jp/azure/virtual-machines/workloads/redhat/redhat-rhui
Azure RHUI については、上記の公開ドキュメントが参考となりますが、ポイントとしてはこの２つになります。

- Red Hat 社でホストされているリポジトリのコンテンツをミラーリングする
- Azure 固有のコンテンツを使用してカスタム リポジトリを作成する

つまり、Red Hat 社のレポジトリをミラーした場所のレポジトリが使用可能ということです。

一般的な、オンデマンドでの利用（Azure VM（RHEL）のご利用料金にライセンス費用が内包されている）の場合は、Azure RHUI にアクセスするための構成が事前に設定されているため、sudo yum update の実行で更新可能となっています。

yum 自体は一般的なRHEL系Linuxのコマンドのため、Azure 特有のことは特にありません。

更新プログラムリストを見る場合は、Updateinfo から確認できますし、セキュリティ重要度を知りたい場合は、 list security all オプションを書けばよいです。

> sudo yum updateinfo list security all

kernel に関するアップデートの情報を除外したい場合は、--exclude=kernel*  リストを追加すればよいです。
「--exclude」 指定することで除外されます。

> sudo yum updateinfo list security all --exclude=kernel*

（実行サンプル）

[root@RHEL8 ]# sudo yum updateinfo list security all --exclude=kernel*
Last metadata expiration check: 2:47:56 ago on Wed 21 Aug 2024 05:22:03 AM UTC.
RHSA-2024:2621 Important/Sec. bpftool-4.18.0-477.55.1.el8_8.x86_64
RHSA-2024:3810 Moderate/Sec.  bpftool-4.18.0-477.58.1.el8_8.x86_64
RHSA-2024:4740 Moderate/Sec.  bpftool-4.18.0-477.64.1.el8_8.x86_64
RHSA-2024:5255 Important/Sec. bpftool-4.18.0-477.67.1.el8_8.x86_64


実際にパッケージを適用の際は、yum updateinfo を yum update に変更し、オプションはそのままで実施することで、除外したものを適用することが可能です。

Microsoft は Windows な会社なので、Microsoft Azure でRHEL使ってくれる人が増えるといいなと思っています。
