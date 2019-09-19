#決定木
簡単なところは実行メインで記載している。詳しくははじパタなどを読むと良い。

# Classification and Regression Trees(CARTs)
木のいいところは，非線形のデータに用いることができ，正則化などが必要ないところである。下記に基本的な例を記す。CARTの決定木は2分木である。
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

### 回帰に用いる
回帰の場合，結果は離散値となる。その場合，下記のようにモデルを構築する，min_samples_leafでは，葉に最低含まれるトレーニングデータの割合を示している。
```
dt = DecisionTreeRegressor(max_depth=8,
             min_samples_leaf=0.13,
            random_state=3)
```

#### モデルの誤差　
モデルの予測誤差は，ノイズ，バリアンス，およびバイアスに分解することができる。バイアスはデータ自体に含まれているものなのでモデル構築の際に操作することができないが，バリアンスとバイアスは別である。バリアンスは予測結果の分散であり，バイアスは真の値と予測結果の差である。そのため，バリアンすとバイアスはトレードオフの関係にあり，バリアンスが小さい場合は学習不足，バイアスが小さい場合は過学習になっている可能性がある。そのため，それら２つがちょうどよくなるようにモデルを構築する必要があり誤差の正規化が必要である。
しかし，直接的にgeneralization errorを見ることはできないので，K-FoldCVなど，cross variationを用いて，トレーニングデータをvalidとtrainingに分けて検討することが一般である。

# ensemble learning
CARTsは非線形であり，正則化の必要もなく，わかりやすく簡単であった。しかし，過学習が起きやすかったり，1変数に大きく影響を受けすぎるなどの問題もある。そこで用いられることが多いのがアンサンブル学習である。アンサンブル学習では，個々に学習した複数の学習機を用いて得られた推測結果をマージして１つの推測結果として用いる手法である。一番簡単なのはVoting Classifierで，`from sklearn.ensemble import VotingClassifier`より使用できる。　
これでは，単純に多くの判別機によって支持された予想結果を真の予想結果として採用するが，通常はその際に複数のモデルそれぞれに重みを与えるのが一般的であり，単純にこれを用いることはほとんどない。　

### bagging
アンサンブル学習のうちの一種。上で説明した`Voting Classifier`が１つのトレーニングデータに対して複数のアルゴリズムを用いたのに対して，`bagging`では１つのアルゴリズムのみを用いる。
`bagging`では`boostrap`を用いている。`boostrap`とは簡単に言えば，重複を許した抽出(復元抽出)をすることである。（N個のデータから重複ありでN個のサブセットを作成することを繰り返す）`bagging`では`boostrap`により得られたトレーニングデータのサブセットに対して，同じアルゴリズムの複数の予測機により学習をし，それぞれの複数のモデルを作成する。このようにして作成された複数の予測機により得られた推測結果を，マージして全体の結果として出力するのが *bagging*　である。
```
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import BaggingClassifier
dt = DecisionTreeClassifier(random_state=1)
bc = BaggingClassifier(base_estimator=dt, n_estimators=50, random_state=1)
```
あとはfit，predictするなり好きに使える。　

#### out of bag evaluation (OOB evaluation)
ブーストラップでは，N個のデータから重複ありでN個のデータからなる集合を作成していた。この際，重複を許しているので当然利用されないデータが生じる。この利用されなかったデータをOOBといい，だいたい36%くらいになる。この使われなかったデータを用いてモデルを評価する方法をOOB evaluationという。その場合，
```
bc = BaggingClassifier(base_estimator=dt,
            n_estimators=50,
            oob_score=True,
            random_state=1)
```
のように明示することにより，学習予測をしたのちに，
```
bc.oob_score_
```
で，scoreを得ることができる。

