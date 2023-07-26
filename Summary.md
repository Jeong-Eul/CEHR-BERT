# CEHR-BERT: Incorporating temporal information from structured EHR data to improve prediction tasks  

<blockquote>
Author: Chao Pang, Xinzhuo Jiang, Krishna S. Kalluri, Matthew Spotnitz<br>    
Data source: IColumbia University Iriving Medical Center-York Presbyterian Hospital<br>
Date: Nov, 2021  
</blockquote>
<br>
<p align = "center"> <img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/architecture.jpg?raw=true" width = 60%></p>
<br>

## Introduction  
## Related work
## Method  

### Data  
저자는 Columbia University Irving Medical Center-New York Presbyterian Hospital (CUIMCNYP)에서 EHR 데이터를 가져온 후, OHDSI라는 open-science community가 개발한 데이터 모델인 OMOP(Observational Medical Out-comes Partnership)라는 양식에 맞게 변환하였다.<br>
<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/OHDSI-OurJourney.png?raw=true" width = 50%></p>
<br>
EHR 데이터는 병원마다 용어도 다르고, 양식도 다른데 OHDSI는 이러한 다양한 용어와 양식을 가진 EHR 데이터를 통일된 용어와 양식으로 바꾸어 준다. OMOP는 수치형 데이터와 clinical domain(visits, conditions, procedures, medications, lab tests, vital signs 등)가 포함되어 있다. (Physionet에 확인해보니, MIMIC-IV를 OMOP로 변환한 데이터셋이 있다.)
<br>

<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/ohdsi.jpg?raw=true" width = 50%></p>
<p align ='center'><a href="https://www.ohdsi.org/data-standardization/">참조: OHDSI 웹페이지 링크</a></p>

저자는 변환한 OMOP 데이터 셋에서 conditions, procedures, medication의 3가지 clinical domain 만을 활용했다고 밝혔다.

### Data processing and patient representation(sequence)  

논문에서 제시한 환자의 시퀀스를 표현하는 벡터를 이해하기 위해 필요한 notation은 다음 사진과 같다.  

<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/notation1.jpg?raw=true" width = 70%></p>

<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/notation2.jpg?raw=true" width = 70%></p>


### Embedding Technique for input of BERT  

patient history를 representation 하기 위한 4가지 embedding이 있다.  
<blockquote>
1) Concept embedding<br>  
2) Visit embedding<br>  
3) Time embedding<br>  
4) Age embedding  
</blockquote>

<br>

---  

<b>Concept embedding</b>: Concept embeddings were used to capture the numeric representations of the concept codes based on underlying cooccurrence statistics  
$\to$ OMOP 데이터 셋 양식에 따르면 concept table의 "concept_code"라는 열이 있다. 아래 사진은 MIMIC을 OMOP로 변환한 데이터셋에서의 예시이다. 그림에서 볼 수 있듯 "concept_code"라고 하는 것은 짧은 자연어(구; Phrase)로 되어 있다. 
<br>
<p align ="center"><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/concept_code_MIMIC_OMOP.jpg?raw=true" width = 40% height = 600></p>
<br>

아마도 환자의 visit 안에 여러 개의 concept_code가 시간과 매칭되어 있을 것이다. 이렇게 자연어로 표현된 "concept_code"를 co-occurrence statistics에 기반한 real vector로 바꾸어 임베딩을 한다.  
<br>
여기서 co-occurence statistics가 명확하게 무엇인지 잘 모르겠다. 동시발생행렬을 의미하는 것인가?  

등장하는 단어(구)의 빈도를 바탕으로 수치로 바꾸어주는 일종의 변환이 있을 것이라 생각된다.  
예를 들어 "you say goodbye and i say hello" 라는 문장이 있으면 "you"의 동시 발생 빈도는 [0, 1, 0, 0, 0, 0] 이 되는 것처럼 말이다.  

<p align ="center"><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/co-occurence.jpg?raw=true" width = 60%></p>

<br>

<b>Visit segment embedding</b>: Visit segment embedding은 기존 BERT에서 segment 토큰과 똑같은 역할을 한다. 참고로 비슷한 연구인 BEHRT에서도 사용했다.  

<p align ="center"><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/BEHRT.jpg?raw=true"></p>

<br>

<b>Time embedding</b>: 개인적으로 다른 EHR 데이터의 Bert 적용한 다양한 연구와 비교하여 ATT와 더불어 본 논문의 차별점이 될 수 있는 본 논문에서의 필살기라 생각된다. <i>간단하게 요약하자면 Time2Vec 이라는 방법론이 있는데 이는 시계열을 잘 representation 할 수 있는 방법이다.</i> 논문의 이름은 "Time2Vec: Learning a Vector Representation of Time" 이다. 시간에 대한 Representation을 수행하는데 특히 주기 패턴과 비주기 패턴에 강건하며 time resolution을 변경하더라도 중요한 정보는 담아내고, 여러 모델에 쉽게 적용이 가능하다는 특징이 있다.

<p align ="center"><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/t2v.png?raw=true"></p>

<br>
이 정보는 나에게 있어서 본 논문의 아키텍처보다 더 재밌는 사실이었다. 내가 하고 싶은 연구에 꼭 필요한 방법인 것 같다는 생각이 들었고, time을 representation 하는 다른 방법도 비교해보고 싶기 떄문에 이 논문은 꼭 시간내서 읽어야겠다. <b>(Lab meeting 등을 이용하여도 좋을 것 같다.)</b>  

<b>Age embedding</b>: 자세하게 나와있지는 않지만 환자가 visit 할 때의 나이를 순차적으로 이어 붙인 것 같다.  

---

