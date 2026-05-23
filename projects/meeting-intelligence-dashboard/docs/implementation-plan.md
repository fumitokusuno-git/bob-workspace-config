# 実装計画 - Meeting Intelligence Dashboard

## 📋 概要

本ドキュメントでは、Meeting Intelligence Dashboard MVP版の実装計画を詳細に定義します。

---

## 🎯 実装の目標

### MVP（最小実行可能製品）の定義
- **目的**: 最速で価値検証を行う
- **期間**: 2-3週間
- **スコープ**: 手動入力 → AI分析 → レポート生成の基本フロー
- **前提**: テキスト議事録ベースの分析であり、一部指標は推定値として扱う
- **入力方針**: `名前: 発言` 形式を推奨し、自由記述も許容するが分析信頼度を明示する

### 成功基準
1. ✅ 議事録を入力してAI分析が実行できる
2. ✅ 5つのKPIスコアが表示される
3. ✅ 具体的な改善提案が生成される
4. ✅ Markdownレポートがダウンロードできる
5. ✅ 過去の会議データが保存・表示される

---

## 🗓️ 実装スケジュール

### Week 1: 基礎実装（設計完了済み）

#### Day 1-2: プロジェクトセットアップ ✅
- [x] プロジェクト構造の設計
- [x] README作成
- [x] KPI定義書作成
- [x] ワイヤーフレーム作成
- [x] データモデル設計
- [x] AIプロンプト設計

#### Day 3-4: HTMLプロトタイプ（基本UI）
- [ ] HTML構造の実装
- [ ] CSS スタイリング（レスポンシブ対応）
- [ ] 入力フォームの実装
- [ ] バリデーション機能

#### Day 5-7: AI分析機能
- [ ] OpenAI API連携
- [ ] プロンプト実装
- [ ] レスポンス処理
- [ ] エラーハンドリング

### Week 2: コア機能実装

#### Day 8-10: KPI計算とダッシュボード
- [ ] KPI自動採点ロジック
- [ ] スコアカード表示
- [ ] レーダーチャート実装（Chart.js）
- [ ] 改善提案の表示

#### Day 11-12: レポート生成
- [ ] Markdown生成ロジック
- [ ] ダウンロード機能
- [ ] JSON エクスポート

#### Day 13-14: データ永続化
- [ ] LocalStorage実装
- [ ] 会議履歴の表示
- [ ] トレンドグラフ

### Week 3: テスト・改善・ドキュメント

#### Day 15-17: テストと改善
- [ ] サンプルデータでのテスト
- [ ] 実際の会議データでのテスト
- [ ] バグ修正
- [ ] UI/UX改善

#### Day 18-19: ドキュメント整備
- [ ] 使い方ガイド作成
- [ ] 技術仕様書作成
- [ ] デモ動画作成

#### Day 20-21: リリース準備
- [ ] 最終調整
- [ ] パフォーマンス最適化
- [ ] セキュリティチェック
- [ ] 社内展開準備

---

## 🏗️ 技術スタック

### フロントエンド
```
HTML5
├── セマンティックHTML
├── アクセシビリティ対応（ARIA）
└── SEO最適化

CSS3
├── CSS Variables（テーマ管理）
├── Flexbox / Grid（レイアウト）
├── Media Queries（レスポンシブ）
└── Animations（UX向上）

JavaScript (ES6+)
├── Vanilla JS（フレームワークなし）
├── Async/Await（非同期処理）
├── Modules（コード分割）
└── LocalStorage API（データ永続化）
```

### ライブラリ
```
Chart.js v4.x
├── レーダーチャート
├── 折れ線グラフ
└── プログレスバー

Marked.js
└── Markdown生成

DOMPurify
└── XSS対策
```

### API
```
OpenAI API
├── Model: GPT-4
├── Max Tokens: 4000
└── Temperature: 0.3（一貫性重視）
```

### APIキー運用方針
- **個人検証用MVP**: ブラウザに設定したAPIキーを利用するローカル運用を許容
- **共有利用・社内展開**: クライアントから直接APIを呼ばず、バックエンドプロキシ経由を前提とする
- **設計上の注意**: 静的配布を前提とした公開環境ではクライアント直叩きを避ける

