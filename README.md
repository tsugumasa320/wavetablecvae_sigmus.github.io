<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax:{inlineMath:[['\$','\$'],['\\(','\\)']],processEscapes:true},CommonHTML: {matchFontHeight:false}});</script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

このWebサイトは、第136回音楽情報科学研究発表会 デモ・萌芽・議論セッションで発表の
"Wavetable合成の為のアトリビュート操作型CVAEエフェクターの初期検討"についてです。

# 概要

<img width="1474" alt="Overview" src="https://user-images.githubusercontent.com/35299183/214529868-13d7e573-4f37-4c3e-ba25-e906b7363d98.png">

本研究は、CVAE(Conditional Variational AutoEncoder)[^1]により波形1周期分のデータ(以下、ウェーブテーブル)の再構成と条件付け生成を行い、
ウェーブテーブル合成におけるデータ・ドリブンな音色の操作を実現する。

教師ラベルとしては、音響特徴を基に、brightness, richness, fullness, noisinessを表すアトリビュートラベルを算出し、
波形の時間依存性を捉えるために畳み込みとアップサンプリングを用いたCVAEを設計した。

データセットは、Adventure Kid Research & Technology[^2]が提供している
モノラルのウェーブテーブル(Single Cycle Waveform)4158件を用いて学習を行い、
アトリビュートラベルを条件付けに用いてウェーブテーブルの生成を行う。

テストデータにおいて、再構成の忠実度と正規化されたアトリビュートの変化を定性的・定量的に確認し、
データ・ドリブンなエフェクトの創出を目標とする。(まだやっていない)

# 導入

オーディオ・エフェクトは、音楽制作やライブパフォーマンス、テレビ、映画、ゲームなど様々なメディアで広く利用され、
特に音楽制作においては、音声のダイナミクス・空間性・音色・ピッチなどを操作するために使われている。

近年では、[^3][^4][^5]の様に深層生成モデルを用いて、Raw audio dataに対するスタイル変換やオーディオ・エフェクトが可能なモデルが出てきている。
例えば、DDSPは深層生成モデルの一部に古典的な信号処理に基づいた機構を組み込む事で、音の表現力やリアルさと操作性を実現している。

対して、本研究では合成音における基本的な音生成方式[^6]であるウェーブテーブル合成の一部に深層生成モデルを組み込むことによって、
既存のウェーブテーブル合成に親しんでいる音楽製作者や演奏者にとって使いやすく実用的で表現手段の拡張となる手法を検討する。

<img width="892" alt="スクリーンショット 2023-01-30 14 07 42" src="https://user-images.githubusercontent.com/35299183/215392159-981f4355-e68e-4eba-933e-b17f554f59a7.png">

[^9]は深層生成モデルを用いたウェーブテーブル合成の拡張の可能性を示している。
しかし、3つのパラメータからアップサンプリングを行い生成を行うというモデル構成上、
限られた範囲のウェーブテーブルしか生成ができない。
また、深層生成モデルを用いて潜在空間をランダムにトラバースする様なアプローチ[^10]もあるが、
オーディオの特徴に関係するパラメータを用いて操作する事が直感的かつ有用な音生成に繋がると考える。

本研究では、CVAEを用いて再構成と条件付け生成を行う。
音響特徴量を基に算出したアトリビューションによって条件付けし、
多様なウェーブテーブルを対象に、ユーザーが理解する事ができるパラメータで
直感的かつ新たなオーディオ・エフェクト効果を得る事を目的とする

# 事前知識

<!--
## Data-driven parametric synthesis[]

1980年代から音声認識や音声合成で使用されるHMMを初めとした統計モデルが活躍
HMMの後継として深層生成モデルがあり、これはデータを元に特徴表現を獲得して音を生成する。
生成した音はデータと知覚的に類似しているが単に再現したものではなく、パラメトリックして再構成される
-->

## ウェーブテーブル合成
ウェーブテーブル合成は、合成音における基本的な音生成方式であり、[^6]
波形1周期分の情報を保存したものをテーブルと呼び、テーブルの繰り返し速度を変える事で任意の周波数の発音を可能にするものである。
多くのデジタル音響合成の核となる技術であり、オシレーターやLFO(Low Freaquency Oscillator)として他の合成方式でも使用される。
また、ウェーブテーブルでは、テーブルサイズが128~2048のものが用いられる事が多いが、
近年のソフトウェアプラグインでは、より広い周波数を表現する為に大きく設定する事が多い。
以下に例を示す。

|  ソフトウェア  |  テーブルサイズ  |
| ---- | ---- |
|  Serum[^13], Pigments[^14],Vital[^15]  |  2048  |
|  Ableton Wavetable[^16]  |  1024  |

