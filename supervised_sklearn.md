# sckit-learnを使った教師あり学習  

## EDA(探索的データ分析)  
irisデータセットなどなら`pyplot`の`scatter_matrix()`などで一気に目視するのがわかりやすい。  
binaryファイルの場合，
```
plt.figure()
sns.countplot(x='education', hue='party', data=df, palette='RdBu')
plt.xticks([0,1], ['No', 'Yes'])
plt.show()
```  
のように，seabornのcountplotなどでカウント結果をプロットするのがわかりやすい。  
ヒートマップのオススメは，
`sns.heatmap(df.corr(), square=True, cmap='')`

## skleanの導入  
skleanでは，columnが特徴，rowがサンプルを示している必要がある。また，通常欠損値を受け付けない。
例えば，knnでn=6の判別機を作成し適応する場合は以下のようにする。
```
from sklearn.neighbors import KNeighborsClassifier  
knn = KNeighborsClassifier(n_neighbors=6)  
knn.fit(x_train, y_train)
#予想
y_pred = knn.predict(X_test)
#精度の表示
print(knn.score(X_test, y_test))
```  
のようになる。nが小さすぎると過学習を引き起こす。

*めも*
通常のnumpy array(139,)を，たて一列に表示させるには`.reshape(-1,1)`でいける。

### trainデータとテストデータを分ける  
#### 単純に
```
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.3, random_state=42)
```  

##  線形回帰
#### 最小二乗法
`from sklearn.linear_model import LinearRegression`

### 正則化  
#### リッジ回帰  
正則化（制限を設ける）することで過学習を防いでいる。説明変数間に相関が強く多重共線性を防ぎたいときに有効。通常のSGD回帰のw,bへの重みづけに加えてL2ノルム（係数のユークリッド長）のペナルティを誤差に加えている。  
`from sklearn.linear_model import Ridge, RidgeCV`  

#### ラッソ回帰
L1ノルムを加えることで正則化している。係数の絶対値に制限を設けている。
`from sklearn.linear_model import Lasso, LassoCV`  
*大事な係数の値を大きくし，重要でない因子の係数を0にすることを自動でやってくれる点で非常に優れたアルゴリズムである。* しかし，リッジ回帰もラッソ回帰も正則化する際の係数を自分で設定する必要がある。

#### 平均二乗誤差(MSE)
*線形回帰モデルの性能を数値化する効果的な手法の一つで、実際の値とモデルによる予測値との誤差の平均値です。*
`rmse = np.sqrt(mean_squared_error(y_test, y_pred))`のようにする。
平均との差で行なったやつで割ると（ルート取る前），決定係数になる。  

## モデルの評価  
1%がエラーで99%が正常のデータに対して，分類を行った結果全てを正常と判断してしまったとしても，その精度は99%となってしまう。偽陽性などを考慮してモデルを評価，構築することが重要となってくる。　　
それらを確認する際には  
```
from sklearn.metrics import classification_report, confusion_matrix
print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```
のようにする。  
ここでロジスティック回帰とROC curveの話が出てきた，共に2値問題に対して用いるものである。ロジスティック回帰はシグモイド関数を用いた二値問題の回帰分析の手法。 *詳しくははじパタを復習！*   
ROC曲線は，thresholdsを変えた時に，横軸に偽陽性率，縦軸に陽性率でプロットした結果である。定義からもわかるが二値問題に対して使用するものである。間違って正しいと認識してしまうことが無い，理想的なモデルでは「のように，偽陽性0で陽性率100になる。  
```
from sklearn.metrics import roc_curve
y_pred_prob = logreg.predict_proba(X_test)[:,1]
fpr, tpr, thresholds = roc_curve(y_test, y_pred_prob)
# Plot ROC curve
plt.plot([0, 1], [0, 1], 'k--')
plt.plot(fpr, tpr)
plt.show()
```
#### AUC  
ROC曲線では「　のような形になるのが最高のモデルであった。そのため，ROC曲線の下の部分の面積が大きいほど良いモデルということになり，この下の面積のことを *AUC* という。  
`from sklearn.metrics import roc_auc_score`
`cv_auc = cross_val_score(logreg, X_test, y_test, cv=5, scoring='roc_auc')`

#### ハイパーパラメター  
kNNや，ラッソ回帰などでは，最適化のために自分で設定する必要のあるパラメタが存在し，それらはハイパーパラメターと呼ばれる。これらを最適化するためにまず考えられるのは，cross variationで様々なパターンを試し最適のものを決定する方法である。この操作を行う場合，scikit learnでは`GridSearchCV`が便利であり，複数のハイパーパラメタを同時に組み合わせて比較することも可能である。  
```
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import GridSearchCV
c_space = np.logspace(-5, 8, 15)
param_grid = {'C': c_space}
logreg = LogisticRegression()
logreg_cv = GridSearchCV(logreg, param_grid, cv=5)
logreg_cv.fit(X, y)
# Print the tuned parameters and score
print("Tuned Logistic Regression Parameters: {}".format(logreg_cv.best_params_))
print("Best score is {}".format(logreg_cv.best_score_))
```
注意として，ロジスティック回帰はL1，L2ノルムを決定するペナルティを設定するハイパラメタも存在する。`param_grid = {'C': s_space, 'penalty': ['l1', 'l2']}`
ただし，`GridSearchCV`は計算量が多くなってしまうことがあるので，その場合は`RandomizedSearchCV`を使うのが良い  

## 実際のデータ  
実際のデータを処理する場合，多くの欠損値やカテゴリー変数が含まれており，sklearnが処理できない形をしていることが多いため，事前処理をすることが重要になってくる。

#### カテゴリー変数
例えば，カテゴリー変数をダミー変数に変更するにはpandasの`pd.get_dammies()`が便利。`pd.get_dummies(df, drop_first=True)`で必要ないダミー変数の列を作成しないことができる。  

#### 欠損値の扱い
実際のデータだと欠損値が0や9999だったりするため，sklearnで簡単に処理するためにはまず，それらを`NaN`にすることが望ましい。 `df[df == '?'] = np.nan`のように
また，欠損値を全て除くなら`.dropna()`などがあるが，非欠損値のなかで平均を取りそれを欠損値に当てはめる方法もある。cvで最適パラメタを探しながら，欠損値処理も含めパイプラインで処理する例e
```
from sklearn.preprocessing import Imputer
from sklearn.pipeline import Pipeline

imp = Imputer(missing_values='NaN', strategy='most_frequent', axis=0)
clf = SVC()

#パイプラインで実行ste
steps = [('imputation', imp),
        ('SVM', clf)]
pipeline = Pipeline(steps)

parameters = {'SVM__C':[1, 10, 100],
              'SVM__gamma':[0.1, 0.01]}

cv = GridSearchCV(pipeline, param_grid=parameters)
y_pred = cv.predict(X_test)

#確認X,te
print("Accuracy: {}".format(cv.score(X_test, y_test)))
print(classification_report(y_test, y_pred))
print("Tuned Model Parameters: {}".format(cv.best_params_))
```
