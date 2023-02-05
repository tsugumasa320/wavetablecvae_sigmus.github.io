<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax:{inlineMath:[['\$','\$'],['\\(','\\)']],processEscapes:true},CommonHTML: {matchFontHeight:false}});</script> <script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

このWebサイトは、第136回音楽情報科学研究発表会 デモ・萌芽・議論セッションで発表の
“Wavetable合成の為のアトリビュート操作型CVAEエフェクターの検討”についてです。

### 目次

- [序論](#序論)
- [前提知識](#前提知識)
- [提案手法](#提案手法)
- [結果](#結果)
- [まとめ](#まとめ)

### 序論

#### 目的
  - 深層生成モデルによるデータ・ドリブンなオーディオ・エフェクトの創出

<img width="661" alt="スクリーンショット 2023-02-06 0 47 51" src="https://user-images.githubusercontent.com/35299183/216829590-fd2292f1-2f5b-4f98-889a-a6f615ebb8de.png">

##### ユースケース案

  1. 使用したいウェーブテーブルを選び、モデルに入力
  2. つまみ(アトリビュートラベル)を動かし、音色を調整する
  3. 出力されたウェーブテーブルを用いて演奏を行う

#### 背景
  - オーディオ・エフェクトは様々なメディアや音楽制作で重要な役割を果たしている
  - 深層生成モデルによって、アトリビュートに基づいた新しいエフェクトが作れないか？
  - 最も基本的な音生成方式であるウェーブテーブル合成を、CVAEを用いてアップデートする
    - 知覚に基づいたアトリビュートラベルを音響特長量から算出
    - オシレーターに使用するウェーブテーブルを上記ラベルで条件付け生成

### 前提知識
  
#### ウェーブテーブル合成
  - 任意の波形1周期分の情報を保存(以下、ウェーブテーブル)
  - ウェーブテーブルの繰り返し速度を変える事でオシレーターとして任意の音高を出力する
  - デジタル音響合成の基礎となる技術であり、多くのシンセサイザーで用いられる

##### 参考動画
<iframe width="560" height="315" src="https://www.youtube.com/embed/k81hoZODOP0?start=17" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
  
#### CVAE(Conditional Variational Autoencoder)
  - Encoder-Decoderネットワークに基づいた、確率的生成モデルの一種
  - 入力データに対して条件付きで生成を行うことが可能
 
### 提案手法

<img width="429" alt="スクリーンショット 2023-02-06 1 13 53" src="https://user-images.githubusercontent.com/35299183/216830929-169b2213-c489-4c05-92d8-46cb35d0a326.png">

#### アトリビュートラベルの算出
  - ウェーブテーブルの分析は、静的音色について表される音響特徴量を用いる必要がある。
  - Kreković[^1]は、brightness, richness, fullnessの３種類のアトリビュートラベルを音響特長量から計算
  - 上記と同様の手法を用いて、条件付けに使用するラベルを算出

#### データセット
  - Adventure Kid Research & Technology[^2]が提供しているモノラルのウェーブテーブル(Single Cycle Waveform)4158件を使用

#### モデル構成
  - 波形の時間依存性を捉えるために、 畳み込みとアップサンプリングを行うモデルを設計
  - Encoder-Decoderの全層に条件付けを実施

#### 損失関数

![スクリーンショット 2023-02-06 1 20 03](https://user-images.githubusercontent.com/35299183/216831207-e6c03af1-f912-4441-9b13-cbd3cba33e55.png)

  - 音響信号の特徴と波形は一意に対応するものでなく、位相が異なっていても同様のスペクトルを得る事がある
  - 特徴を正確に捉える為にSTFTによって、スペクトルを計算
  - スペクトルの分解能を上げる為に、6つ分のウェーブテーブルを連結し、(引用入れる)スペクトル距離を用いて損失とする

$$ S(x,y) =  \frac{||STFT(x) - STFT(y)||_F}{||STFT(x)||_F} + log(||STFT(x) -STFT(y)||_1) $$
 
STFTはShort Term Fourier Transformのことであり、 $\|\|・\|\|_F$ , $\|\|・\|\|_1$ はそれぞれフロべニウスノルム、L1ノルムである。
    
### 結果

#### 再構成品質
  - テストデータにおいて、再構成品質と条件付け生成の結果を確認

<img width="653" alt="スクリーンショット 2023-02-06 1 44 42" src="https://user-images.githubusercontent.com/35299183/216832429-6054285c-20f1-4cb3-a086-027fc3e6b114.png">

(単音と曲の音声も載せる)

<img width="653" alt="スクリーンショット 2023-02-06 1 43 32" src="https://user-images.githubusercontent.com/35299183/216832368-e7272d40-264b-4b73-beea-123b513a2be8.png">

### まとめ

#### 結論
  - ウェーブテーブル合成における新しいエフェクトの可能性を示した
    - CVAEを用いたウェーブテーブルの再構成と条件付けにおいて、更なる検証が必要ではあるが一定の成果を得た

#### 今後の展望
  - 更なる再構成品質向上を可能にするモデル構成
  - 新しいエフェクト創出の為の、ラベル抽出手法の探索
  - UIの検討（下記は、検討中のUIイメージ)

<img width="1502" alt="スクリーンショット 2023-02-06 1 55 14" src="https://user-images.githubusercontent.com/35299183/216832970-d43c6374-f9c5-40db-af94-c7c90889d79e.png">

### 謝辞
 This work was supported by Cybozu Labs youth.

### 関連文献