<iframe width="560" height="315" src="https://www.youtube.com/embed/k81hoZODOP0?start=17" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## CVAE

CVAE（Conditional Variational Autoencoder）は、
Encoder-Decoderネットワークに基づいた、確率的生成モデルの一種であり、
入力データに対して条件付きで生成を行うことができる。
本研究では、ウェーブテーブルの時間依存性を捉えるために、
Encoder-Decoderにそれぞれ畳み込みとアップサンプリングを採用している。
Encoderは入力データとラベルから低次元に圧縮された潜在表現を抽出し、
Decoderは潜在表現とラベルを用いて出力データを生成する。

## 音響特徴量とアトリビュート
 
 岩宮ら[^11]は音色を、静的音色、準静的音色、動的音色、準動的音色の4種類に分類している。
 静的音色とは、時間的な変化のない音(定常音)について、周波数スペクトルによって規定される音であり、
 ウェーブテーブルの分析は、静的音色について表される音響特徴量を用いる必要がある。
 
 Kreković[^9]は,ウェーブテーブルに対して、brightness, richness, fullnessの３種類のアトリビュートラベルを
 パワースペクトラムのスペクトルセントロイド,奇数時倍音の相対エネルギー,スペクトルスプレッドから算出した。
 
 単一のウェーブテーブルからこれらの特徴を計算する為に、波形を6つ分繰り返し連結しスペクトルの分解能を高めている。

<img width="1000" alt="ラベル分布" src="https://user-images.githubusercontent.com/35299183/214603563-da043596-6d6d-402f-9475-2cfcae662825.png">


# 提案手法
　　既存のウェーブテーブル合成ユーザーにとって使いやすいものとする為には、
 4つの要件を満たすことが重要だと考える。
 
 1つ目は、より実用に近い大きなデータサイズを持つウェーブテーブルを学習に用いる事である。
 Adventure Kid Research & Technology[^2]が提供している
 テーブルサイズが600[sample]のモノラルのウェーブテーブル(Single Cycle Waveform)4158件を用いて学習を行なった。
 
 <!--
 2つ目は、リアルタイム演奏を想定し、ウェーブテーブル生成にかかる時間を少なく保つ事である。
 []は、ウェーブテーブルの特徴表現を圧縮するために、周波数領域に変換し学習を行なっている。
 しかし、推論/生成の際に周波数領域と時間領域の変換には時間を要する。
 本研究では、時間領域で学習を行い、推論/生成の高速化を図った.
 （ここは検証が必要）
 -->
 
 2つ目は、ウェーブテーブルの特徴を捉えるためのラベル付手法である。
 ここでは、[^9]らの研究成果に基づき、brightness, richness, fullnessの３種類のアトリビュートラベルを算出した。
 尚、音響特長量を算出後に正規化手法としてYeo-Johnson変換とMinMax-Normalizeを行い0~1の範囲にスケーリングを行なった。
  
 3つ目は、より広範な種類のウェーブテーブルを生成可能にするネットワーク構成である。
 Encoder-Decoderネットワークを用いて、入力データを再構成する様に学習を行い、
 生成の際に、条件付けを行うことにより、多様な種類のウェーブテーブルに対応可能にした。

 ## モデル構成
 
<img width="817" alt="スクリーンショット 2023-01-29 20 59 23" src="https://user-images.githubusercontent.com/35299183/215324590-bee3a9b2-f9b7-4d2a-add0-ebb8633b72f6.png">
 
 このネットワークは[^5]らの研究にヒントを得ており、
 畳み込み層とアップサンプリング層を経る事で、波形の時間依存性を捉えるように設計されている。
 また、より条件付けをモデルに強制させるために、Encoder-Decoderの全層のチャンネルにラベル情報を連結させている。
 
 <img width="553" alt="スクリーンショット 2023-01-29 17 42 23" src="https://user-images.githubusercontent.com/35299183/215315225-e0858365-280d-4ffc-9595-09891673884f.png">

 ## 損失関数
 
 音声の特徴は波形に一意に対応するものでなく、
 位相が異なっていても同様のスペクトルを得ることがある。
 その為、スペクトラムによって損失を計算している。
 [^9]は6つ分のウェーブテーブルを繰り返し連結する事でスペクトルの分解能を向上させており、
 同様の手法を使用する。
 また、Engelら[^4]は波形間の距離を推定するためマルチスケールスペクトル距離を提案している。
 ウェーブテーブルにおいては、波形の長さが短いことから、繰り返し連結を行った後の最大テーブルサイズのみでスペクトル距離を算出した。
 以下の様に定義する。
 
 $$ S(x,y) =  \frac{||STFT(x) - STFT(y)||_F}{||STFT(x)||_F} + log(||STFT(x) -STFT(y)||_1) $$
 
 STFTはShort Term Fourier Transformのことであり、 $\|\|・\|\|_F$ , $\|\|・\|\|_1$ はそれぞれフロべニウスノルム、L1ノルムである。

