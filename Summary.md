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

- 기존 EHR의 BERT 적용 관련 연구들은 시간의 간격을 두고 수집되는 EHR 데이터의 Temporal 특성을 완전히 모델링하지 못했음  
- Text로 표현된 medical history는 임베딩 효과가 좋지만, temporal interval이 있는 경우에는 그러지 못했음  
- BERT는 입력된 문장 다음 문장을 예측하는 NSP, 입력 문장의 일부 단어를 가린 후 입력 문장을 가려진 단어를 포함하여 복원하는 MLM 으로 학습이 이루어지는데, EHR 데이터 같은 경우에 환자 전체 기록을 연속된 하나의 문서로 볼 수 없음 -> 새로운 학습 방법이 필요함
<br>

## Related work

 - BEHRT: 입력 값으로 diagnosis code, SEP 토큰 사용 -> 다양한 clinical domain(procedure, medication 등)을 충분히 활용하지 않았음  
 - G-BERT: 적은 데이터셋(20K patient)로 약물 추천시스템을 위해 개발되었음  
 - Med-BERT: MLM과 더불어 sub task로서 LOS가 7일 이상일지 아닐지를 예측하여 훈련되었음. 하지만  age, visit segment embedding, sep 토큰을 사용하지않았으며 temporal information을 활용하지 않았음  
 - Peng et al."Temproal Self Attention Network for Medical Concept Embedding", 2019, Che et al."Recurrent Neural Network for Multivariate Time Series with Mssing Values", 2018: 시간 정보를 통합하려는 연구들(연속된 visit 또는 lab value 사이의 temproal 특징 추출) -> 본 논문에서는 이 두가지 연구를 참고하고 artificial time token의 도입으로 시간 정보를 모델링하는 새로운 접근법을 제안  
<br>

## Method  

### Data  
저자는 Columbia University Irving Medical Center-New York Presbyterian Hospital (CUIMCNYP)에서 EHR 데이터를 가져온 후, OHDSI라는 open-science community가 개발한 데이터 모델인 OMOP(Observational Medical Out-comes Partnership)라는 양식에 맞게 변환하였다.<br>
<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/OHDSI-OurJourney.png?raw=true" width = 50%></p>
<br>
EHR 데이터는 병원마다 용어도 다르고, 양식도 다른데 OHDSI는 이러한 다양한 용어와 양식을 가진 EHR 데이터를 통일된 용어와 양식으로 바꾸어 준다. OMOP는 수치형 데이터와 clinical domain(visits, conditions, procedures, medications, lab tests, vital signs 등)가 포함되어 있다. (Physionet에 확인해보니, MIMIC-IV를 OMOP로 변환한 데이터셋이 있다.)
<br>
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

<b>Concept embedding</b>: <br>Concept embeddings were used to capture the numeric representations of the concept codes based on underlying cooccurrence statistics<br>  
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

<b>Visit segment embedding</b>: <br>Visit segment embedding은 기존 BERT에서 segment 토큰과 똑같은 역할을 한다. 참고로 비슷한 연구인 BEHRT에서도 사용했다.  
<br>
<p align ="center"><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/BEHRT.jpg?raw=true"></p>

<br>

<b>Time embedding</b>:<br> 개인적으로 다른 EHR 데이터의 Bert 적용한 다양한 연구와 비교하여 ATT와 더불어 본 논문의 차별점이 될 수 있는 본 논문에서의 필살기라 생각된다. <i>간단하게 요약하자면 Time2Vec 이라는 방법론이 있는데 이는 시계열을 잘 representation 할 수 있는 방법이다.</i> 논문의 이름은 "Time2Vec: Learning a Vector Representation of Time" 이다. 시간에 대한 Representation을 수행하는데 특히 주기 패턴과 비주기 패턴에 강건하며 time resolution을 변경하더라도 중요한 정보는 담아내고, 여러 모델에 쉽게 적용이 가능하다는 특징이 있다.

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

MIMIC의 경우에 omop_data_csv/condition_occurrence.csv, procedure_occurrence.csv, drug_exposure.csv에 존재하는 Concept code(concept code는 자연어, 식별자의 형태로 저장되어 있음. 자연어의 경우엔 그대로 사용하고, 식별자인 경우엔 json 파일을 이용해 매핑을 해야하는 형태로 이해하였음)를 가지고 {Concept code : SCRIPT} 의 딕셔너리 형태로 저장된 achilles_json/condition_treemap.json(여기에 AGE 포함), achilles_json/drug_treemap.json, achilles_json/procedure_treemap.json 을 매핑시켜 데이터를 구성할 수 있을 것 같다.
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

