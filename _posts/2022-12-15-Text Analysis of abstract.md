---
layout : single
title : "[Python] 논문데이터 텍스트분석-(2)"
categories : Python
tags : 텍스트분석, 논문
toc : true
---

---

<center><I>사이언스온에서 수집한 도시소멸지수 논문데이터 텍스트마이닝을 실시하였다.</I></center>

<center><I>텍스트마이닝은 텍스트 형태로 이루어진 비정형 및 반정형 데이터에 대하여 자연어처리를 이용하여 텍스트로부터 가치와 의미있는 정보를 찾아내는 기술이다.</I></center>

---

```python
import os
import re
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import rc 
import seaborn as sns

rc('font', family='AppleGothic') 
plt.rcParams['axes.unicode_minus'] = False  

pd.set_option('display.max.rows',1000)
pd.set_option('mode.chained_assignment', None) #오류제거

import warnings
warnings.simplefilter('ignore')
```

```python
from konlpy.tag import Mecab, Okt, Kkma
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from nltk.corpus import stopwords
from collections import Counter
```

```python
df = pd.read_csv('./AB_소멸위험_도시소멸.csv', encoding='utf-8-sig')
df.info()
```

![12_before](../../images/2022-12-15-Text Analysis of abstract/12_before.png)



# 1. 컬럼 조회

## 1.1 불필요한 컬럼 삭제

```python
del_col = ['저널명','ISSN','ISBN','권호Id','권번호','호번호','소장정보유무','논문제어번호','저자',
           '원문유무','초록유무','페이지정보','DOI','원문URL','ScienceON상세링크','ScienceON모바일링크','학위구분']
```

```python
df.drop(del_col, axis=1, inplace=True)
```

```python
df.info()
```

![13_after](../../images/2022-12-15-Text Analysis of abstract/13_after.png)

```python
plt.figure(figsize=(15,5))
sns.countplot(x='DB코드', data=df)
```

![output_8_1](../../images/2022-12-15-Text Analysis of abstract/output_8_1.png)

> DIKO : 학외논문 / JAKO : 국내논문 / CFKO : 학술회의 

```python
plt.figure(figsize=(20,8))
sns.countplot(x='발행년', data=df)
```

![output_11_1](../../images/2022-12-15-Text Analysis of abstract/output_11_1.png)

```python
ab = df['초록']
```

```python
ab[1]
```

![11_abstract](../../images/2022-12-15-Text Analysis of abstract/11_abstract.png)

> &amp;#xD; 등 불필요한 문자를 제거해 주는 정규화 작업이 필요함 



# 2. Mecab

## 2.1 특수문자 제거하기

```python
import re
compile = re.compile("[^ ㄱ-ㅣ가-힣]+")
for i in range(len(ab)):

    a = compile.sub("",ab[i])
    ab[i] = a
```

![03_print(ab)](../../images/2022-12-15-Text Analysis of abstract/03_print(ab)-1602646.png)



## 2.2 형태소 분석

> 형태소 분석이란 형태소를 비롯하여 어근, 접두사/접미사, 품사태깅(POS)등 다양한 언어적 속성의 구조를 파악하는 것이다. 

### 2.1.1 명사추출 nouns

```python
mecab = Mecab()

nouns = []
for i in range(len(ab)):
    nouns.append(' '.join([word for word in mecab.nouns(ab[i])
                           if len(word)>1]))
```

```python
nouns[1]
```

![04_nouns](../../images/2022-12-15-Text Analysis of abstract/04_nouns.png)



### 2.2.2 형태소 추출 morphs

```python
mecab = Mecab()

morphs = []
for i in range(len(ab)):
    morphs.append(' '.join([word for word in mecab.morphs(ab[i])
                           if len(word)>1]))
```

```python
morphs[1]
```

![05_morphs](../../images/2022-12-15-Text Analysis of abstract/05_morphs.png)

> 형태소는 불필요한 단어까지 추출하기 때문에 명사추출을 사용함



# 3. CountVectorizer

> CountVectorizer는 단어의 빈도수를 기반 추출 방법 

## 3.1 빈도분석