---

## 📁 ファイル構成

```
meeting-intelligence-dashboard/
├── README.md                          # プロジェクト概要
├── docs/                              # ドキュメント
│   ├── kpi-definitions.md             # KPI定義書
│   ├── implementation-plan.md         # 実装計画（このファイル）
│   ├── user-guide.md                  # 使い方ガイド
│   └── technical-spec.md              # 技術仕様書
├── design/                            # 設計資料
│   ├── wireframe.md                   # ワイヤーフレーム
│   ├── data-model.md                  # データモデル
│   └── prompt-templates.md            # AIプロンプト
├── src/                               # ソースコード
│   ├── index.html                     # メインHTML
│   ├── css/
│   │   ├── variables.css              # CSS変数
│   │   ├── base.css                   # 基本スタイル
│   │   ├── components.css             # コンポーネント
│   │   └── responsive.css             # レスポンシブ
│   └── js/
│       ├── app.js                     # メインアプリ
│       ├── api.js                     # OpenAI API連携
│       ├── kpi.js                     # KPI計算ロジック
│       ├── chart.js                   # グラフ描画
│       ├── storage.js                 # データ永続化
│       ├── report.js                  # レポート生成
│       └── utils.js                   # ユーティリティ
├── examples/                          # サンプルデータ
│   ├── sample-meeting-1.json
│   ├── sample-meeting-2.json
│   └── sample-transcript.txt
└── tests/                             # テスト
    └── test-scenarios.md
```

---

## 🔧 実装の詳細

### 1. HTMLプロトタイプ（Day 3-4）

#### 実装内容
```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meeting Intelligence Dashboard</title>
    <link rel="stylesheet" href="css/variables.css">
    <link rel="stylesheet" href="css/base.css">
    <link rel="stylesheet" href="css/components.css">
    <link rel="stylesheet" href="css/responsive.css">
</head>
<body>
    <!-- ヘッダー -->
    <header class="app-header">
        <h1>📊 Meeting Intelligence Dashboard</h1>
        <p>会議の生産性を可視化し、継続的な改善を支援します</p>
    </header>

    <!-- メインコンテンツ -->
    <main class="app-main">
        <!-- 入力セクション -->
        <section class="input-section">
            <!-- 会議情報入力 -->
            <!-- アジェンダ入力 -->
            <!-- 議事録入力 -->
            <!-- 分析実行ボタン -->
        </section>

        <!-- 分析結果セクション -->
        <section class="analysis-section" style="display: none;">
            <!-- 総合スコア -->
            <!-- KPIスコアカード -->
            <!-- レーダーチャート -->
            <!-- 改善提案 -->
            <!-- レポート出力 -->
        </section>

        <!-- 履歴セクション -->
        <section class="history-section">
            <!-- 会議履歴 -->
            <!-- トレンドグラフ -->
        </section>
    </main>

    <!-- フッター -->
    <footer class="app-footer">
        <p>&copy; 2024 Meeting Intelligence Dashboard</p>
    </footer>

    <!-- JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <script type="module" src="js/app.js"></script>
</body>
</html>
```

#### タスク
- [ ] セマンティックHTMLの実装
- [ ] フォーム要素の実装
- [ ] アクセシビリティ対応（ARIA属性）
- [ ] メタタグの設定

---

### 2. CSS スタイリング（Day 3-4）

#### CSS Variables（variables.css）
```css
:root {
    /* カラーパレット */
    --primary-blue: #2196F3;
    --primary-dark: #1976D2;
    --success-green: #4CAF50;
    --warning-orange: #FF9800;
    --error-red: #F44336;
    
    /* グレースケール */
    --gray-50: #FAFAFA;
    --gray-100: #F5F5F5;
    --gray-900: #212121;
    
    /* タイポグラフィ */
    --font-primary: 'Noto Sans JP', sans-serif;
    --text-base: 1rem;
    --text-xl: 1.25rem;
    
    /* スペーシング */
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 2rem;
    
    /* ボーダー */
    --border-radius: 8px;
    --border-color: var(--gray-300);
    
    /* シャドウ */
    --shadow-sm: 0 2px 4px rgba(0,0,0,0.1);
    --shadow-md: 0 4px 8px rgba(0,0,0,0.1);
}
```