3) Hyper parameter:
<blockquote>
1.BERT ENCODER: 5 layer<br>
2.Multi Head: 8개<br>
3.Droprate: 0.1<br>
4.embedding, hidden dimension: 128<br>
5.Input demension: 300<br>
6.EPOCH: 5<br>
7.Batch size: 32<br>
8.Learning Rate: $2e^{-4}$
</blockquote>
<br>
window size를 300으로한 이유는 300이라는 size가 환자의 history의 90%를 cover할 수 있기 때문이라고 밝혔다. 더불어, 시퀀스가 만약 300을 넘어갈 경우 random slice했다고 한다. 개인적으로 이해하기는 처음 element부터 300까지 혹은 마지막 element부터 뒷 방향으로 300까지 slice를 하는데 이 기준을 랜덤하게 정한 것이 아닐까한다.  
또한 시퀀스가 300 보다 적을 경우 PAD 토큰을 이용하여 padding을 해주었다고 한다.  

### Experiments  

저자는 제안한 모델의 data representation 능력을 확인하기 위해 Disease classification을 진행했다고 한다. 이를 위해 classifier로 Bi-LSTM을 사용했다.  
<p align='center'><img src="https://raw.githubusercontent.com/Jeong-Eul/CEHR-BERT/main/Image/bilstm.webp" width = 70%></p>  
<br>

#### Appendix A. Prediction Tasks  

Target cohort(대상 집단): 초기 그룹  
Outcome cohort(결과 집단): 초기 그룹의 하위 집단  

observation window: patient의 전체 history 중에서 학습에 포함할 데이터
prediction window:  patient의 전체 history 중에서 예측에 포함할 데이터

prediction task는 Target cohort 중에서 outcome cohort 를 잘 분류해 내는 것이다.  

1. <b>T2DM HF(Type 2 diabetes mellitus patients who developed Heart Failure):</b> 대상 집단은 제 2형 당뇨가 있는 환자들로 구성이 되었고, 제 2형 당뇨를 진단받기 전에 제 1형 당뇨, 당뇨증(혈뇨증), 임신성 당뇨병, 2차성 당뇨병 및 신생아 당뇨병이 있는 환자들은 제외했다. 결과 집단은 Heart Falilure를 경험한 사람으로 다음과 같은 기준이 있는 환자를 포함했다. <br>

    1-1 적어도 하나의 높은 BNP 결과가 있으며(least one lab test with high BNP results)
    1-2 기계적 순한 지원(machanical circulatory support)  
    1-3 인공 심장 관련 수술(artificial heart procedure)  
    1-4 이뇨제, 혈관 활성제 또는 투석 절차를 받은 환자들(diuretic agent,
vasoactive agent or dialysis procedure.)

이 기준들은 Table 6,7에 제시된 OMOP concept id를 가지고 판단했다고 한다.  

<p align ='center'><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/table6.jpg?raw=true"></p>
<br>
<p align ='center'><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/table7.jpg?raw=true"></p>
<br>

2. <b>HF readmit(Heart Failure patiens who were readmitted within 30 days):</b>  대상집단은 hert failure 때문에 병원에 방문한 환자들이다. 결과집단은 대상집단 환자들 중 한번 방문한 이후 30일 이내 다시 방문한 환자들이다. 따라서 예측하고자 하는 것은 어떤 환자가 30일 이내에 다시 재방문할 것인지이다. 방문을 의미하는 concept id는 9201, 262이다. 또한 이 경우에 prediction window는 30일이다.  
<br>

3. <b>discharge home death(patients who were discharged and died within one year)</b>: 대상집단은 inpatient - discharged to home 즉, 입원 했다가 퇴원 했던 환자들이며, 결과집단은 퇴원 후 1년 이내 사망한 환자이다. 입원 환자 concept id 9201, 262 / 퇴원 환자는 discharge_to_concept_id 로 구분했으며 prediction window는 360일이다. 
<br>

4. <b>Hospitalization</b>: 특정환자의 첫번째 방문시점 이후 입원 여부에 대한 예측 task로 이해했다. 충분한 데이터포인트를 갖기 위해 observation window 내에 2회 ~ 30회까지 visit이 발생한 환자만을 포함시켰다고한다.



<br>

observation window는 따로 명시되지 않는 한 1년으로 세팅했다. (Hospiralization의 obsevation window는 첫 방문 이후 3년, prediction window는 2년 즉, 처음 방문 이후 3년이 지난 시점으로부터 2년간의 hospitalization에 대한 예측 task)  

