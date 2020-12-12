---
layout:     post
title:      "[코드리뷰]UDA"
subtitle:   "Unsupervised Data Augmentation for Consistency Training"
mathjax: true
tags:
  - Semi-supervised Learning
  - Data augmentation
  - Deep Learning
  - NLP(Natural Language Process) 
---

# [코드리뷰] - [Unsupervised Data Augmentation for Consistency Training](https://arxiv.org/abs/1904.12848), NIPS 2019

딥러닝은 다양한 분야에서 기존 방법론 보다 좋은 성과를 보여주고 있습니다.
Machine Translation, Sentiment Analysis, Question And Answering(Q&A) 등 일부 자연어처리 분야에서는 전통적인 방법론 보다 월등히 앞선 성능을 보이고 있습니다.
따라서 기업들이 앞 다투어 딥러닝을 다양한 분야에 적용하고 서비스를 출시하려고 노력하고 있습니다.
하지만 안타깝게도 딥러닝이 적용된 서비스는 생각보다 찾아보기 힘들며 그 품질 역시 소비자를 만족시키기에 부족합니다.

품질하락의 다양한 이유가 있지만 딥러닝 관점에서 이유를 찾아보면 근본적인 문제는 해당 분야에 대한 Labeled 데이터가 부족하다는 것입니다.
일반적으로 딥러닝 모델를 학습하기 위해서는 많은 Labeled 데이터가 필요합니다.
데이터의 양은 딥러닝 모델의 성능과 비례관계가 있기 때문에 얼마나 데이터를 확보했는가에 따라 딥러닝 모델의 성능이 달라집니다.
따라서 다양한 분야에 딥러닝을 적용하여도 Labeled 데이터가 부족하기 때문에 좋은 성능의 모델을 만들 수 없습니다.

