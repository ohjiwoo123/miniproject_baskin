# miniproject_baskin

## 주제 : 배스킨 라빈스 매장 정보를 웹스크롤링하여 데이터 시각화하기

1. Mr.K와 조를 이루어 2명에서 미니프로젝트를 진행하였습니다. 저는 웹 스크롤링을 메인으로 진행 하였고, 다른 조원은 카카오 지도 api를 활용하여 위도와 경도를 얻고, 데이터 시각화 부분을 진행하였습니다.
2. 제가 메인으로 한 웹 스크롤링 부분의 코드 및 문제 해결과정을 정리를 해보겠습니다.
3. 본 미니 프로젝트는 **"직장인을 위한 데이터 분석 실무 - 스타벅스"** 편을 참고하여 진행하였습니다.

---
## 미니프로젝트 시작 전 고충
1. 스타벅스 공식 홈페이지의 지도와 배스킨 라빈스 공식 홈페이지의 지도의 UI(User Interface)가 달랐다. 그리고 배스킨 라빈스 공식 홈페이지 지도는 크롤링 하기에 불편하다고 느껴져서 카카오 지도에서 크롤링을 하기로 하였다.
2. 각 홈페이지의 html 코드가 다르기 때문에 일일이 소스코드를 분석하여 확인해야한다.
## 오류
1. 카카오 지도 홈페이지를 보면 "서울 배스킨라빈스" 검색 후 URL이 고정되어 있어서 URL의 변화에 따라서 크롤링을 하는 작업은 불가능했다.
2. 카카오 지도에서 "서울 배스킨라빈스" 검색 후 '더보기' 버튼을 누르면 2페이지로 이동한다.
3. 각 페이지로 넘어가는 Element 숫자가 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 이런식으로 점진적인 증가하는 형태가 아니었다.<br>
1 -> 2 -> 3 -> 4 -> 5 가 반복되는 형식이었다.
4. 때로는 첫페이지만 크롤링이 되었었고, 때로는 크롤링 숫자가 변칙적이고 제대로 크롤링이 되지 않았다.
## 문제해결
1. URL이 고정된 것은 다음 페이지를 누르고 크롤링 할 때마다 새로운 soup 객체를 생성하여 크롤링 하여 해결.
2. 더보기 버튼을 누르면 바로 2페이지가 되고, 코드는 `for i in range (2,6)` 2~5 페이지를 반복해야 해서 처음 더보기 버튼을 누를 때는 크롤링하지 않고, for문에서 반복할 때만 크롤링을 진행하였다. 즉, 첫 2페이지를 2번 누르게 된다.
3. Element for Next Button 이 1~5 사이에서 반복된다. Element 5에서 다음 페이지로 넘어 갈 때는 Element 1로 넘어간다.<br> 
`for i in range (2,6)` 안에서 **Element 2 ~ 5** 크롤링을 실행하고 `def next_btn_click()`에서 Element 숫자가 5가 되면 다음 버튼(>)을 누르고 **Element 1** 크롤링을 실행한다.
4. 절차 지향적으로 크롤링 위치를 올바르게 **Positioning**하여 해결.
## 크롤링 영역
```python
# 필요 라이브러리
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import urllib.request
import time
from bs4 import BeautifulSoup
from selenium.webdriver.common.action_chains import ActionChains
import pandas as pd

# 배스킨라빈스 매장 정보를 담을 리스트
baskin_list = []

# 크롤링 페이지 숫자를 세기 위한 count 
count = 0

# 현재 크롤링 진행중인 페이지를 출력하는 함수
def countCrawling() :
    print("Count Crawling : {} page".format(count))

# 스크롤 다운 함수 
def scroll_down():
    # next버튼
    next_btn = 'info.search.page.next'
    # 스크롤 다운 (Scroll Down)
    SCROLL_PAUSE_TIME = 1
    # Get scroll height
    last_height = driver.execute_script("return document.body.scrollHeight")
    # Wait to load page
    time.sleep(SCROLL_PAUSE_TIME)
    # Calculate new scroll height and compare with last scroll height
    new_height = driver.execute_script("return document.body.scrollHeight")
    # 스크롤을 내릴 곳이 없다면
    if new_height == last_height:
        # 다음 페이지 숫자의 버튼을 누른다.
        next_btn_click()

# 더보기 버튼을 누르는 함수 (더보기 버튼을 누르면 자동으로 page num이 2가 된다.)
def more_btn_click():
    # 쉬는시간 1초
    SCROLL_PAUSE_TIME = 1
    # 더보기 버튼
    more_btn = 'info.search.place.more'
    # 더보기 버튼을 찾아서 누른다.
    button = driver.find_element_by_id(more_btn)
    ActionChains(driver).move_to_element(button).click().perform()
    ActionChains(driver).move_to_element(button).click().perform()
    time.sleep(SCROLL_PAUSE_TIME)

# 다음 페이지 숫자를 누르는 함수 
def next_btn_click():
    # 2~5페이지 까지를 xpath로 찾아서 버튼을 누른다.

    for i in range (2,6):
        xPath = '//*[@id="info.search.page.no' + str(i) + '"]'
        driver.find_element_by_xpath(xPath).click()
        time.sleep(2)
        crawling()
        time.sleep(2)
    # 5페이지까지 가면 다음 )>)페이지로 넘긴다. 
    if i == 5:
        # next버튼
        next_btn = 'info.search.page.next'
        button2 = driver.find_element_by_id(next_btn)
        ActionChains(driver).move_to_element(button2).click().perform()
        ActionChains(driver).move_to_element(button2).click().perform()
        crawling()
        time.sleep(2)

# 크롤링 하는 함수 
def crawling():
    global count
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    baskin_soup_list = soup.select("li.PlaceItem")

    for item in baskin_soup_list:
        soup = BeautifulSoup(html, 'html.parser')
        name = item.select('strong')[0]('a')[1].text
        hours = item.select('p.periodWarp')[0]('a')[0].text
        address = item.select('div.addr')[0]('p')[0].text
        cnt_review = item.select('div.rating')[0]('a')[1].text
        starpoint = item.select('div.rating')[0]('span')[0]('em')[0].text
        tel = item.select('div.contact')[0]('span')[0].text

        baskin_list.append([name, hours, address, cnt_review, starpoint, tel])
    count += 1
    countCrawling()

# 크롬 드라이버 세팅 및 크롤링 세팅
driver = webdriver.Chrome('/Users/jwoh/Desktop/pythonScrolling/chromedriver3')
driver.get("https://map.kakao.com/")
# 검색창 찾기
elem = driver.find_element_by_name("q")
# 서울 배스킨 라빈스 입력
elem.send_keys("서울 배스킨라빈스")
# 엔터 입력
elem.send_keys(Keys.RETURN)

SCROLL_PAUSE_TIME = 1
# Get scroll height
last_height = driver.execute_script("return document.body.scrollHeight")
# Wait to load page
time.sleep(SCROLL_PAUSE_TIME)
# Calculate new scroll height and compare with last scroll height
new_height = driver.execute_script("return document.body.scrollHeight")
# 스크롤을 내릴 곳이 없다면
if new_height == last_height:
    # 1페이지 크롤링 
    crawling()
    # 더보기 버튼 클릭
    # 2페이지로 넘어갔지만 크롤링은 하지 않음
    more_btn_click()

# 총 21페이지까지이므로 21 페이지까지 크롤링 
while count < 21 :
    scroll_down()
```

## 표 확인 및 수정 및 저장 영역
```python
# 표 확인 
columns = ['매장명','영업시간','주소','리뷰 갯수', '별점','전화번호']
seoul_baskin_df = pd.DataFrame(baskin_list, columns = columns)

# 맨 마지막 2개는 배스킨 라빈스 매장이 아니므로 삭제해서 데이터 프레임 만들기.
new_df = seoul_baskin_df[:-2]

# 카카오 API 위도, 경도 설정 시 오류 해결을 위한 특정 셀의 값 변경 
# 서울 영등포구 당산로 214 상가1동 1층 111-2,3호 -> 서울 영등포구 당산로 214 상가1동 1층
new_df.iloc[182][2]="서울 영등포구 당산로 214 상가1동 1층"

# 엑셀로 저장
new_df.to_excel('/Users/jwoh/Desktop/pythonScrolling/DataSciencePython/02_개정판/8_BaskinRobbins_Location/files/seoul_baskin_list.xlsx', index=False)
```

## 참고자료 
1. 직장인을 위한 파이썬 데이터분석 깃허브 - https://github.com/Play-with-data/datasalon
2. 익명의 카카오지도 크롤러 깃허브 - https://github.com/wlgh325/python_crawling
