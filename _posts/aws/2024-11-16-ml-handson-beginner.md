---
layout: post
title: "機械学習で画像を分類してみよう(AWS ハンズオン)"
date: 2024-11-16
categories: aws ai
---

- TOC
{:toc}

# 機械学習で画像を分類してみよう

下記ハンズオンを実施した記録

https://dcj71ciaiav4i.cloudfront.net/E33695C0-835B-11EB-8E80-138DCE1C124F/

# はじめに

OCR をするハンズオンみたい

# 機械学習環境のセットアップ

Amazon SageMaker: 機械学習の環境を簡単にセットアップできる

この辺りを効率よく処理する仕組みを提供してるみたい

![]({{site.baseurl}}/images/aws/ml-handson-beginner/ml_workflow.png)

2024/11/16 現在ではハンズオンの説明と少し場所が変わっていた

![]({{site.baseurl}}/images/aws/ml-handson-beginner/notebook.png)

conda_tensorflow_python36 がなかったので、conda_tensorflow2_p310 を選択

![]({{site.baseurl}}/images/aws/ml-handson-beginner/envchoose.png)

実行してみるとエラーが出た。。。

![]({{site.baseurl}}/images/aws/ml-handson-beginner/error.png)

chatGPT を参考にして環境を整えてみた

```shell
conda create -n tensorflow_p36 python=3.6
conda activate tensorflow_p36
pip install tensorflow==1.15

python -m ipykernel install --user --name tensorflow_p36 --display-name "Python (tensorflow_p36)"

```

ノートブックインスタンスを再起動すると tensorflow_p36 が出てきた

![]({{site.baseurl}}/images/aws/ml-handson-beginner/after_reboot.png)

と思ったらなんか消えた。。。よくわからん

そもそもソースに問題があるっぽい

変更前

```python
from skimage.io import ImageCollection,concatenate_images
from PIL import Image
import numpy as np
import pathlib

def load_image_with_label(f):
    label = pathlib.PurePath(f).parent.name
    return np.array(Image.open(f)), label

dataset = ImageCollection("./mnist_png/*/*/*.png", load_func=load_image_with_label)
np_dataset =  np.array(dataset)
X = concatenate_images(np_dataset[:,0])
y = np_dataset[:,1]
```

変更後

```python
from skimage.io import ImageCollection,concatenate_images
from PIL import Image
import numpy as np
import pathlib

def load_image_with_label(f):
    label = pathlib.PurePath(f).parent.name
    return np.array(Image.open(f)), label

dataset = ImageCollection("./mnist_png/*/*/*.png", load_func=load_image_with_label)
X = concatenate_images([item[0] for item in dataset])  # 各タプルの0番目を画像として抽出し結合
y = np.array([item[1] for item in dataset])
```

なんかうまく行ってるっぽい

# scikit-learn による機械学習

## k-近傍法

分類したデータに似ているデータを k 個みて最も多いラベルを採用する。

サンプルコードだと 0, 1 のラベルを 0 とし、2, 3 のラベルを 1 にしている。
そのため、1.1 は 0, 1, 2 が似ているデータとして判断され、0 と 1 のラベルである 0 が多数派として採用される。
predict_proba を使用した場合は確率で計算されるため、0 の確率が 0.66..., 1 の確率が 0.333...として出力される。

サンプルコード

```python
X = [[0], [1], [2], [3]]
y = [0, 0, 1, 1]
from sklearn.neighbors import KNeighborsClassifier
neigh = KNeighborsClassifier(n_neighbors=3)
neigh.fit(X, y)
print(neigh.predict([[1.1]]))
print(neigh.predict_proba([[0.9]]))
```

出力結果

```
[0]
[[0.66666667 0.33333333]]
```

実際にトレーニングデータを読み込ませてみた。X_train[0]は 6 のようだ

![]({{site.baseurl}}/images/aws/ml-handson-beginner/k_1.png)

答え合わせ。正解！！

![]({{site.baseurl}}/images/aws/ml-handson-beginner/k_2.png)

時間計測

```python
%%time
neigh.predict(X_test[0:100])
```

結果

