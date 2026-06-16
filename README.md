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
| Python version | |
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
| F3 | | O/X | main.ipynb 셀 __ 또는 main.py line __ | 00:__-00:__ |
| F4 | | O/X | main.ipynb 셀 __ 또는 main.py line __ | 00:__-00:__ |
| F5 | | O/X | main.ipynb 셀 __ 또는 main.py line __ | 00:__-00:__ |
## 8. 실행 방법
원하는 만큼 주식코드를 (예:삼성전자 005930) list에 넣은 뒤 실행시키면 인기순으로 웹 서버에서 출력됩니다.
## 9. 주요 코드 설명
```
def get_stock_data(code):
    url = f'https://finance.naver.com/item/main.nhn?code={code}'
    response = requests.get(url)
    html = response.text
    soup = BeautifulSoup(html, 'html.parser')

    name = soup.select_one('div.wrap_company h2 a').get_text()
    price = soup.select_one('#chart_area div.rate_info span.blind')
    trading_value = soup.select_one('#chart_area div.rate_info span.sp_txt10+em span.blind')
    trade_raw = int(trading_value.get_text().replace(',', ''))
    return {
        'code': code,
        'name': name,
        'price': price.get_text(),
        'trade_str': f"{trading_value.get_text()} 백만 원",
        'trade_raw': trade_raw
    }
```
주식의 이름, 현재가, 거래대금을 반환하는 함수입니다. 이때 거래대금은 ,가 포함된 문자열이 반환되므로 replace 함수를 통해 ,를 없앤 뒤 정수로 변환해 이후 정렬을 위해 사용합니다.

```
@route('/')
def index():
    stock_list = []
    
    # 설정한 종목코드를 돌며 데이터 수집
    for code in STOCK_CODES:
        data = get_stock_data(code)
        stock_list.append(data)
        
    # 💡 핵심: 수집된 종목들을 거래대금(trade_raw) 기준으로 정렬!
    # reverse=True 이므로 내림차순(가장 큰 금액이 맨 위로) 정렬됩니다.
    stock_list.sort(key=lambda x: x['trade_raw'], reverse=True)

    lines = []
    for i, s in enumerate(stock_list, 1):
        lines.append(f"{i}위: {s['name']} ({s['code']}) 현재가: {s['price']} - 거래대금: {s['trade_str']}")
    
    return "<br>".join(lines)
```
거래대금 순으로 정렬 후 웹 서버에 출력해주는 함수입니다. 직전에 정수로 변환한 거래대금을 정렬에 이용합니다.

## 10. 구현 결과
1위: SK하이닉스 (000660) 현재가: 2,382,000 - 거래대금: 7,940,145 백만 원  
2위: 삼성전자 (005930) 현재가: 343,000 - 거래대금: 5,677,739 백만 원  
3위: NAVER (035420) 현재가: 242,000 - 거래대금: 617,536 백만 원  
4위: 한미반도체 (042700) 현재가: 328,500 - 거래대금: 364,247 백만 원  
5위: 카카오 (035720) 현재가: 40,450 - 거래대금: 58,067 백만 원  
## 11. 한계점 및 개선 가능성
웹 사이트에 접속할 때 주식 정보가 수집되지만 접속 이후에 재접속을 하지 않으면 실시간으로 변동을 반영하지 못한다는 한계가 있습니다. time을 활용한 반복문으로 개선가능성이 있습니다.
