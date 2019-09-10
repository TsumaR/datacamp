# Machine Learning with the Experts: School Budgets


##  復習的内容
今までの講座で習った操作について復習していたり，深掘りしていたり。

#### データをカテゴリー化する
文字列のまま処理すると，メモリの消費が激しく処理速度も落ちる。
```
categorize_label = lambda x: x.astype('category')
df[LABELS] = df[LABELS].apply(categorize_label, axis=0)
```

#### log loss
supervised_sklearn.mdで，スパムメールの例があった。1%だけがスパムメールのデータに対して全てがスパムメールでないと予想するモデルを組むと，accuracyは99%になってしまう。このようなことを避ける評価基準に *log loss* がある。log lossを用いれば，実際のラベルからどれくらい違ったかをみることができる。モデルが間違ったラベルを高い確信度で出力すると指数関数的に値が大きくなる。

## モデル作成

#### シンプルなモデルを作成　
データ解析を行う場合，最初にシンプルなモデルを作成することが望ましい。シンプルなモデルを適用することで，どのようなアプローチを取るべきかの指針が得られるためである。

## 自然言語処理
bag-of-words，１つずつ単語を処理する方法。文法や語順などを無視してしまうが単純でわかりやすい。
```
from sklearn.feature_extraction.text import CountVectorizer

#トークンパターンの設定
TOKENS_ALPHANUMERIC = '[A-Za-z0-9]+(?=\\s+)'

df.Position_Extra.fillna('', inplace=True)
vec_alphanumeric = CountVectorizer(token_pattern=TOKENS_ALPHANUMERIC)

vec_alphanumeric.fit(df.Position_Extra)
```
それだと順番を無視してしまうので，n個ごとに区切って処理する方法がある。

#### 数字と文字が混ざっているデータ　
文字列だけのデータと数字だけのデータに分ける。文字列だけのデータに対しては全てを統合しbag-of-wordsを行い，数字データにはimputer()処理する。その後統合してからone vs restでロジスティック回帰を行う。
```
from sklearn.preprocessing import FunctionTransformer

#文字列，文字列だけ取得
get_text_data = FunctionTransformer(lambda x: x['text'], validate=False)
get_numeric_data = FunctionTransformer(lambda x: x[['numeric', 'with_missing']], validate=False)

process_and_join_features = FeatureUnion(
            transformer_list = [
                ('numeric_features', Pipeline([
                    ('selector', get_numeric_data),
                    ('imputer', Imputer())
                ])),
                ('text_features', Pipeline([
                    ('selector', get_text_data),
                    ('vectorizer', CountVectorizer())
                ]))
             ]
        )

# Instantiate nested pipeline: pl
pl = Pipeline([
        ('union', process_and_join_features),
        ('clf', OneVsRestClassifier(LogisticRegression()))
    ])

# Fit pl to the training data
pl.fit(X_train, y_train)

# Compute and print accuracy
accuracy = pl.score(X_test, y_test)
print("\nAccuracy on sample data - all data: ", accuracy)
```
パイプラインを使うことでモデルの変更の際に必要な書き換えを一部にすることができるなどの利点もある。

### 精度の向上　
上のモデルでは，区切り文字を全て同等として扱った上に，1単語ずつの区切りでしか処理しなかった。言語処理の際に利用した`CountVectorizer()`に少し変更を加えるだけでモデルの精度ははるかに向上する。
`CountVectorizer(ngram_range=(1, 2))`では，bag-of-wordsで二連続の文字も１つの特徴量として扱うようにできる。　相互作用を考慮するなら`PolynomialFeatures`を用いるのが良い。これを用いると，２つの特徴（単語のペアや単語）が同時に存在するときの重要性を考慮に入れることができる。同様の処理ができるものの使い方の例を示すと，`SparseInteractions(degree=2)`のようになり，degreeを指定する必要がある。
`from sklearn.feature_extraction.text import HashingVectorizer`


詳しくは[jupyter_notebook](notebooks/1.0-full-model.ipynb)をみると良い。
