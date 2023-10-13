# NLP
이미지 기반의 딥러닝 (이미지 분류, 이미지 객체 탐지)을 학습하였으니, 자연어 처리에 대해 학습하고자 한다.
자연어에 대한 학습이 끝나면, 평소 궁금했던 음성 데이터의 처리에 대해서도 학습하고 싶다.


# 학습 로그


<br />



* 09.26 : 자연어 처리의 가장 기본인 토크나이저에 대하여 학습을 하였다. 딥러닝 이전의 전통적인 머신러닝 자연어 처리에 사용되었던 sklearn의 CountVectorizer (향후에는 쓰이지 않을 것 같다.)과 tensorflow keras의 Tokenizer에 대하여 학습을 하였다.  
전통적인 자연어 처리 머신러닝 (감정 분류)이 어떠한 방식으로 이루어 졌는지 매우 단순하게 구현해 보았다.

머신러닝은 TF-Idf값이 들어있는 word bag을 통하여 학습을 한 뒤, 분류하는 방식으로 이루어져 있었다.  
참고 링크 : https://medium.com/@bedigunjit/simple-guide-to-text-classification-nlp-using-svm-and-naive-bayes-with-python-421db3a72d34

* 09.27 : 어제 공부하였던 keras의 Tokenizer은 단순히 index 번호에 따라 one-hot-encoding 으로 vectorize 하므로 단어 간의 유사성을 파악하지도 못하고, 의미적 문법적으로 정보를 함축하고 있지도 않다. 전통적인 머신러닝에는 대입해서 사용할 수 있겠지만 딥러닝에서 사용하기 위해서 한가지 절차를 더 밟게 되는데, 바로 Word Embedding이다. Word Embedding은 연관성 있는 단어들을 군집화하여 multi-dimension 공간에 vector 로 표시하고, 단어나 문장을 vector space 로 끼워 넣음 (embedding)을 뜻한다. 이전에 머신러닝때 피쳐의 수를 줄이기 위해 사용하였던 PCA와 유사한 것 같았다. 단어 갯수가 1만개라고 하였을 때, Tokenizer을 이용하면 1만개의 숫자로 정수 인코딩이 되고, 이를 to_categorical을 이용하여 원 핫 인코딩을 해주면 각 단어당 1만개의 컬럼(이 중 딱 1개의 값만 1을 가지고, 나머지는 0)을 가지게 된다. 이 단어들을 그냥 학습에 사용한다고 해도 10000x10000이라는 매우 큰 비효율적인 의미없는 큰 정보들을 이용하게 된다. 이때 word2vec을 이용하여 약 300개의 군집정도로 함축한다고 하면, 각 단어당 300개의 컬럼을 가지게 된다 ( (1x10000) x (10000x300)  =  (1x300), 내적).  이렇게 임베딩 된 벡터들은 코사인 유사도를 통하여 단어간의 유사성도 파악이 가능하고, 의미적 문법적 정보도 함축하고 있고, 전이 학습도 가능하여 크기도 줄어들어 학습에도 사용하기 용이해지게 된다.
* 09.28 : 강의상으로는 word2vec 이후 바로 RNN으로 넘어가지만, 좀 더 자세히 알아보고 싶어 따로 공부해 보았다. word2vec에서의 가장 큰 문제점이라 한다면, 출력층에서 소프트맥스 함수를 지난 단어 집합 크기의 벡터와 실제값인 원-핫 벡터와의 오차를 구하고 이로부터 임베딩 테이블에 있는 '모든 단어'에 대한 임베딩 벡터 값을 업데이트한다는 것이다. 물론 임베딩 벡터 값을 업데이트 한다는 것은 중요하고, 더 정밀한 벡터값이 나올 수 있다. 그러나, 현재 학습을 하고 있는 단어와 별 연관 관계가 없는 수많은 단어의 임베딩 벡터값까지 업데이트하는 것은 비효율적이다. 특히 단어의 수가 많으면 많을수록 훨씬 더 많은 시간이 걸리게 된다. 이를 해결하기 위한 것이 네거티브 샘플링인데, 네거티브 샘플링은 Word2Vec이 학습 과정에서 전체 단어 집합이 아니라 일부 단어 집합에만 집중할 수 있도록 하는 방법이다. 무작위로 주변 단어가 아닌 단어들을 일부 가져오고, 하나의 중심 단어에 대해서 전체 단어 집합보다 훨씬 작은 단어 집합을 만들어놓고 마지막 단계를 이진 분류 문제로 변환하여 주변 단어들을 긍정(positive), 랜덤으로 샘플링 된 단어들을 부정(negative)으로 레이블링한다.  

