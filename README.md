# 마이크로프로세서응용설계 기말 실기평가
## 1. 기본 정보
- 학번:2019732056
- 이름:최진환

- 작품명:주식 인기순 나열 프로그램
## 2. 선택 기능
- 선택 기능 1: F3. Python 데이터 처리
- 선택 기능 2: F4. 웹 서버 또는 웹 API
- 선택 기능 3: F5. 외부 연동 또는 저장 기능
## 3. 작품 목적
웹 스크래핑을 통해 Naver-Pay 증권에서 주식 정보를 받아 Python을 이용하여 인기순으로 나열하기 위한 데이터 처리 후 Bottle을 이용해 웹 서버에 출력
## 4. 사용한 하드웨어
| 구분 | 사용 부품 |
|---|---|
| Raspberry Pi | Raspberry Pi4 |
## 5. 사용한 소프트웨어 및 라이브러리
| 구분 | 내용 |
|---|---|
| 실행 환경 | Jupyter Lab / Jupyter Notebook / Python script |
| Python version | 3.13.5 |
| 사용 라이브러리 | BeautifulSoup, requests, bottle |
| Web framework | bottle |
| External service or storage | 해당 없음 |
## 6. 전체 시스템 동작 설명
1. 인기순으로 정렬하고 싶은 주식코드를 list에 입력받습니다.
2. bottle을 이용해 웹 서버를 엽니다.
3. 주식코드에 해당하는 Naver-Pay 증권 웹사이트에서 해당 주식의 이름, 현재가, 거래대금 정보를 수집하는 함수가 작동합니다.
4. 수집된 정보가 거래대금 순으로 정렬되고 웹 서버에 출력됩니다.
## 7. 선택 기능별 구현 증거표
| 선택 기능 | 기능명 | 구현 여부 | 코드 위치 | 영상 시간 |
|---|---|---|---|---|
| F3 | Python 데이터 처리 | O | main.ipynb line 19,31,33 | 00:__-00:__ |
| F4 | 웹 서버 또는 웹 API | O | main.ipynb line 23~38 | 00:__-00:__ |
| F5 | 외부 연동 또는 저장 기능 | O | main.ipynb line 7~21 | 00:__-00:__ |
## 8. 실행 방법
원하는 만큼 주식코드를 (예:삼성전자 005930) list에 넣은 뒤 실행시키면 인기순으로 웹 서버에서 출력됩니다.
## 9. 주요 코드 설명
```
def get_stock_data(code):
    url = 'https://finance.naver.com/item/main.nhn?code=' + code
    
    response = requests.get(url)
    response.raise_for_status()
    html = response.text
    soup = BeautifulSoup(html, 'html.parser')
    
    name = soup.select_one('div.rate_info dl.blind strong').get_text()
    price = soup.select_one('div.rate_info div.today span.blind').get_text()
    trade_amount = soup.select_one('table.no_info span.sp_txt9+em span.blind').get_text()
    trade_value_s = soup.select_one('table.no_info span.sp_txt10+em span.blind').get_text()
    trade_value = int(trade_value_s.replace(',',''))

    return {'name':name, 'price':price, 'trade_amount':trade_amount, 'trade_value_s':trade_value_s, 'trade_value':trade_value}
```
주식의 이름, 현재가, 거래대금을 반환하는 함수입니다. 이때 거래대금은 ,가 포함된 문자열이 반환되므로 replace 함수를 통해 ,를 없앤 뒤 정수로 변환해 이후 정렬을 위해 사용합니다.

```
@route('/')
def index():
    stock_data = []
    lines = []

    for code in codes:
        stock_data.append(get_stock_data(code))

    stock_data.sort(key = lambda x:x['trade_value'], reverse=True)

    for i,s in enumerate(stock_data):
        lines.append(f"인기 {i+1}위: {s['name']} / 현재가:{s['price']} / 거래량:{s['trade_amount']} / 거래대금:{s['trade_value_s']} 백만 원")

    return '<br>'.join(lines)

run(host='0.0.0.0', port=8080)
```
거래대금 순으로 정렬 후 웹 서버에 출력해주는 함수입니다. 직전에 정수로 변환한 거래대금을 정렬에 이용합니다.

## 10. 구현 결과
인기 1위: SK하이닉스 / 현재가:2,411,000 / 거래량:1,199,260 / 거래대금:2,865,233 백만 원  
인기 2위: 삼성전자 / 현재가:335,500 / 거래량:6,428,334 / 거래대금:2,146,618 백만 원  
인기 3위: 한화오션 / 현재가:138,800 / 거래량:3,293,983 / 거래대금:451,787 백만 원  
인기 4위: 삼성전기 / 현재가:2,023,000 / 거래량:216,158 / 거래대금:435,291 백만 원  
인기 5위: 현대차 / 현재가:619,000 / 거래량:358,432 / 거래대금:222,153 백만 원  
## 11. 한계점 및 개선 가능성
웹 사이트에 접속할 때 주식 정보가 수집되지만 접속 이후에 재접속을 하지 않으면 실시간으로 변동을 반영하지 못한다는 한계가 있습니다. time을 활용한 반복문으로 개선가능성이 있습니다.
