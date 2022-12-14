---
layout : single
title : "[Python] 논문데이터 수집"
categories : Python
tags : openAPI, 논문
toc : true
---



> 논문 데이터를 수집하기 위해  [사이언스온](https://scienceon.kisti.re.kr/apigateway/api/main/mainForm.do) 사이트에서 openAPI 인증키 발급을 받은 후 사용하였다.

 

# 1. 라이브러리 불러오기


```python
#!pip install pycrypto

import requests
from bs4 import BeautifulSoup
import pandas as pd
```

## 1.1 AES256 Util 설치


```python
import crypto
import sys
sys.modules['Crypto'] = crypto
```


```python
from Crypto.Cipher import AES
from urllib.parse import quote
import base64


class AESTestClass:
    def __init__(self, plain_txt, key):
        # iv, block_size 값은 고정입니다.
        self.iv = 'jvHJ1EFA0IXBrxxz'
        self.block_size = 16
        self.plain_txt = plain_txt
        self.key = key

    def pad(self):
        number_of_bytes_to_pad = self.block_size - len(self.plain_txt) % self.block_size
        ascii_str = chr(number_of_bytes_to_pad)
        padding_str = number_of_bytes_to_pad * ascii_str
        padded_plain_text = self.plain_txt + padding_str
        return padded_plain_text

    def encrypt(self):
        cipher = AES.new(self.key.encode('utf-8'), AES.MODE_CBC, self.iv.encode('utf-8'))
        padded_txt = AESTestClass.pad(self)
        encrypted_bytes = cipher.encrypt(padded_txt.encode('utf-8'))
        encrypted_str = base64.urlsafe_b64encode(encrypted_bytes).decode("utf-8")
        encrypted_encoded = quote(encrypted_str)
        return encrypted_encoded

```

## 1.2 토큰 발급


```python
import requests
import traceback
import AES256Util
import datetime
import re
import json


################################################### 사용자 정보 입력 #####################################################

MAC_address = "맥주소 입력하세요"
clientID = "발급받은 client_id를 입력하세요"
key = "발급받은 인증키를 입력해주세요."
#######################################################################################################################
time = ''.join(re.findall("\d", datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')))
plain_txt = json.dumps({"datetime": time, "mac_address": MAC_address}).replace(" ", "")
refreshToken = ""
accessToken = ""


# Access Token 및 Refresh Token 발급
def generateToken():
    encryption = AES256Util.AESTestClass(plain_txt, key)
    encrypted_txt = encryption.encrypt()
    target_URL = "https://apigateway.kisti.re.kr/tokenrequest.do?client_id=" + clientID + "&accounts=" + encrypted_txt

    try:
        response = requests.get(target_URL)
        # API 호출 결과값 출력
        print(response.text)

        # 발급된 토큰(Refresh Token, Access Token) global 변수 입력
        json_object = json.loads(response.text)
        global refreshToken, accessToken
        refreshToken = json_object['refresh_token']
        accessToken = json_object['access_token']
        print(refreshToken)
        print(accessToken)

    except Exception:
        traceback.print_exc()

# Access Token 재발급
def regenerateToken():
    target_URL = "https://apigateway.kisti.re.kr/tokenrequest.do?refreshToken=" + refreshToken + "&client_id=" + clientID

    try:
        response = requests.get(target_URL)
        # API 호출 결과값 출력
        print(response.text)

        # 발급된 토큰(Access Token) 변수 입력
        json_object = json.loads(response.text)
        global accessToken
        accessToken = json_object['access_token']
        print(accessToken)

    except Exception:
        traceback.print_exc()


# 함수 호출
generateToken() # 토큰키 발급 


```



## 1.3 OpenAPI


```python
from urllib import parse
import requests

target="ARTI"
clientID = "711fb02bbe0f4a387665f7e116919e5e969686bca39006fa2483973f7c0b9dd2"
accessToken = "02d118b7e7d092ed492728ff7676ac0ec5c37ac689a5b6b481ea02e5c72755b6"
rowcount = '100' #최대100개


if __name__=='__main__':
    

    """
    검색어를 인코딩 합니다.
    검색어 필드에 관한것은 ScienceOn Api GateWay를 참고 해주세요.
    """
    query = parse.quote("{\"AB\":\"도시소멸\"}") #KW : 키워드, AB : 초록, BI : 전체

    target_URL = ("https://apigateway.kisti.re.kr/openapicall.do?"+
    "client_id=" +clientID+
    "&token="+accessToken+
    "&version=1.0"+
    "&action=search"+
    "&target="+target+
    "&searchQuery="+query+
    "&rowCount="+rowcount)

    """ 검색할 쿼리를 입력하여 논문 검색 api에 request를 요청하고 response를 받는다. """
    response=requests.get(target_URL)
    context = response.text
```

# 2. 파싱하기


```python
from os import name
import xml.etree.ElementTree as et
import bs4
from lxml import html
from urllib.parse import urlencode, quote_plus, unquote
```


```python
xml_obj = bs4.BeautifulSoup(context,'lxml-xml')
```


```python
rows= xml_obj.findAll("record")
```


```python
len(rows)
```


    100

![01_rows](../../images/2022-12-14-Collecting thesis/01_rows.png)

## 2.1 컬럼명 파싱하기


```python
A = rows[0].findAll('item')
row_list = []
for a in A:
    row_list.append(a['metaName'])
```


```python
row_list
```


    ['CN','DB코드','저널제어번호','출판사(발행기관)','저널명','ISSN','ISBN','권호Id','권번호','호번호','발행년','소장정보유무','논문제어번호','논문명','초록','저자',
     '원문유무','초록유무','페이지정보','DOI','원문URL','ScienceON상세링크','ScienceON모바일링크','키워드','학위구분']



## 2.2 데이터 파싱하기


```python
rows[0].text.split('\n')
```

![02_data parsing](../../images/2022-12-14-Collecting thesis/02_data parsing.png)


```python
item_list = []
value_list = []
for i in range(0, len(rows)):
    item_list.append(rows[i].text.split('\n'))
    value_list.append(item_list[i][1:26]) #필요없는 앞뒤 데이터 삭제하여 컬럼수와 개수 맞춰줌
```


```python
len(value_list[0]) 
```


    25

![03_value_list](../../images/2022-12-14-Collecting thesis/03_value_list.png)

## 2.3 데이터 프레임

- 소멸위험 논문의 수가 적은것같아 도시소멸 논문까지 수집함


```python
# AB : 소멸위험
소멸위험 = pd.DataFrame(value_list, columns = row_list)
소멸위험['CN'].unique()
```


```python
# AB : 도시소멸 
도시소멸 = pd.DataFrame(value_list, columns = row_list)
도시소멸['CN'].unique()
```


```python
# merge outerjoin해서 중복값 제거 
df = pd.merge(소멸위험, 도시소멸, how='outer')
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 192 entries, 0 to 191
    Data columns (total 25 columns):
     #   Column          Non-Null Count  Dtype 
    ---  ------          --------------  ----- 
     0   CN              192 non-null    object
     1   DB코드            192 non-null    object
     2   저널제어번호          192 non-null    object
     3   출판사(발행기관)       192 non-null    object
     4   저널명             192 non-null    object
     5   ISSN            192 non-null    object
     6   ISBN            192 non-null    object
     7   권호Id            192 non-null    object
     8   권번호             192 non-null    object
     9   호번호             192 non-null    object
     10  발행년             192 non-null    object
     11  소장정보유무          192 non-null    object
     12  논문제어번호          192 non-null    object
     13  논문명             192 non-null    object
     14  초록              192 non-null    object
     15  저자              192 non-null    object
     16  원문유무            192 non-null    object
     17  초록유무            192 non-null    object
     18  페이지정보           192 non-null    object
     19  DOI             192 non-null    object
     20  원문URL           192 non-null    object
     21  ScienceON상세링크   192 non-null    object
     22  ScienceON모바일링크  192 non-null    object
     23  키워드             192 non-null    object
     24  학위구분            192 non-null    object
    dtypes: object(25)
    memory usage: 39.0+ KB

```python
df['초록'][0]
```


    0      본 연구의 목적은 지방소멸위험지수를 활용하여 전국 시·군·구의 소멸위험을 거시적으로...
    1      본 연구는 현재 우리나라에서 가장 심각하게 사회 및 경제적으로 부각되고 있는 인구감...
    2      본 논문은 어촌 지역사회의 당면한 문제들을 토대로 지속가능한 내생적 개발의 관점에서...
    3      2005년부터 본격화된 정부의 출산장려 정책에도 불구하고 2018년 우리나라의 합계...
    4      본 연구는 기술보증기금의 기술사업성 평가자료를 활용하여 1인 창조기업의 경영역량과 ...
                                 ...                        
    187    본 연구는 지방에 소재한 대학의 지역사회 협력 방안을 제시하고자 한다. 한국의 지방...
    188    본 연구는 무분별하게 형성되어 복잡해 보이는 지역이라 하더라도 지역의 공간구조를 인...
    189    우리나라는 대기오염물질 저감을 위해 청정연료 사용, 배출허용기준 강화 등 다양한 정...
    190    최근 인구감소와 고령화에 따른 생산인구의 감소는 다양한 경제적·사회적 문제들을 야기...
    191    도시공간에 있어 오랜 시간 동안 도시공간에 축적되어온 역사 환경과 문화적 요소들은 ...
    Name: 초록, Length: 192, dtype: object



# 3. 데이터 저장


```python
df.to_csv('./AB_소멸위험_도시소멸.csv', encoding='utf-8-sig') #utf-8, cp949, euc-kr 모두 인코딩이 깨져 utf-8-sig해줌 
```