# 実験

## 実験目的
 
 ①. 再構成誤差 -> クオリティ
 
 ②. KL -> 連続性
 
 ③. Conditionig -> 操作性
 
 この3つが上手くいっているか確認する。

 ・ データセット
 
 Adventure Kid Research & Technology[^2]が提供している
 モノラルのウェーブテーブル(Single Cycle Waveform)で、
 サンプリングレートは44.1kHz,データサイズ600sampleのデータ4158件を使用した。
 
 ・ ラベル
 
 ラベルは、[^12]と同様の手法で[0,1]の範囲かつ知覚的に線形になる様に変換している。
 
 $$ f(x) = {log[(e^k - 1) ・ \frac{x-b_l}{b_u-b_l} + 1 ] \over k} $$
 
 $b_l$はラベルの下限を表し、 $b_u$ は上限を表す。定数 $k$ は対数変換後のデータ分布を制御するものである.
 スペクトル密度は7.5それ以外の特徴は5.5を用いている。
 
 上記のラベルに加えて、noisinessを表す特徴としてゼロ交差率を追加し、 $k=5.5$ で分布の変換を行い、
 算出したラベルの分布を示す。固定長のデータであり、窓幅はデータ長と同様の幅とし、窓関数は用いていない。
 
 ・ 評価指標
 
 再構成誤差と操作の影響を定性的に確認した。
 
 （画像載っける）
 
 ・ 実験プロトコル（実験再現できるくらい書く）
 
 学習回数は,10000epochで学習率1e-4,
 betaを指数関数的に上がる様に設定.
  
　　・ 実験結果 + 考察
  
 ・ richnessはうまくいく、他はうまくいかない
 
# 結論

 ・ 研究背景
 
 ・　　なにを明らかにしようとしたか
　
 ・　　どのように明らかにしようとしたか
 
　　・　　結果何がわかったか（＝結論）

# future work

 ・ やり残したことではなく、夢のあることを書く(この研究の先にどんな面白い研究が出てきうるのか)
 
  ・ AR-VAEやFader-Netの適用
 
  ・　ギターや他のシンセ方式への応用

# 謝辞

　・サイボウズラボユース

# 引用

[^1]: Kingma, Durk P., et al. "Semi-supervised learning with deep generative models." Advances in neural information processing systems 27 (2014).

[^2]: "Adventure Kid Research & Technology (AKRT)" https://www.adventurekid.se/akrt/

[^3]: Engel, Jesse, et al. "Neural audio synthesis of musical notes with wavenet autoencoders." International Conference on Machine Learning. PMLR, 2017.

[^4]: Engel, Jesse, et al. "DDSP: Differentiable digital signal processing." arXiv preprint arXiv:2001.04643 (2020).

[^5]: Caillon, Antoine, and Philippe Esling. "RAVE: A variational autoencoder for fast and high-quality neural audio synthesis." arXiv preprint arXiv:2111.05011 (2021).

[^6]: コンピュータ音楽 : 歴史・テクノロジー・アート, Curtis Roads (著), 青柳龍也・小坂直敏・平田圭二・堀内靖雄 (訳・監修), 後藤真孝・引地孝文・平野砂峰旅・松島俊明(訳), 東京電機大学出版局, 2001年

[^9]: Kreković, Gordan. "DEEP CONVOLUTIONAL OSCILLATOR: SYNTHESIZING WAVEFORMS FROM TIMBRAL DESCRIPTORS." (2022).

[^10]: Hyrkas, Jeremy. "WaVAEtable Synthesis." CMMR: 263.

[^11]: 岩宮眞一郎：音響サイエンスシリーズ1 音色の感性学， コロナ社，pp.64-67， 2010.

[^12]: Kreković, G., Pošćić, A., Petrinović, D., “An Algorithm for Controlling Arbitrary Sound Synthesizers Using Adjectives”, Jorunal of New Music Reserach, 2016., DOI:10.1080/09298215.2016.1204325

[^13]: xfer records, Serum, https://xferrecords.com/products/serum

[^14]: Auturia, Pigments, https://www.arturia.com/products/software-instruments/pigments/overview

[^15]: Vital Audio, Vital, https://vital.audio/

[^16]: Ableton, Wavetable, https://www.ableton.com/ja/packs/wavetable/


<!--

[2] Robert Bristow-Johnson, “Wavetable synthesis 101, a
fundamental perspective,” in Audio Engineering Society
Convention 101. Audio Engineering Society, 1996.

[3] Andrew B. Horner and James W. Beauchamp, Instrument Modeling and Synthesis, pp. 375–397, Springer
New York, New York, NY, 2008.

-->

