<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax:{inlineMath:[['\$','\$'],['\\(','\\)']],processEscapes:true},CommonHTML: {matchFontHeight:false}});</script> <script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

このWebサイトは、第136回音楽情報科学研究発表会 デモ・萌芽・議論セッションで発表の
“Wavetable合成の為のアトリビュート操作型CVAEエフェクターの検討”についての発表資料です

## 目次

- [序論](#序論)
- [前提知識](#前提知識)
- [提案手法](#提案手法)
- [結果](#結果)
- [まとめ](#まとめ)

## 序論

### 目的

- 深層生成モデルによるデータ・ドリブンなオーディオ・エフェクトの創出
- "Wavetable effector"を提案

<img width="805" alt="スクリーンショット 2023-02-07 23 41 13" src="https://user-images.githubusercontent.com/35299183/217276003-bf56ca9b-ea70-4748-a261-0b7e8a824d72.png">

### ユースケース案

  ①. 使用したいウェーブテーブルを選び、モデルに入力
  
  ②. つまみ(アトリビュートラベル)を動かし、音色を調整する
  
  ③. 出力されたウェーブテーブルを用いて演奏を行う

### 背景
  - オーディオ・エフェクトは様々なメディアや音楽制作で重要な役割を果たしている
  - 近年発展している深層生成モデルによって、データ・ドリブンな新しいエフェクトが作れないか？
  - 基本的な音生成方式[^1]であるウェーブテーブル合成を、CVAEを用いてアップデートする
    - 知覚に基づいたアトリビュートラベルを音響特長量から算出
    - オシレーターに使用するウェーブテーブルを上記ラベルで条件付け生成

## 前提知識
  
### ウェーブテーブル合成
  - 任意の波形1周期分の情報を保存(以下、ウェーブテーブル)
  - ウェーブテーブルの繰り返し速度を変える事でオシレーターとして任意の音高を出力する
  - デジタル音響合成の基礎となる技術であり、多くのシンセサイザーで用いられる

#### 参考動画
<iframe width="560" height="315" src="https://www.youtube.com/embed/k81hoZODOP0?start=17" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
  
### CVAE(Conditional Variational Autoencoder)
  - Encoder-Decoderネットワークに基づいた、確率的生成モデルの一種
  - 入力データに対して条件付きで生成を行うことが可能
 
## 提案手法

以下にモデルの概要を示す

<img width="612" alt="model" src="https://user-images.githubusercontent.com/35299183/217061772-4cbb9f9d-6951-4b8a-9e3f-fb170f7989f0.png">

### ①. データセット

  - Adventure Kid Research & Technology[^2]が提供しているモノラルのウェーブテーブル(Single Cycle Waveform)4158件を使用

### ②. アトリビュートラベルの算出
  - ウェーブテーブルの分析は、静的音色[^6]について表される音響特徴量を用いる必要がある
  - Kreković[^3]は、brightness, richness, fullnessの３種類のアトリビュートラベルを音響特長量から計算しており、同様の手法を使用しデータセットからラベルを抽出する
  - 上記のラベルをCVAEの学習と条件付け生成に用いる

### ③. モデル構成
  - 波形の時間依存性を捉えるために、 畳み込みとアップサンプリングを行うモデルを設計
  - Encoder-Decoderの全層に条件付けを実施

<!--

WIP

| Function | Layer |  |
| - | - | - |
| Encoder | 4-layer Convolutional Network |  - Conv(i=1, o=32, k=6, s=2, p=0) + LReLU + BN<br> - Conv(i=32, o=64, k=8, s=3, p=0) + LReLU + BN<br> - Conv(i=64, o=128, k=7, s=3, p=0) + LReLU + BN<br> - Conv(i=128, o=256, k=6, s=3, p=0) + LReLU + BN |
|  | 1-layer Liner Network |  - Linear(i=2888, 0=256) + LReLU<br> - Linear(i=256, o=16) × 2 (in parallel) |
| Decoder | 1-layer Liner Network |  - Linear(i=256, 0=2888) + LReLU |
|  | 3-layer Upsampling + ResNet Network |  - LReLU + TrConv(i=128, o=64, k=6, s=1, p=0)<br> - LReLU + Conv((i=64, o=64, k=, s=1, p=0) × 3<br> - LReLU + TrConv(i=64, o=32, k=8, s=1, p=0)<br> - LReLU + Conv((i=32, o=32, k=4, s=1, p=0) × 3<br> - LReLU + TrConv(i=32, o=16, k=7, s=1, p=0)<br> - LReLU + Conv((i=16, o=16, k=4, s=1, p=0) × 3<br> - LReLU + TrConv(i=16, o=16, k=7, s=1, p=0)<br> - LReLU + Conv((i=16, o=16, k=4, s=1, p=0) × 3 |
|  | 1- layer Output Dense: |  - Conv1D(1=8, o=64, k=4, s=1, p=0) + Tanh |

(下記の様な解説入れる）

Table 2 Table showing configurations of the VAEs for the image-based datasets. In the Encoder Linear Stack, the last layer has two parallel linear layers for computing the mean and log standard deviation of the latent vectors respectively. Conv: 2-dimensional convolutional layer, TrConv:
2-dimensional transposed convolutional layer, i: input channels, o: output channels, k: kernel size, s: stride, p: padding, d: dropout probability,
SELU: Scaled Exponential Linear Unit (291. ReLU: Rectifier Linear Unit

-->

### ④. 損失関数

<!--
![スクリーンショット 2023-02-06 1 20 03](https://user-images.githubusercontent.com/35299183/216831207-e6c03af1-f912-4441-9b13-cbd3cba33e55.png)
メモ：横長の図にする
-->

  - 音響信号は位相が異なっていても同じスペクトルを得る事がある
  - 特徴を正確に捉える為にSTFT[^7]を行い、スペクトルからロスを計算
  - スペクトルの分解能を上げる為に、6つ分のウェーブテーブルを連結し、下記のスペクトル距離を用いる 
 
 $$ S(x,y) =  \frac{||STFT(x) - STFT(y)||_F}{||STFT(x)||_F} + log(||STFT(x) -STFT(y)||_1) $$
 
   - $\|\|・\|\|_F$ , $\|\|・\|\|_1$ はそれぞれフロべニウスノルム、L1ノルムである
   - 上記スペクトル距離は、Engelら[^4]やCaillonら[^5]が使用しているマルチスペクトル距離を参考にした
    
## 結果

### 再構成品質
  - テストデータにおいて、再構成品質と条件付け生成の結果を確認
  - (チャンピオンデータ3つ選定して載せる)

<img width="653" alt="スクリーンショット 2023-02-06 1 44 42" src="https://user-images.githubusercontent.com/35299183/216832429-6054285c-20f1-4cb3-a086-027fc3e6b114.png">

(何かの曲で再合成して載せる。案としては正弦波を3つのラベルでそれぞれ動かしたものとか)

<img width="653" alt="スクリーンショット 2023-02-06 1 43 32" src="https://user-images.githubusercontent.com/35299183/216832368-e7272d40-264b-4b73-beea-123b513a2be8.png">

## まとめ

### 結論
  - ウェーブテーブル合成における新しいエフェクトの可能性を示した
    - CVAEを用いたウェーブテーブルの再構成と条件付けにおいて、更なる検証が必要ではあるが再構成と条件付け生成において、一定の成果を得る事が出来た

### 今後の展望
  - 更なる再構成品質向上を可能にするモデル構成の検討
  - 新しいエフェクト創出の為の、ラベル抽出手法の探索
  - UIの検討（下記は、検討中のUIイメージ)

<img width="1502" alt="スクリーンショット 2023-02-06 1 55 14" src="https://user-images.githubusercontent.com/35299183/216832970-d43c6374-f9c5-40db-af94-c7c90889d79e.png">

## 謝辞
 This work was supported by Cybozu Labs youth.

## 関連文献, 脚注

[^1]: コンピュータ音楽 : 歴史・テクノロジー・アート, Curtis Roads (著), 青柳龍也・小坂直敏・平田圭二・堀内靖雄 (訳・監修), 後藤真孝・引地孝文・平野砂峰旅・松島俊明(訳), 東京電機大学出版局, 2001年

[^2]: "Adventure Kid Research & Technology (AKRT)" https://www.adventurekid.se/akrt/

[^3]: Kreković, Gordan. "DEEP CONVOLUTIONAL OSCILLATOR: SYNTHESIZING WAVEFORMS FROM TIMBRAL DESCRIPTORS." (2022).

[^4]: Engel, Jesse, et al. "DDSP: Differentiable digital signal processing." arXiv preprint arXiv:2001.04643 (2020).

[^5]: Caillon, Antoine, and Philippe Esling. "RAVE: A variational autoencoder for fast and high-quality neural audio synthesis." arXiv preprint arXiv:2111.05011 (2021).

[^6]: 静的音色とは、時間的な変化のない音(定常音)について、周波数スペクトルによって規定される音

[^7]: STFTとは、 Short Term Fourier Transformのこと
