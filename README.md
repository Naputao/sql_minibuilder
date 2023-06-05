~~目前只需要改commands/DB.py就行了~~

commands中的执行功能都差不多写完了，tokenize已完成，AST写了一点

可以完善AST，也可以实现AST与具体功能的结合

自己实现的功能先自己建个分支比较好

# 任务

## SQL解释器

### 1. 目标完成情况

目标： 

#### 1.1 对单串命令文本的token化

文件：（`tokenizer.py`）

完成情况：

- [x] 总体

输入：
```
"""
SELECT id, name, this
FROM table1
WHERE id = 1 AND this < 2.3;
"""

```

输出：
```
[(Token.Keyword.DML, 'SELECT'), (Token.Text.Whitespace, ' '), (Token.Name, 'id'), (Token.Punctuation, ','), (Token.Text.Whitespace, ' '), (Token.Name, 'name'), (Token.Punctuation, ','), (Token.Text.Whitespace, ' '), (Token.Name, 'this'), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Keyword, 'FROM'), (Token.Text.Whitespace, ' '), (Token.Keyword, 'table1'), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Text.Whitespace, ' '), (Token.Keyword, 'WHERE'), (Token.Text.Whitespace, ' '), (Token.Name, 'id'), (Token.Text.Whitespace, ' '), (Token.Operator.Comparison, '='), (Token.Text.Whitespace, ' '), (Token.Literal.Number.Integer, '1'), (Token.Text.Whitespace, ' '), (Token.Keyword, 'AND'), (Token.Text.Whitespace, ' '), (Token.Name, 'this'), (Token.Text.Whitespace, ' '), (Token.Operator.Comparison, '<'), (Token.Text.Whitespace, ' '), (Token.Literal.Number.Float, '2.3'), (Token.Punctuation, ';')]
```

#### 1.2 在token基础上，抽象出AST

文件：`AST_builder.py`

完成情况：

- [x] 对SELECT的实现

- [X] 对FROM的实现

- [x] 对WHERE的基础实现

- [ ] 对WHERE中AND、OR的正确顺序判断

- [ ] 对CREATE的实现

- [ ] 对UPDATE的实现

- [ ] 对主键`PRIMARY`的实现

- [ ] 对非空`NOT NULL`的实现


**目前已部分完成，只到能够解析查询命令（SELECT）的地方**

输入：
```
SELECT id, name, this
FROM table1
WHERE id = 1 AND this < 2.3;
```

期望输出：
```
statements: [
    - SelectStmt {
        type: "select_stmt"
        - clauses: [
            - SelectClause {
                type: "select_clause"
                - columns: [
                    - "id"
                    - "name"
                    - "this"
                ]
            }
            - FromClause {
                type: "from_clause"
                - tables: [
                    - "table1"
                ]
            }
            - WhereClause {
                type: "where_clause"
                - exprs: [
                    operator: "AND"
                    left: exprs[
                        operator: "="
                        left: "id"
                        right: 1
                    ]
                    right: exprs[
                        operator: "<"
                        left: "this"
                        right: 2.3
                    ]
                ]
            }
        ]
    }
]
```

**实际输出**：
（可以看到`WHERE`被利用了两次，一次是作为`CLAUSE`，一次是作为`EXPRESSION`）
```
SELECT                                             level:AST_KEYWORDS.CLAUSE
-- id
-- name
-- this
FROM                                               level:AST_KEYWORDS.CLAUSE
-- table1
WHERE                                              level:AST_KEYWORDS.CLAUSE
--WHERE                                            level:AST_KEYWORDS.EXPRESSION
---- {'left': 'id', 'op': '=', 'right': 1}
--AND                                              level:AST_KEYWORDS.EXPRESSION
---- {'left': 'this', 'op': '<', 'right': 2.3}
```

**用语解释**：

- statement: 需要执行的一条语句，以`;`结尾

- clause: 语句中的子句，一般一个查询的SQL语句可以被分为`select caluse`，`from clause`, `where clause`




#### 1.3 在AST基础上，实现对`sql_commands`的调用，完成语句

文件：`sql_commands/main.py`（示例用，之后可能会改）

- [x] 完成示例文件，演示如何结合AST与实现的`sql_commands`，通过FROM语句获得databse中的表

- [ ] 完成AST与执行代码关于WHERE的结合

- [ ] 完成AST与执行代码关于SELECT的结合

- [ ] 更多

### 2. 目标总览

## SQL代码执行
### 1. 方法完成情况

#### 交给XJW

- [x] delete方法（不完善，删去的对象没有保存于本地）

- [x] drop方法（有待争议，是丢掉整个对象，还是丢掉表中的所有行，但保留列属性？）

#### 交给CXL

- [x] select方法

- [x] where方法

#### 交给YS

- [x] insert方法

- [x] update方法

- [x] create方法

#### 交给LZH

#### 好像差不多都完成了，有问题之后再改


### 2. 方法总览

**delete**

```python
def delete(
    self,
    table: pd.DataFrame,
    del_rows: list[int]
) -> bool:
```


**drop**

是drop整张表，删除其对象，还是

```python
def drop(
    self,
    table: pd.DataFrame,
) -> bool:
```

**create**

很明显这个还需要更多属性，例如，主键的`PRIMARY`，序列化的`SERIAL`，非空的`NOT NULL`

```python
def delete(
    self,
    table: pd.DataFrame,
    attributes: list[str],
    types: list[str]
) -> bool:
```


**insert**

```python
def insert(
    self,
    table: pd.DataFrame,
    attributes: list[str],
    values: list
) -> bool:
```


**update**

```python
def update(
    self,
    table: pd.DataFrame,
    update_rows: list[int],
    attributes: list[str],
    values: list
) -> bool:
```


**select**
```python
def select(
    self,
    table: pd.DataFrame,
    sel_columns: list[int],
    sel_rows: list[int]
) -> pd.DataFrame:
```

**where**
```python
def where(
    self,
    table: pd.DataFrame,
    attribute: str,
    value,
    operand
) -> list[int]:
```