```
CPU times: user 392 ms, sys: 109 ms, total: 501 ms
Wall time: 609 ms
```

## 決定木

こんな感じでルールを作成して振り分ける

![]({{site.baseurl}}/images/aws/ml-handson-beginner/ketteigi.png)

画像分類をする場合は「N 番目の画素が K 以上かどうか」という人間が理解しにくいルールができあがる

このコードは[0, 0]のラベルに 1 を[1, 1]のラベルに 1 を設定している。
そして[2, 2]の予測をしている。[2, 2]は近い[1, 1]のラベルがさいようされるため 1 になる。

```python
from sklearn import tree
X = [[0, 0], [1, 1]]
y = [0, 1]
clf = tree.DecisionTreeClassifier()
clf = clf.fit(X, y)
print(clf.predict([[2., 2.]]))
print(clf.predict_proba([[2., 2.]]))
```

実際にトレーニングデータを読み込ませてみた

```py
X = X_train
y = y_train
from sklearn import tree
clf = tree.DecisionTreeClassifier()
clf = clf.fit(X, y)
print(clf.predict([X_test[0]]))
print(clf.predict_proba([X_test[0]]))
```

結果

```
['6']
[[0. 0. 0. 0. 0. 0. 1. 0. 0. 0.]]
```

正解！

実行時間

```py
%%time
clf.predict(X_test[0:100])
```

結果

```
CPU times: user 1.77 ms, sys: 0 ns, total: 1.77 ms
Wall time: 2.5 ms
```

決定木の方が早い

## 機械学習の結果の評価

予測結果

```python
y_predict = clf.predict(X_test)
```

予測結果と正解を突き合わせる

```py
from sklearn.metrics import accuracy_score
accuracy_score(y_test, y_predict)
```

結果

```
0.8777
```

詳細な結果の出力

```py
from sklearn.metrics import confusion_matrix
confusion_matrix(y_test, y_predict)
```

出力

```
array([[ 907,    1,    8,    8,    3,   10,   20,    3,    9,    7],
       [   1, 1076,    7,    5,    2,    4,    2,    8,    9,    5],
       [  13,   11,  891,   19,   14,   11,   11,   15,   23,    9],
       [   9,   14,   32,  829,    7,   36,    4,   17,   29,   27],
       [   2,    2,   13,    5,  809,    9,   15,    9,   18,   55],
       [  19,    3,    7,   40,    8,  743,   20,    2,   22,   21],
       [  11,    7,   16,    4,   14,   19,  881,    1,   21,    7],
       [   4,    7,   17,    8,   12,    3,    3,  982,   12,   30],
       [  10,   19,   38,   23,   12,   24,   22,    8,  782,   23],
       [   8,    3,    9,   20,   53,   17,    0,   31,   24,  877]])
```

例えば、0 のうち正しく 0 と判定されたのが 907 個。0 なのに 1 と判定されたのが 1 個ある。ということを示す。

1 は比較的、正確な判定がされていそう。誤判定が多いものとしては、4 と 9, 3 と 5 などがあった。

## その他のアルゴリズム

Random Forest を使用した予測

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
clf = RandomForestClassifier(max_depth=2, random_state=0)
clf.fit(X, y)
```

結果

```
0.6283
```

低いやないかい！max_depth を大きくすると精度が上がるみたいなのでしてみる。

2 → 4 に変更

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
clf = RandomForestClassifier(max_depth=4, random_state=0)
clf.fit(X, y)
```

上がった！

```
0.8042
```

8 に変更すると 0.9267 まで上がり、10 だと 0.9481 になった！

10 のときの結果詳細はこんな感じ

```
array([[ 962,    0,    2,    0,    1,    1,    3,    0,    7,    0],
       [   0, 1096,    9,    1,    4,    1,    1,    1,    2,    4],
       [   5,    4,  962,    6,    7,    0,    7,   15,    9,    2],
       [   0,    5,   16,  927,    3,   12,    4,   12,   10,   15],
       [   2,    0,    2,    0,  891,    0,   10,    2,    4,   26],
       [   9,    6,    1,   18,    1,  830,    9,    0,    8,    3],
       [   5,    5,    0,    1,    3,   10,  953,    0,    4,    0],
       [   0,   11,   19,    1,   13,    0,    0, 1000,    5,   29],
       [   0,   12,    4,    7,    5,    6,    5,    1,  899,   22],
       [   4,    7,    4,   16,   21,    1,    1,   15,   12,  961]])
```