### Random Forests
baggingでは，アルゴリズムは何を用いることも可能であった。一方でRandom Forestsでは決定木しか用いることができない。
![random forests](https://github.com/TsumaR/datacamp/blob/master/images/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-09-16%2019.37.27.png)
ランダムフォレストではまず，全体のデータセットからN個のブーストラップ標本を作成する。ここまではバギングと一緒である。ランダムフォレストはここで，各ブーストラップ標本において，全変数のうちからd個の特徴量をランダムに選択し，そのd個の特徴量を用いてそれぞれの決定木を成長させる。(このdの値は通常全特徴量の平方数である。)　
そのようにして得られたN個の決定木により得られた予想結果をmergeすることで全体の予想結果とするのがランファムフォレストである。
```
from sklearn.ensemble import RandomForestRegressor
rf = RandomForestRegressor(n_estimators=25,
            random_state=2)
rf.fit(X_train, y_train)
```
重要だった特徴量の表示
```
importances = pd.Series(data=rf.feature_importances_,
                        index= X_train.columns)
importances_sorted = importances.sort_values()

importances_sorted.plot(kind='barh', color='lightgreen')
plt.title('Features Importances')
plt.show()
```
## ブースティングの進化　

### アダブースト
2クラス問題の識別器であり，多クラスに対応するためには1対他識別器を構成する必要がある。　
先ほどバギングでは単純に，弱識別器を別々で作成し，多数決で予想結果を出力していた。一方，アダブーストは弱識別器の学習結果に従って学習データに重みが付与される。その結果のちに学習する識別器ほど誤りの多い学習データに集中して学習するようになる。　
弱識別器の数が大きすぎると過学習が生じるので，交差検証法などで選ぶ必要がある。
![adaboost](https://github.com/TsumaR/datacamp/blob/master/images/adaboost.png)
```
from sklearn.ensemble import AdaBoostClassifier
dt = DecisionTreeClassifier(max_depth=2, random_state=1)
ada = AdaBoostClassifier(base_estimator=dt, n_estimators=180, random_state=1)
ada.fit(X_train, y_train)
y_pred_proba = ada.predict_proba(X_test)[:,1]
ada_roc_auc = roc_auc_score(y_test, y_pred_proba)
```
ここまでははじパタに詳しく書いてある。

### Gradient Boosting (GB)
コンペで主に使われているいわゆる勾配ブースティング。アダブーストと異なり，学習データへの重みは微調整されず，各予測機は１つ前の予測機の残余誤差をラベルとして利用する。下の図はCARTに対して勾配ブースティングを行なっている様子を表している。
![GB](https://github.com/TsumaR/datacamp/blob/master/images/gradiant_boosting.png)
[参考](https://qiita.com/Quasi-quant2010/items/10f7ad4ed2e11004990f)
回帰の場合，
```
y(pred) = y1 +　ηr1 + … + ηrN
```
が予想結果として出力される。以下に簡単な例を示す。　
```
from sklearn.ensemble import GradientBoostingRegressor

gb = GradientBoostingRegressor(max_depth=4,
            n_estimators=200,
            random_state=2)
```

### Stochastic Gradient Boosting  (SGB)
勾配ブースティングでは各ステップで全てのデータを用いて学習したが，無作為非復元抽出した部分サンプルから各ステップのツリー構築を行うのが確率勾配ブースティングである。sklearnでは変数を変更するだけで使用することができる。
```
#subsample=xを加える

sgbr = GradientBoostingRegressor(max_depth=4,
            subsample=0.9,
            max_features=0.75,
            n_estimators=200,                                
            random_state=2)
```

## ハイパラメータチューニング
いろいろな方法がある。ハイパーパラメータの一覧は`.get_params()`で取得できる。ランダムフォレスト回帰の例。
```
from sklearn.model_selection import GridSearchCV
from sklearn.metrics import mean_squared_error

#ハイパーパラメータ候補の設定
params_rf = {
    'n_estimators':[100,350,500],
    'max_features':['log2','auto','sqrt'],
    'min_samples_leaf':[2,10,30]
}

grid_rf = GridSearchCV(estimator=rf,
                       param_grid=params_rf,
                       scoring='neg_mean_squared_error',
                       cv=3,
                       verbose=1,
                       n_jobs=-1)

grid_rf.fit(X_train, X_test)

best_model = grid_rf.best_estimator_
y_pred = best_model.predict(X_test)
rmse_test = MSE(y_test, y_pred)**(1/2)

print('Test RMSE of best model: {:.3f}'.format(rmse_test))                    
```
