# Git Bash × GitHub SSH接続 完全ガイド

新人エンジニア向けに、GitHubのSSH接続設定から日常的なGit操作まで、実務で困らないよう丁寧に解説します。

---

## 📋 目次

1. [初期設定（最初の1回だけ）](#初期設定最初の1回だけ)
2. [リポジトリのクローン](#リポジトリのクローン)
3. [日常的な作業フロー](#日常的な作業フロー)
4. [プルリクエスト（PR）の作成](#プルリクエストprの作成)
5. [よくあるトラブルと解決法](#よくあるトラブルと解決法)

---

## 初期設定（最初の1回だけ）

### ステップ1: Gitの初期設定

Git Bashを開いて、あなたの名前とメールアドレスを設定します。

```bash
# 名前を設定（GitHubのユーザー名でOK）
git config --global user.name "あなたの名前"

# メールアドレスを設定（GitHubに登録したメール）
git config --global user.email "your-email@example.com"

# 設定確認
git config --global --list
```

**ポイント**: このメールアドレスはGitHubのアカウントと同じものを使いましょう。

---

### ステップ2: SSHキーの生成

SSHキーは、パスワードなしで安全にGitHubと通信するための鍵です。

```bash
# SSHキーを生成（メールアドレスは自分のものに変更）
ssh-keygen -t ed25519 -C "your-email@example.com"

# 保存場所を聞かれたら、そのままEnterキーを押す
# パスフレーズを聞かれたら、設定してもしなくてもOK（空でEnter可）
```

**生成される場所**: `C:\Users\あなたのユーザー名\.ssh\id_ed25519`

---

### ステップ3: SSH Agentの起動とキーの追加

```bash
# SSH Agentを起動
eval "$(ssh-agent -s)"

# 生成したSSHキーを追加
ssh-add ~/.ssh/id_ed25519
```

**「Agent pid 1234」のような表示が出ればOK！**

---

### ステップ4: GitHubにSSH公開鍵を登録

#### 4-1. 公開鍵をコピー

```bash
# 公開鍵の内容を表示（これをコピーする）
cat ~/.ssh/id_ed25519.pub
```

**`ssh-ed25519 AAAA...` から始まる文字列全体をコピー**してください。

#### 4-2. GitHubに登録

1. GitHub（https://github.com）にログイン
2. 右上のアイコン → **Settings**
3. 左メニュー → **SSH and GPG keys**
4. **New SSH key** ボタンをクリック
5. **Title**: 「My PC」など、わかりやすい名前
6. **Key**: さっきコピーした公開鍵を貼り付け
7. **Add SSH key** をクリック

---

### ステップ5: 接続テスト

```bash
# GitHubに接続できるか確認
ssh -T git@github.com
```

**成功メッセージ例**:
```
Hi あなたのユーザー名! You've successfully authenticated, but GitHub does not provide shell access.
```

このメッセージが出れば設定完了です！🎉

---

## リポジトリのクローン

既存のGitHubリポジトリを自分のPCにコピーします。

### ステップ1: SSH URLを取得（詳細図解）

#### 方法1: リポジトリページから取得（推奨）

1. **GitHubのリポジトリページを開く**
   - 例: https://github.com/give159/Gitbush
   - ブラウザでリポジトリのトップページにアクセス

2. **緑色の「Code」ボタンを探す**
   - ページ右上付近にあります
   - 「Add file」ボタンの隣です
   
3. **「Code」ボタンをクリック**
   - ドロップダウンメニューが開きます

4. **「SSH」タブをクリック**
   - デフォルトで「HTTPS」が選択されていることがあります
   - 必ず **「SSH」** を選択してください
   
5. **URLをコピー**
   - 📋 アイコン（コピーボタン）をクリック
   - または、手動で選択してコピー
   - **形式**: `git@github.com:ユーザー名/リポジトリ名.git`
   - **例**: `git@github.com:give159/Gitbush.git`

```
┌─────────────────────────────────────┐
│  < > Code  ▼                        │  ← ここをクリック
├─────────────────────────────────────┤
│  HTTPS   SSH   GitHub CLI           │  ← SSH を選択
├─────────────────────────────────────┤
│  git@github.com:give159/Gitbush.git │  ← これをコピー
│  📋                                  │  ← コピーボタン
└─────────────────────────────────────┘
```

#### 方法2: URLバーから手動作成

現在のHTTPS URLからSSH URLを作成できます。

**変換例**:
```
HTTPS形式:
https://github.com/give159/Gitbush

↓ 変換 ↓

SSH形式:
git@github.com:give159/Gitbush.git
```

**変換ルール**:
- `https://github.com/` → `git@github.com:`
- 最後に `.git` を追加

---

### 📂 様々なリポジトリタイプでのSSH URL取得方法

GitHubには色々な種類のリポジトリがあります。それぞれの場合でのSSH URLの探し方を説明します。

#### パターン1: 通常のパブリックリポジトリ

**特徴**: 誰でも見られる公開リポジトリ

**SSH URLの場所**:
1. リポジトリのトップページ
2. 緑色の「Code」ボタン → SSH
3. `git@github.com:ユーザー名/リポジトリ名.git`

**例**:
```bash
git@github.com:torvalds/linux.git
git@github.com:facebook/react.git
```

---

#### パターン2: プライベートリポジトリ

**特徴**: 自分または招待された人だけが見られるリポジトリ（🔒マークが付いている）

**SSH URLの場所**:
- 通常のリポジトリと**全く同じ手順**
- 「Code」ボタン → SSH
- **重要**: SSH鍵を設定していないとアクセスできません

**例**:
```bash
git@github.com:your-company/private-project.git
```

**注意点**:
```bash
# プライベートリポジトリをクローン
git clone git@github.com:your-company/private-project.git

# もしPermission deniedエラーが出たら:
# → SSH鍵が正しく設定されているか確認
# → リポジトリへのアクセス権限があるか確認
```

---

#### パターン3: フォーク（Fork）したリポジトリ

**特徴**: 他の人のリポジトリを自分のアカウントにコピーしたもの

**SSH URLの場所**:
1. **自分のフォーク**のページに行く（元のリポジトリではない！）
2. URL例: `https://github.com/あなたのユーザー名/リポジトリ名`
3. 「Code」ボタン → SSH

**よくある間違い**:
```bash
# ❌ 元のリポジトリのURL（これだとPushできない）
git@github.com:original-owner/repo.git

# ✅ 自分のフォークのURL（こっちを使う）
git@github.com:your-username/repo.git
```

**確認方法**:
- リポジトリ名の下に「forked from 元のリポジトリ名」と表示されている
- URLのユーザー名が**自分のユーザー名**になっているか確認

---

#### パターン4: Organization（組織）のリポジトリ

**特徴**: 会社やチームの組織アカウント配下のリポジトリ

**SSH URLの場所**:
- 通常と同じ手順
- 「Code」ボタン → SSH
- `git@github.com:組織名/リポジトリ名.git`

**例**:
```bash
# 組織のリポジトリ
git@github.com:microsoft/vscode.git
git@github.com:google/go.git
git@github.com:your-company/team-project.git
```

**注意点**:
- 組織のメンバーでない場合、プライベートリポジトリは見られません
- 組織の管理者から招待を受ける必要があります

---

#### パターン5: Gist（コードスニペット共有）

**特徴**: 小さなコードやメモを共有するサービス

**SSH URLの場所**:
1. Gistのページ（`https://gist.github.com/ユーザー名/ID`）
2. 右上の「Embed」ボタンの隣にある「Clone via HTTPS」をクリック
3. 表示されるURLから手動でSSH形式に変換

**変換例**:
```bash
# GistのHTTPS URL
https://gist.github.com/1234567890abcdef.git

# SSH URLに変換
git@gist.github.com:1234567890abcdef.git
```

**クローン例**:
```bash
git clone git@gist.github.com:1234567890abcdef.git
```

---

#### パターン6: GitHub Enterprise（企業版GitHub）

**特徴**: 企業が独自に運用しているGitHubサーバー

**SSH URLの場所**:
- 通常のGitHubと同じ手順
- ただし、ドメインが違う

**形式**:
```bash
# 通常のGitHub
git@github.com:ユーザー名/リポジトリ名.git

# GitHub Enterprise
git@github.your-company.com:ユーザー名/リポジトリ名.git
git@git.your-company.co.jp:ユーザー名/リポジトリ名.git
```

**注意点**:
- 会社のSSH鍵設定ポリシーに従う
- VPN接続が必要な場合もあります

---

### 🔍 SSH URL 一覧表（まとめ）

| リポジトリタイプ | SSH URL形式 | 探す場所 |
|:---|:---|:---|
| 通常のリポジトリ | `git@github.com:user/repo.git` | Codeボタン → SSH |
| プライベート | `git@github.com:user/repo.git` | Codeボタン → SSH |
| フォーク | `git@github.com:あなた/repo.git` | **自分の**フォーク → SSH |
| Organization | `git@github.com:org/repo.git` | Codeボタン → SSH |
| Gist | `git@gist.github.com:ID.git` | 手動変換 |
| Enterprise | `git@github.company.com:user/repo.git` | Codeボタン → SSH |

---

### 💡 どのリポジトリタイプか見分けるコツ

```
👤 個人リポジトリ
https://github.com/ユーザー名/リポジトリ名
└─ ユーザー名が個人名

🏢 Organizationリポジトリ  
https://github.com/組織名/リポジトリ名
└─ 組織名（microsoft, google など）

🔱 フォーク
リポジトリ名の下に「forked from 元/リポジトリ」表示
└─ 自分のアカウント配下にある

🔒 プライベート
リポジトリ名の横に鍵マーク🔒
└─ アクセス権限が必要

📝 Gist
https://gist.github.com/ で始まる
└─ コードスニペット用
```

### ステップ2: クローン実行

```bash
# 作業したいディレクトリに移動（例: デスクトップ）
cd ~/Desktop

# リポジトリをクローン（コピーしたSSH URLを貼り付け）
git clone git@github.com:give159/Gitbush.git

# クローンしたディレクトリに移動
cd Gitbush
```

**実行例**:
```bash
$ git clone git@github.com:give159/Gitbush.git
Cloning into 'Gitbush'...
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 10 (delta 0), reused 10 (delta 0), pack-reused 0
Receiving objects: 100% (10/10), done.
```

**これで準備完了！** ファイルの編集ができます。

---

## 🔍 SSH URL確認のチェックリスト

自分が正しいSSH URLをコピーできているか確認しましょう！

### ✅ 正しいSSH URLの特徴

```bash
# ✅ 正しい形式
git@github.com:give159/Gitbush.git
git@github.com:ユーザー名/リポジトリ名.git

# ポイント:
# ① 「git@github.com:」で始まる
# ② 「https://」ではない
# ③ 「.git」で終わる
# ④ ユーザー名とリポジトリ名が「/」で区切られている
```

### ❌ よくある間違い

```bash
# ❌ HTTPSになっている（SSHではない）
https://github.com/give159/Gitbush.git

# ❌ .gitがない
git@github.com:give159/Gitbush

# ❌ 「:」が「/」になっている
git@github.com/give159/Gitbush.git
```

### 🔧 間違えてHTTPSでクローンした場合

すでにHTTPSでクローンしてしまった場合は、リモートURLをSSHに変更できます。

```bash
# 現在のリモートURLを確認
git remote -v

# SSH URLに変更
git remote set-url origin git@github.com:give159/Gitbush.git

# 変更確認
git remote -v
# origin  git@github.com:give159/Gitbush.git (fetch)
# origin  git@github.com:give159/Gitbush.git (push)
```

**これで準備完了！** ファイルの編集ができます。

---

## 日常的な作業フロー

### 基本的な流れ（初めてのプッシュ）

```bash
# 1. 作業ブランチを作成して切り替え
git checkout -b feature/my-first-feature

# 2. ファイルを編集（VSCodeなどで）
# 例: README.mdを編集

# 3. 変更内容を確認
git status

# 4. 変更をステージングエリアに追加
git add .
# または特定のファイルだけ追加
git add README.md

# 5. コミット（変更内容を記録）
git commit -m "READMEに使い方を追加"

# 6. GitHubにプッシュ（初回）
git push -u origin feature/my-first-feature
```

---

### 次回以降のプッシュ（2回目以降）

```bash
# 1. ファイルを編集

# 2. 変更を確認
git status

# 3. 変更を追加
git add .

# 4. コミット
git commit -m "機能Xを実装"

# 5. プッシュ（次回以降は短く書ける）
git push
```

**ポイント**: 初回は `-u origin ブランチ名` が必要ですが、2回目以降は `git push` だけでOKです！

---

## プルリクエスト（PR）の作成

コードレビューしてもらうために、プルリクエストを作成します。

### ステップ1: GitHubでPRを作成

1. GitHubのリポジトリページを開く
2. **Compare & pull request** ボタンが表示されるのでクリック
   - または **Pull requests** タブ → **New pull request**
3. ベースブランチを選択（通常は `main` または `develop`）
4. タイトルと説明を記入
   - **タイトル例**: 「ユーザー登録機能を実装」
   - **説明例**: 「何を変更したか」「なぜ変更したか」を書く
5. **Create pull request** をクリック

---

## よくあるコマンド一覧

```bash
# 現在のブランチを確認
git branch

# 別のブランチに切り替え
git checkout main

# 最新の変更を取得
git pull

# コミット履歴を確認
git log --oneline

# 変更を一時的に退避
git stash

# 退避した変更を戻す
git stash pop

# リモートリポジトリの情報を確認
git remote -v
```

---

## よくあるトラブルと解決法

### ❌ エラー: Permission denied (publickey)

**原因**: SSH鍵が正しく設定されていない

**解決法**:
```bash
# SSH Agentに鍵が追加されているか確認
ssh-add -l

# 追加されていなければ再度追加
ssh-add ~/.ssh/id_ed25519

# 接続テスト
ssh -T git@github.com
```

---

### ❌ エラー: fatal: not a git repository

**原因**: Gitリポジトリの中にいない

**解決法**:
```bash
# リポジトリのディレクトリに移動
cd ~/Desktop/Gitbush
```

---

### ❌ コンフリクト（競合）が発生した

**原因**: 同じファイルの同じ箇所を複数人が編集した

**解決法**:
```bash
# 最新の変更を取得
git pull

# コンフリクトしているファイルを開いて手動で修正
# <<<<<<< HEAD と >>>>>>> の部分を確認

# 修正後、再度コミット
git add .
git commit -m "コンフリクトを解決"
git push
```

---

## 📝 実務で役立つTips

### ✅ コミットメッセージの書き方

```bash
# 良い例
git commit -m "ユーザー登録機能を実装"
git commit -m "バグ修正: ログイン時のエラー処理"
git commit -m "リファクタリング: 重複コードを削減"

# 悪い例
git commit -m "修正"
git commit -m "aaa"
```

**ポイント**: 「何をしたか」が一目でわかるように書く！

---

### ✅ ブランチ名の命名規則

```bash
# 機能追加
git checkout -b feature/user-registration

# バグ修正
git checkout -b fix/login-error

# リファクタリング
git checkout -b refactor/clean-code
```

---

### ✅ 作業前に必ずやること

```bash
# 最新のmainブランチに切り替え
git checkout main

# 最新の変更を取得
git pull

# 新しいブランチを作成
git checkout -b feature/new-feature
```

これで他の人の変更とコンフリクトしにくくなります！

---

## 🎓 まとめ

1. **初期設定は最初の1回だけ**（SSH鍵の生成と登録）
2. **基本フロー**: clone → ブランチ作成 → 編集 → add → commit → push
3. **プッシュ**: 初回は `git push -u origin ブランチ名`、次回以降は `git push`
4. **PR作成**: GitHubのWebページで作成
5. **困ったら**: `git status` でまず状態を確認！

**この手順を繰り返せば、実務でのGit操作は完璧です！** 💪

参考リポジトリ: https://github.com/give159/Gitbush