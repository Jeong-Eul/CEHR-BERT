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

<b>Concept embedding</b>: Concept embeddings were used to capture the numeric representations of the concept codes based on underlying cooccurrence statistics  
$\to$ OMOP 데이터 셋 양식에 따르면 concept table의 "concept_code"라는 열이 있다. 아래 사진은 MIMIC을 OMOP로 변환한 데이터셋에서의 예시이다. 그림에서 볼 수 있듯 "concept_code"라고 하는 것은 짧은 자연어(구; Phrase)로 되어 있다. 
<br>
<p align ="center"><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/Image/concept_code_MIMIC_OMOP.jpg?raw=true>" width = 40% height = 600></p>
<br>

아마도 환자의 visit 안에 여러 개의 concept_code가 시간과 매칭되어 있을 것이다. 이렇게 자연어로 표현된 "concept_code"를 co-occurrence statistics에 기반한 real vector로 바꾸어 임베딩을 한다.  
<br>
여기서 co-occurence statistics가 명확하게 무엇인지 잘 모르겠다. 동시발생행렬을 의미하는 것인가?  

등장하는 단어(구)의 빈도를 바탕으로 수치로 바꾸어주는 일종의 변환이 있을 것이라 생각된다.  
예를 들어 "you say goodbye and i say hello" 라는 문장이 있으면 "you"의 동시 발생 빈도는 [0, 1, 0, 0, 0, 0] 이 되는 것처럼 말이다.  



## Experiment  
