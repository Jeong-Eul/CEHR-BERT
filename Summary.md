# CEHR-BERT: Incorporating temporal information from structured EHR data to improve prediction tasks  

<blockquote>
Author: Chao Pang, Xinzhuo Jiang, Krishna S. Kalluri, Matthew Spotnitz<br>    
Data source: IColumbia University Iriving Medical Center-York Presbyterian Hospital<br>
Date: Nov, 2021  
</blockquote>
<br>
<p align = "center"> <img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/architecture.jpg?raw=true" width = 60%></p>
<br>

## Introduction  
## Related work
## Method  

### Data  
저자는 Columbia University Irving Medical Center-New York Presbyterian Hospital (CUIMCNYP)에서 EHR 데이터를 가져온 후, OHDSI라는 open-science community가 개발한 데이터 모델인 OMOP(Observational Medical Out-comes Partnership)라는 양식에 맞게 변환하였다.<br>
<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/OHDSI-OurJourney.png?raw=true" width = 50%></p>
<br>
EHR 데이터는 병원마다 용어도 다르고, 양식도 다른데 OHDSI는 이러한 다양한 용어와 양식을 가진 EHR 데이터를 통일된 용어와 양식으로 바꾸어 준다. OMOP는 수치형 데이터와 clinical domain(visits, conditions, procedures, medications, lab tests, vital signs 등)가 포함되어 있다. (Physionet에 확인해보니, MIMIC-IV를 OMOP로 변환한 데이터셋이 있다.)
<br>

<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/ohdsi.jpg?raw=true" width = 50%></p>
<p align ='center'><a href="https://www.ohdsi.org/data-standardization/">참조: OHDSI 웹페이지 링크</a></p>

저자는 변환한 OMOP 데이터 셋에서 conditions, procedures, medication의 3가지 clinical domain 만을 활용했다고 밝혔다.

### Data processing and patient representation(sequence)  

논문에서 제시한 환자의 시퀀스를 표현하는 벡터를 이해하기 위해 필요한 notation은 다음 사진과 같다.  

<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/notation1.jpg?raw=true" width = 70%></p>
<p align ='center'><img src="https://github.com/Jeong-Eul/CEHR-BERT/blob/main/notation2.jpg?raw=true" width = 70%></p>


### Embedding Technique for input of BERT



## Experiment  
