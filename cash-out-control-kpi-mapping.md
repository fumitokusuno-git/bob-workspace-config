# キャッシュアウトコントロール - KPI計算データ項目マッピング

## 1. KPI計算に必要なデータ項目一覧

### KPI-001: 日次支払予測額

**計算ロジック**:
```sql
SELECT 
    支払予定日,
    サプライヤー種別名,
    チャネル名,
    支払条件名,
    SUM(支払金額) AS 予測支払額
FROM (
    -- 確定支払（請求済み）
    SELECT 
        p.scheduled_payment_date AS 支払予定日,
        st.supplier_type_name AS サプライヤー種別名,
        ch.channel_name AS チャネル名,
        pt.payment_terms_name AS 支払条件名,
        p.payment_amount AS 支払金額
    FROM payment p
    JOIN supplier s ON p.supplier_id = s.supplier_id
    JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
    JOIN invoice i ON p.invoice_id = i.invoice_id
    JOIN procurement_detail pd ON i.invoice_id = pd.invoice_id
    JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
    JOIN reservation r ON rd.reservation_id = r.reservation_id
    JOIN channel ch ON r.channel_id = ch.channel_id
    JOIN payment_terms pt ON s.payment_terms_id = pt.payment_terms_id
    WHERE p.payment_status_id IN ('PS001', 'PS002', 'PS003', 'PS004')
    
    UNION ALL
    
    -- 予測支払（未請求）
    SELECT 
        pd.scheduled_payment_date AS 支払予定日,
        st.supplier_type_name AS サプライヤー種別名,
        ch.channel_name AS チャネル名,
        pt.payment_terms_name AS 支払条件名,
        pd.procurement_amount AS 支払金額
    FROM procurement_detail pd
    JOIN supplier s ON pd.supplier_id = s.supplier_id
    JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
    JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
    JOIN reservation r ON rd.reservation_id = r.reservation_id
    JOIN channel ch ON r.channel_id = ch.channel_id
    JOIN payment_terms pt ON s.payment_terms_id = pt.payment_terms_id
    WHERE pd.invoice_id IS NULL
    AND pd.procurement_status = '確定'
) AS combined
WHERE 支払予定日 BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '30 days'
GROUP BY 支払予定日, サプライヤー種別名, チャネル名, 支払条件名
ORDER BY 支払予定日;
```

**必要なテーブル**:
- [`payment`](payment) - 支払情報
- [`invoice`](invoice) - 請求情報
- [`procurement_detail`](procurement_detail) - 仕入明細
- [`reservation_detail`](reservation_detail) - 予約明細
- [`reservation`](reservation) - 予約
- [`supplier`](supplier) - サプライヤー
- [`supplier_type`](supplier_type) - サプライヤー種別
- [`channel`](channel) - チャネル
- [`payment_terms`](payment_terms) - 支払条件

**必要なデータ項目**:
- [`payment.scheduled_payment_date`](payment.scheduled_payment_date)
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`procurement_detail.scheduled_payment_date`](procurement_detail.scheduled_payment_date)
- [`procurement_detail.procurement_amount`](procurement_detail.procurement_amount)
- [`procurement_detail.invoice_id`](procurement_detail.invoice_id)
- [`procurement_detail.procurement_status`](procurement_detail.procurement_status)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)
- [`supplier.payment_terms_id`](supplier.payment_terms_id)
- [`reservation.channel_id`](reservation.channel_id)

---

### KPI-002: 週次キャッシュアウト実績

