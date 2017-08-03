---
title: Ubuntu Setup
---

# 開発用Ubuntuマシンのセットアップ


## apt-getのアップデート
apt-getはUbuntuで使うことのできる便利なパッケージ管理ツール。
これを使うことで、様々なプログラムやライブラリを簡単にインストールすることができる。
apt-get自体の更新は以下のコマンドで行う。
``sudo apt-get update`` apt-getのパッケージリストを更新する。

``sudo apt-get upgrade`` apt-getでインストールしたパッケージの更新する。インストールしたパッケージの数や種類によってはかなり時間がかかる場合がある。


## テキストエディタ Atom のインストール

最新のDebianパッケージ(.deb)をAtom公式ウェブサイトからダウンロード。
ダウンロードフォルダへ移動し、以下のコマンドでインストールする。
dpkgはDebianパッケージをインストールするためのツールである。

```shellsudo dpkg --install atom-amd64.deb
```

### オプション
 追加で以下のアドオンをAtomに追加することを推奨する。
Edit > Preferences > Install から検索、インストールすることができる。
 - sublime-style-column-selection: shiftを押しながら左クリックで複数行を同時選択できるようになる。複数行を一気に書き換えたいときに便利


### 便利コマンド
`ctrl+shift+m` マークダウンファイル(.md)のリアルタイムプレビューができる

## Gitのセットアップ

1. インストール

    apt-getでインストールできる。
    `` sudo apt-get install git ``

2. ユーザー名とメールアドレスの設定

    Gitを使い始めるためには、まずこれらを設定する必要がある。
    各コミットにこれらの情報が登録されるので、プライバシーについては要注意。
    Githubアカウントを持っていてメールアドレスを公開したくない場合、<アカウント名>@users.noreply.github.comを設定すると良い。

    ```bash
    $ git config --global user.name "Taro Suzuki"
    $ git config --global user.email taro@users.noreply.github.com
    ```

    正しく登録されているかは``git config --list``で確認できる。

3. (オプション)Githubとの連携

    Gitをインストールしたら、Githubのアカウントと連携しておくと便利である。
    連携するにはパスワードのようなものを用いてコンピュータとGithubアカウントを互いに承認する必要があるが、このパスワードに相当するものがsshである。
    sshは暗号化のプロトコルであり、コンピュータ上に秘密鍵、Githubアカウントに公開鍵を登録しておくことによって不正なアクセスを防ぐことができる。

    1. 秘密鍵/公開鍵ペアの生成

        ターミナルを開き、ssh秘密鍵/公開鍵を生成する。Emailアドレスは、Githubアカウントを登録したメールアドレスを用いる。

        ```bash
        $ ssh-keygen -t rsa -C "email_address@example.com"
        ```
        これを実行するとホームフォルダ下の隠しディレクトリ /.ssh に秘密鍵と公開鍵が生成される。正常に作成されているかは``$ ls ~/.ssh``で確認できる。

        ```bash
        $ ls ~/.ssh
        id_rsa    id_rsa.pub
        ```

        id_rsaが秘密鍵、id_rsa.pubが公開鍵である。

    2. 公開鍵の登録

        公開鍵が記されたファイルを開き、中身をコピーする。id_rsa.pubは"ssh-rsa"で始まりメールアドレスで終わるが、これらを含めファイル内の内容をすべてコピーして良い。

        ブラウザでGithubにログインし、 Settings > SSH and GPG keys > New SSH key の順にたどっていく。Titleには分かりやすい名前(Lab PCなど)、Keyには先程コピーした公開鍵を入力し、Add SSH Keyをクリックして登録を完了する。これにより、現在のマシンからGithubにアクセスしたときに秘密鍵と公開鍵の照合が行われ、登録済みのユーザーとして認識されるようになる。

    3. 登録完了の確認

        正常にgithubと連携できているかを確認する。

        ```bash
        $ ssh -T git@github.com
        ```

        以下のメッセージが出てきたら、yesと入力して続行する。
        ```
        The authenticity of host 'github.com (<github IP address>)' can't be established.
        RSA key fingerprint is SHA256:<github SHA256 hash>.
        Are you sure you want to continue connecting (yes/no)?
        ```

        接続に成功していれば以下のメッセージが表示される。
        ```
        Warning: Permanently added 'github.com,<github IP addree>' (RSA) to the list of known hosts.
        Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
        ```



## FlightCtrl開発のためのセットアップ
