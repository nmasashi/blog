---
layout: post
title: 仕組みから学ぶ生成AI入門
date: 2025-12-01
categories: reading genAI
---

![image]({{site.baseurl}}/images/reading/genai_nyumon/bookcover.jpg)

この本読んだのでメモ

## 第 1 章 ディープラーニングの基礎知識

- **線形多項分類器**

線形関数でデータ分類する。数式はこんな感じ

$$
y = w_1x_1 + w_2x_2 + … + b
$$

サンプルソース

```python
model = models.Sequential(name='linear_model')
model.add(layers.Input(shape=(784,), name='input'))
model.add(layers.Dense(10, activation='softmax', name='softmax'))

model.summary()
```

出力

![image]({{site.baseurl}}/images/reading/genai_nyumon/linear_model.png)

$$
y = w_1x_1 + w_2x_2 + … w_784x_784 + b
$$

みたいな式が 10 個あるイメージ。だから Param は(784+1)\*10 個になる

- **多層ニューラルネットワーク**

線形多項分類器の全結合層を複数重ねたもの。
画像分類にもしようできるが、画像特化というわけではない

- 畳み込みフィルター

  - 畳み込みフィルターを多段に重ねることにより画像分類に有効
  - 例えば縦線を検出する用の畳み込みフィルターをかけることで縦線を抽出してその部分の画像だけ抜き出す

    → 結果として画像サイズが小さくなる

  - この畳み込みフィルターを逆にデータを流していくと画像生成が可能（きれいな画像にはならないきがする）

ソース例

```python
model1 = models.Sequential(name='conv_filter_model1')
model1.add(layers.Input(shape=(784,), name='input'))
model1.add(layers.Reshape((28, 28, 1), name='reshape'))
model1.add(layers.Conv2D(2, (5, 5), padding='same',
                         kernel_initializer=edge_filter,
                         bias_initializer=tf.constant_initializer(-0.2),
                         activation='relu', name='conv_filter'))

model1.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/conv_filter.png)

フィルターのサイズ 5\*5=25 にバイアスを足した 26 がフィルター一つのパラメータになる。それが 2 枚あるから 26\*2 で 52 が Param になる。

## 第 2 章 変分オートエンコーダによる画像生成

### 2.1.1 オートエンコーダと潜在空間

全結合層を用いたオートエンコーダのイメージ

```txt
入力画像
↓
全結合層（ノード数：256）
↓
全結合層（ノード数：128）
↓
全結合層（ノード数：16）← 潜在空間
↓
全結合層（ノード数：128）
↓
全結合層（ノード数：256）
↓
全結合層（ノード数：784）
↓
出力画像
```

ソース例

```python
model = models.Sequential(name='AutoEncoder')
model.add(layers.Input(shape=(784,), name='input'))
model.add(layers.Dense(256, activation='relu', name='feedforward1'))
model.add(layers.Dense(128, activation='relu', name='feedforward2'))
model.add(layers.Dense(16, activation='relu', name='feedforward3'))
model.add(layers.Dense(128, activation='relu', name='feedforward4'))
model.add(layers.Dense(256, activation='relu', name='feedforward5'))
model.add(layers.Dense(784, activation='sigmoid', name='output'))

model.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/autoencoder00.png)

- ノード数を減らして、その後ノード数を増やして入力データと同じような出力が得られるような学習を行う
- ノード数を一時的に減らしているため、全く同じ画像の出力は困難
- ノード数が一番少なくなる層を潜在空間と呼ぶ
- 潜在空間より前の層をエンコーダ、後ろの層をデコーダと呼ぶ

学習後の入出力結果（上が入力画像で下が出力画像）

![image]({{site.baseurl}}/images/reading/genai_nyumon/autoencoder01.png)

若干ぼやけているけどなんか再現できていそう

0~9 の数字を学習したので、それ以外を入力するとこんな感じになる

![image]({{site.baseurl}}/images/reading/genai_nyumon/autoencoder02.png)

このオートエンコーダのデコーダ部分を切り取って、潜在空間に対応する値をいれると画像出力モデルとして利用ができる。
しかしこのままだと潜在空間に何を入れたらいいのかはよくわからない。

### 2.1.2 転置畳み込みフィルターによる画像生成

転置畳み込み: 畳み込みの逆で、データサイズを大きくしていくイメージ

エンコーダコード

