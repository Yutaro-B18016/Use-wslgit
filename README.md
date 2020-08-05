# Git for Windowsの代わりにwslgitを利用する.

Git for Windowsの代わりにwslgitを用いて,WSL側のgitをWindows側から利用する方法を示す.

もし,WSL2で[ubuntu-wsl2-systemd-script](https://github.com/DamionGans/ubuntu-wsl2-systemd-script)等を利用し,systemdを利用できるようなハックをしている場合,wsl+linuxコマンドで実行されるwslコマンドは常にルートディレクトリで実行されるようになってしまうため,この方法は実質利用できない.
おとなしく,WSL側だけでwslのgitを利用しよう.

## WSLにgitをインストールする.

インストール方法は選択したディストーションによって異なるが,以降,Ubuntuを利用しているとする.

WSLの端末を開いて,

```bash:bash
sudo apt update

sudo apt -y upgrade

sudo apt -y autoremove

sudo apt -y autoclean
```

アップデートを行い.

```bash:bash
sudo apt -y install git
```
を実行する.

インストール出来たのか確認するために下記のコマンドを実行する.

```bash:bash
git version
```

そうすると,

```
git version 2.25.1
```

のように表示される.

## wslgitをインストールする.

バージョン更新が楽なのでChocolateyを利用する.

Chocolateyのインストールに関しては,以下に記載済みなので,そちらを参照のこと.

[Java_Install_for_Games](https://github.com/softwarereliabilitylab/Java_Install_for_Games)

コマンドプロンプト又はPowershellを**管理者権限**で起動し,以下のコマンドを実行する.

```Powershell:Powershell or cmd (管理者)
choco install -y wslgit
```

インストールが完了すれば,**新しいコマンドプロンプトかPowershell**のターミナルで

```Powershell:Powershell or cmd
wslgit version
```
と入力してWSLのgitの確認と同じ出力であれば問題ない.

次にwslgitをgit.exeとして利用できるようにする.

リネームでもいいが更新されても利用できるようにするためにシンボリックリンクを利用する.

パスが設定されているフォルダならば,どこでもいいが,今回は%USERPROFILE%にbinというフォルダを作ってそこにパスを通し,そこにgit.exeというシンボリックを貼ることにする.

コマンドプロンプトから%USERPROFILE%にbinというフォルダを作成する.

```Powershell:Powershell or cmd
mkdir %USERPROFILE%\bin
```

パス通すところもコマンドプロンプトでやりたかったが,1024文字以上設定するとそれ以上は消えて大変な目にあってしまうので,GUIのシステムのプロパティから設定する.


**環境変数の編集には十分注意すること**

エクスプローラー->左サイドにあるPCを選択->右クリック->プロパティ->左サイドにあるシステムの詳細設定->環境変数->システムの環境変数にあるPathを選択->編集->新規->%USERPROFILE%\binを入力(%USERPROFILE%はC:Users\ユーザー名を表す.わかるのならそれを設定した方が良い)->ok->ok->ok

その後,%USERPROFILE%\bin\にgit.exeというシンボリックリンクを貼る.

コマンドプロンプトを**管理者権限**で開き,以下のコマンドを実行する.

```cmd:cmd (管理者)
mklink "%USERPROFILE%\bin\git.exe" "C:\ProgramData\chocolatey\lib\WSLGit\wslgit.exe"
```

**新しいコマンドプロンプトかPowershell**を開き,WSLと同じように,

```Powershell:cmd or Powershell
git version
```

と入力して出力が同じであれば問題ない.

## gitの設定を行う.

gitの設定はwsl側で行う.

- githubにSSHで接続出来るようにする.

pushする時,又はプライベートリポジトリのcloneを行う時にパスワードを要求されるので,githubにsshで接続出来るようにする.

sshの設定のためのディレクトリをなければ作る.

```bash:bash
mkdir ~/.ssh
```

次に,鍵を作成する.暗号強度が下記以上であり,githubが対応していれば何でもいい.(鍵の作成のみcmdやPowershellでもいいがパスに気をつけること)

```
ssh-keygen -t rsa -b 4096
```

と入力すると,

```
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa):
```

と鍵の保存場所と名前を聞かれるので,適宜入力(入力がなければ~/.ssh/id_rsa)に作成される.

ここではデフォルトのままとする.

```
Enter passphrase (empty for no passphrase):
```

パスフレーズの設定である.

鍵を使用する時に結局パスフレーズを聞かれるようになるため,何も入力せずにEnterで良い.

こうして,出来た鍵をgithubに登録する.

```bash:bash
cat ~/.ssh/id_rsa.pub|clip.exe
```

指定した名前に.pub拡張子を加えたものが公開鍵である.
上記のようにcatで表示させ,それをクリップボードにコピーしている.

これをgithubに貼り付ける.

[SSH and GPG Keys](https://github.com/settings/keys)

上記のNew SSH keyをクリックし,keyと表示されるところに貼り付ける.
そして,Add SSH keyを追加をクリックするとgithubに登録は完了である.

githubへのssh接続に使う鍵を設定する.

```
Host github github.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa
  User git
```

といった内容のファイルを,.ssh直下に**config**という名前で作成する.
~/.ssh/id_rsaは各自の秘密鍵のパスを設定する.

このままでは,権限で怒られてしまうので,

```bash:bash
chmod -R 600 ~/.ssh/

chmod 700 .ssh
```
.sshの中身の権限を600に,.ssh自体を700に設定する.

ここまで,済めば設定が問題ないかを実際にssh接続をして確認する.

```bash:bash
ssh -T git@github.com
```

と入力すると初回接続であれば,

```bash:bash
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

known_hostsに追加するか聞かれるので,yesと入力する.

接続に成功すると以下のように表示される.Githubのユーザー名は各自のものである.

```bash:bash
Hi Githubのユーザー名! You've successfully authenticated, but GitHub does not provide shell access.
```
上記のようなメッセージが表示されない場合,権限かconfigファイルの鍵のパスの設定が間違っている可能性が高い.
鍵は漏洩すると大変である**鍵の扱いには注意すること**.

- gitにユーザー名とemailアドレスを設定する.

```bash:bash
git config --global user.name "ユーザー名"

git config --global user.email emailアドレス
```

- httpsでcloneしsshでpush出来るようにする.

sshよりもhttpsでcloneする方が早いし,推奨であるため,httpsでcloneしてもpushはsshで行うようにする.

```bash:bash
git config --global 'url.git@github.com:.pushinsteadof' 'https://github.com/'
```

プライベートリポジトリに関してはhttpsでcloneするとユーザー名とパスワードが聞かれてしまうため,sshでcloneするようにする.

以上で設定は全て終わりである.

WSL側からだけでなくWindows側からも利用可能なのかを確かめるためには,プライベートリポジトリをsshでコマンドプロンプト又はPowershellでcloneする.
あるいはcloneしたpush権限のあるリポジトリにpushすることで確認できるであろう.

## gitをアップデートする.

VSCodeは**優秀なテキストエディタ**であり,gitも統合されている.
上記設定を行った後,VSCodeを起動すると下記のような警告が発生するかもしれない.


```VSCode:VSCode
インストールされている Git 2.25.1 には既知の問題があります。Git 機能を正常に動作させるために、Git を 2.27 以上に更新してください。
```

発生しない場合問題は無いので以降は行う必要がない.

これは,WSL上のUbuntuにあるgitのバージョンが古く脆弱性が存在するためである.(Ubuntu版はパッチ適応されてるってどっかで聞いたような気もするけど,Git for Windows前提だから警告が出るのかもしれない)

[Gitに認証データ窃取の脆弱性、アップデートを](https://news.mynavi.jp/article/20200418-1018783/)

[CVE-2020-5260](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-5260)

[CVE-2020-5260 in Ubuntu](https://people.canonical.com/~ubuntu-security/cve/2020/CVE-2020-5260.html)

そのため,gitを更新する.

新しいgitを提供しているppaを追加する.

```bash:bash
sudo add-apt-repository ppa:git-core/ppa
```

追加していいかの確認が出るので

```bash:bash
Press [ENTER] to continue or Ctrl-c to cancel adding it.
```

Enterを押す.

そして,アップデートコマンドを実行する.

```bash:bash
sudo apt update

sudo apt -y upgrade

sudo apt -y autoremove

sudo apt -y autoclean
```

そしてgitのバージョンを確認すると,

```bash:bash
git version
```

```bash:bash
git version 2.28.0
```

のようにバージョンが上がり,VSCodeを起動し直しても警告が出なくなるはずである.
