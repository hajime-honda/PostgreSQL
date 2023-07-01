# PostgreSQL
- PostgreSQLに関する個人的なメモです。
- ChatGPTに聞いている項目は確認する必要があります。

## window関数関係

### count(*) over()とは...(ChatGPT)
- 

## JSON関係

### jsonとjsonbの違い
公式ドキュメントから
> json型とjsonb型というデータ型は、ほとんど 同一の入力値セットを受け入れます。 現実的に主要な違いは効率です。 jsonデータ型は入力テキストの正確なコピーで格納し、処理関数を実行するたびに再解析する必要があります。 jsonbデータ型では、分解されたバイナリ形式で格納されます。 格納するときには変換のオーバーヘッドのため少し遅くなりますが、処理するときには、全く再解析が必要とされないので大幅に高速化されます。 また jsonb型の重要な利点はインデックスをサポートしていることです。

引用元：[8.14. JSONデータ型](https://www.postgresql.jp/document/14/html/datatype-json.html)

### json(or jsonb)型オブジェクトのアクセス方法

| 演算子 | 返り値の型 | 取得対象 |
|---|---|---|
| json -> text | json | keyのvalue |
| json ->> text | text | keyのvalue |
| json -> integer | json | arrayの要素 |
| json ->> integer | text | arrayの要素 |
| json #> text | json | pathのvalue |
| json #>> text | text | pathのvalue |
| jsonb[text] | jsonb | keyのvalue |

### ->, ->>

- 存在しないkeyを指定するとnullが返ってくる
- ->> でアクセスするとtext型で取得する

``` sql
select
  (`{"name" : "Bob"}`::json) -> 'name';

-- "Bob"

select
  (`{ "name" : "Bob" }`::json) ->'address';

-- null

select ('[1, "hoge", null]'::json)->0; 

-- 1
```

### #>, #>>
- #>は json/jsonb型を返し、#>>はtext型を返す。

``` sql
select ('{"address":{"state":"Tokyo"}}'::jsonb)#>'{"address"}';

-- {"state": "Tokyo"}
```

## 関数

### JSON_ARRAY_LENGTH 関数
- JSON 文字列の外部配列内の要素数を返します

#### 構文
  ``` sql
  json_array_length('JSON配列オブジェクト' [, エラーを返す代わりにnullを返すかどうか(boolean) ] ) 
  ```

#### 戻り値

  要素数(integer)

#### 例
  ``` sql
  select json_array_length('[11,12,13,{"f1":21,"f2":[25,26]},14]'); 

  -- json_array_length 
  -- -----------------
  -- 5
  ```

### quote_literal 関数
- 指定された文字列を引用付き文字列として返します。

#### 構文

``` sql
quote_literal(string)
```

#### 戻り値
- 入力文字列のデータ(char または varchar)と同じデータ型の文字列を返す

#### 例

``` sql
select quote_literal(catid), catname
from category
order by 1,2;

-- quote_literal |  catname
-- --------------+-----------
-- '1'           | MLB
-- '10'          | Jazz
-- '11'          | Classical
-- '2'           | NHL
-- '3'           | NFL
-- '4'           | NBA
-- '5'           | MLS
-- '6'           | Musicals
-- '7'           | Plays
-- '8'           | Opera
-- '9'           | Pop
-- (11 rows)
```

### json_to_record 関数(ChatGPT)
- JSONデータをテーブルのレコードに変換します。

#### 例

``` sql
SELECT
  *
FROM
  json_to_record('{"key1": "value1", "key2": "value2"}') AS t(key1 text, key2 text);

-- key1  |  key2
-- -------+-------
-- value1 | value2
```

### json_array_elements 関数(ChatGPT)
- JSON配列を展開して、各要素を単一の行として返す関数です。

### array_to_json 関数(ChatGPT)
- 配列をJSON形式に変換するために使用されます。

#### 例
``` sql
select
  array_to_json(ARRAY[1, 2, 3]);

-- [1, 2, 3]
```


### array_agg関数(ChatGPT)
- 特定のカラムの値を集約して配列にまとめるために使用されます

#### 例
``` sql
SELECT
  array_agg(column_name)
FROM
  table_name GROUP BY group_column;
```

### to_jsonb関数(ChatGPT)
- JSONB（バイナリJSON）形式に変換するために使用されます

## ストプロ関係

### どんな型があるか(ChatGPT)

1. スカラー型（Scalar Types）:
   - 整数型（integer, bigint, smallint）
   - 浮動小数点型（real, double precision）
   - 文字列型（character varying, text）
   - 日付と時刻型（date, time, timestamp, interval）
   - ブール型（boolean）
   - UUID型（uuid）
   - その他の数値型（numeric, decimal）
1. テーブル型（Table Types）:
   - テーブル型（table）
   - レコード型（record）
1. 配列型（Array Types）:
   - 整数配列（integer[]）
   - 文字列配列（text[]）
   - 日付配列（date[]）
   - その他の配列型（任意の基本データ型の配列）
1. ユーザー定義型（User-Defined Types）:
   - CREATE TYPEコマンドを使用してユーザーが定義したカスタムデータ型
1. 特殊な型:
   - カーソル型（cursor）
   - レコードセット型（recordset）

### 基本的な書き方(ChatGPT)

``` sql
CREATE FUNCTION function_name(arg1 data_type, arg2 data_type, ...)
  RETURNS return_data_type
AS $$
BEGIN
  -- 処理するコードをここに記述します
  -- ...
  RETURN return_value;
END;
$$ LANGUAGE plpgsql;
```

### SELECTした結果を返す方法(ChatGPT)

- RECORD型 または TABLE型を使用する方法がある

RECORD型 の場合
``` sql
CREATE FUNCTION function_name(arg1 data_type, arg2 data_type, ...)
  RETURNS SETOF RECORD
AS $$
DECLARE
  result RECORD;
BEGIN
  -- SELECT文を実行して結果を取得
  SELECT column1, column2, ...
  INTO result
  FROM table_name
  WHERE condition;

  RETURN NEXT result;
END;
$$ LANGUAGE plpgsql;
```

TABLE型 の場合
``` sql
CREATE FUNCTION function_name(arg1 data_type, arg2 data_type, ...)
  RETURNS TABLE (column1 data_type, column2 data_type, ...)
AS $$
BEGIN
  -- SELECT文を実行して結果を取得し、直接返す
  RETURN QUERY
  SELECT column1, column2, ...
  FROM table_name
  WHERE condition;
END;
$$ LANGUAGE plpgsql;
```

### EXECUTE 文はどういった命令か？(ChatGPT)

- 動的にSQL文を実行するために使用される命令です。これにより、実行時に動的なクエリを生成し、実行することができます。

``` sql
EXECUTE 'SELECT * FROM my_table WHERE id = $1' USING some_id;
```

### ループする方法(ChatGPT)

- 一般的な方法では「for」を使う

``` sql
CREATE FUNCTION loop_table(my_table my_table_type)
  RETURNS void
AS $$
DECLARE
  row my_table_type;
BEGIN
  FOR row IN SELECT * FROM my_table LOOP
    -- ループ内で行ごとの処理を行う
    -- row.column_name を使用して列の値にアクセス
    -- ...
  END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### エラーを発生させる方法(ChatGPT)

- RAISEステートメントを使用する

``` sql
CREATE FUNCTION my_function()
  RETURNS void
AS $$
BEGIN
  IF some_condition THEN
    RAISE EXCEPTION 'An error occurred.';
  END IF;

  -- 他の処理を続ける...
END;
$$ LANGUAGE plpgsql;
```

### デバッグ方法(ChatGPT)

- RAISE NOTICEステートメントの使用
- RAISE EXCEPTIONステートメントの使用
- SELECT ステートメントの使用
- デバッグツールの使用
- ログの監視

### RAISE NOTICE で変数の確認をする

``` sql
-- 変数の中身を表示する（「%」に変数の値がセットされます）
raise info '%' , 変数名;　

-- 例
raise info '%' , num1;        -- 変数num1の中身を表示する
raise info 'num1=%' , num1;   -- このように書くと「num1=XX」と表示し、見やすくなります
```

引用元：[【PostgreSQL】変数の値を画面に出す（raise）【デバッグ用にも】](https://postgresweb.com/post-2852)

### json化1(ChatGPT)

- jsonb_build_object関数を使用して行をJSONオブジェクトに変換し、jsonb_agg関数を使用してJSONオブジェクトの配列を作成できる

``` sql
CREATE FUNCTION convert_table_to_json_array()
  RETURNS jsonb
AS $$
DECLARE
  json_array jsonb;
BEGIN
  SELECT jsonb_agg(jsonb_build_object('column1', column1, 'column2', column2, ...))
  INTO json_array
  FROM my_table;

  RETURN json_array;
END;
$$ LANGUAGE plpgsql;
```

### json化2(ChatGPT)

- to_json関数を使用してTABLE型をJSON形式に変換することができる

``` sql
CREATE FUNCTION convert_table_to_json(my_table my_table_type)
  RETURNS json
AS $$
BEGIN
  RETURN to_json(my_table);
END;
$$ LANGUAGE plpgsql;
```

### フィルタリング処理作成例(ChatGPT)

``` sql
CREATE OR REPLACE FUNCTION build_filter_query(filter JSON)
RETURNS TEXT AS $$
DECLARE
    column_name TEXT;
    operator TEXT;
    filter_value TEXT;
    query TEXT := 'SELECT * FROM your_table WHERE 1=1';
BEGIN
    FOR filter_condition IN SELECT * FROM json_array_elements(filter) LOOP
        column_name := filter_condition->>'field';
        operator := filter_condition->>'operator';
        filter_value := filter_condition->>'value';

        IF operator = 'and' THEN
            query := query || ' AND ' || column_name || ' = ' || quote_literal(filter_value);
        ELSIF operator = 'or' THEN
            query := query || ' OR ' || column_name || ' = ' || quote_literal(filter_value);
        END IF;
    END LOOP;

    RETURN query;
END;
$$ LANGUAGE plpgsql;
```

### ページネーションの作成例(ChatGPT)

``` sql
CREATE FUNCTION get_paged_data(page_num integer, page_size integer)
  RETURNS TABLE (id integer, name text, age integer)
AS $$
BEGIN
  RETURN QUERY
    SELECT id, name, age
    FROM my_table
    ORDER BY id
    LIMIT page_size
    OFFSET (page_num - 1) * page_size;
END;
$$ LANGUAGE plpgsql;
```

## 参考サイト
- [PostgreSQLでjsonを扱う方法を色々調べてみた](https://dev.classmethod.jp/articles/postgresql-json-handling/)
- [PostgreSQLのjson/jsonb型](https://zenn.dev/snagasawa/articles/postgresql_json_and_jsonb_type)
- [JSON_ARRAY_LENGTH 関数](https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/JSON_ARRAY_LENGTH.html)
- [40.8. エラーとメッセージ](https://www.postgresql.jp/docs/9.4/plpgsql-errors-and-messages.html)
- [【PostgreSQL】変数の値を画面に出す（raise）【デバッグ用にも】](https://postgresweb.com/post-2852)