#### タスク
- [ ] CSS変数の定義
- [ ] 基本スタイルの実装
- [ ] コンポーネントスタイルの実装
- [ ] レスポンシブデザインの実装

---

### 3. AI分析機能（Day 5-7）

#### OpenAI API連携（api.js）

MVPではAIとフロントエンドの責務を明確に分離します。

- **AIの責務**
  - 決定事項の抽出
  - 発言分析
  - 沈黙や停滞の兆候推定
  - アジェンダ消化状況の推定
  - KPIの一次評価と根拠生成
  - 改善提案の生成
  - `analysis_confidence` / `data_quality_flags` / `assumptions` の返却

- **フロントエンドの責務**
  - 必須入力と入力形式の検証
  - スコア範囲の検証
  - 総合スコア・グレード・差分の再計算
  - 表示用データへの正規化
  - 推定値ラベルの付与
  - エラーと品質低下の表示

```javascript
// api.js
export class OpenAIService {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseURL = 'https://api.openai.com/v1';
        this.model = 'gpt-4';
    }

    async analyzeMeeting(meetingData) {
        const prompt = this.buildPrompt(meetingData);
        
        try {
            const response = await fetch(`${this.baseURL}/chat/completions`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${this.apiKey}`
                },
                body: JSON.stringify({
                    model: this.model,
                    messages: [
                        {
                            role: 'system',
                            content: this.getSystemPrompt()
                        },
                        {
                            role: 'user',
                            content: prompt
                        }
                    ],
                    temperature: 0.3,
                    max_tokens: 4000,
                    response_format: { type: 'json_object' }
                })
            });

            if (!response.ok) {
                throw new Error(`API Error: ${response.status}`);
            }

            const data = await response.json();
            return JSON.parse(data.choices[0].message.content);
        } catch (error) {
            console.error('AI分析エラー:', error);
            throw error;
        }
    }

    getSystemPrompt() {
        // prompt-templates.md のシステムプロンプトを使用
        return `あなたは会議分析の専門家です...`;
    }

    buildPrompt(meetingData) {
        // prompt-templates.md のユーザープロンプトを使用
        return `以下の会議について、5つのKPIを評価し...`;
    }
}
```

#### タスク
- [ ] OpenAI API連携の実装
- [ ] プロンプト生成ロジック
- [ ] レスポンス解析
- [ ] エラーハンドリング
- [ ] ローディング表示

---

### 4. KPI計算ロジック（Day 8-10）

#### KPI計算（kpi.js）
```javascript
// kpi.js
export class KPICalculator {
    calculateOverallScore(kpiScores) {
        const weights = {
            pre_meeting_readiness: 0.25,
            decision_density: 0.25,
            voice_diversity: 0.15,
            silent_cost_reduction: 0.15,
            agenda_adherence: 0.20
        };

        let totalScore = 0;
        for (const [kpi, weight] of Object.entries(weights)) {
            totalScore += kpiScores[kpi].score * weight;
        }

        return Math.round(totalScore);
    }

    getGrade(score) {
        if (score >= 90) return 'S';
        if (score >= 80) return 'A';
        if (score >= 70) return 'B';
        if (score >= 60) return 'C';
        if (score >= 50) return 'D';
        return 'E';
    }

