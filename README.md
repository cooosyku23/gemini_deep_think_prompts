# gemini_deep_think_prompts

アプリケーション開発で使えるGemini 2.5 Deep Think用プロンプト集

## 概要

Gemini 2.5 Deep Thinkモードを活用して、アプリケーションのコードレビューを自動化するためのプロンプト集です。各プロンプトは「ワーカー（Worker）」と「バリデーター（Validator）」の2つの役割を逐次実行し、構造化されたYAML形式でレビュー結果を出力します。

## プロンプト一覧

### 1. アーキテクチャ評価 (01_Architecture_Evaluation.md)

**概要**
ソフトウェアアーキテクチャの設計原則違反や保守性・拡張性の問題を検出します。SOLID原則の違反、関心の分離の不備、結合度と凝集度の問題、循環依存などを評価対象とします。

**検出項目**
- 単一責任の原則（SRP）違反
- オープン/クローズドの原則（OCP）違反
- 神クラス（God Class）
- 循環依存
- 不適切な依存関係

**使用例**
```
<コードをプロンプトに貼り付け>

入力例：
class ReportGenerator:
    def generate_report(self, data, format):
        # データ取得
        db_data = self._fetch_data_from_db(data)
        # ビジネスロジック
        processed_data = self._process_data(db_data)
        # フォーマット処理
        if format == 'pdf':
            report = self._format_as_pdf(processed_data)
        elif format == 'csv':
            report = self._format_as_csv(processed_data)
        return report

出力例：
findings:
  - id: "ARCH-001"
    severity: "High"
    recommendation: "Schedule"
    reasoning: "ReportGeneratorクラスが、データ取得、ビジネスロジック、複数形式へのフォーマットという3つの異なる責務を持っており、単一責任の原則(SRP)に違反している。"
    impact_detail: "この設計では、データ取得ロジックの変更がフォーマット処理に影響したり、新しいフォーマットの追加が既存コードの大規模な修正を必要としたりするため、保守コストが増大する。"
    ...
```

### 2. セキュリティ評価 (02_Security_Evaluation.md)

**概要**
OWASP Top 10などの既知の脆弱性パターンを検出し、実害に直結するセキュリティリスクを評価します。SQLインジェクション、XSS、認証・認可の欠陥、セキュリティ設定不備などを対象とします。

**検出項目**
- SQLインジェクション
- クロスサイトスクリプティング（XSS）
- 認証・認可の欠陥
- 安全でない直接オブジェクト参照（IDOR）
- セキュリティ設定不備

**使用例**
```
<コードをプロンプトに貼り付け>

入力例：
def get_user(user_id):
    conn = sqlite3.connect('app.db')
    cursor = conn.cursor()
    query = "SELECT * FROM users WHERE id = '" + user_id + "'"
    cursor.execute(query)
    return cursor.fetchone()

出力例：
findings:
  - id: "SEC-001"
    severity: "Critical"
    recommendation: "Fix Now"
    impact_notes: [I, AuthZ]
    reasoning: "ユーザーからの入力を直接SQLクエリに連結しており、SQLインジェクション攻撃に対して脆弱である。"
    impact_detail: "攻撃者が細工したuser_idを送信することで、データベース内の任意の情報を読み取り、変更、または削除できる可能性がある。"
    repro: |
      1. user_idパラメータに ' OR 1=1 -- を設定してリクエストを送信する。
      2. 本来アクセスできないはずのユーザー情報が返却されることを確認する。
    ...
```

### 3. パフォーマンス評価 (03_Performance_Evaluation.md)

**概要**
アプリケーションの性能劣化やスケーラビリティ問題を検出します。アルゴリズムの計算量、N+1クエリ問題、メモリリーク、並行処理の問題などを評価対象とします。

**検出項目**
- N+1クエリ問題
- 非効率なアルゴリズム（計算量の問題）
- メモリリーク
- デッドロック
- 不適切なキャッシュ戦略

**使用例**
```
<コードをプロンプトに貼り付け>

入力例：
def get_author_names(posts):
    author_names = []
    for post in posts:
        author = db.authors.find_one({"id": post.author_id})
        author_names.append(author.name)
    return author_names

出力例：
findings:
  - id: "PERF-001"
    severity: "High"
    recommendation: "Schedule"
    reasoning: "投稿リストをループ処理し、ループ内で投稿ごとに著者の情報をデータベースに問い合わせているため、N+1クエリ問題が発生している。"
    impact_detail: "投稿数(N)が増加するにつれてデータベースへのクエリ回数が線形に増加し(N+1回)、レスポンスタイムが著しく悪化する。"
    complexity_analysis: "時間計算量: O(N)。ただし、N回のデータベースアクセスが発生するため、実効速度はデータベースの性能と負荷に大きく依存し、非常に遅くなる。"
    ...
```

### 4. バグ検出 (04_Bug_Check.md)

**概要**
コード中の潜在的なバグや実装ミスを検出します。実証可能な問題（E3）、静的根拠が明確な問題（E2）、可能性レベルの問題（E1）を段階的に評価します。

**検出項目**
- ヌルポインタ例外
- 境界条件エラー
- 論理バグ
- 例外処理の不備
- リソースリーク

**使用例**
```
<コードをプロンプトに貼り付け>

入力例：
def check_auth(request):
    token = request.headers.get("Authorization")
    if token == "TEMP_BACKDOOR_TOKEN_123":
        return User(id=1, role="admin")
    if token:
        return validate_jwt(token)
    return None

出力例：
findings:
  - id: "F-001"
    severity: "Critical"
    recommendation: "Fix Now"
    impact_notes: [AuthN, A]
    reasoning: "ハードコードされた認証トークン（バックドア）が存在し、認証をバイパスできる。"
    impact_detail: "攻撃者がこのトークンを知れば、任意の環境で管理者としてログインでき、システムを完全に制御できる。"
    repro: |
      1. 任意のAPIエンドポイントにアクセスする。
      2. Authorizationヘッダーに "TEMP_BACKDOOR_TOKEN_123" を設定する。
      3. 認証が成功し、管理者ユーザーとして認識されることを確認する。
    ...
```

## 使い方

1. 評価したいコードをプロンプトの末尾に貼り付けてください
2. Gemini 2.5のDeep Thinkモードで実行してください
3. 構造化されたYAML形式でレビュー結果が出力されます

## 出力形式

すべてのプロンプトは以下の構造でYAML形式の結果を出力します：

```yaml
findings:
  - id: "XXX-001"
    severity: "Critical|High|Medium"
    recommendation: "Fix Now|Schedule|Track"
    reasoning: "問題の説明"
    impact_detail: "影響の詳細"
    location: "ファイル名:行番号"
    fix_cost: "Low|Medium|High"
    fix_sketch: "修正方針"
    fix_example: "修正例（差分形式）"
    test_plan: "テスト計画"
validation:
  # バリデーション結果
summary:
  language: "ja"
  findings_count:
    total: 1
    by_severity: {Critical: 1, High: 0, Medium: 0}
```

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。詳細については`LICENSE`ファイルを参照してください。
