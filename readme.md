# NarrativeBoard

講演会・授業・シンポジウムなどで使用する、リアルタイム質問・コメント表示システムです。

参加者がスマートフォンから投稿したコメントを、司会・運営側が確認し、会場スクリーンへ表示できます。

開発：
柊 和佑  
中部大学 人文学部 メディア情報社会学科

---

# システム構成

NarrativeBoard は4つのHTMLで構成されています。

```text
/index.html
  イベント生成

/post.html
  投稿画面

/admin.html
  管理画面

/display.html
  会場表示画面
```

---

# 必要なもの

・Supabase アカウント  
・GitHub Pages または Web サーバ  
・ブラウザ（Chrome 推奨）

---

# Supabase 初期設定

## 1. Supabase プロジェクト作成

Supabase で新しいプロジェクトを作成します。

推奨設定：

```text
Enable Data API：ON
Automatically expose new tables：OFF
Enable automatic RLS：ON
```

---

## 2. APIキー確認

以下を控えます。

```text
Project URL
Publishable Key
```

HTML内の：

```javascript
const SUPABASE_URL = "...";
const SUPABASE_KEY = "...";
```

へ設定してください。

---

# テーブル作成

Supabase の SQL Editor に以下を入力して実行します。

```sql
create table public.events (
  id uuid primary key default gen_random_uuid(),
  event_name text not null,
  admin_token text,
  created_at timestamptz default now(),
  is_active boolean default true
);

create table public.comments (
  id uuid primary key default gen_random_uuid(),
  event_id uuid references public.events(id) on delete cascade,
  user_name text,
  comment text not null,
  status text default 'pending',
  display_order integer default 0,
  is_pinned boolean default false,
  created_at timestamptz default now()
);

create table public.logs (
  id uuid primary key default gen_random_uuid(),
  event_id uuid references public.events(id) on delete cascade,
  comment_id uuid references public.comments(id) on delete cascade,
  action_type text not null,
  operator_name text,
  memo text,
  created_at timestamptz default now()
);
```

---

# RLS・権限設定

SQL Editor に、RLS設定SQLを入力して実行してください。

本リポジトリに含まれる：

```text
supabase_rls.sql
```

を使用してください。

---

# GitHub Pages 配置

以下をGitHubへアップロードします。

```text
index.html
post.html
admin.html
display.html
```

GitHub Pages を ON にします。

例：

```text
https://username.github.io/NarrativeBoard/
```

---

# 使用方法

## 1. イベント作成

`index.html` を開きます。

イベント名を入力し、

```text
イベント生成
```

を押します。

以下のURLが生成されます。

・投稿HTML  
・管理HTML  
・表示HTML  

---

## 2. 投稿画面

参加者へ：

```text
post.html?event=...
```

を共有します。

QRコード化すると便利です。

参加者は：

・名前（任意）  
・コメント  

を入力して投稿できます。

投稿直後には会場表示されません。

---

## 3. 管理画面

司会・運営担当者のみ：

```text
admin.html?event=...&token=...
```

を使用します。

このURLには管理用トークンが含まれています。

第三者へ共有しないでください。

管理画面では：

・公開  
・非公開  
・却下  
・固定表示  
・本文編集  

が可能です。

---

## 4. 表示画面

会場スクリーンには：

```text
display.html?event=...
```

を表示します。

公開設定されたコメントのみが表示されます。

OBS のブラウザソースとしても使用できます。

---

# コメント状態

| 状態 | 内容 |
|---|---|
| pending | 未確認 |
| public | 公開中 |
| hidden | 非公開 |
| rejected | 却下 |

---

# セキュリティ

NarrativeBoard は Supabase RLS を使用しています。

・投稿画面  
→ コメント投稿のみ可能

・表示画面  
→ 公開コメントのみ取得可能

・管理画面  
→ 管理トークン付きURLのみ更新可能

注意：

```text
service_role key
```

は絶対にHTMLへ記載しないでください。

使用するのは：

```text
Publishable Key
```

のみです。

---

# 推奨運用

## 講演会前

・イベント作成  
・表示PC接続  
・QRコード生成  
・管理URLを司会者へ共有

---

## 講演会中

・司会がコメント確認  
・適切なコメントのみ公開  
・固定表示を必要に応じて使用

---

## 講演会後

・不要イベントを削除  
・必要に応じてログ保存

---

# 注意事項

・管理URLは参加者へ共有しない  
・公開GitHubへ service_role key を置かない  
・長文投稿が多い場合は表示件数を調整する  
・学外公開イベントでは投稿監視を推奨

---

# 今後の拡張候補

・CSV出力  
・投稿停止ボタン  
・イベント終了モード  
・リアクション機能  
・QRコード自動生成  
・画像投稿  
・OBS向けデザインテーマ切替  
・認証ログイン対応

---

# ライセンス

研究・教育用途向け。

必要に応じて各組織で改変して使用してください。