    getGradeDescription(grade) {
        const descriptions = {
            'S': '理想的な会議',
            'A': '優秀な会議',
            'B': '良好な会議',
            'C': '改善の余地あり',
            'D': '要改善',
            'E': '抜本的な見直しが必要'
        };
        return descriptions[grade];
    }
}
```

#### タスク
- [ ] KPI計算ロジックの実装
- [ ] グレード判定ロジック
- [ ] スコア表示の実装
- [ ] アニメーション効果

---

### 5. グラフ描画（Day 8-10）

#### Chart.js実装（chart.js）
```javascript
// chart.js
export class ChartRenderer {
    renderRadarChart(canvasId, kpiScores) {
        const ctx = document.getElementById(canvasId).getContext('2d');
        
        new Chart(ctx, {
            type: 'radar',
            data: {
                labels: [
                    '事前準備度',
                    '決定密度',
                    '発言多様性',
                    'サイレント削減',
                    'アジェンダ遵守'
                ],
                datasets: [
                    {
                        label: '今回の会議',
                        data: [
                            kpiScores.pre_meeting_readiness.score,
                            kpiScores.decision_density.score,
                            kpiScores.voice_diversity.score,
                            kpiScores.silent_cost_reduction.score,
                            kpiScores.agenda_adherence.score
                        ],
                        borderColor: '#2196F3',
                        backgroundColor: 'rgba(33, 150, 243, 0.2)'
                    },
                    {
                        label: '目標値',
                        data: [80, 70, 75, 85, 80],
                        borderColor: '#4CAF50',
                        borderDash: [5, 5],
                        backgroundColor: 'rgba(76, 175, 80, 0.1)'
                    }
                ]
            },
            options: {
                scales: {
                    r: {
                        min: 0,
                        max: 100,
                        ticks: { stepSize: 20 }
                    }
                }
            }
        });
    }

    renderTrendChart(canvasId, historyData) {
        // トレンドグラフの実装
    }
}
```

#### タスク
- [ ] レーダーチャートの実装
- [ ] トレンドグラフの実装
- [ ] プログレスバーの実装
- [ ] レスポンシブ対応

---

### 6. レポート生成（Day 11-12）

#### Markdown生成（report.js）
```javascript
// report.js
export class ReportGenerator {
    generateMarkdown(analysisResult) {
        const { meeting_info, analysis_result } = analysisResult;
        
        let markdown = `# 会議分析レポート\n\n`;
        markdown += `## 会議情報\n`;
        markdown += `- 会議名: ${meeting_info.title}\n`;
        markdown += `- 日時: ${meeting_info.date} ${meeting_info.time}\n`;
        markdown += `- 参加者: ${meeting_info.participants.join(', ')}\n`;
        markdown += `- 時間: ${meeting_info.duration_minutes}分\n\n`;
        
        markdown += `## 総合評価\n`;
        markdown += `- スコア: ${analysis_result.overall_score}/100\n`;
        markdown += `- 評価: ${analysis_result.grade}\n\n`;
        
        markdown += `## KPI詳細\n\n`;
        // KPIスコアの詳細を追加
        
        markdown += `## 改善提案\n\n`;
        // 改善提案を追加
        
        return markdown;
    }