0 と 1 など全く誤判定しないものも出てきていた。

# 深層学習への入り口

## 深層学習への入り口

今回は 0~9 まで振り分けるのでこんな感じのニューラルネットワークになる
![]({{site.baseurl}}/images/aws/ml-handson-beginner/nn.png)

エラー出た。。。

```
import tensorflow as tf

model = tf.keras.models.Sequential([
  tf.keras.layers.Dense(10)
])

loss_fn = tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True)

model.compile(optimizer='adam',
              loss=loss_fn,
              metrics=['accuracy'])

model.fit(X_train, y_train, epochs=5)

---------------------------------------------------------------------------
ModuleNotFoundError                       Traceback (most recent call last)
Cell In[50], line 1
----> 1 import tensorflow as tf
      3 model = tf.keras.models.Sequential([
      4   tf.keras.layers.Dense(10)
      5 ])
      7 loss_fn = tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True)

ModuleNotFoundError: No module named 'tensorflow'
```

tensorflow がないようなのでインストール

```
!pip show tensorflow
!pip install tensorflow
```

再実施してみると実行できたがなんかいろいろ警告される。設定の競合が発生しているみたい。。。
（よくわからないから放置）

```
2024-11-17 16:55:54.219320: E external/local_xla/xla/stream_executor/cuda/cuda_fft.cc:477] Unable to register cuFFT factory: Attempting to register factory for plugin cuFFT when one has already been registered
WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
E0000 00:00:1731862554.244962    8139 cuda_dnn.cc:8310] Unable to register cuDNN factory: Attempting to register factory for plugin cuDNN when one has already been registered
E0000 00:00:1731862554.253613    8139 cuda_blas.cc:1418] Unable to register cuBLAS factory: Attempting to register factory for plugin cuBLAS when one has already been registered
2024-11-17 16:55:54.283925: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
To enable the following instructions: AVX2 AVX512F FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
2024-11-17 16:55:57.478907: E external/local_xla/xla/stream_executor/cuda/cuda_driver.cc:152] failed call to cuInit: INTERNAL: CUDA error: Failed call to cuInit: CUDA_ERROR_NO_DEVICE: no CUDA-capable device is detected
Epoch 1/5
2024-11-17 16:55:58.027508: W external/local_xla/xla/tsl/framework/cpu_allocator_impl.cc:83] Allocation of 188160000 exceeds 10% of free system memory.
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 4s 2ms/step - accuracy: 0.7690 - loss: 19.0956
Epoch 2/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 5s 2ms/step - accuracy: 0.8771 - loss: 6.0534
Epoch 3/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 3s 2ms/step - accuracy: 0.8837 - loss: 5.6435
Epoch 4/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 3s 2ms/step - accuracy: 0.8838 - loss: 5.5011
Epoch 5/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 5s 2ms/step - accuracy: 0.8872 - loss: 5.1775
<keras.src.callbacks.history.History at 0x7facb470d120>
```

この結果は 5 回繰り返して学習を行い、88.72%の正答率を学習データ内では記録していることをしめしている。繰り返しの回数を増やすと一般的に正答率は上がる。

モデルの評価

```python
predictions = model.predict(X_test)

from scipy.special import softmax
probability = softmax(predictions, axis = 1)
y_predict = np.argmax(probability, axis=1)

accuracy_score(y_predict, y_test)
```

結果は 0.8884 になっている。トレーニング中とほぼ同じ

kerase を使用した場合も同じ結果になった

```python
model.evaluate(X_test,  y_test, verbose=2)
```

[欠損値, 評価指標]という形式で出力されている

```
[5.392171859741211, 0.8884000182151794]
```

## 畳み込みニューラルネットワーク

画像認識は畳み込みを使用すると画像の輪郭などを取り出すことができる

