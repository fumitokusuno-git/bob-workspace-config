# キャッシュアウトコントロール - データ項目一覧

## 1. トランザクションエンティティ

### 1.1 予約（Reservation）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 予約ID | reservation_id | VARCHAR(20) | ○ | 予約の一意識別子 | 全KPI |
| 予約番号 | reservation_number | VARCHAR(20) | ○ | 顧客向け予約番号 | - |
| 顧客ID | customer_id | VARCHAR(20) | ○ | 顧客の一意識別子 | - |
| チャネルID | channel_id | VARCHAR(10) | ○ | ビジネスチャネル識別子 | KPI-008, KPI-009 |
| 予約日時 | reservation_datetime | TIMESTAMP | ○ | 予約が確定した日時 | KPI-009 |
| 旅行開始日 | travel_start_date | DATE | ○ | 旅行の開始日 | - |
| 旅行終了日 | travel_end_date | DATE | ○ | 旅行の終了日 | - |
| 予約ステータス | reservation_status | VARCHAR(20) | ○ | 予約の状態（確定/キャンセル等） | - |
| 総販売額 | total_sales_amount | DECIMAL(15,2) | ○ | 顧客への総販売額（税込） | - |
| 総原価額 | total_cost_amount | DECIMAL(15,2) | ○ | サプライヤーへの総支払予定額 | KPI-001 |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 1.2 予約明細（Reservation Detail）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 予約明細ID | reservation_detail_id | VARCHAR(20) | ○ | 予約明細の一意識別子 | 全KPI |
| 予約ID | reservation_id | VARCHAR(20) | ○ | 親予約のID（外部キー） | 全KPI |
| 明細番号 | detail_number | INT | ○ | 予約内の明細連番 | - |
| サプライヤーID | supplier_id | VARCHAR(20) | ○ | サービス提供サプライヤー | KPI-006, KPI-007 |
| サービス種別ID | service_type_id | VARCHAR(10) | ○ | サービス種別（宿泊/交通等） | 全KPI |
| サービス名 | service_name | VARCHAR(200) | ○ | サービスの具体名 | - |
| 利用日 | service_date | DATE | ○ | サービス利用日 | - |
| 数量 | quantity | INT | ○ | 利用数量（人数、泊数等） | - |
| 単価 | unit_price | DECIMAL(15,2) | ○ | 顧客向け単価 | - |
| 販売額 | sales_amount | DECIMAL(15,2) | ○ | 明細の販売額 | - |
| 原価単価 | unit_cost | DECIMAL(15,2) | ○ | サプライヤーへの支払単価 | - |
| 原価額 | cost_amount | DECIMAL(15,2) | ○ | 明細の原価額 | KPI-001 |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 1.3 仕入明細（Procurement Detail）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 仕入明細ID | procurement_detail_id | VARCHAR(20) | ○ | 仕入明細の一意識別子 | 全KPI |
| 予約明細ID | reservation_detail_id | VARCHAR(20) | ○ | 関連予約明細ID（外部キー） | 全KPI |
| サプライヤーID | supplier_id | VARCHAR(20) | ○ | 仕入先サプライヤー | KPI-006, KPI-007 |
| 仕入日 | procurement_date | DATE | ○ | 仕入確定日 | KPI-009 |
| 仕入金額 | procurement_amount | DECIMAL(15,2) | ○ | 仕入金額（税込） | KPI-001, KPI-002 |
| 消費税額 | tax_amount | DECIMAL(15,2) | ○ | 消費税額 | - |
| 支払予定日 | scheduled_payment_date | DATE | ○ | 当初の支払予定日 | KPI-001, KPI-005 |
| 請求ID | invoice_id | VARCHAR(20) |  | 関連請求ID（請求後に設定） | - |
| 仕入ステータス | procurement_status | VARCHAR(20) | ○ | 仕入の状態（確定/キャンセル等） | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 1.4 請求（Invoice）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 請求ID | invoice_id | VARCHAR(20) | ○ | 請求の一意識別子 | 全KPI |
| 請求番号 | invoice_number | VARCHAR(30) | ○ | サプライヤー発行の請求番号 | - |
| サプライヤーID | supplier_id | VARCHAR(20) | ○ | 請求元サプライヤー | KPI-006, KPI-007 |
| 請求日 | invoice_date | DATE | ○ | 請求書発行日 | - |
| 請求受領日 | invoice_received_date | DATE | ○ | 請求書を受領した日 | KPI-007 |
| 請求金額 | invoice_amount | DECIMAL(15,2) | ○ | 請求総額（税込） | KPI-001, KPI-002 |
| 消費税額 | tax_amount | DECIMAL(15,2) | ○ | 消費税額 | - |
| 支払期限日 | payment_due_date | DATE | ○ | 支払期限日 | KPI-001, KPI-005 |
| 支払予定日 | scheduled_payment_date | DATE | ○ | 実際の支払予定日 | KPI-001 |
| 請求ステータス | invoice_status | VARCHAR(20) | ○ | 請求の状態（未払/支払済等） | - |
| 備考 | remarks | VARCHAR(500) |  | 請求に関する備考 | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 1.5 支払（Payment）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 支払ID | payment_id | VARCHAR(20) | ○ | 支払の一意識別子 | 全KPI |
| 請求ID | invoice_id | VARCHAR(20) | ○ | 関連請求ID（外部キー） | 全KPI |
| サプライヤーID | supplier_id | VARCHAR(20) | ○ | 支払先サプライヤー | KPI-006, KPI-007 |
| 支払予定日 | scheduled_payment_date | DATE | ○ | 支払予定日 | KPI-001, KPI-005 |
| 支払実行日 | payment_execution_date | DATE |  | 実際の支払実行日 | KPI-002, KPI-005, KPI-007 |
| 支払金額 | payment_amount | DECIMAL(15,2) | ○ | 支払金額 | KPI-002, KPI-003 |
| 支払方法ID | payment_method_id | VARCHAR(10) | ○ | 支払方法（振込/手形等） | KPI-002 |
| 支払ステータスID | payment_status_id | VARCHAR(10) | ○ | 支払ステータス | KPI-002, KPI-005 |
| 振込手数料 | transfer_fee | DECIMAL(10,2) |  | 振込手数料 | - |
| 承認者ID | approver_id | VARCHAR(20) |  | 支払承認者 | - |
| 承認日時 | approved_at | TIMESTAMP |  | 承認日時 | - |
| 備考 | remarks | VARCHAR(500) |  | 支払に関する備考 | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 1.6 予算（Budget）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 予算ID | budget_id | VARCHAR(20) | ○ | 予算の一意識別子 | KPI-003 |
| 会計年度 | fiscal_year | INT | ○ | 会計年度 | KPI-003 |
| 会計月 | fiscal_month | INT | ○ | 会計月（1-12） | KPI-003 |
| サプライヤー種別ID | supplier_type_id | VARCHAR(10) |  | サプライヤー種別（NULL=全体） | KPI-003 |
| チャネルID | channel_id | VARCHAR(10) |  | チャネル（NULL=全体） | KPI-003 |
| 予算金額 | budget_amount | DECIMAL(15,2) | ○ | 予算金額 | KPI-003 |
| 備考 | remarks | VARCHAR(500) |  | 予算に関する備考 | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

