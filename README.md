---

# データ登録アプリケーションにて登録するデータ
ユーザー一覧
血液型
好きな教科

---

# 実装したコードの詳細な説明
今回実装したコードについて、データ形式やデータ型を含めて詳細に説明します。

---

## モデルの変更

`models/user.py` に以下の変更を加えました。

### 変更内容
```python
zodiac_sign = CharField()  # 星座フィールド追加
```

### 説明
- **データ型**: `CharField`
  - Peewee のフィールド型で、文字列を格納するために使用。
  - 星座（例: "牡羊座", "魚座"）を保存するために使用します。
- **データベースの構造**:
  - テーブル名: `user`
  - カラム名: `zodiac_sign`
  - 格納されるデータ形式: テキスト（文字列）

---

## 星座のリストの定義

`routes/user.py` 内で、星座のリストを定義しました。

### 変更内容
```python
ZODIAC_SIGNS = [
    "牡羊座", "牡牛座", "双子座", "蟹座", "獅子座",
    "乙女座", "天秤座", "蠍座", "射手座", "山羊座",
    "水瓶座", "魚座"
]
```

### 説明
- **データ形式**: リスト
- **用途**:
  - 星座をドロップダウン選択肢として提供するために使用。
- **リストの内容**:
  - 12星座が日本語で文字列として格納されています。

---

## ルートの変更

### `add` 関数

```python
zodiac_sign = request.form['zodiac_sign']  # 星座データを取得
User.create(name=name, age=age, zodiac_sign=zodiac_sign)
```

- **`request.form['zodiac_sign']`**:
  - フォームから送信された星座の値を取得します。
  - **データ型**: 文字列
- **`User.create`**:
  - `name`, `age`, `zodiac_sign` を受け取り、新しいユーザーをデータベースに保存します。
  - **データ型**:
    - `name`: 文字列
    - `age`: 整数
    - `zodiac_sign`: 文字列

---

### `edit` 関数

```python
user.zodiac_sign = request.form['zodiac_sign']  # 星座データを更新
```

- **`request.form['zodiac_sign']`**:
  - 編集フォームから送信された星座の値を取得します。
  - **データ型**: 文字列
- **`user.zodiac_sign`**:
  - 取得した星座の値をデータベースに保存します。

---

## HTML の変更

### ユーザー追加画面 (`user_add.html`)

```html
<label>星座:
    <select name="zodiac_sign" required>
        {% for sign in zodiac_signs %}
        <option value="{{ sign }}">{{ sign }}</option>
        {% endfor %}
    </select>
</label><br>
```

- **`<select>` タグ**:
  - 星座を選択するためのドロップダウンを提供します。
  - **データ形式**: ユーザーが選択した星座の値が文字列として送信されます。
- **`{% for sign in zodiac_signs %}`**:
  - 定義した `ZODIAC_SIGNS` リストをループして選択肢を生成します。

---

### ユーザー編集画面 (`user_edit.html`)

```html
<label>星座:
    <select name="zodiac_sign" required>
        {% for sign in zodiac_signs %}
        <option value="{{ sign }}" {% if user.zodiac_sign == sign %}selected{% endif %}>
            {{ sign }}
        </option>
        {% endfor %}
    </select>
</label><br>
```

- **機能**:
  - 編集画面で現在選択されている星座を表示し、変更が可能。
- **`{% if user.zodiac_sign == sign %}selected{% endif %}`**:
  - 現在の星座が選択された状態でドロップダウンを表示します。

---

### ユーザー一覧画面 (`user_list.html`)

```html
<thead>
    <tr>
        <th>&nbsp;</th>
        <th>名前</th>
        <th>年齢</th>
        <th>星座</th>
    </tr>
</thead>
<tbody>
    {% for user in items %}
    <tr>
        <td><a href="{{ url_for('user.edit', user_id=user.id) }}">編集</a></td>
        <td>{{ user.name }}</td>
        <td>{{ user.age }}</td>
        <td>{{ user.zodiac_sign }}</td>
    </tr>
    {% endfor %}
</tbody>
```

- **追加した列**:
  - 星座 (`<th>星座</th>`)
  - 保存された星座が表示されます。

---

## データベースのマイグレーション

### 方法 1: データベースを再作成
```bash
rm my_database.db
python
>>> from models import User
>>> User.create_table()
```

- **注意点**: この方法はデータをすべて削除するため、開発環境でのみ使用してください。

### 方法 2: マイグレーションスクリプトの実行
```python
from playhouse.migrate import SqliteMigrator, migrate
from models.db import db
from models.user import User

migrator = SqliteMigrator(db)

with db.atomic():
    migrate(
        migrator.add_column('user', 'zodiac_sign', User.zodiac_sign)
    )
```

- **効果**: データを保持したまま、`zodiac_sign` カラムを追加します。

---

## 全体のデータフロー

1. ユーザー追加画面で名前、年齢、星座を入力し、送信。
2. データベースに `name`, `age`, `zodiac_sign` が保存される。
3. 一覧画面で全ユーザーの名前、年齢、星座が表示される。
4. 編集画面で各フィールドを編集可能。
5. 更新後、変更内容がデータベースに保存される。

---

## 利用するデータ型のまとめ

| フィールド名       | データ型    | 説明                |
|--------------------|------------|---------------------|
| `name`            | 文字列     | ユーザーの名前       |
| `age`             | 整数       | ユーザーの年齢       |
| `zodiac_sign`     | 文字列     | ユーザーの星座       |

---