```

/home/ec2-user/anaconda3/envs/python3/lib/python3.10/site-packages/keras/src/layers/convolutional/base_conv.py:107: UserWarning: Do not pass an `input_shape`/`input_dim` argument to a layer. When using Sequential models, prefer using an `Input(shape)` object as the first layer in the model instead.
  super().__init__(activity_regularizer=activity_regularizer, **kwargs)
Epoch 1/5
2024-11-17 17:23:26.463169: W external/local_xla/xla/tsl/framework/cpu_allocator_impl.cc:83] Allocation of 188160000 exceeds 10% of free system memory.
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 25s 13ms/step - accuracy: 0.8964 - loss: 2.8233
Epoch 2/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 25s 13ms/step - accuracy: 0.9814 - loss: 0.0616
Epoch 3/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 43s 14ms/step - accuracy: 0.9840 - loss: 0.0505
Epoch 4/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 36s 12ms/step - accuracy: 0.9879 - loss: 0.0388
Epoch 5/5
1875/1875 ━━━━━━━━━━━━━━━━━━━━ 24s 13ms/step - accuracy: 0.9900 - loss: 0.0300
```

普通のニューラルネットワークより時間はかかったが、精度は各段に上がった。

## カラー画像の分類

エラーがでたので一部書き換え

```python
from skimage.io import ImageCollection,concatenate_images
from PIL import Image
import numpy as np
import pathlib

labelmap = {'airplane':0,
            'automobile':1,
            'bird':2,
            'cat':3,
            'deer':4,
            'dog':5,
            'frog':6,
            'horse':7,
            'ship':8,
            'truck':9}

def load_image_with_label(f):
    label = pathlib.PurePath(f).parent.name
    return np.array(Image.open(f)), labelmap[label]

train_set = ImageCollection("./cifar10/train/*/*.png", load_func=load_image_with_label)
test_set = ImageCollection("./cifar10/test/*/*.png", load_func=load_image_with_label)

# この部分書き換え
X_train_cl = concatenate_images([item[0] for item in train_set])
y_train_cl = np.array([item[1] for item in train_set])
X_test_cl = concatenate_images([item[0] for item in test_set])
y_test_cl = np.array([item[1] for item in test_set])
```

畳み込みニューラルネットワークの実装
color 版なので input_shape は(縦, 横, 3)になる。この 3 は RGB。

```python
import tensorflow as tf

input_shape =(32,32,3)
model = tf.keras.models.Sequential([
    tf.keras.layers.Reshape(input_shape),
    tf.keras.layers.Conv2D(32, (3, 3), activation='relu', input_shape=input_shape),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(10)
    ])

loss_fn = tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True)

model.compile(optimizer='adam',
              loss=loss_fn,
              metrics=['accuracy'])

model.fit(X_train_cl, y_train_cl, epochs=5)
```

繰り返し 5 回で 71%

```
313/313 ━━━━━━━━━━━━━━━━━━━━ 8s 22ms/step - accuracy: 0.1606 - loss: 57.5797
Epoch 2/5
313/313 ━━━━━━━━━━━━━━━━━━━━ 8s 16ms/step - accuracy: 0.3423 - loss: 1.8909
Epoch 3/5
313/313 ━━━━━━━━━━━━━━━━━━━━ 5s 14ms/step - accuracy: 0.5050 - loss: 1.4385
Epoch 4/5
313/313 ━━━━━━━━━━━━━━━━━━━━ 4s 14ms/step - accuracy: 0.6363 - loss: 1.1086
Epoch 5/5
313/313 ━━━━━━━━━━━━━━━━━━━━ 7s 21ms/step - accuracy: 0.7103 - loss: 0.8742
```

テストデータで試してみると

```python
model.evaluate(X_test_cl,  y_test_cl, verbose=2)
```

結果

```
[3.8070476055145264, 0.3005000054836273]
```

低い！！30%しかない。データが多様になるとより高度なアルゴリズムやより多くのデータが必要にある。

Epoch 回数を 10 回に増やすとトレーニングデータでの制度は 89%まで上昇したが、テストデータでの結果には 0.6%程度の差がみられるだけだった。

# コスト

一日かけてやって1ドルくらいだった。