**計算ロジック**:
```sql
SELECT 
    DATE_TRUNC('week', p.payment_execution_date) AS 週,
    st.supplier_type_name AS サプライヤー種別名,
    ch.channel_name AS チャネル名,
    pm.payment_method_name AS 支払方法名,
    SUM(p.payment_amount) AS 支払実績額,
    COUNT(*) AS 支払件数
FROM payment p
JOIN supplier s ON p.supplier_id = s.supplier_id
JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
JOIN invoice i ON p.invoice_id = i.invoice_id
JOIN procurement_detail pd ON i.invoice_id = pd.invoice_id
JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
JOIN reservation r ON rd.reservation_id = r.reservation_id
JOIN channel ch ON r.channel_id = ch.channel_id
JOIN payment_method pm ON p.payment_method_id = pm.payment_method_id
WHERE p.payment_status_id = 'PS005'  -- 支払完了
AND p.payment_execution_date IS NOT NULL
GROUP BY 週, サプライヤー種別名, チャネル名, 支払方法名
ORDER BY 週 DESC;
```

**必要なテーブル**:
- [`payment`](payment)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)
- [`channel`](channel)
- [`payment_method`](payment_method)
- [`invoice`](invoice)
- [`procurement_detail`](procurement_detail)
- [`reservation_detail`](reservation_detail)
- [`reservation`](reservation)

**必要なデータ項目**:
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`payment.payment_method_id`](payment.payment_method_id)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)
- [`reservation.channel_id`](reservation.channel_id)

---

### KPI-003: 月次キャッシュアウト予算達成率

**計算ロジック**:
```sql
SELECT 
    fp.fiscal_year AS 会計年度,
    fp.fiscal_month AS 会計月,
    COALESCE(st.supplier_type_name, '全体') AS サプライヤー種別名,
    COALESCE(ch.channel_name, '全体') AS チャネル名,
    b.budget_amount AS 予算金額,
    COALESCE(actual.actual_amount, 0) AS 実績金額,
    CASE 
        WHEN b.budget_amount > 0 THEN 
            ROUND((COALESCE(actual.actual_amount, 0) / b.budget_amount * 100), 2)
        ELSE 0 
    END AS 予算達成率
FROM budget b
JOIN fiscal_period fp ON b.fiscal_year = fp.fiscal_year 
    AND b.fiscal_month = fp.fiscal_month
LEFT JOIN supplier_type st ON b.supplier_type_id = st.supplier_type_id
LEFT JOIN channel ch ON b.channel_id = ch.channel_id
LEFT JOIN (
    SELECT 
        EXTRACT(YEAR FROM fp.period_start_date) AS fiscal_year,
        EXTRACT(MONTH FROM fp.period_start_date) AS fiscal_month,
        s.supplier_type_id,
        r.channel_id,
        SUM(p.payment_amount) AS actual_amount
    FROM payment p
    JOIN supplier s ON p.supplier_id = s.supplier_id
    JOIN invoice i ON p.invoice_id = i.invoice_id
    JOIN procurement_detail pd ON i.invoice_id = pd.invoice_id
    JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
    JOIN reservation r ON rd.reservation_id = r.reservation_id
    JOIN fiscal_period fp ON p.payment_execution_date BETWEEN fp.period_start_date AND fp.period_end_date
    WHERE p.payment_status_id = 'PS005'
    GROUP BY fiscal_year, fiscal_month, s.supplier_type_id, r.channel_id
) actual ON b.fiscal_year = actual.fiscal_year 
    AND b.fiscal_month = actual.fiscal_month
    AND (b.supplier_type_id = actual.supplier_type_id OR b.supplier_type_id IS NULL)
    AND (b.channel_id = actual.channel_id OR b.channel_id IS NULL)
ORDER BY fp.fiscal_year DESC, fp.fiscal_month DESC;
```

**必要なテーブル**:
- [`budget`](budget)
- [`fiscal_period`](fiscal_period)
- [`payment`](payment)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)
- [`channel`](channel)

**必要なデータ項目**:
- [`budget.fiscal_year`](budget.fiscal_year)
- [`budget.fiscal_month`](budget.fiscal_month)
- [`budget.budget_amount`](budget.budget_amount)
- [`budget.supplier_type_id`](budget.supplier_type_id)
- [`budget.channel_id`](budget.channel_id)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`fiscal_period.period_start_date`](fiscal_period.period_start_date)
- [`fiscal_period.period_end_date`](fiscal_period.period_end_date)