사실 이 네거티브 샘플링은 이미지 객체 탐지 딥러닝 모델들에서 본 기억이 있다. 특히 기억에 남는 것이, 객체 탐지 모델의 가장 초창기 모델인 R-CNN에서 파지티브 샘플과 네거티브 샘플의 비율을 각 feature extraction, classification, bounding box regression 부분들의 각 부분마다 다르게 설정하여 학습하였기에, 매우 비효율적이라는 생각을 하며 공부했던 기억이 난다.

* 09.29 : Skip_gram with Negative samples를 구현한 코드를 보던 중, keras의 Embedding layer에 대하여 의문이 생겨 조사를 해 보았다. 나는 Embedding을 할 때, Tokenizer를 통하여 정수로 인코딩된 값들을 to_categorical을 통하여 원 핫 인코딩을 한 뒤, embedding table과의 내적을 통하여 embedding vector을 구한다고 알고 있었다. 그러나, 이 코드에서는 정수인코딩한 값을 그대로 Embedding layer에 인풋으로 집어 넣으면, 아웃풋으로 embedding table에서 해당되는 값의 Embedding vector가 나왔다. 이 과정이 이해가 안되서 keras의 Embedding layer코드 페이지를 살펴보니, embedding layer의 weights는 input 값과 내적하는 것이 아닌, 단순히 '몇번째 row의 value로 바꿀것이냐' 를 선택하는 것이였다. 즉, 깊게 생각할 것도 없이 (단어 갯수 x 임베딩 차원)의 가중치가 모델안에 있으면, 3번째 단어의 임베딩 벡터를 구하고 싶으면 저 가중치에서 3번째 줄을 단순히 가져오는 것이였다. 뭔가 허무하긴 하지만, 생각해보니 머신러닝에 사용하기 위한 데이터가 아니면 굳이 원핫인코딩을 하지 않아도 그냥 정수값을 딥러닝 네트워크에 투입하면 된다는 것을 깨달았다.
* 09.30 : RNN에 대해서 자세하게 학습을 해 보았다. 이전에도 공부한 적이 있는 모델이였지만, 세부적으로 들여다보니 이해가 되지 않는 부분이 있어 공들여서 살펴 보았다. 가장 의문이였던 점 중 하나는 torch.nn에 있는 nn.RNN에서 output의 size가 hidden_size와 같다는 점, 즉 내가 원하는 dimension의 output을 얻기 위해서는 hidden_size를 원하는 사이즈로 설정해 주어야 한다는 것이였다. 이전에 구현했던 RNN에서는, hidden layer를 통과한 hidden state에 nn.Linear을 통하여 원하는 사이즈의 output을 얻을 수 있었는데, nn.RNN에서는 그렇치 않다는 것이였다. 이유가 무엇인지 들여다보니 nn.RNN에서의 output은 최종 hidden state를 출력하는 것이였다. 즉 원하는 output을 얻기 위해서는 nn.RNN에서의 output에 추가로 nn.Linear을 통하여 가중치를 곱해주어야 한다는 것이다. 이전에는 복잡해서 대충 이해하고 넘어갔던 모델을, 시간을 들여 차근차근 단계별로 공부하여 어떻게 된 구조이고 어떠한 방식으로 작동하는 것인지 완벽하게 이해하게 되었다.
* 10.01 : 어제까지 학습하였던 내용들을 바탕으로 직접 코드들을 작성해보았다. 문장들을 tokenizer을 통해 정수 인코딩을 한 뒤 패딩을 통해 같은 길이의 시퀀스를 가진 문장들로 변환해 주었고, 이후 Embedding layer을 통과하여 embedding 후 RNN을 통하여 분류하는 모델을 구현해 보았다.
* 10.03 : 평소 딥러닝 프레임워크로 파이토치만을 사용하였지만, 강의에서 keras를 사용하기에 keras로 LSTM/GRU의 코드를 작성해 사용해 보았는데, keras가 훨씬 더 코드가 간단하고 편리하다는 느낌을 받았다. torch.nn의 LSTM과 keras의 LSTM의 차이점이라 한다면, 디폴트로 나오는 output값이 다르다는 것이다. nn.LSTM의 경우 디폴트가 many to many에 쓰이는, 최종 히든 레이어값을 전부 출력하게 되고, 이에 따라 many to one에서 사용하기 위해서는 output[:,-1]을 통하여 가장 마지막 시퀀스의 최종 은닉층 값을 가져와야 한다. Keras의 LSTM은 반대로, 디폴트가 마지막 시퀀스의 최종 은닉층 값을 가져오게 되어 있고, return_sequences = True로 설정하면 모든 시퀀스의 최종 히든 레이어 통과값을 가져오게 되고, return_state=True로 설정하면 마지막 은닉층 값과 마지막 셀 상태를 반환하게 된다.
* 10.05 : 지금까지 사용해왔던 tokenizer는 띄어쓰기나 특수문자들을 기준으로 분리해 낸 다음 번호를 매기는 것이였다. 그러나 한국어나 일본어 같은 경우 띄어쓰기 만으로는 단어들끼리의 구별이 안되므로 (특히 품사같이 무조건 다른 단어와 붙어있는 경우) 다른 방법이 필요한데, 이때 크게 2가지 방법이 있다. 하나는 전통적인 자연어 처리에 사용되는 것처럼 모든 단어들에 대한 단어집을 만들어 놓는 사전 기반 방식인데, 이를 위해서는 전문적인 지식도 필요하고, 매우 많은 노력이 들어가게 된다. 또 다른 방법이 sub-word 방식으로, 하나의 단어는 의미 있는 여러 단어들의 조합으로 구성된 경우가 많기 때문에, 단어를 여러 단어로 분리하여 보겠다는 방법을 의미한다.
* 10.06 : 개체명 인식(Named Entity Recognition)에 대하여 학습을 하였는데, 학습을 위한 데이터셋을 보기만 해도 얼마나 많은 노력이 들어갔는지 가늠이 되지 않을 정도였다. 각 단어마다 어떤 품사인지, 어떤 종류의 개체명인지, 그 개체명의 시작인지 중간인지 등 매우 많은 노력이 들어간 데이터셋 이였는데, 확실히 개체명 같은 고유어에 대해서 탐색하기 위해서는 이런 자세한 데이터셋이 필요할 것 같다. 최근의 NER은 어떠한 모델일지 추후에 공부하는 것이 기대된다. 
* 10.08 : 문득 cross entropy loss를 구할 때 정답 라벨 값을 원핫인코딩을 안해주어도 되는지 의문이 들었다. 생각해보니 이미지 분류때도 그렇고, 그동안 딥러닝 모델들을 사용하면서 라벨값을 원핫인코딩 해준 적은 없는 것 같은데, 그러면 cross entropy loss 함수에 자동으로 포함이 되어있는 건지 궁금해져서 찾아보니, 그렇다고 한다. 라벨 값을 integer로 주면, 자동으로 원핫 후에 loss를 구하여 output으로 내주는 함수였던 것이다. 그리고 만약 라벨값이 정수이면, sparse categorical cross entropy를 사용하는 것이 결과는 똑같지만 연산량이 좀 더 줄어든다고 한다.
* 10.10 : seq2seq (encoder-decoder)모델을 공부하던 도중 Image Captioning이라는 모델을 알게되어 호기심에 조사를 해 보았다. 이전에 사용해보았던 CNN모델인 GoogleNet을 Encoder로 사용하여 이미지의 특징을 추출하고, 이 특징들을 임베딩 벡터 차원으로 만든다. 그 후에 이 이미지에 대한 문장을 기존에 하던 것처럼 tokenizer를 사용해 단어별로 정수 인코딩을 해준 뒤, 임베딩 레이어를 통하여 임베딩 벡터 차원으로 만들고, 아까 추출한 이미지의 벡터를 가장 처음에 오게 붙인다. 이렇게 되면 기존 문장의 시퀀스 길이+1의 시퀀스가 생기게 되고, 이미지의 값을 LSTM에 인풋으로 넣는것으로 Decoder가 작동하게 된다.  기본적으로는 여지껏 배운 것들을 합치는 느낌이라 재미도 있고 이해도 되었지만, 문장을 만들때의 매커니즘을 생각해보면, 각 seq마다 hidden layer을 통과한 뒤에 가장 확률이 높은 단어를 선택하며 진행하게 되는데, 이렇게 된다면 맥락적인 부분에서 어색한 부분이 생겨날 수도 있고, 만약 하나의 단어가 잘못되면 그의 기반한 모든 결과가 망가지는 위험이 있다. 이를 보완하기 위해 Beam Search라는 것이 있다고 하는데, 이것에 관련된 코드를 차근차근 살펴보며 공부를 해보기로 결심했다.
* 10.12 : 
