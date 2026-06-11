# BAR RIVER 開発ログ

## プロジェクト概要
- **店舗名**: BAR RIVER
- **オーナー**: りいやさん
- **開発者**: りいやさん（初心者）× Claude
- **リポジトリ**: github.com/Electry-lab/RIVER
- **本番URL**: electry-lab.github.io/RIVER/
- **DB**: Supabase（haqenuqasavvgzyvufuv）

---

## システム構成

### ファイル構成
| ファイル | 用途 |
|---|---|
| index.html | スタッフ用注文・会計アプリ |
| owner.html | オーナー用管理アプリ |
| practice.html | 練習用アプリ（Supabase接続なし） |
| manifest.json | PWA設定（スタッフ用） |
| manifest-owner.json | PWA設定（オーナー用） |
| manifest-practice.json | PWA設定（練習用） |
| icon-192.png / icon-512.png | 本番アイコン |
| icon-practice-192.png / icon-practice-512.png | 練習用アイコン（グレー） |

### Supabaseテーブル
| テーブル | 内容 |
|---|---|
| sessions | 営業セッション（開始〜閉店） |
| groups | お客様グループ（卓） |
| orders | 注文明細 |
| staff | スタッフ情報 |
| menus | メニュー・価格（Supabaseで管理） |

### スプレッドシート（Google Apps Script）
| ファイル | 用途 |
|---|---|
| 経営ダッシュボード.gs | 売上・バック・伝票の月次集計 |
| スタッフ管理.gs | バック計算・給与明細・源泉徴収 |
| 損益管理.gs | 売上−人件費−経費＝営業利益 |

---

## バック計算ルール

### ドリンクバック（100円/杯）
対象：テキーラ クエルボ、クライナー、コカボム、アネホ、グラス、缶
- 指名なし卓：全員対象
- 指名あり卓：他スタッフ（other）のみ対象

### シャンパンバック（10%）
対象：シャブリ、ヴーヴイエロー、ペリエ白、モエアイス、モエピカ、モエネク、ペリエロゼ、ウルトラ、ソウメイ、ベルエポ、ドンペリ白、ドンペリロゼ、アルマンドゴールド、アルマンドロゼ
- 全卓対象

### カスタム追加メニューのバック
- 自分・スタッフから追加：シャンパンバックと同じ10%
- お客様から追加：ノーカウント

### 指名バック（10%）
- 指名あり卓のみ
- 計算式：（税抜合計 − シャンパン合計）× 10% ÷ 指名人数

### バック対象外スタッフ
- りいや（オーナー）はバック計算から除外

---

## スタッフ設定
| 名前 | 権限 |
|---|---|
| なおと | 編集・閉店・会計OK |
| りん | 編集・閉店・会計OK |
| りょうせい〜かなた | 通常スタッフ |
| けんしろう | 編集・会計OK |
| いっこう | 編集・会計OK |
| りいや | オーナー（スタッフログイン不可・バック対象外） |
| テスト | テスト用 |

---

## 主な修正履歴

### 2026/06/09（試運転開始日）
- アイコン作成・差し替え（Gemini生成・グレー練習用も）
- manifest-owner.json・manifest-practice.json追加
- NULLセッション問題の修正
- ズームタップ無効化（user-scalable=no）
- カスタムメニューのバック計算追加
- りいやをスタッフログインから除外・バック対象外に
- グループタップ→注文確認画面に変更（スタッフ・オーナー共通）
- 営業開始モーダル追加（継続・新規分岐）
- スタッフ自動ログイン（localStorage保持）
- メニューをSupabaseテーブル（menus）に移行
- 練習アプリ（practice.html）作成

### 2026/06/10
- セッション複数端末共有対応（localStorageからSupabaseベースに）
- グループカードに入店時刻・会計時刻表示
- groupsテーブルにcreated_atカラム追加
- ヘッダーに🔄リロードボタン追加
- 10秒自動更新に変更
- oCreateGroupにセッションチェック追加

### 2026/06/11〜12（初営業・深夜トラブル対応）
- 余分なセッションが大量生成される問題が発生
- loadData()のたびにcurSessionId=nullになる根本バグ発見・修正
- 0時またぎ営業でセッションが切れる問題→未閉店ベース判定に変更
- 本日タブに閉店済みデータが残る問題修正
- SQLで迷子グループ（かおりちゃん・かのん）をセッションに再紐付け
- 0円グループ大量発生→SQLで削除

---

## 現在の既知の課題
- スプレッドシートとの連携（人件費の自動取得）がまだ未実装
- LINE連携（出勤・日払い申告の自動連動）は将来予定
- 顧客台帳は今後追加予定
- オフライン対応（Wi-Fi切れ時のローカル保存）は検討中

---

## SQL Editor よく使うクエリ

### 未閉店セッション確認
```sql
SELECT id, started_at, closed_at FROM sessions 
WHERE closed_at IS NULL ORDER BY started_at DESC;
```

### 余分なセッションを一括閉店
```sql
UPDATE sessions SET closed_at = now()
WHERE closed_at IS NULL AND id != '使いたいセッションID';
```

### メニュー価格変更
```sql
UPDATE menus SET price = 新価格 WHERE name = '商品名';
```

### グループのセッション修正
```sql
UPDATE groups SET session_id = 'セッションID' WHERE name = 'グループ名';
```

### 0円グループ削除
```sql
DELETE FROM groups 
WHERE id NOT IN (SELECT DISTINCT group_id FROM orders)
AND closed = false;
```

---

## 今後のロードマップ
1. アプリ安定稼働の確認（現在進行中）
2. スプレッドシートへの初回データ反映・確認
3. 顧客台帳の追加
4. LINE連携（出勤・日払い申告）
5. オフライン対応
6. 他店舗への展開（メニューをSupabaseで管理してるので対応しやすい）