    downloadMarkdown(content, filename) {
        const blob = new Blob([content], { type: 'text/markdown' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = filename;
        a.click();
        URL.revokeObjectURL(url);
    }

    downloadJSON(data, filename) {
        const blob = new Blob([JSON.stringify(data, null, 2)], { 
            type: 'application/json' 
        });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = filename;
        a.click();
        URL.revokeObjectURL(url);
    }
}
```

#### タスク
- [ ] Markdown生成ロジック
- [ ] JSON エクスポート
- [ ] ダウンロード機能
- [ ] ファイル名の自動生成

---

### 7. データ永続化（Day 13-14）

#### LocalStorage実装（storage.js）

MVPでは保存データを「履歴一覧用サマリー」と「詳細データ」に分け、デフォルトではサマリー中心に保存します。履歴比較の単位は、まず `meeting_info.title` が同じ会議同士に限定します。

```javascript
// storage.js
export class StorageService {
    constructor() {
        this.storageKey = 'meeting_history';
        this.maxMeetings = 500;
    }

    saveMeeting(meetingData) {
        const history = this.getHistory();
        history.unshift(meetingData);
        
        // 最大件数を超えたら古いデータを削除
        if (history.length > this.maxMeetings) {
            history.pop();
        }
        
        localStorage.setItem(this.storageKey, JSON.stringify(history));
    }

    getHistory() {
        const data = localStorage.getItem(this.storageKey);
        return data ? JSON.parse(data) : [];
    }

    getMeeting(meetingId) {
        const history = this.getHistory();
        return history.find(m => m.meeting_id === meetingId);
    }

    deleteMeeting(meetingId) {
        const history = this.getHistory();
        const filtered = history.filter(m => m.meeting_id !== meetingId);
        localStorage.setItem(this.storageKey, JSON.stringify(filtered));
    }

    clearHistory() {
        localStorage.removeItem(this.storageKey);
    }

    getStorageUsage() {
        const data = localStorage.getItem(this.storageKey);
        const bytes = new Blob([data || '']).size;
        const mb = (bytes / 1024 / 1024).toFixed(2);
        return { bytes, mb };
    }
}
```

#### タスク
- [ ] LocalStorage実装
- [ ] 会議履歴の保存・取得
- [ ] サマリー保存と詳細保存の分離
- [ ] 同一会議名ベースの履歴比較
- [ ] データの削除機能
- [ ] ストレージ使用量の監視

---

## 🧪 テスト計画

### テストシナリオ

#### 1. 基本フロー
```
1. 会議情報を入力
2. アジェンダを入力
3. 議事録を入力
4. 「AI分析を実行」ボタンをクリック
5. 分析結果が表示される
6. レポートをダウンロード
```

#### 2. エラーケース
```
- APIキーが未設定
- 必須項目が未入力
- 議事録が短すぎる
- API呼び出しが失敗
- ネットワークエラー
```

#### 3. エッジケース
```
- 非常に長い議事録（10,000文字以上）
- 参加者が1名のみ
- 会議時間が5分未満
- 決定事項が0件
```

### テストデータ

#### サンプル1: 理想的な会議
```json
{
  "meeting_title": "週次定例会議",
  "date": "2024-01-15",
  "time": "14:00",
  "duration_minutes": 60,
  "participants": ["田中太郎", "佐藤花子", "鈴木一郎"],
  "agenda": "1. 進捗報告\n2. 課題検討\n3. アクション決定",
  "transcript": "田中: 本日のアジェンダは3つです..."
}
```

---

## 🚀 デプロイ計画

### ローカル実行
```bash
# 1. ファイルをダウンロード
# 2. index.html をブラウザで開く
# 3. OpenAI APIキーを設定
# 4. 使用開始
```

### 将来の展開（Phase 2以降）
- GitHub Pages でのホスティング
- Vercel / Netlify でのデプロイ
- PWA化（オフライン対応）
- モバイルアプリ化

---

## 📊 進捗管理

### マイルストーン

| マイルストーン | 期限 | ステータス |
|--------------|------|-----------|
| 設計完了 | Week 1 Day 2 | ✅ 完了 |
| HTMLプロトタイプ | Week 1 Day 4 | ⏳ 予定 |
| AI分析機能 | Week 1 Day 7 | ⏳ 予定 |
| ダッシュボード | Week 2 Day 10 | ⏳ 予定 |
| レポート生成 | Week 2 Day 12 | ⏳ 予定 |
| データ永続化 | Week 2 Day 14 | ⏳ 予定 |
| テスト完了 | Week 3 Day 17 | ⏳ 予定 |
| ドキュメント | Week 3 Day 19 | ⏳ 予定 |
| リリース | Week 3 Day 21 | ⏳ 予定 |

---

## 🎯 次のステップ

1. **このプランをレビュー**
   - スケジュールの妥当性確認
   - 優先順位の調整

2. **実装モードへ切り替え**
   - Code モードまたは Advanced モードで実装開始
   - HTMLプロトタイプから着手

3. **定期的な進捗確認**
   - 毎日の進捗をTODOリストで管理
   - 問題があれば早期に対応

---

## 📝 備考

### 技術的な課題
- OpenAI APIのレート制限対策
- ブラウザのLocalStorage容量制限
- 大量データの処理パフォーマンス
- テキスト議事録ベース分析における推定精度の限界
- 入力形式の揺れによる分析品質のばらつき
- クライアント直叩き時のAPIキー保護

### 将来の拡張機能（Phase 2以降）
- 音声認識機能（Whisper API）
- Box連携
- Teams連携
- リアルタイム分析
- 複数プロジェクトの管理
- チーム分析機能