Time embedding은  visit에 대한 절대적인 시간 정보를 포함하고 있으며 Age embedding은 visit에 대한 상대적인 시간 정보를 포함하고 있기 때문에 계절적 패턴과 나이와 관련된 질병(2형 당뇨 등)의 condition을 잘 capture 할 수 있다.

이를 바탕으로 BERT의 입력이 될 Input vector를 만든다.  
[상세과정]  

1. Concept embedding + Visit segment embedding  =>  CV 라고 표현한다면
2. Concat([Age emnedding, Time embedding, CV]) => Concatenated Embeddings  
3. Fully Connected layer(Concatenated Embeddings) => temporal concept embeddings  

즉 3번의 Temporal concept embedding이 Bert의 입력이 된다.(FC layer의 출력차원은 Concept embedding 차원과 같은 차원으로 만들어 주는 것 같다.)

<p align ="center"><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/temporal_concept_emb_describe.jpg?raw=true" width = 70%></p>

### Training method  

<blockquote>
1) MLM<br>  
2) VTP<br>  
</blockquote>

<br>

<b>MLM(Masked Language Model)</b>: Masked Language Model을 도입함으로써 BERT 는 문맥에서 빠진 단어를 맞추기 위해 양방향으로 문맥을 살피며 문장 전체를 이해하게 된다. 이러한 MLM을 CEHR-BERT에 적용하였다고한다. 입력되는 시퀀스(Temporal concept embedding)의 일부를 랜덤하게 mask 하고, mask 된 부분을 맞추도록 훈련된다.  

<p align = 'center'><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/MLM.jpg?raw=true" width = 50%></p>

<br>

<b>VTP</b>: VTP는 Visit Type Prediction의 약어이다. 요약하면, 위에서 임베딩한 medical concept이 들어있는 embedding vector를 환자의 visit type으로 변환하는 작업이다. 때문에 VTP는 일종의 language translation task로 볼 수 있다. 저자는 다양한 visit type(방문 유형)이 다양한 medical concept와 서로 연관이 있다라는 가정을 세웠다. 예시를 들어줬으면 좋겠지만 설명이 없어 아쉬웠다. 나름대로 해석을 해보자면 다음과 같은 연관이 있을 것 같다.  

example 1. emergency visit 인 경우 응급 상황에 처한 환자들에게 즉각적인 응급 치료와 처치를 제공하는 것이 목적이므로 medical concept에는 기록 시간 빈도가 높다거나 응급 질환에 자주 사용되는 약물들의 사용이 있을 것 같다.  

example 2. oup patient visit 인 경우 일반적인 건강 검진, 진단, 치료, 상담이 목적이므로 예약된 시간에 의사와 상담이 이루어진다. 따라서 medical concept에는 기록 시간 빈도가 낮다거나 일반적인 약물들의 사용이 있을 것 같다.  

이렇듯 저자의 말대로 visit type과 medical concept 사이에 연관이 있기 때문에 Visit Type을 잘 맞추도록 하는 sub task를 통해 BERT 모델에게 추가적인 문맥을 파악할 수 있도록 할 수 있다.  

<p align = 'center'><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/NTM.jpg?raw=true" width =45%></p>


<b>VTP를 구현하는 법</b>: 환자의 Visit type(Inpatient visit, Outpatient visit, emergency visit, Masked visit) sequence, Age Embedding, Time embedding 을 같은 과정(Temporal transformation)을 거쳐 일정 크기의 벡터로 만들고, 이를 디코더의 입력으로 넣는다. 그 다음 Encoder(BERT)에서 나온 d_model 차원의 벡터를 받아(key와 value를 받지 않을까 싶다) Multihead Attention으로 퓨전해주어 표현학습하고, 이를 기반으로 다시 Visit type이 무엇이었는지 맞추는 과정을 수행한다. BERT의 laguage translation과 정확히 동일한 과정으로 진행되는 것 같다.  
<br>
이 모든 내용을 기반으로 모델의 아키텍처를 이해하기 쉽게 다시 그리면 다음과 같다.  

<br>

<p align ="center"><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/model_architecture_reproduction.jpg?raw=true"></p>

<p align ="center"><i>그림에서 Visit segment + Concept Embedding에 randomly masked 가 추가되어야 함(7/25 update)</i></p>

## Experiment  

### Experiment Setup  

1) Patient selection(2.4M 명의 환자와 그에 따른 184.7M 개의 관측치가 생성됨):  
<blockquote>적어도 1회 이상 방문한 환자<br>  
관측치가 5개 이상인 환자</blockquote>  

<br>

2) Clinical Domain:  
<blockquote>1. Condition<br>
    2. Procedure<br>
    3. Medication</blockquote>
<br>
<p align='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/data_example.jpg?raw=true>" width = 80%></p>  

MIMIC의 경우에 omop_data_csv/condition_occurrence.csv, procedure_occurrence.csv, drug_exposure.csv에 존재하는 Concept code를 가지고 {Concept code : SCRIPT} 의 딕셔너리 형태로 저장된 achilles_json/condition_treemap.json(여기에 AGE 포함), achilles_json/drug_treemap.json, achilles_json/procedure_treemap.json 을 매핑시켜 데이터를 구성할 수 있을 것 같다.  
<br>
<p align='center'>Physionet에서 제공하는 MIMIC-IV의 OMOP 데이터 파일</p>
<p align ="center"><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/physionet_data_files_img.jpg?raw=true"></p>
<br>
<p align='center'>OMOP.csv 예시</p>
<p align ="center"><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/OMOP_CSV_example.jpg?raw=true"></p>
<br>
<p align='center'>concept code를 매핑하는데 사용하는 json 파일:Procedure</p>
<p align ="center"><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/procedure_treemap.jpg?raw=true"></p>

<br>

