#教師なし学習

#### tips
推定したクラスのクラスタ中心を取得する。
```
centroids = model.cluster_centers_
```
データをscatter plotして，KMeansなど教師なし学習の結果をプロットの色として表示して，クラスタの中心をプロットしてあげると見やすい絵を得ることができる。![KMenas例](https://github.com/TsumaR/datacamp/blob/master/images/ex_kmeans.svg)

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
```
from scipy.cluster.hierarchy import linkage, dendrogram
mergings = linkage(samples, method='complete')

dendrogram(mergings,
           labels=varieties,
           leaf_rotation=90,
           leaf_font_size=6,
)
plt.show()
```
注意しないといけないのは，scipyの階層的クラスタリングはsklearnのパイプライン処理に対応していないので`Normalizer()`をパイプラインで使用できず以下のように処理する必要がある。
```
from sklearn.preprocessing import normalize
normalized_movements = normalize(movements)
```

#### t-SNE
詳しくはここでは説明しない，論文を読むこと。データに応じて変数を変更する必要があることなど注意点が多い。最近ではUMAPに取って代われれている(バイオインフォ界だけではないはず。)
```
from sklearn.manifold import TSNE

#モデルの作成。learning_rateは一般に50~200が良いことが多い。
model = TSNE(learning_rate=200)

tsne_features = model.fit_transform(samples)
xs = tsne_features[:,0]
ys = tsne_features[:,1]
```
## 次元削減(PCA)
実際のデータではノイズが多く含まれるデータを扱うことになるため，重要な要素だけを抽出し処理することで教師なし学習が可能になる。そのために最も用いられるのがPCAである。PCAは2ステップからなる。　
```
from sklearn.decomposition import PCA
model = PCA()
pca_features = mdoel.fit_transform(grains)
xs = pca_features[:,0]
ys = pca_features[:,1]
plt.scatter(xs, ys)
plt.show()

#ピアソンの相関係数を求める。
correlation, pvalue = pearsonr(xs, ys)
```
実際に観察するのに必要な次元数をPCAで決定するのが良い。Seuratのtutorialでもやっているので思い出すと良い。

*NMF* 非負値行列因子分解というのもある。0以上の値のmatrixに対して次元削減を行う。原点からの距離が重要な点となる。基本PCAで良さそう。
文書推薦などのカウントデータ（0以上)のスコアデータに対しては引き算による複雑性が生じず結果を扱いやすいので用いられることが多いよう。レコメンドではPCAが使われることはほとんどない模様。
遺伝子発現データもNMFから見えてくるものがありそうだけど。。
PCAと比較してどのように利点があるのか調べる必要がある。レコメンドとかと組み合わせたらツールの開発につながりそう。
類似はcosinse類似度により見ていた。