## 2. マスタエンティティ

### 2.1 サプライヤー（Supplier）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| サプライヤーID | supplier_id | VARCHAR(20) | ○ | サプライヤーの一意識別子 | 全KPI |
| サプライヤーコード | supplier_code | VARCHAR(20) | ○ | サプライヤーコード | - |
| サプライヤー名 | supplier_name | VARCHAR(200) | ○ | サプライヤー正式名称 | - |
| サプライヤー名カナ | supplier_name_kana | VARCHAR(200) |  | サプライヤー名カナ | - |
| サプライヤー種別ID | supplier_type_id | VARCHAR(10) | ○ | サプライヤー種別 | 全KPI |
| 支払条件ID | payment_terms_id | VARCHAR(10) | ○ | 支払条件 | KPI-001, KPI-007 |
| 郵便番号 | postal_code | VARCHAR(10) |  | 郵便番号 | - |
| 住所 | address | VARCHAR(300) |  | 住所 | - |
| 電話番号 | phone_number | VARCHAR(20) |  | 電話番号 | - |
| 担当者名 | contact_person | VARCHAR(100) |  | 担当者名 | - |
| メールアドレス | email | VARCHAR(100) |  | メールアドレス | - |
| 銀行名 | bank_name | VARCHAR(100) |  | 振込先銀行名 | - |
| 支店名 | branch_name | VARCHAR(100) |  | 振込先支店名 | - |
| 口座種別 | account_type | VARCHAR(10) |  | 口座種別（普通/当座） | - |
| 口座番号 | account_number | VARCHAR(20) |  | 口座番号 | - |
| 口座名義 | account_holder | VARCHAR(100) |  | 口座名義 | - |
| 取引開始日 | contract_start_date | DATE |  | 取引開始日 | - |
| 取引終了日 | contract_end_date | DATE |  | 取引終了日 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 2.2 顧客（Customer）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 顧客ID | customer_id | VARCHAR(20) | ○ | 顧客の一意識別子 | - |
| 顧客番号 | customer_number | VARCHAR(20) | ○ | 顧客番号 | - |
| 顧客名 | customer_name | VARCHAR(200) | ○ | 顧客名 | - |
| 顧客名カナ | customer_name_kana | VARCHAR(200) |  | 顧客名カナ | - |
| 顧客区分 | customer_category | VARCHAR(20) |  | 顧客区分（個人/法人） | - |
| 郵便番号 | postal_code | VARCHAR(10) |  | 郵便番号 | - |
| 住所 | address | VARCHAR(300) |  | 住所 | - |
| 電話番号 | phone_number | VARCHAR(20) |  | 電話番号 | - |
| メールアドレス | email | VARCHAR(100) |  | メールアドレス | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

