# news data processing by KoBert


## 뉴스 카테고리 분류
------------


<br>
<br>

## 문장 띄어쓰기 확인
-----------

+ 데이터셋 : 총 12,600 개의 기사에서 특정 문장을 제거한 뒤 286,502의 data 추출
  + ['o','『', '(', '┌','│', '└', 'ㄴ', '┌','├','◎', '[', '■', 'ㄱ', '-', '.', '<']로 시작하는 문장 제거
  + ['쪽', '-', '>', ']']로 끝나는 문장 제거
  + [')']
  + train data : 183,360개, dataset 의 64%
  + val data : 45,841, dataset 의 20%
  + test data : 57,301, dataset 의 16%
  + Test2 data : OCR 결과파일 내 기사 텍스트를 이용해 모델 평가
+ KoBertTokenizer loading 오류로 직접 가져왔다.   [[github]](https://github.com/monologg/KoBERT-Transformers/blob/master/kobert_transformers/tokenization_kobert.py)
+ 'monologg/kobert'
  + 10 epoch : best model
    + validation ( loss : 0.0308, f1 : 0.9300)
  + 13 epoch
    + validation ( loss : 0.0060, f1 : 0.9811 )
    + test( 기사 원문의 xml 파일 ) : ( loss : 0.0354, f1 : 0.9293)

+ 한계
  + 한글에 띄어쓰기, 붙여쓰기 둘 다 허용하는 단어가 많기 때문에 원문의 띄어쓰기와 다를 수 있다.

<br>
<br>

출처 
* [BERT를 이용한 한국어 띄어쓰기 모델 만들기 - 01. 데이터 준비](https://bhchoi.github.io/post/nlp/dev/bert_korean_spacing_01/)
* [BERT를 이용한 한국어 띄어쓰기 모델 만들기 - 02. 데이터 전처리](https://bhchoi.github.io/post/nlp/dev/bert_korean_spacing_02/)
* [BERT를 이용한 한국어 띄어쓰기 모델 만들기 - 03. 모델](https://bhchoi.github.io/post/nlp/dev/bert_korean_spacing_03/)
* [BERT를 이용한 한국어 띄어쓰기 모델 만들기 - 04. 학습](https://bhchoi.github.io/post/nlp/dev/bert_korean_spacing_04/)
* [Docs : pytorch lightning 사용법](https://pytorch-lightning.readthedocs.io/en/latest/common/lightning_module.html)
* [Docs : pytorch lightning trainer 메소드 종류](https://pytorch-lightning.readthedocs.io/en/stable/common/trainer.html)

<br>
<br>

## 뉴스 단락 연결 여부 검사  
-----------
 

### 문장 연결성 파악

<br>

+ 문장 연결성 파악
  + 단락으로 분리된 이전 문장과 다음 문장이 자연스럽게 연결되는지 확인
  
    + MASK 에 들어갈 단어 예측
      + MASK : 단락이 분리되었을 때 이어지는 단어
      + 2개의 단락에서 문장이 끊어졌을 때 1개의 문장으로
붙여지는지 확인
    + 다음 문장 예측 : 두 문장 사이에 유사도 측정

<br>

+ 단락 유사도 파악
    + 이전 단락과 다음 단락이 자연스럽게 연결되는지 확인
      + 단락의 주제가 같은지 파악 ⇒ 한 페이지에 있는 여러 기사들의 주제가 거의 동일하기 때문에 무의미하다고 생각되어 실행하지 않았다.
      + MASK 에 들어갈 단어 예측
        + MASK : 단락이 분리되었을 때 이어지는 단어
      + 다음 문장 예측 : 두 문장 사이에 유사도 측정

<br>


### 모델

1. 다음 문장 예측
  + 모델에 테스트할 앞뒤 단락을 넣었을 때 T/F 를 잘 판별해낸다면, 두단락의 유사도/연결성이 있는지, 한 문장으로 볼 수 있을 지 확인이 가능하다고 가정
  + 데이터셋 : 총 12,600 개의 기사에서 455,188 개의 data 추출
    + train data : 291,320 개, dataset 의 64%
    + validation data : 72,830 개, dataset 의 20%
    + test data : 91,038 개, dataset 의 16%
    + Test2 data : OCR 결과파일 내 기사 텍스트를 이용해 모델 평가

    + 데이터셋의 형태 : [(이전 문장, 다음 문장, label )], label 이 1 이면 오답, 0 이면 정답
      + [(' 의미에서 종택의 이름도 그렇게 정했다고 한다. 현판은 ... ', '과잉진료와 오진으로 인한 피해가 조기 검진의 효과보다 크고,유방암 사망률...', 1 ),   
    ('나 정물사진은 절제된 프레이밍과 세련된 컬러와 톤이 ...',  '작가가 발표한 작품들은 누구나 폭넓게 이해할 수 있는 소재와 표현방식...', 0 ) ], ...

  + 방법

    + 학습된 BertForNextSentencePrediction 모델을 기사 데이터로
추가 학습

    + 균형 데이터이기 때문에 f1-score 을 보지 않고 정확도만 보아도
될 것으로 생각했다.
    + 확률값과 정답([0, 1])의 차이가 심하기 때문에 loss 값은
중요하지 않다고 판단했다.
    + 'klue/bert-base'
      + 3 epoch
        + validation ( loss : 0.3855, accuracy : 0.9246 )
        + test ( loss : 0.3870, accuracy : 0.9230 )    
    + 'snunlp/KR-Medium'
      + 2 epoch : best model
        + validation ( accuracy : 0.981)
        + test ( loss : 0.3316, accuracy : 0.9807)
      + 3 epoch
        + validation ( loss : 0.3333, accuracy : 0.9793 )
        + Test ( loss : 0.3329, accuracy : 0.9796)
        + Test2 ( 기사 원문의 xml 파일 ) : (loss : 0.3654, accuracy : 0.9459 )

  2. Mask 에 들어갈 단어 예측 ( 진행 중 )
