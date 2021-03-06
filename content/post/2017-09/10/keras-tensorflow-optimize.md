+++
title = "TensorFlow を Intel CPU で最適化してみた"
description = "TensorFlow を Intel CPU で最適化してみた"
date = "2017-09-10T20:21:19+09:00"
categories = ["Programming"]
tags = ["Python", "TensorFlow", "Machine Learning"]
archives = ["2017-09"]
url = "post/2017-09/10/keras-tensorflow-optimize"
thumbnail = "/img/2017-09/10/mnist.png"
+++

自分のローカル環境(MacBook)で少しでも実行速度を改善しようとしたときの
メモです。

<!--more-->

### はじめに

自分のローカル環境(MacBook 12inch, 2016, SkyLake CPU) は決して速いマシンではないです。
ファンレスなので熱くなりやすいし、そもそも TDP 5W クラスなのであまり無理はできません。

それでもやはりローカル環境もある程度使うことを考えておかないと、ちょっとしたことを
試すには不便です。

というのも、Keras + TensorFlow に最初に手をつけるであろうサンプル (`keras/examples/mnist_cnn.py`)
を物は試しで動かしてみたところ、77分程かかってしまいました。

### まずは通常のインストール

自分の環境は、以下のようになっています。

```
bash$ sw_vers
ProductName:    Mac OS X
ProductVersion: 10.12.6
BuildVersion:   16G29
bash$ python --version
Python 3.6.2
bash$ 
```

この環境に**素の**Keras+TensorFlow を仕込んでみます。環境を壊したくないので venv
を使います。普通に`pip` でインストールしていきます。

```
bash$ python3 -m venv keras-example
bash$ cd keras-example/
bash$ . bin/activate
(keras-example) bash$ pip install pillow
(keras-example) bash$ pip install h5py
(keras-example) bash$ pip install matplotlib
(keras-example) bash$ pip install keras
(keras-example) bash$ pip list
Package         Version
--------------- -------
cycler          0.10.0
h5py            2.7.0
Keras           2.0.6
matplotlib      2.0.2
numpy           1.13.1
olefile         0.44
Pillow          4.2.1
pip             9.0.1
pyparsing       2.2.0
python-dateutil 2.6.1
pytz            2017.2
PyYAML          3.12
scipy           0.19.1
setuptools      28.8.0
six             1.10.0
Theano          0.9.0
(keras-example) bash$ pip install tensorflow
(keras-example) bash$ pip list
Package           Version
----------------- ---------
backports.weakref 1.0rc1
bleach            1.5.0
cycler            0.10.0
h5py              2.7.0
html5lib          0.9999999
Keras             2.0.6
Markdown          2.6.8
matplotlib        2.0.2
numpy             1.13.1
olefile           0.44
Pillow            4.2.1
pip               9.0.1
protobuf          3.3.0
pyparsing         2.2.0
python-dateutil   2.6.1
pytz              2017.2
PyYAML            3.12
scipy             0.19.1
setuptools        28.8.0
six               1.10.0
tensorflow        1.2.1
Theano            0.9.0
Werkzeug          0.12.2
wheel             0.29.0
(keras-example) bash$ 
```

ここまでで必要なものは一式入ったはずです。

### サンプルの実行