---

### KPI-004: 支払予測精度（MAPE）

**計算ロジック**:
```sql
WITH prediction_actual AS (
    SELECT 
        pd.scheduled_payment_date AS 予測日,
        pd.procurement_amount AS 予測金額,
        p.payment_execution_date AS 実行日,
        p.payment_amount AS 実績金額,
        ABS(p.payment_amount - pd.procurement_amount) / p.payment_amount AS 絶対誤差率,
        DATEDIFF(day, pd.scheduled_payment_date, p.payment_execution_date) AS 予測期間日数,
        st.supplier_type_name AS サプライヤー種別名,
        ch.channel_name AS チャネル名
    FROM procurement_detail pd
    JOIN payment p ON pd.invoice_id = (
        SELECT invoice_id FROM invoice WHERE invoice_id = p.invoice_id
    )
    JOIN supplier s ON pd.supplier_id = s.supplier_id
    JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
    JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
    JOIN reservation r ON rd.reservation_id = r.reservation_id
    JOIN channel ch ON r.channel_id = ch.channel_id
    WHERE p.payment_status_id = 'PS005'
    AND p.payment_execution_date IS NOT NULL
)
SELECT 
    CASE 
        WHEN 予測期間日数 <= 7 THEN '1週間先'
        WHEN 予測期間日数 <= 14 THEN '2週間先'
        WHEN 予測期間日数 <= 30 THEN '1ヶ月先'
        ELSE '1ヶ月超'
    END AS 予測期間,
    サプライヤー種別名,
    チャネル名,
    ROUND(AVG(絶対誤差率) * 100, 2) AS MAPE,
    COUNT(*) AS サンプル数
FROM prediction_actual
GROUP BY 予測期間, サプライヤー種別名, チャネル名
ORDER BY 予測期間, サプライヤー種別名;
```

**必要なテーブル**:
- [`procurement_detail`](procurement_detail)
- [`payment`](payment)
- [`invoice`](invoice)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)
- [`reservation`](reservation)
- [`channel`](channel)

**必要なデータ項目**:
- [`procurement_detail.scheduled_payment_date`](procurement_detail.scheduled_payment_date)
- [`procurement_detail.procurement_amount`](procurement_detail.procurement_amount)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_status_id`](payment.payment_status_id)

---

### KPI-005: 支払遅延率

**計算ロジック**:
```sql
SELECT 
    DATE_TRUNC('month', p.payment_execution_date) AS 月,
    st.supplier_type_name AS サプライヤー種別名,
    CASE 
        WHEN DATEDIFF(day, p.scheduled_payment_date, p.payment_execution_date) = 0 THEN '遅延なし'
        WHEN DATEDIFF(day, p.scheduled_payment_date, p.payment_execution_date) BETWEEN 1 AND 3 THEN '1-3日遅延'
        WHEN DATEDIFF(day, p.scheduled_payment_date, p.payment_execution_date) BETWEEN 4 AND 7 THEN '4-7日遅延'
        WHEN DATEDIFF(day, p.scheduled_payment_date, p.payment_execution_date) > 7 THEN '8日以上遅延'
        ELSE '早期支払'
    END AS 遅延区分,
    COUNT(*) AS 件数,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY DATE_TRUNC('month', p.payment_execution_date)), 2) AS 構成比