```python
from sklearn.feature_extraction.text import CountVectorizer

CV = CountVectorizer()
cntvec = CV.fit_transform(nouns)
cntvec
```

```python
CV = CountVectorizer()
cntvec = CV.fit_transform(nouns)
cntvec_df = pd.DataFrame(cntvec.toarray(), columns=CV.get_feature_names())
csv_CV = pd.DataFrame(cntvec_df.sum().sort_values(ascending=False))
```

![06_csv_CV](../../images/2022-12-15-Text Analysis of abstract/06_csv_CV.png)



## 3.2 불용어 처리

> 문장에 불필요한 단어를 불용어라고 한다. 이 과정을 통해 키워드의 품질을 높일 수 있다. 

```python
stop_words = csv_CV[csv_CV[0]==1]
```

![07_stop_words](../../images/2022-12-15-Text Analysis of abstract/07_stop_words.png)

> **한번 언급된 단어들은 중요하지 않다고 판단되어 불용어 처리함**

```python
mecab = Mecab()

nouns = []
for i in range(len(ab)):
    nouns.append(' '.join([word for word in mecab.nouns(ab[i]) #명사추출
                           if word not in stop_words #불용어제거
                           if len(word)>1])) #1글자 단어 제거 
```



# 4. TfidfVectorizer

*count기반의 특징추출은 단순 빈도만을 계산하기 때문에 조사, 관사처럼 의미는 없지만 문장에 많이 등장하는 단어들을 높게 쳐주기 때문에 유의미한 결과를 얻기 힘들 수 있다. TF-IDF는 이러한 단어들에 일종의 패널티를 줘서 CountVectorizer한계점을 해결할 수 있다.* 

```python
from sklearn.feature_extraction.text import TfidfVectorizer

TV = TfidfVectorizer()
tfidfvec = TV.fit_transform(nouns)
tfidfvec
```

```
<192x4667 sparse matrix of type '<class 'numpy.float64'>'
	with 19788 stored elements in Compressed Sparse Row format>
```

> 192x4667은 192는 행의 개수 4667은 단어의 개수를 의미함

```python
tfidfvec.toarray()[0]
```

```
array([0.        , 0.        , 0.06146696, ..., 0.        , 0.        ,
       0.06821469])
```

> Toarray()함수를 통해 문서 내의 단어가 등장한 횟수에 따라 값이 부여된 것을 볼 수 있다. 

```python
TV.vocabulary_
```

![08_vocabulary](../../images/2022-12-15-Text Analysis of abstract/08_vocabulary.png)

> Vacabulary_ 를 통해 객체가 가지고 있는 정보를 확인할 수 있다. dictionary형태의 값을 가지게 되는데, 각 value가 의미하는 칼럼의 위치가 되고 key는 해당 컬럼의 단어를 의미한다. 

> "연구"는 행렬의 2625번째 컬럼을 의미한다. 

```python
from sklearn.feature_extraction.text import TfidfVectorizer

TV = TfidfVectorizer(ngram_range=(2,2)) #ngram : 바이그램 모형을 사용하여 2개의 연속된 단어묶음 추출
tfidfvec = TV.fit_transform(nouns)
tfidfvec_df = pd.DataFrame(tfidfvec.toarray(), columns=TV.get_feature_names())
csv_TV = pd.DataFrame(tfidfvec_df.sum().sort_values(ascending=False))
csv_TV
```

![09_csv_TV](../../images/2022-12-15-Text Analysis of abstract/09_csv_TV.png)

```python
a = csv_TV.reset_index()
a['first'] = a['index'].apply(lambda x : x.split(' ')[0])
a['last'] = a['index'].apply(lambda x : x.split(' ')[1])
csv_TV = a[a['first']!=a['last']]
```

> 앞뒤 같은 경우를 방지하기 위해 같은 단어들은 모두 제거함 

```python
csv_TV.reset_index(drop=True)
```

![10_csv_TV](../../images/2022-12-15-Text Analysis of abstract/10_csv_TV.png)

```python
csv_TV.to_csv('./ngram_전체.csv', encoding='euc-kr')
```