Keras のサンプルは、[Keras のソース](https://github.com/fchollet/keras/) にありますので、
`git clone`します。

```
(keras-example) bash$ git clone https://github.com/fchollet/keras.git
  :
(keras-example) bash$ cd keras/
(keras-example) bash$ cd examples/
```

さて、examples に mnist_cnn.py がありますのでいよいよ実行です。
最後に所要時間を見たいので、`time`で計測します。

```
(keras-example) bash$ time python mnist_cnn.py
Using TensorFlow backend.
Downloading data from https://s3.amazonaws.com/img-datasets/mnist.npz
11386880/11490434 [============================>.] - ETA: 0sx_train shape: (60000, 28, 28, 1)
60000 train samples
10000 test samples
Train on 60000 samples, validate on 10000 samples
Epoch 1/12
2017-08-05 20:08:03.226901: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
2017-08-05 20:08:03.226944: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
2017-08-05 20:08:03.226956: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speed up CPU computations.
2017-08-05 20:08:03.226967: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
60000/60000 [==============================] - 339s - loss: 0.3367 - acc: 0.8976 - val_loss: 0.0840 - val_acc: 0.9751
Epoch 2/12
60000/60000 [==============================] - 375s - loss: 0.1182 - acc: 0.9654 - val_loss: 0.0537 - val_acc: 0.9828
Epoch 3/12
60000/60000 [==============================] - 293s - loss: 0.0907 - acc: 0.9730 - val_loss: 0.0463 - val_acc: 0.9858
Epoch 4/12
60000/60000 [==============================] - 367s - loss: 0.0732 - acc: 0.9786 - val_loss: 0.0386 - val_acc: 0.9877
Epoch 5/12
60000/60000 [==============================] - 431s - loss: 0.0645 - acc: 0.9812 - val_loss: 0.0341 - val_acc: 0.9888
Epoch 6/12
60000/60000 [==============================] - 399s - loss: 0.0574 - acc: 0.9830 - val_loss: 0.0310 - val_acc: 0.9893
Epoch 7/12
60000/60000 [==============================] - 433s - loss: 0.0530 - acc: 0.9845 - val_loss: 0.0317 - val_acc: 0.9894
Epoch 8/12
60000/60000 [==============================] - 367s - loss: 0.0472 - acc: 0.9862 - val_loss: 0.0312 - val_acc: 0.9897
Epoch 9/12
60000/60000 [==============================] - 389s - loss: 0.0456 - acc: 0.9866 - val_loss: 0.0331 - val_acc: 0.9890
Epoch 10/12
60000/60000 [==============================] - 460s - loss: 0.0415 - acc: 0.9873 - val_loss: 0.0296 - val_acc: 0.9897
Epoch 11/12
60000/60000 [==============================] - 417s - loss: 0.0383 - acc: 0.9882 - val_loss: 0.0292 - val_acc: 0.9907
Epoch 12/12
60000/60000 [==============================] - 315s - loss: 0.0376 - acc: 0.9885 - val_loss: 0.0269 - val_acc: 0.9910
Test loss: 0.0268840175608
Test accuracy: 0.991

real    77m16.629s
user    203m6.948s
sys     21m3.707s
#--------------------------------------------------------------------------------
(keras-example) bash$ brew install bazel swig
```

77分かかりました。1epoch あたり 300～400秒前後といったところでしょうか。

ここで気になるのは、

```
2017-08-05 20:08:03.226901: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
```

というメッセージです。上記は「SSE4.2向けにコンパイルされてないけど、CPUには SSE4.2があるよ」といった
メッセージなので、TensorFlow をコンパイルし直せば速くなりそうな気がします。

同じようなメッセージがいくつかありましたのでピックアップすると、

* SSE4.2
* AVX
* AVX2
* FMA

の４つがありました。これらをどうにかしましょう。

### TensorFlow の再コンパイル

TensorFlow を自分でコンパイルするためには、`bazel`、`swig`、と TensorFlow のソースが
必要となります。まずは下準備から。

```
(keras-example) bash$ brew install bazel swig
(keras-example) bash$ git clone https://github.com/tensorflow/tensorflow.git
(keras-example) bash$ cd tensorflow
(keras-example) bash$ pip list | grep tensorflow
tensorflow        1.2.1
(keras-example) bash$ git checkout v1.2.1
Note: checking out 'v1.2.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at b4957ffc6... Merge pull request #11156 from av8ramit/1.2.1
(keras-example) bash$ 
```

ここまでで準備は OK。次に `configure` を走らせます。いくつか入力を求められますが、
MacBook 12inch の場合は全部 N ですね(OpenCL って macOS でサポートされているのでは？
と思い試したもののエラーでビルドできませんでした)。

```
(keras-example) bash$ ./configure
Please specify the location of python. [Default is /Users/masao/work/keras-example/bin/python]:
Found possible Python library paths:
  /Users/masao/work/keras-example/lib/python3.6/site-packages
Please input the desired Python library path to use.  Default is [/Users/masao/work/keras-example/lib/python3.6/site-packages]

Using python library path: /Users/masao/work/keras-example/lib/python3.6/site-packages
Do you wish to build TensorFlow with MKL support? [y/N]
No MKL support will be enabled for TensorFlow
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:
Do you wish to build TensorFlow with Google Cloud Platform support? [y/N]
No Google Cloud Platform support will be enabled for TensorFlow
Do you wish to build TensorFlow with Hadoop File System support? [y/N]
No Hadoop File System support will be enabled for TensorFlow
Do you wish to build TensorFlow with the XLA just-in-time compiler (experimental)? [y/N]
No XLA support will be enabled for TensorFlow
Do you wish to build TensorFlow with VERBS support? [y/N]
No VERBS support will be enabled for TensorFlow
Do you wish to build TensorFlow with OpenCL support? [y/N]
No OpenCL support will be enabled for TensorFlow
Do you wish to build TensorFlow with CUDA support? [y/N]
No CUDA support will be enabled for TensorFlow
Extracting Bazel installation...
...................
INFO: Starting clean (this may take a while). Consider using --async if the clean takes more than several minutes.
Configuration finished
(keras-example) bash$ 
```

configure が終わればビルド、ですが、ここで `bazel` が登場します。またオプション指定をここで
行います。
上述の SSE4.2、AVX、AVX2、FMA、の４つをサポートするために、`bazel build` に `--copt=`として
オプション指定していきます。

```
bazel build -c opt --copt=-msse4.2 --copt=-mavx --copt=-mavx2 --copt=-mfma //tensorflow/tools/pip_package:build_pip_package
```

で、これを実行します。がこれが非常に遅い。マシンが熱くなってしまったせいもありますが、
速いマシンでバイナリを作ってしまうのが良さそうです。

```
(keras-example) bash$ bazel build -c opt --copt=-msse4.2 --copt=-mavx --copt=-mavx2 --copt=-mfma //tensorflow/tools/pip_package:build_pip_package
(keras-example) bash$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
Target //tensorflow/tools/pip_package:build_pip_package up-to-date:
  bazel-bin/tensorflow/tools/pip_package/build_pip_package


INFO: Elapsed time: 5944.145s, Critical Path: 243.19s

real    99m4.470s
user    0m0.350s
sys     0m0.409s
(keras-example) bash$ 
```

99分って...。ビルドが終わったら、これを pip のパッケージにします。

```
(keras-example) bash$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
2017年 8月 6日 日曜日 00時51分34秒 JST : === Using tmpdir: /var/folders/h4/41q8jgjn7lbbyl8jz50sz0jh0000gn/T/tmp.XXXXXXXXXX.n9wL8WD9
~/work/keras-example/tensorflow/bazel-bin/tensorflow/tools/pip_package/build_pip_package.runfiles ~/work/keras-example/tensorflow
~/work/keras-example/tensorflow
/var/folders/h4/41q8jgjn7lbbyl8jz50sz0jh0000gn/T/tmp.XXXXXXXXXX.n9wL8WD9 ~/work/keras-example/tensorflow
2017年 8月 6日 日曜日 00時51分40秒 JST : === Building wheel
warning: no files found matching '*.dll' under directory '*'
warning: no files found matching '*.lib' under directory '*'
~/work/keras-example/tensorflow
2017年 8月 6日 日曜日 00時52分00秒 JST : === Output wheel file is in: /tmp/tensorflow_pkg
(keras-example) bash$ ls -l /tmp/tensorflow_pkg/
total 61952
-rw-r--r--  1 masao  wheel  31719305  8  6 00:52 tensorflow-1.2.1-cp36-cp36m-macosx_10_12_x86_64.whl
(keras-example) bash$ 
```

あとは今はいっている `tensorflow`を置き換える(uninstall して作成済パッケージをinstall)だけです。

```
(keras-example) bash$ pip uninstall -y tensorflow
(keras-example) bash$
(keras-example) bash$ pip install /tmp/tensorflow_pkg/tensorflow-1.2.1-cp36-cp36m-macosx_10_12_x86_64.whl
Processing /tmp/tensorflow_pkg/tensorflow-1.2.1-cp36-cp36m-macosx_10_12_x86_64.whl
Requirement already satisfied: six>=1.10.0 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: bleach==1.5.0 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: backports.weakref==1.0rc1 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: html5lib==0.9999999 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: wheel>=0.26 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: markdown>=2.6.8 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: werkzeug>=0.11.10 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: numpy>=1.11.0 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: protobuf>=3.2.0 in /Users/masao/work/keras-example/lib/python3.6/site-packages (from tensorflow==1.2.1)
Requirement already satisfied: setuptools in /Users/masao/work/keras-example/lib/python3.6/site-packages (from protobuf>=3.2.0->tensorflow==1.2.1)
Installing collected packages: tensorflow
Successfully installed tensorflow-1.2.1
(keras-example) bash$
```

### 再実行

やっとできたので、再度実行してみます。

```
(keras-example) bash$ time python mnist_cnn.py
Using TensorFlow backend.
x_train shape: (60000, 28, 28, 1)
60000 train samples
10000 test samples
Train on 60000 samples, validate on 10000 samples
Epoch 1/12
60000/60000 [==============================] - 151s - loss: 0.3464 - acc: 0.8953 - val_loss: 0.0731 - val_acc: 0.9773
Epoch 2/12
60000/60000 [==============================] - 189s - loss: 0.1096 - acc: 0.9676 - val_loss: 0.0521 - val_acc: 0.9832
Epoch 3/12
60000/60000 [==============================] - 177s - loss: 0.0819 - acc: 0.9758 - val_loss: 0.0439 - val_acc: 0.9859
Epoch 4/12
60000/60000 [==============================] - 179s - loss: 0.0686 - acc: 0.9794 - val_loss: 0.0388 - val_acc: 0.9876
Epoch 5/12
60000/60000 [==============================] - 187s - loss: 0.0604 - acc: 0.9823 - val_loss: 0.0373 - val_acc: 0.9878
Epoch 6/12
60000/60000 [==============================] - 184s - loss: 0.0546 - acc: 0.9840 - val_loss: 0.0314 - val_acc: 0.9888
Epoch 7/12
60000/60000 [==============================] - 204s - loss: 0.0501 - acc: 0.9847 - val_loss: 0.0322 - val_acc: 0.9883
Epoch 8/12
60000/60000 [==============================] - 197s - loss: 0.0460 - acc: 0.9861 - val_loss: 0.0302 - val_acc: 0.9901
Epoch 9/12
60000/60000 [==============================] - 185s - loss: 0.0435 - acc: 0.9871 - val_loss: 0.0296 - val_acc: 0.9895
Epoch 10/12
60000/60000 [==============================] - 219s - loss: 0.0401 - acc: 0.9880 - val_loss: 0.0294 - val_acc: 0.9900
Epoch 11/12
60000/60000 [==============================] - 209s - loss: 0.0399 - acc: 0.9883 - val_loss: 0.0299 - val_acc: 0.9898
Epoch 12/12
60000/60000 [==============================] - 202s - loss: 0.0361 - acc: 0.9892 - val_loss: 0.0272 - val_acc: 0.9909
Test loss: 0.0272319751683
Test accuracy: 0.9909

real    38m30.351s
user    96m11.136s
sys     23m48.160s
(keras-example) bash$
```

最適化する前は以下のようでした。

```
real    77m16.629s
user    203m6.948s
sys     21m3.707s
```

実は最適化前の時間には、サンプルデータのダウンロード時間が含まれているので
(ベンチマークとしては失敗ですね...)、厳密な比較にはなっていないのですが、
倍位にはなっているのではないかと思います。
（といっても GPU を使うと桁違いに速くなるのでこの程度は誤差の範疇かもしれません）

### まとめ

**労多くして功少なし、** と言い切るのは少し悲しいので、
**「重い処理は速いマシンに任せよう」という教訓を得た**、と前向きに思うようにしますw。