FROM payment p
JOIN supplier s ON p.supplier_id = s.supplier_id
JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
WHERE p.payment_status_id = 'PS005'
AND p.payment_execution_date IS NOT NULL
GROUP BY 月, サプライヤー種別名, 遅延区分
ORDER BY 月 DESC, サプライヤー種別名, 遅延区分;
```

**必要なテーブル**:
- [`payment`](payment)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)

**必要なデータ項目**:
- [`payment.scheduled_payment_date`](payment.scheduled_payment_date)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)

---

### KPI-006: サプライヤー別支払集中度

**計算ロジック**:
```sql
WITH supplier_payment AS (
    SELECT 
        s.supplier_id,
        s.supplier_name AS サプライヤー名,
        st.supplier_type_name AS サプライヤー種別名,
        SUM(p.payment_amount) AS 支払総額
    FROM payment p
    JOIN supplier s ON p.supplier_id = s.supplier_id
    JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
    WHERE p.payment_status_id = 'PS005'
    AND p.payment_execution_date BETWEEN '2026-01-01' AND '2026-12-31'
    GROUP BY s.supplier_id, s.supplier_name, st.supplier_type_name
),
ranked_suppliers AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (ORDER BY 支払総額 DESC) AS ランク,
        SUM(支払総額) OVER () AS 全体支払総額,
        SUM(支払総額) OVER (ORDER BY 支払総額 DESC) AS 累積支払額
    FROM supplier_payment
)
SELECT 
    ランク,
    サプライヤー名,
    サプライヤー種別名,
    支払総額,
    ROUND(支払総額 / 全体支払総額 * 100, 2) AS 構成比,
    ROUND(累積支払額 / 全体支払総額 * 100, 2) AS 累積構成比
FROM ranked_suppliers
WHERE ランク <= 50
ORDER BY ランク;
```

**必要なテーブル**:
- [`payment`](payment)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)

**必要なデータ項目**:
- [`payment.supplier_id`](payment.supplier_id)
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`supplier.supplier_name`](supplier.supplier_name)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)

---

### KPI-007: サプライヤー別平均支払サイト

**計算ロジック**:
```sql
SELECT 
    s.supplier_id,
    s.supplier_name AS サプライヤー名,
    st.supplier_type_name AS サプライヤー種別名,
    pt.payment_terms_name AS 支払条件名,
    pt.payment_days AS 標準支払サイト,
    ROUND(AVG(DATEDIFF(day, i.invoice_received_date, p.payment_execution_date)), 1) AS 実績平均支払サイト,
    ROUND(AVG(DATEDIFF(day, i.invoice_received_date, p.payment_execution_date)) - pt.payment_days, 1) AS 標準との差異,
    COUNT(*) AS 支払件数
FROM payment p
JOIN invoice i ON p.invoice_id = i.invoice_id
JOIN supplier s ON p.supplier_id = s.supplier_id
JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
JOIN payment_terms pt ON s.payment_terms_id = pt.payment_terms_id
WHERE p.payment_status_id = 'PS005'
AND p.payment_execution_date IS NOT NULL
AND i.invoice_received_date IS NOT NULL
GROUP BY s.supplier_id, s.supplier_name, st.supplier_type_name, pt.payment_terms_name, pt.payment_days
HAVING COUNT(*) >= 3
ORDER BY 実績平均支払サイト DESC;
```

**必要なテーブル**:
- [`payment`](payment)
- [`invoice`](invoice)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)
- [`payment_terms`](payment_terms)

**必要なデータ項目**:
- [`invoice.invoice_received_date`](invoice.invoice_received_date)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.supplier_id`](payment.supplier_id)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`supplier.supplier_name`](supplier.supplier_name)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)
- [`supplier.payment_terms_id`](supplier.payment_terms_id)
- [`payment_terms.payment_days`](payment_terms.payment_days)

---

### KPI-008: チャネル別支払構成比

**計算ロジック**:
```sql
SELECT 
    DATE_TRUNC('month', p.payment_execution_date) AS 月,
    ch.channel_name AS チャネル名,
    ch.channel_type AS チャネル種別,
    st.supplier_type_name AS サプライヤー種別名,
    SUM(p.payment_amount) AS 支払金額,
    ROUND(SUM(p.payment_amount) * 100.0 / SUM(SUM(p.payment_amount)) OVER (PARTITION BY DATE_TRUNC('month', p.payment_execution_date)), 2) AS 構成比
FROM payment p
JOIN invoice i ON p.invoice_id = i.invoice_id
JOIN procurement_detail pd ON i.invoice_id = pd.invoice_id
JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
JOIN reservation r ON rd.reservation_id = r.reservation_id
JOIN channel ch ON r.channel_id = ch.channel_id
JOIN supplier s ON p.supplier_id = s.supplier_id
JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
WHERE p.payment_status_id = 'PS005'
GROUP BY 月, ch.channel_name, ch.channel_type, st.supplier_type_name
ORDER BY 月 DESC, 支払金額 DESC;
```