<br>

#### 4 prediction tasks result  

<br>
<p align = "center"><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/experiment_result.jpg?raw=true"></p>
<br>

*R-BERT: No pretrained CEHR-BERT  

위 실험 결과를 통해 알 수 있는 것:
1. sequence를 모델링 할 수 있는 BERT, LSTM이 base model(LR, XGB)보다 성능이 좋았다.  
2. 모든 Task에서 pre trained model이 그렇지 않은 것 보다 성능이 좋았다.  
3. BERT 기반 모델(R-BERT, BEHRT, MedBERT)이 LSTM 보다 좋았다. -> Attention 때문일까?  
4. 대부분의 Task 에서 제안 모델이 1등, Med-BERT가 2등을 달성했다.  
4. 하지만 Hospitalization 의 경우 Med-BERT 보다 LSTM의 성능이 더 좋았다. -> 이유를 분석하여 제시하지 않았다.    
3. 제안 모델인 CHERT-BERT가 모든 예측 task에서 성능이 좋았다.  

#### Ablation study: ATT, Temporal concept embedding, Visit Prediction Task가 정말 효과가 있었는가?  

<p align ="center"><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/experiment_result_abl.jpg?raw=true"></p>

1. <b>ATT 토큰 유무에 따른 성능차이 비교</b> - M-BERT(no token), B-BERT(only sep token), CEHR-BERT
 
    어떠한 seperate 토큰을 사용하지 않은 M-BERT 보다 SEP 토큰을 사용한 B-BERT가 거의 모든 Task에서 성능이 좋았으며 SEP 토큰 대신 visit에 대한 시간 정보를 사용한 CEHR-BERT 가 성능이 제일 좋았다. 따라서 visit 별 시간 정보를 활용하는 것이 효과가 있었다.  

    또한 논문의 Discussion에서 ATT가 잠재공간에서 적절한 representation을 학습했음을 보여주는 figure를 제공했다. 아래 그림은 ATT 임베딩 값을 PCA를 사용하여 2차원 벡터로 각각 축소한 후 좌표평면에 plot한 결과이다. 그림을 보면, Time internal의 크기에 따라 $W_{n}$ 토큰, $M_{n}$ 토큰, $LT$ 토큰이 오른쪽에서 위쪽으로 linearly 하게 증가하고 있다. 더불어, week 토큰은 서로 군집을 이루고 있었다. 또한 Visit의 시작과 끝을 나타내는 토큰인 VS, VE는 visit의 interval과는 의미론적으로 거리가 멀기 때문에 ATT 토큰과 상태적으로 떨어져 있으며 군집을 이루고 있었다. 이는 납득할만한 결과이며 토큰의 semantic vector 를 잘 학습하였음을 알 수 있다.
    <br>
    <br>
    <p align ="center"><img src = "https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/ATT_visualization.jpg?raw=true" width = 70%></p>
    <br>
2. <b>Temporal concept embedding의 효과</b> - ART-BERT(no temporal concept embedding), CEHR-BERT  

    ALT-BERT는 temporal concept embedding을 사용하지 않고, Time embedding, age embedding, concept embedding을 더했다. 성능은 CEHR-BERT가 더 높았으며, temporal concept embedding이 효과가 있었다. 이는 temporal concept embedding이 FC layer를 통해 weight와 bias를 학습할 수 있어서 세 가지 임베딩을 잘 Fusion 하도록 학습이 되었기 때문이 아닐까 싶다.  


3. <b>Visit Prediction Task의 효과</b> - NS-BERT(no VTP), CEHR-BERT  
    결과는 Hospitalization 예측 task를 제외하고 CEHR-BERT가 성능이 좋았다. 저자는 이를 실험을 위해 데이터를 Train 75%, Validation 10%, Test 15%로 4-fold로 분리했고, 각 fold를 evaluation 했는데, 이때 수행한 cross validation의 무작위 변동 때문이라고 설명했다.(잘 이해가 안됨/seed 고정을 했다면 각 fold에 들어가는 train, val은 고정일 텐데..?)  

    <p align ='center'><img src ="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/4-fold_issue.jpg?raw=true"></p>


    아무튼 Visit Prediection Task는 애초에 각 visit의 concept이 visit type과 연관이 있을 것이라는 가정에 기반한 Sub task이므로, BERT의 representation을 학습하는데 좋은 전략일 것이라고 직관적으로 이해할 수 있었으며, 실험결과 또한 이를 증명했다.  

    