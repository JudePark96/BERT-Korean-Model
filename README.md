# BERT with SentencePiece for Korean text
This is a repository of Korean BERT model with SentencePiece tokenizer.

* BERT-Base, Korean Model: 12-layer, 768-hidden, 12-heads, 110M parameters(비공개)

## SentencePiece tokenizer 학습
 한국어 전용 BERT 모델을 만들기 위해 Google의 [SentencePiece](https://github.com/google/sentencepiece)을 사용하였습니다. 약 1억 8천만 문장(위키피디아, 뉴스 데이터)을 활용하여 32,000개의 vocabulary (subwords)를 학습하였습니다. 모델 type은 <code>--model_type</code> 옵션을 이용하여 bpe type을 사용 하였습니다. 
 <br>
 
```python
import sentencepiece as spm

parameter = '--input={} --model_prefix={} --vocab_size={} --model_type={}'

input_file = 'corpus.txt'
vocab_size = 32000
prefix = 'bert_kor'
model_type = 'bpe'
cmd = parameter.format(input_file, prefix, vocab_size, model_type)

spm.SentencePieceTrainer.Train(cmd)
```   
<br>
BERT 모델에 SentencePiece 사전을 사용하기 위해서 [PAD], [CLS], [SEP], [MASK]등의 token을 추가해줘야 합니다. 이를 위해 <code>--user_defined_symbols</code> 옵션을 적용하였습니다.
<br>
<br>
 
```python
import sentencepiece as spm

parameter = '--input={} --model_prefix={} --vocab_size={} --user_defined_symbols={}'

input_file = 'corpus.txt'
vocab_size = 32000
prefix = 'bert_kor'
user_defined_symbols = '[PAD],[UNK],[CLS],[SEP],[MASK]'
cmd = parameter.format(input_file, prefix, vocab_size,user_defined_symbols)

spm.SentencePieceTrainer.Train(cmd)
```   
<br>

SentencePiece 출력 예시
```python
import sentencepiece as spm

sp = spm.SentencePieceProcessor()
sp.Load('{}.model'.format(prefix))
token = sp.EncodeAsPieces('나는 오늘 아침밥을 먹었다.')
print(token)

['▁나는', '▁오늘', '▁아침', '밥을', '▁먹었다', '.']
```   
## 사전 학습 데이터 준비  
create_pretraining_data.py를 사용하여 BERT pre-training에 적합한 <code>.tfrecord</code> 파일 형식으로 변환하였습니다. 학습 데이터의 구성은 "Next sentence prediction" Task을 위해 한 줄에 한 문장씩 구성하고 Document 사이에는 빈 줄을 삽입할 것을 권장하고 있습니다.
 
~~~
라 토스카(La Tosca)는 1887년에 프랑스 극작가 사르두가 배우 사라 베르나르를 위해 만든 작품이다.
1887년 파리에서 처음 상연되었다.
1990년 베르나르를 주인공으로 미국 뉴욕에서 재상연되었다.
1800년 6월 중순의 이탈리아 로마를 배경으로 하며, 당시의 시대적 상황 하에서 이야기가 전개된다.
1900년, 사르두의 연극은 푸치니의 오페라 토스카로 새롭게 각색되었다.
베르디는 사드루의 각본에서 "갑작스런 종결" 부분을 수정할 것을 권하지만, 사르루는 이를 거절한다.
후에, 푸치니 또한 사르두의 각본에서 "갑작스런 종결부분"을 수정할 것을 제안하지만 끝내 사르두를 설득하지 못했다.

2008년 하계 올림픽의 복싱 남자 라이트급 종목은 8월 11일일부터 8월 24일까지 중화인민공화국의 베이징에 있는 베이징 노동자 체육관에서 열렸다.
27개국에서 27명의 선수가 참가하였다.
2008년 하계 올림픽 복싱 남자 라이트급 경기는 개최 도시인 베이징에 있는 베이징 노동자 체육관에서 경기가 열렸다.

도리데 시는 일본 이바라키현의 남부에 있는 시이다.
간토 평야에 위치하고 도네 강과 고카이 강에 접하고 있다.
이 때문인지 일찍이 수해가 많았다.
현재에도 시 남서부의 대지를 제외하면 시역이 많은 부분이 침수의 위험성이 있다.
그러나 최근에 도네 강, 고카이 강 등의 제방의 고기능화에 의해 하천의 범람에 의한 침수 피해는 거의 없어졌다.
한편 집중호우에 의해 시내의 저지 등에서는 도로가 일부 침수하는 등의 피해가 일어난다.
~~~
## BERT Pre-training
학습 시간 문제로 **한국어 위키데이터(2019.01 dump file, 약 350만 문장)** 을 사용하여 학습을 진행하였으며, 모델의 하이퍼파라미터는 논문과 동일하게 사용하였습니다. 오리지널 논문과 동일하게 구축하고자 n-gram masking은 적용하지 않았습니다. 학습 방법은 논문에 서술된 것처럼 128 length 90%, 512 length 10%씩 학습을 진행하여, 총 100만 step을 진행했습니다. 
<br>
<br>
* 학습 파라미터(seq_length=128)
```python
learning_rate = 1e-4
train_batch_size = 256 
max_seq_length = 128
masked_lm_prob = 0.15
max_predictions_per_seq = 20
num_train_steps = 900000
```   

* 학습 파라미터(seq_length=512)
```python
learning_rate = 2e-5
train_batch_size = 256 
max_seq_length = 512
masked_lm_prob = 0.15
max_predictions_per_seq = 77
num_train_steps = 100000
```   

## 성능 평가  
BERT Model 성능 평가는 한국어 SQuAD Task [KorQuAD](https://korquad.github.io/)로 평가하였습니다. 성능 비교를 위해 BERT-Multilingual Model과 실험을 진행하였으며, 하이퍼파라미터는 기본값으로 사용하였습니다. 성능 결과는 아래와 같습니다.   

| Model | F1(Dev Set 기준) |
|:---:|:---:|
| BERT-Base, Multilingual Cased | 89.9% |
| **BERT-Base, Korean Model(our model)** | 87.8% |
<br>

## Step별 성능 평가
Base Model 기준으로 총 120만 step을 학습 시켜보았습니다. 각 step별 KorQuAD Task의 F1 Score는 아래와 같습니다. 100만 Step 이상 학습을 진행했을 경우에도 F1 Score가 늘어나는 것을 확인할 수 있었습니다. 

* Base Model(12-layer, 768-hidden, 12-heads)<br>

| Step | seq_length | F1 |
|:---:|:---:|:---:|
| 40만 | 128 | 73.98% |
| 60만 | 128 | 77.15% |
| 90만 | 128| 78.64% |
| 100만 | 512 | 87.8% |
| 120만 | 512 | 88.19% |

## 기타
  Base model은 한국어 Pre-training 모델 연구와 실험을 위한 한국어 BERT Model입니다. 모델 사용시 반드시 출처를 밝혀주시길 바랍니다.
  <br>
  <br>
  한국어 BERT Model 학습이나 모델 사용 관련하여 궁금한 사항이 있으시면 oh31400@naver.com 메일로 문의 부탁립니다. 










