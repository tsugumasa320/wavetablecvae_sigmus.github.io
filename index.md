<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax:{inlineMath:[['\$','\$'],['\\(','\\)']],processEscapes:true},CommonHTML: {matchFontHeight:false}});</script> <script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

<!--このWebサイトは、第136回音楽情報科学研究発表会 デモ・萌芽・議論セッションで発表の
“Wavetable合成の為のアトリビュート操作型CVAEエフェクターの検討”についての発表資料です-->
## CVAEを用いたウェーブテーブル合成の意味的な音色操作についての検討: SEMANTIC CONTROL OF WAVETABLE SYNTHESIS USING CVAE

## 目次

- [序論](#序論)
- [前提知識](#前提知識)
- [提案手法](#提案手法)
- [結果](#結果)
- [まとめ](#まとめ)

## 概要

![スクリーンショット 2023-02-15 17 27 14](https://user-images.githubusercontent.com/35299183/218973246-b803175d-42c8-4192-845d-eb04d16501b7.png)

本研究では、ウェーブテーブル合成という音響合成方式において、深層生成モデルを用いて意味的な音色操作を行う手法を提案する。
ウェーブテーブル合成とは、1周期分の波形(以下、ウェーブテーブルと称する）を繰り返し読み出す事で音響合成を行う方式である。

提案手法では、Conditional Variational Autoencoder (CVAE) [^1]を用いて、ウェーブテーブルの条件付け生成を行う。
条件付けには、音響特徴に基づいて算出した明るさ(bright), 温かさ(warm), リッチさ(rich)という3つのラベルを用いる。
また、ウェーブテーブルの時間依存性を考慮するために、畳み込みとアップサンプリングを用いたCVAEのアーキテクチャを設計する。
推論時の速度を高める為に、モデルは時間領域のデータで学習される

実験には、Adventure Kid Research & Tech-nology [^2]が提供するモノラルのウェーブテーブル4158件をデータセットとして用いる。
実験の結果、提案手法は、テストデータに対して高い再構成品質を示し、アトリビュートラベルを変化させることでウェーブテーブルの音色を操作できることを定性的・定量的に示す。
本研究は、データに基づいた意味的な音色生成を実現することで、直感性及び簡易性を高める事を目標とする。

### ユースケース案

  1. 使用したいウェーブテーブルを選び、モデルに入力
  2. つまみ(セマンティックラベル)を動かし、音色を調整する
  3. 出力されたウェーブテーブルを用いて演奏を行う

## 序論

### 背景・課題

  - シンセサイザーは音楽制作やパフォーマンスにおいて重要な役割を果たしている
    - 思い描いた音色を生成するには知識と経験が必要
  - 音楽製作者や演奏家に人気のある"ウェーブテーブル合成"に注目
    - 最もシンプルな合成方式であり、他合成手法への展開も見据えてフォーカスして検討

### 目的

- 深層生成モデルによる**データに基づいた意味的な音色操作**の実現
  - 音色の探索を直感的かつ簡易的にする事で、使いやすさを向上する


## 前提知識

### Wavetable Synthesis

<img width="1025" alt="スクリーンショット 2023-02-13 22 52 23" src="https://user-images.githubusercontent.com/35299183/218475920-1e032d15-1914-4f21-b944-ea1a86478802.png">

  - デジタル音響合成の基礎となる技術
  - 1周期分の波形（ウェーブテーブル）を保存し繰り返し読み出す事で音を生成
  - 繰り返し速度を変える事で任意の音高を出力する

<details>
<summary><span style="color: ｂlue; ">>参考動画(クリックすると開きます)</span></summary>

<iframe width="560" height="315" src="https://www.youtube.com/embed/k81hoZODOP0?start=17" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

</details>

### CVAE(Conditional Variational Autoencoder)
  - Encoder-Decoderネットワークに基づいた、確率的生成モデルの一種
  - 入力データに対して条件付きで生成を行うことが可能

## 提案手法

<img width="1045" alt="スクリーンショット 2023-02-08 2 53 14" src="https://user-images.githubusercontent.com/35299183/217326549-51ad3ac1-50cd-4521-99c9-c6e4e7a05840.png">

### ①. データセット

  - Adventure Kid Research & Technology[^2]が提供しているモノラルのSingle Cycle Waveform 4158件を使用

### ②. アトリビュートラベルの算出
  - Wavetableの分析は、静的音色[^6]について表される音響特徴量を用いる必要がある
  - Kreković[^3]は、bright, warm, richの３種類のアトリビュートラベルを音響特長量から計算しており、同様の手法を使用しデータセットからラベルを抽出する
  - 上記のラベルをCVAEの学習と条件付け生成に用いる

### ③. モデル構成
  - 波形の時間依存性を捉えるために、 畳み込みとアップサンプリングを行うモデルを設計
  - Encoder-Decoderの全層に条件付けを実施

<!--

WIP

| Function | Layer                               |                                                                                                                                                                                                                                                                                                                                                                                                         |
| -------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Encoder  | 4-layer Convolutional Network       | - Conv(i=1, o=32, k=6, s=2, p=0) + LReLU + BN<br> - Conv(i=32, o=64, k=8, s=3, p=0) + LReLU + BN<br> - Conv(i=64, o=128, k=7, s=3, p=0) + LReLU + BN<br> - Conv(i=128, o=256, k=6, s=3, p=0) + LReLU + BN                                                                                                                                                                                               |
|          | 1-layer Liner Network               | - Linear(i=2888, 0=256) + LReLU<br> - Linear(i=256, o=16) × 2 (in parallel)                                                                                                                                                                                                                                                                                                                             |
| Decoder  | 1-layer Liner Network               | - Linear(i=256, 0=2888) + LReLU                                                                                                                                                                                                                                                                                                                                                                         |
|          | 3-layer Upsampling + ResNet Network | - LReLU + TrConv(i=128, o=64, k=6, s=1, p=0)<br> - LReLU + Conv((i=64, o=64, k=, s=1, p=0) × 3<br> - LReLU + TrConv(i=64, o=32, k=8, s=1, p=0)<br> - LReLU + Conv((i=32, o=32, k=4, s=1, p=0) × 3<br> - LReLU + TrConv(i=32, o=16, k=7, s=1, p=0)<br> - LReLU + Conv((i=16, o=16, k=4, s=1, p=0) × 3<br> - LReLU + TrConv(i=16, o=16, k=7, s=1, p=0)<br> - LReLU + Conv((i=16, o=16, k=4, s=1, p=0) × 3 |
|          | 1- layer Output Dense:              | - Conv1D(1=8, o=64, k=4, s=1, p=0) + Tanh                                                                                                                                                                                                                                                                                                                                                               |

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
  - AM/FM/加算/減算合成やLFOなどへの応用の検討
  - UIの検討（下記は、検討中のUIイメージ)

<img width="1502" alt="スクリーンショット 2023-02-06 1 55 14" src="https://user-images.githubusercontent.com/35299183/216832970-d43c6374-f9c5-40db-af94-c7c90889d79e.png">

## 謝辞
 This work was supported by Cybozu Labs youth.

## 関連文献, 脚注

[^1]: Kingma, Durk P., et al. “Semi-supervised learning with deep generative models.” Advances in neural information processing systems 27 (2014).

[^2]: "Adventure Kid Research & Technology (AKRT)" https://www.adventurekid.se/akrt/

[^3]: Kreković, Gordan. "DEEP CONVOLUTIONAL OSCILLATOR: SYNTHESIZING WAVEFORMS FROM TIMBRAL DESCRIPTORS." (2022).

[^4]: Engel, Jesse, et al. "DDSP: Differentiable digital signal processing." arXiv preprint arXiv:2001.04643 (2020).

[^5]: Caillon, Antoine, and Philippe Esling. "RAVE: A variational autoencoder for fast and high-quality neural audio synthesis." arXiv preprint arXiv:2111.05011 (2021).

[^6]: 静的音色とは、時間的な変化のない音(定常音)について、周波数スペクトルによって規定される音

[^7]: STFTとは、 Short Term Fourier Transformのこと