```python
encoder = models.Sequential(name='encoder')
encoder.add(layers.Input(shape=(32*32,), name='encoder_input'))
encoder.add(layers.Reshape((32, 32, 1), name='reshape'))
encoder.add(layers.Conv2D(32, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_filter1')) # (16, 16, 32)
encoder.add(layers.Conv2D(64, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_filter2')) # (8, 8, 64)
encoder.add(layers.Conv2D(128, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_filter3')) # (4, 4, 128)
encoder.add(layers.Flatten(name='flatten'))
encoder.add(layers.Dense(2, name='embedding_space'))

encoder.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/encoder.png)

32 \* 32 ピクセルの画像を畳み込みフィルターで畳み込みつつ最終的に 2 次元データに落とし込んでいる

デコーダコード

```python
decoder = models.Sequential(name='decoder')
decoder.add(layers.Input(shape=(2,), name='decoder_input'))
decoder.add(layers.Dense(4 * 4 * 128, name='expand'))
decoder.add(layers.Reshape((4, 4, 128), name='reshape'))
decoder.add(layers.Conv2DTranspose(64, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_transpose1')) # (8, 8, 64)
decoder.add(layers.Conv2DTranspose(32, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_transpose2')) # (16, 16, 32)
decoder.add(layers.Conv2DTranspose(1, (3, 3), strides=2, padding='same',
                activation='sigmoid', name='conv_transpose3')) # (32, 32, 1)
decoder.add(layers.Flatten(name='flatten'))

decoder.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/decoder.png)

転置畳み込みフィルターを使用して二次元データを 32\*32 ピクセルの画像に戻している

エンコーダとデコーダを組み合わせたもの

```python
model = models.Model(inputs=encoder.inputs[0],
                     outputs=decoder(encoder(encoder.inputs[0])),
                     name='CNN_AutoEncoder')
model.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/cnn_autoencoder.png)

潜在空間の様子を見てみるとこのようになっており、各データがクラスタを構成しているのがわかる

![image]({{site.baseurl}}/images/reading/genai_nyumon/cnn_ce_graph.png)

それぞれの座標で見てみるとこんな感じに判定されるみたい

![image]({{site.baseurl}}/images/reading/genai_nyumon/cnn_ce_graph02.png)

この2次元の潜在空間に値をいれると画像生成ができる。しかし、クラスターがまばらでありクラスター間もすきまがある。
このため、クラスター間の部分では複数画像が合わさった画像が生成されてしまっている。

### 変分オートエンコーダへの拡張

エンコーダから出た値をそのまま潜在空間に配置するのではなく、正規分布を使用しそれをもとにデコーダを学習させる。
その際に、学習させるデータを選ぶ関数をサンプラーと呼ばれるみたい

エンコーダのコード

```python
encoder = models.Sequential(name='encoder')
encoder.add(layers.Input(shape=(32*32,), name='encoder_input'))
encoder.add(layers.Reshape((32, 32, 1), name='reshape'))
encoder.add(layers.Conv2D(32, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_filter1')) # (16, 16, 32)
encoder.add(layers.Conv2D(64, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_filter2')) # (8, 8, 64)
encoder.add(layers.Conv2D(128, (3, 3), strides=2, padding='same',
                activation='relu', name='conv_filter3')) # (4, 4, 128)
encoder.add(layers.Flatten(name='flatten'))
encoder.add(layers.Dense(4, name='mean_and_log_var'))

encoder.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/vae_encoder.png)

サンプラーのコード

```python
def get_samples(x): # x: encoder output
    num_examples = tf.shape(x)[0]
    means, log_vars = x[:, 0:2], x[:, 2:4]
    std_samples = tf.random.normal(shape=(num_examples, 2))
    samples = means + tf.exp(0.5 * log_vars) * std_samples
    return samples

sampler = models.Sequential(name='sampler')
sampler.add(layers.Input(shape=(4,), name='sampler_input'))
sampler.add(layers.Lambda(get_samples, name='sampled_embedding'))

sampler.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/vae_sampler.png)

デコーダは同じ

こんな感じで合体させる。これに誤差関数を平均二乗誤差をとるようなものを設定して学習をする。
（この辺からよくわからんくなった。）

```python
model_inputs = encoder.inputs[0]
model_outputs = layers.Concatenate(name='prediction_with_mean_log_var')(
    [encoder(model_inputs), decoder(sampler(encoder(model_inputs)))])

model = models.Model(inputs=model_inputs, outputs=model_outputs,
                     name='Variational_AutoEncoder')
model.summary()
```

![image]({{site.baseurl}}/images/reading/genai_nyumon/vae.png)

結果はこんな感じ

![image]({{site.baseurl}}/images/reading/genai_nyumon/vae_result01.png)

変分オートエンコーダを使用する前はx軸が-40 ~ -程度だったが、今回は-4 ~ 4程度になっており、クラスターが均等に広がっていることがわかる。

シグモイド関数を使用して0 ~ 1で表現するとこんな感じ

![image]({{site.baseurl}}/images/reading/genai_nyumon/vae_result02.png)

以前のものより判別できる画像（複数要素が入っていない）が多い

![image]({{site.baseurl}}/images/reading/genai_nyumon/vae_result03.png)
