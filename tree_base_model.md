#決定木

### Classification and Regression Trees
木のいいところは，非線形のデータに用いることができ，正則化などが必要ないところである。下記に基本的な例を記す。
```
from sklearn.tree import DecisionTreeClassifier
dt = DecisionTreeClassifier(max_depth=6, random_state=SEED)
dt.fit(X_train, y_train)
y_pred = dt.predict(X_test)
acc = accuracy_score(y_pred, y_test)
```
根から始まり，葉で終わる。各ノードでは１つの特徴量(f)に対して，ある値(sp)と比較した時にどうかで，leftとrightに分けていく。このfとspは，そのノードの時点で得られる情報(IG)が最大になるものを選ぶようになっている。この際，IGの最大化の判断はエントロピーなどの考え方を用いる。上の例でエントロピーを用いる場合次のようにコードの一部を書き換える必要がある。
```
dt_entropy = DecisionTreeClassifier(max_depth=8, criterion='entropy', random_state=1)
```
ただし，得られる結果はデフォルトの`gini-index`と同様であり，`gini-index`の方が通常早いため，基本はデフォルトのままで良い。(ジニ係数)

#### 回帰に用いる
回帰の場合，結果は離散値となる。その場合，下記のようにモデルを構築する，min_samples_leafでは，葉に最低含まれるトレーニングデータの割合を示している。
```
dt = DecisionTreeRegressor(max_depth=8,
             min_samples_leaf=0.13,
            random_state=3)
```