**必要なテーブル**:
- [`payment`](payment)
- [`reservation`](reservation)
- [`channel`](channel)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)

**必要なデータ項目**:
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`reservation.channel_id`](reservation.channel_id)
- [`channel.channel_name`](channel.channel_name)
- [`channel.channel_type`](channel.channel_type)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)

---

### KPI-009: チャネル別平均支払リードタイム

**計算ロジック**:
```sql
SELECT 
    ch.channel_name AS チャネル名,
    ch.channel_type AS チャネル種別,
    st.supplier_type_name AS サプライヤー種別名,
    ROUND(AVG(DATEDIFF(day, 
        CASE 
            WHEN ch.channel_type = '顧客予約型' THEN r.reservation_datetime
            WHEN ch.channel_type = '事前仕入型' THEN pd.procurement_date
            ELSE r.reservation_datetime
        END, 
        p.payment_execution_date)), 1) AS 平均リードタイム日数,
    COUNT(*) AS 件数
FROM payment p
JOIN invoice i ON p.invoice_id = i.invoice_id
JOIN procurement_detail pd ON i.invoice_id = pd.invoice_id
JOIN reservation_detail rd ON pd.reservation_detail_id = rd.reservation_detail_id
JOIN reservation r ON rd.reservation_id = r.reservation_id
JOIN channel ch ON r.channel_id = ch.channel_id
JOIN supplier s ON p.supplier_id = s.supplier_id
JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
WHERE p.payment_status_id = 'PS005'
AND p.payment_execution_date IS NOT NULL
GROUP BY ch.channel_name, ch.channel_type, st.supplier_type_name
ORDER BY 平均リードタイム日数 DESC;
```

**必要なテーブル**:
- [`payment`](payment)
- [`procurement_detail`](procurement_detail)
- [`reservation`](reservation)
- [`channel`](channel)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)

**必要なデータ項目**:
- [`reservation.reservation_datetime`](reservation.reservation_datetime)
- [`procurement_detail.procurement_date`](procurement_detail.procurement_date)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`channel.channel_name`](channel.channel_name)
- [`channel.channel_type`](channel.channel_type)

---

### KPI-010: 支払額異常検知率

**計算ロジック**:
```sql
WITH payment_stats AS (
    SELECT 
        s.supplier_id,
        s.supplier_name,
        st.supplier_type_name,
        AVG(p.payment_amount) AS 平均支払額,
        STDDEV(p.payment_amount) AS 標準偏差
    FROM payment p
    JOIN supplier s ON p.supplier_id = s.supplier_id
    JOIN supplier_type st ON s.supplier_type_id = st.supplier_type_id
    WHERE p.payment_status_id = 'PS005'
    AND p.payment_execution_date >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY s.supplier_id, s.supplier_name, st.supplier_type_name
    HAVING COUNT(*) >= 10
),
anomaly_detection AS (
    SELECT 
        p.payment_id,
        p.payment_execution_date,
        ps.supplier_name,
        ps.supplier_type_name,
        p.payment_amount,
        ps.平均支払額,
        ps.標準偏差,
        ABS(p.payment_amount - ps.平均支払額) / ps.標準偏差 AS Z_SCORE,
        CASE 
            WHEN ABS(p.payment_amount - ps.平均支払額) > 2 * ps.標準偏差 THEN 1
            ELSE 0
        END AS 異常フラグ
    FROM payment p
    JOIN payment_stats ps ON p.supplier_id = ps.supplier_id
    WHERE p.payment_status_id = 'PS005'
    AND p.payment_execution_date >= CURRENT_DATE - INTERVAL '30 days'
)
SELECT 
    DATE_TRUNC('month', payment_execution_date) AS 月,
    supplier_type_name AS サプライヤー種別名,
    COUNT(*) AS 総支払件数,
    SUM(異常フラグ) AS 異常件数,
    ROUND(SUM(異常フラグ) * 100.0 / COUNT(*), 2) AS 異常検知率
FROM anomaly_detection
GROUP BY 月, supplier_type_name
ORDER BY 月 DESC, 異常検知率 DESC;
```