이를 해결하기 위하여 Semi-supervised Learning을 주로 적용합니다.
이 방법론은 Labeled 데이터 뿐만 아니라 Unlabeled 데이터를 활용하여 모델의 성능을 향상시키는 방법론입니다.
오늘 포스팅에서는 최근 Semi-superivsed Learning 방법론 중 좋은 성능을 보이고 있는 `UDA` 에 대해 다루도록 하겠습니다.
이 글은 **[Unsupervised Data Augmentation for Consistency Training](https://arxiv.org/abs/1904.12848)** 논문을 참고하여 정리하였음을 먼저 밝힙니다.
논문을 간단하게 리뷰하고 pytorch 라이브러리를 이용하여 코드를 구현한 내용을 자세히 설명드리겠습니다.
논문 그대로를 리뷰하기보다는 *생각을 정리하는 목적으로 제작*하고 있기 때문에 실제 내용과 다른점이 존재할 수 있습니다. 
혹시 제가 잘못 알고 있는 점이나 보안할 점이 있다면 댓글 부탁드립니다.

#### Short Summary
이 논문의 큰 특징 3가지는 아래와 같습니다.

1. Vision, NLP 분야에서 효과적인 **Data Augmenation** 방법론을 제시하고 이를 Semi-superivsed Learning에 활용합니다. 
2. **Consistency Loss와 Supervised Loss를 이용**하여 모델을 학습하는 Semi-supervised Learning 방법론을 제안합니다.  
3. Labeled 데이터를 10% 정도 사용하여도 해당방법론이 Fully Superivsed 방법론 보다 **좋은 성능**을 보일 수 있다는 것을 실험적으로 증명하였습니다.

## 논문 리뷰

해당 논문에서는 제시한 방법론을 두가지 분야(NLP, Vision) 에 적용하고 실험결과를 제시합니다.
이 글에서는 UDA논문을 NLP에 적용하는 방법에 대해 자세하게 다루기 위하여 Vision에 대한 내용은 포함하고 있지 않습니다.
따라서 Vision 분야에 UDA를 적용하는 방법을 알고 싶으신 분들은 [Youngerous BLOG]() 를 참조하시기 바랍니다.

### Overview

![](/img/in-post/2020/2020-12-13/overview.png)
<center>UDA overview</center>

UDA는 Labeled 데이터와 Unlabeled 데이터를 함께 학습에 활용하는 Semi-supervised Learning 방법론입니다.
Labeled 데이터를 활용하여 Supervised Loss를 구성하고 Unlabeled 데이터를 이용하여 Consistency Loss를 구성합니다.
이 두개의 Loss를 합쳐서 Final Loss를 구성하고 이를 학습에 활용합니다.

Supervised Loss는 일반적인 분류 학습에 활용하는 Cross_entropy Loss이므로 Labeled 데이터의 문장 $x_l$와 라벨 $y$가 있으면 쉽게 구성할 수 있습니다.
반면 Consistency Loss를 만들기 위해서는 두개의 의미가 비슷한 문장이 필요합니다.
따라서 Unlabeled 데이터의 문장 $x_u$ 뿐만 아니라 $x_u$와 비슷한 의미를 지니지만 문법적, 단어의 표현이 다른 문장 $\hat{x_u}$을 생성해야 합니다.
UDA는 이를 위하여 Back Translation, TD-IDF 등의 Data Augmentation 방법을 제시합니다.

Data Augmentation을 통해 생성된 문장을 분류모델에 넣으면 특정 라벨에 속할 확률분포 $p_{\theta}(y|\hat{x})$를 추출할 수 있습니다.
또한 원본 문장을 분류모델에 넣으면 특정 라벨에 속할 확률분포 $p_{\theta}(y|x)$를 추출할 수 있습니다.
이 두 분포의 차이인 KL-Divergence $D( p_{\theta}(y|x) || p_{\theta}(y|\hat{x} )$ 를 계산하여 Consistency Loss로 활용합니다.

이 방법을 활용하면 적은 Label 데이터와 많은 Unlabeled 데이터로 좋은 성능을 도출할 수 있습니다.

### BackGround

##### Unsupervised Data Augmentation

Data Augmenation은 원본 데이터를 이용하여 인공 데이터를 생성하는 방법을 의미합니다.
일부 단어를 변경하거나 문법 및 문장의 내용을 수정하여 데이터를 증강하는 등의 다양한 방법들이 있습니다.

![](/img/in-post/2020/2020-12-13/text_augmentation.png)
<center>Text Augentation 분류 <a href="http://dsba.korea.ac.kr/seminar/?pageid=3&mod=document&uid=1328">[LINK]</a></center>

일반적으로 Data Augmentation은 단순히 데이터를 증강하는 것이 아니라 원본 데이터와 표현은 다르지만 유사한 의도(label)을 갖고 있는 데이터를 생성하는 것이 목표입니다.
따라서 label을 보존하면서 데이터를 생성하는 방법에 대해 최근에 다양한 방법론들이 연구되고 있습니다.

> Data Augmenation과 관련된 [세미나 영상](https://www.youtube.com/watch?v=UVtMqh3agQY&feature=youtu.be) 입니다. Data Augmentation에 대해 궁금하신 분들은 참고 바랍니다.

본 논문에서는 다양한 NLP Data Augmenation 방법중에서 Back-translation을 활용합니다.
Back-translation은 번역기 두개를 이용하여 인공 문장을 생성하는 방법입니다.

![](/img/in-post/2020/2020-12-13/back_translation.png)
<center>Back-Translation 예시</center>

Back-translation을 하기 위해서는 두개의 번역기가 필요합니다.
예를들어 위 그림에서 번역기A는 언어1(한글)에서 언어2(영어)로 번역해주는 Seq2Seq 모델이고, 번역기B는 언어2(영어)에서 언어1(한글)로 번역해주는 Seq2Seq 모델입니다.
Back-translation 과정은 원본문장을 번역기A에 넣고 그 번역된 문장을 번역기B에 넣어 새로운 문장을 생성하는 것을 의미합니다. 
번역기의 품질이 매우 뛰어나더라도 언어간 미묘한 간극에 의하여 번역이 달라질 수 있으며 동일한 의미를 지니더라도 번역기의 학습데이터에 따라 다르게 표현될 수 있습니다.
즉 원본문장과 의미, 의도(label)는 비슷하지만 표현이 다른 인공문장을 생성할 수 있습니다.

본 논문에서는 원본문장과 인공문장의 차이(diversity)를 크게 하기 위하여 번역기B에 Temperature Sampling 방법을 적용합니다.
Temperature Sampling 이란 번역기의 Decoder에서 생성된 단어의 생성 확률을 특정 비율(temperature)로 조정한 다음 sampling 방법을 이용하여 문장을 번역하는 것을 의미합니다.

![](/img/in-post/2020/2020-12-13/teperatue_sampling.png)
<center>Temperature Sampling 예시</center>

좀더 구체적으로 설명해보자면 번역기는 일반적으로 Encoder와 Deocder로 구성된 Sequence to Sequence 모델입니다.
문장을 넣으면 Encoder는 문장의 정보를 압축하고 Deocder는 autoregressive하게 매 시점 특정 token(단어)이 현 시점에서 등장할지에 대한 확률이 추출됩니다.
Temperature라는 파라미터를 이용하여 추출된 확률을 변화할 수 있습니다. 

<center>$\hat{p_i} = f_{\tau}(p)_i = \frac{p_i^{\frac{1}{\tau}}}{\sum_j p_j^{\frac{1}{\tau}}}$</center>
$p_i$ : 번역기 decoder에서 추출된 단어 $i$가 등장할 확률  
$\hat{p_i}$ : Temperatue가 적용된 단어 $i$가 등장할 확률  
$\tau$ : Temperatue를 의미하며 사용자 설정 파라미터  

Temperature가 1보다 작으면 등장할 확률이 높은 단어의 확률을 높이는 역할을 하며
Temperature가 1보다 크면 등장할 확률이 낮은 단어와 높은 단어의 차이를 적게 하는 역할을 합니다.
따라서 Temperatue를 증가시켜 Sampling하면 다양한(Diversity) 문장을 생성할 수 있으며, Temperatue를 감소시켜 Sampling하면 문법에(Robust) 비교적 맞는 문장을 생성할 수 있습니다.  

본 논문에서는 Temperatue를 0.9로 고정하고 Back-translation을 이용하여 인공문장을 생성합니다.

##### Consistency Training

Consistency Training 이란 모델이 데이터의 작은 변화에 민감하지 않게(Robust) 만드는 방법입니다.
작은 변화란 사람이 봤을 때 label에 큰 영향을 주지 않을 정도의 noise를 의미합니다.

![](/img/in-post/2020/2020-12-13/adversarial_training.png)
<center>Adversarial Training 예시</center>

예를 들어 팬더의 그림이 있을 때 Gaussian Noise를 추가할 경우 사람이 봤을 때는 여전히 팬더라고 대부분 생각합니다.
반면 딥러닝 모델은 사람과는 다르게 이러한 미묘한 변화를 크게 받아들이며 분류 모델인 경우 팬더가 아닌 다른 것으로 예측합니다.
이를 극복하기 위하여 일반적으로 Regulization Term을 추가하거나 변형한 데이터가 원본의 Label을 예측할 수 있도록 학습하여 Consistancy Training을 적용합니다.
Semi-supervised Learning에서 Consistency Tranining 은 조금 다른 뜻으로 활용됩니다.
Unlabeled 데이터를 모델 학습에 활용하기 위하여 노이즈가 추가된 데이터와 추가되지 않은 데이터가 동일한 label을 갖어야 한다는 Consistaency Training의 아이디어를 활용합니다.

![](/img/in-post/2020/2020-12-13/fixmatch_example.png)
<center>FixMatch Consistency Training 예시</center>

위 그림은 FixMatch의 Consistency Traning 예시입니다.
Unlabeled 데이터를 이용하여 Strong Augmented 데이터와, Week Augmented 데이터를 만듭니다.
그리고 구축한 딥러닝 모델에 넣어 각 데이터가 어떤 것을 예측하였는지 확률을 추출합니다.
이 확률을 기반으로 두 augmented 데이터의 label이 같도록 학습하는 것이 semi-supervised Learning에서 Consistency Training을 활용하는 방법입니다.

![](/img/in-post/2020/2020-12-13/consistency_training.png)
<center>Unsupervised Consistency Loss 예시</center>

본 논문 역시 noise가 추가된 문장과 원본 문장(Raw Data)의 label이 같도록 학습하는 방식으로 Consistency Training을 적용합니다.
여기서 Noise가 추가된 문장은 Back-translation을 이용하여 원본 데이터로부터 생성된 인공문장을 의미합니다.
두 데이터를 동일한 모델에 넣고 각각 분포를 추출한 다음 두 분포가 일치하도록 KL-Divergence를 이용하여 Loss를 구성하고 학습합니다.
즉 두 데이터의 분포를 일치 시킴으로써 두 데이터 사이의 Label 정보를 일치 시키는 것으로 볼 수 있습니다.
Consistency Loss의 식은 아래와 같습니다.

<center>$D( p_{\theta}(y|x) || p_{\theta}(y|x,e)$</center>
$e$ : 인공 문장을 생성할 때 삽입된 Noise  
$p_{\theta}(y|x)$ : 원본 문장의 분류 확률 분포    
$p_{\theta}(y|x,e)$ : 인공 문장의 분류 확률 분포     

##### Final Loss

UDA에서 제시한 방법론의 핵심은 <u>Label 데이터</u>로는 **Supervised Loss**를 구성하고 <u>Unlabeled 데이터</u>로는 **Consistency Loss**를 구성한 후 이 두개의 Loss를 합하여 모델을 학습하는 것입니다.



Label 데이터로 Supervised Loss를 만드는 방법은 일반적인 Cross-Entropy Loss를 만드는 방법과 같습니다.
Label 데이터의 문장 $x$을 모델에 넣어 실제 라벨인 $y$ 에 대한 확률 $p_{\theta}(y|x)$를 추출하고 이를 이용하여 Cross-Entropy Loss $-log p_{\theta}(y|x)$ 를 구성합니다.

Unlabel 데이터로 Consistency Loss를 구성하는 방법은 원본데이터와 인공데이터 사이의 KL-Divergence Loss를 계산하는 것 입니다.
TD-IDF, Back Translation 등을 활용하여 Unlabel 데이터의 문장 $x$으로부터 인공 문장 $\hat{x}$ 을 생성합니다.
인공문장 $\hat{x}$ 파라미터 $\theta$ 가 고정되어 있는 모델에 넣어 $p_{\theta}(y|x)$









## Reference
- [[BLOG]](https://nlp.stanford.edu/blog/maximum-likelihood-decoding-with-rnns-the-good-the-bad-and-the-ugly/#:~:text=Temperature%20sampling%20is%20a%20standard,semantic%20distortions%20in%20the%20process.) Maximum Likelihood Decoding with RNNs - the good, the bad, and the ugly
- [[PAPER]](https://arxiv.org/abs/1904.12848) Unsupervised Data Augmentation for Consistency Training, Qizhe at el, NIPS 2019
- [[GITHUB]](https://github.com/google-research/uda) Unsupervised Data Augmentation Tensorflow Implementation