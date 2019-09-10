#教師なし学習

#### tips
推定したクラスのクラスタ中心を取得する。
```
centroids = model.cluster_centers_
```
データをscatter plotして，KMeansなど教師なし学習の結果をプロットの色として表示して，クラスタの中心をプロットしてあげると見やすい絵を得ることができる。![KMenas例](file:///Users/wagatsuma/Downloads/ex_kmeans.svg)

## クラスタの評価

#### ラベルがついている場合
pandasの`pd.crosstab(df['labels'], df['species'])`で一目わかりやすい結果を得ることができる。
#### ラベルがついていない場合(KMeans)
inertiaを用いる。inertiaは各クラスター内の二乗誤差のこと。modelにfitした後，`model.inertia_`とすることで取得できる。小さければ良いわけではなく，クラスター数が大きくなると必然的に低い値になるのでプロットするなどしてちょうど良いスコアを得るのが望ましい。

#### 標準化
KMeansではアルゴリズムの性質上，分散の大きい特徴を重要な特徴として捉えてしまう。このような場合標準化をするのが望ましい。標準化は平均を0，分散を1にしており，値を0~1に収める正規化とは異なる。
`from sklearn.preprocessing import StandardScaler`

##階層的クラスタリング
SciPyの`linkage()`で階層的クラスタリングを行い，`dendrogram()`で結果を図示することができる。
