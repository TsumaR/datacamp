## DataCampのGit講座のノート  

### 基本操作
Gitでは，編集しているファイルやディレクトリと，その操作履歴をまとめてrepositoryという。このような現存のファイル以外の情報は，repositoryのrootディレクトリに存在する`.git`ディレクトリに保存される。
##### gitの状態を知る
repositoryの状態を教えてくれる。`git status`で`add`しておらずstageに存在しないファイルや，`add`されstageに存在するファイルなど，レポジトリの状況を知ることができる。
`git diff`で変更したファイルについて教えてくれる。`git diff directory`でディレクトリを指定して作業ツリーとの比較を行える。ファイル名を指定すればそのファイルのバージョン間でどのような変更があったかを知ることができる。`-`が消去，`+`が追加を表す。  
`git diff -r HEAD`では，`-r`で比較するバージョンを指定し，`HEAD`が最新のcommitを意味するため，最新のコミットとstaging areaの状態を比較することができる。`HEAD~2`で２つ前ということを意味する。  

##### gitに変更をsaveする  
`git add filename`でstaging areaにファイルを加える。この操作はファイルに変更があるたびにすることが望ましい。  
`git commit`で，staging areaに存在する変更点を全て保存する。`-m`で変更内容を記載する，間違って記載してしまった際は`--amed`を前に加えることで修正できる。  

##### 一部の変更だけをcommitする  
`git add path/to/file`とすれば，変更内容をcommitしたいファイルだけをstaging areaに登録し，commitすることができる。間違えてstageしてしまった場合は`git reset HEAD`で，直前の`add`を取り消すことができる。  

##### 変更を取り消す  
まだ`git add`しておらず，stageに存在しないファイルに関しては`git checkout -- filename`で，最後の`gitadd`以降の変更を取り消すことができる。


##### gitの変更履歴を確認する  
`git log`でcommitの履歴を確認できる。スペースキーでページを進めることができ，qで閉じることができる。最新の変更が最上部に表示される。後ろにファイルパスを記載すれば特定のファイルの変更履歴を確認できる。  

##### hash
各commitは識別されるためにhashという40桁のidを持つ。`git log`で確認できる。あるcommitの詳細を確認したい際には`git show 0da2f7`のように，確認したいhashの最初の数文字を投げれば良い。  
２つのcommitの間の違いを確認したい際は`git diff HEAD~1..HEAD~3`のようにする。

##### commmitの重要な情報だけ表示
`git annotate file`

##### .gitignore
temporaryファイルなど，gitで追跡したくないファイルも存在する。それらは`.gitignore`に保存することでgitに追跡されない。`.gitignore`の中に無視したいディレクトリやファイルのリストを渡すことで，それらはgitの保存から逃れる。この際，ワイルドカードを使用することもできる。
追跡していないファイルで消去を行いたい場合`git clean -f filename`でできる。

### Gitの仕組み  
##### データ保存の仕組み  
各commitは作者や日付，メッセージなどのメタデータを含むのと同時にtreeをもつ。各treeはcommitした際にレポジトリが保持しているファイル名とその場所が保存されている。treeが持つ各ファイルにはblobという，commitが起きた際のファイル内容を簡潔に記載したbinaryファイルが存在する。変更がなされていないときは以前のcommit内容に結びつくようになっている。`git add`されるまでは当然のごとくtreeやblobが存在しない。
[!gitの仕組み](https://assets.datacamp.com/production/repositories/1545/datasets/71c08f5726f7192c0303d4ce84d4ecb9336c6fa5/gds_2_1_diagram.svg)

##### gitの設定を変更する  
`git config --list`で現在のセッティングを確認できる。ローカルセッティングだけを確認するなら`git config --list --local`のようにflexibleに利用できる。
`git config --global setting value`で全体に対する設定を変更できる。通常使用しない方が良いが，メールアドレスを変更したい場合などに利用する。その場合は`git config --global user.email rep.loop@datacamp.com`のように指定する。