**必要なテーブル**:
- [`payment`](payment)
- [`supplier`](supplier)
- [`supplier_type`](supplier_type)

**必要なデータ項目**:
- [`payment.payment_amount`](payment.payment_amount)
- [`payment.payment_execution_date`](payment.payment_execution_date)
- [`payment.payment_status_id`](payment.payment_status_id)
- [`payment.supplier_id`](payment.supplier_id)
- [`supplier.supplier_name`](supplier.supplier_name)
- [`supplier.supplier_type_id`](supplier.supplier_type_id)

---

## 2. データソースマッピング

### 2.1 予約管理システム
**提供データ**:
- 予約（Reservation）
- 予約明細（Reservation Detail）
- 顧客（Customer）

**連携方式**: バッチ連携（日次）またはAPI連携（リアルタイム）

---

### 2.2 支払管理システム
**提供データ**:
- 仕入明細（Procurement Detail）
- 請求（Invoice）
- 支払（Payment）

**連携方式**: バッチ連携（日次）またはAPI連携（リアルタイム）

---

### 2.3 会計システム
**提供データ**:
- 予算（Budget）
- 会計期間（Fiscal Period）

**連携方式**: バッチ連携（月次）

---

### 2.4 マスタ管理システム
**提供データ**:
- サプライヤー（Supplier）
- チャネル（Channel）
- サプライヤー種別（Supplier Type）
- サービス種別（Service Type）
- 支払条件（Payment Terms）
- 支払方法（Payment Method）
- 支払ステータス（Payment Status）

**連携方式**: バッチ連携（日次）またはマスタ同期

---

## 3. データ品質要件

### 3.1 必須項目の完全性
- すべての必須項目（○マーク）は NULL 不可
- 外部キー項目は参照先テーブルに存在する値のみ許可

### 3.2 データ整合性
- [`支払.支払金額`](payment.payment_amount) = 関連する [`請求.請求金額`](invoice.invoice_amount) の合計
- [`予約.総原価額`](reservation.total_cost_amount) = 関連する [`予約明細.原価額`](reservation_detail.cost_amount) の合計
- [`支払実行日`](payment.payment_execution_date) >= [`請求受領日`](invoice.invoice_received_date)

### 3.3 タイムリネス
- トランザクションデータ: 発生後24時間以内に連携
- マスタデータ: 変更後24時間以内に連携
- KPI計算: 日次バッチで更新（深夜実行）

### 3.4 精度要件
- 金額項目: 小数点以下2桁まで
- 日付項目: YYYY-MM-DD形式
- タイムスタンプ: YYYY-MM-DD HH:MI:SS形式

---

## 4. 実装優先順位

### フェーズ1（MVP）: 基本KPIの実装
1. KPI-001: 日次支払予測額
2. KPI-002: 週次キャッシュアウト実績
3. KPI-003: 月次キャッシュアウト予算達成率

### フェーズ2: 予測精度とサプライヤー分析
4. KPI-004: 支払予測精度（MAPE）
5. KPI-006: サプライヤー別支払集中度
6. KPI-007: サプライヤー別平均支払サイト

### フェーズ3: チャネル分析と異常検知
7. KPI-008: チャネル別支払構成比
8. KPI-009: チャネル別平均支払リードタイム
9. KPI-005: 支払遅延率
10. KPI-010: 支払額異常検知率