---

### 2.3 チャネル（Channel）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| チャネルID | channel_id | VARCHAR(10) | ○ | チャネルの一意識別子 | KPI-008, KPI-009 |
| チャネルコード | channel_code | VARCHAR(10) | ○ | チャネルコード | - |
| チャネル名 | channel_name | VARCHAR(100) | ○ | チャネル名 | - |
| チャネル種別 | channel_type | VARCHAR(20) | ○ | 顧客予約型/事前仕入型/月次一括精算型 | KPI-008, KPI-009 |
| 表示順 | display_order | INT | ○ | 表示順序 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

**初期データ例**:
- CH001: 顧客予約型（店舗窓口）
- CH002: 顧客予約型（Webサイト）
- CH003: 顧客予約型（電話予約）
- CH004: 事前仕入型（パッケージツアー）
- CH005: 月次一括精算型（法人契約）

---

### 2.4 サプライヤー種別（Supplier Type）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| サプライヤー種別ID | supplier_type_id | VARCHAR(10) | ○ | サプライヤー種別の一意識別子 | 全KPI |
| サプライヤー種別コード | supplier_type_code | VARCHAR(10) | ○ | サプライヤー種別コード | - |
| サプライヤー種別名 | supplier_type_name | VARCHAR(100) | ○ | サプライヤー種別名 | - |
| 表示順 | display_order | INT | ○ | 表示順序 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

**初期データ例**:
- ST001: 宿泊施設
- ST002: 航空会社
- ST003: 鉄道会社
- ST004: バス会社
- ST005: 観光施設
- ST006: 飲食店
- ST007: その他

---

### 2.5 サービス種別（Service Type）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| サービス種別ID | service_type_id | VARCHAR(10) | ○ | サービス種別の一意識別子 | 全KPI |
| サービス種別コード | service_type_code | VARCHAR(10) | ○ | サービス種別コード | - |
| サービス種別名 | service_type_name | VARCHAR(100) | ○ | サービス種別名 | - |
| サプライヤー種別ID | supplier_type_id | VARCHAR(10) | ○ | 関連サプライヤー種別 | - |
| 表示順 | display_order | INT | ○ | 表示順序 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

**初期データ例**:
- SV001: 宿泊（ホテル）
- SV002: 宿泊（旅館）
- SV003: 航空券（国内線）
- SV004: 鉄道券（新幹線）
- SV005: 鉄道券（在来線）
- SV006: バス（高速バス）
- SV007: バス（観光バス）
- SV008: 入場券（観光施設）
- SV009: 食事（昼食）
- SV010: 食事（夕食）

---

### 2.6 支払条件（Payment Terms）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 支払条件ID | payment_terms_id | VARCHAR(10) | ○ | 支払条件の一意識別子 | KPI-001, KPI-007 |
| 支払条件コード | payment_terms_code | VARCHAR(10) | ○ | 支払条件コード | - |
| 支払条件名 | payment_terms_name | VARCHAR(100) | ○ | 支払条件名 | - |
| 支払サイト日数 | payment_days | INT | ○ | 標準支払サイト（日数） | KPI-001, KPI-007 |
| 締日 | closing_day | INT |  | 締日（1-31、NULL=締日なし） | KPI-001 |
| 支払日 | payment_day | INT |  | 支払日（1-31、NULL=都度払い） | KPI-001 |
| 説明 | description | VARCHAR(200) |  | 支払条件の説明 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

**初期データ例**:
- PT001: 即時払い（支払サイト0日）
- PT002: 月末締め翌月末払い（支払サイト30-60日）
- PT003: 月末締め翌々月10日払い（支払サイト40-70日）
- PT004: 15日締め当月末払い（支払サイト15-30日）
- PT005: 都度払い（利用後7日以内）

---

### 2.7 支払方法（Payment Method）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 支払方法ID | payment_method_id | VARCHAR(10) | ○ | 支払方法の一意識別子 | KPI-002 |
| 支払方法コード | payment_method_code | VARCHAR(10) | ○ | 支払方法コード | - |
| 支払方法名 | payment_method_name | VARCHAR(100) | ○ | 支払方法名 | - |
| 手数料率 | fee_rate | DECIMAL(5,4) |  | 手数料率（%） | - |
| 表示順 | display_order | INT | ○ | 表示順序 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

**初期データ例**:
- PM001: 銀行振込
- PM002: 手形
- PM003: 小切手
- PM004: 現金
- PM005: クレジットカード

---

### 2.8 支払ステータス（Payment Status）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 支払ステータスID | payment_status_id | VARCHAR(10) | ○ | 支払ステータスの一意識別子 | KPI-002, KPI-005 |
| 支払ステータスコード | payment_status_code | VARCHAR(10) | ○ | 支払ステータスコード | - |
| 支払ステータス名 | payment_status_name | VARCHAR(100) | ○ | 支払ステータス名 | - |
| 表示順 | display_order | INT | ○ | 表示順序 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |

**初期データ例**:
- PS001: 支払予定
- PS002: 承認待ち
- PS003: 承認済み
- PS004: 支払処理中
- PS005: 支払完了
- PS006: 支払保留
- PS007: 支払キャンセル

---

### 2.9 会計期間（Fiscal Period）

| 項目名 | 論理名 | データ型 | 必須 | 説明 | KPI関連 |
|--------|--------|----------|------|------|---------|
| 会計期間ID | fiscal_period_id | VARCHAR(10) | ○ | 会計期間の一意識別子 | KPI-003 |
| 会計年度 | fiscal_year | INT | ○ | 会計年度 | KPI-003 |
| 会計月 | fiscal_month | INT | ○ | 会計月（1-12） | KPI-003 |
| 期間開始日 | period_start_date | DATE | ○ | 期間開始日 | KPI-003 |
| 期間終了日 | period_end_date | DATE | ○ | 期間終了日 | KPI-003 |
| 営業日数 | business_days | INT | ○ | 営業日数 | - |
| 有効フラグ | is_active | BOOLEAN | ○ | 有効/無効フラグ | - |
| 作成日時 | created_at | TIMESTAMP | ○ | レコード作成日時 | - |
| 更新日時 | updated_at | TIMESTAMP | ○ | レコード更新日時 | - |