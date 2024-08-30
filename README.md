# Today's-KBO-win-rate-Prediction
KBO 승률 예측 프로그램
<br>
> 당일날 공개되는 라인업을 입력했을 때 어떤 팀이 이길지 승률을 예측하는 모델
  
## member
- 권도원
- 신혜승
- 임정규

## overview

![Final Model Structure](https://github.com/user-attachments/assets/7be93a9f-7d08-4efd-8fc9-cca8ed38f0e2)


## purpose
24년도 기준 많은 인기를 끌고 있는 KBO의 당일의 승부 예측 프로그램을 만들어 사람들의 흥미와 관심을 높이자!


## procedure
1. 자료 준비 과정에서 api를 제공하는 웹 사이트를 찾지 못해 웹 크롤링을 사용해서 자료를 준비 (https://statiz.sporki.com/)
   
        !pip install beautifulsoup4
        import requests
        from bs4 import BeautifulSoup
        import pandas as pd
        
        url = 'https://statiz.sporki.com/player/?m=rival&p_no=10058&pos=pitching&year=2023&ct_code=&pa='
        
        response = requests.get(url)
        
        html = response.text
        soup = BeautifulSoup(html, 'html.parser')
        temp = soup.find_all("table")[0]
        
        df = pd.DataFrame(columns = ["이름","타석","실질타석","타수","득점","안타","2루타","3루타","홈런","루타수","타점","볼넷","몸에맞는볼","고의4구","삼진","병살타","희생 번트","희생 플라이","타율","출루율","장타율","OPS","투구수","avLI","RE24","추가한 승리확률"])
        df
        
        df.to_csv('선수 이름.csv', index=False, encoding='utf-8-sig')
        
        i = 1
        #투수마다 상대한 타자의 수가 다르기 때문에 타자수를 rows로 측정
        rows = temp.find_all('tr')
        len(rows)
        for i in range(len(rows)):
            #table 클래스의 <tr>에 해당하는 모든 줄을 긁어옴
            temp2 = temp.find_all("tr")[i]
            #긁어온 줄 중에서 필요한 텍스트(이름,타율 등의 지표들)만을 data에 저장
            data = [td.get_text(strip=True) for td in temp2.find_all('td')]
            # Check if data has enough elements before accessing them
            if len(data) >= 26:
                df.loc[i-1] = data
            else:
                print(f"Skipping row {i} due to insufficient data.")
                # Or handle the case where data is incomplete in a way that suits your needs
        
        
        #팀별 엔트리 주소
        url = "https://statiz.sporki.com/team/?m=seasonBacknumber&t_code=7002&year=2024"
        
        # URL에서 HTML 내용 가져오기
        response = requests.get(url)
        html_content = response.text
        
        # BeautifulSoup으로 HTML 파싱
        soup = BeautifulSoup(html_content, 'html.parser')
        
        # 선수 이름 추출
        players = []
        for item in soup.find_all("div", class_="item away"):
            name_link = item.find("a")
            if name_link:
                name = name_link.text.strip()
                players.append(name)
        
        # 추출한 선수 이름 출력
        for player in players:
            print(player)
        with open('HanWha Eagles Team.txt', 'w', encoding='utf-8') as f:
            for player in players:
                f.write(player + '\n')

   <img width="628" alt="KakaoTalk_20240830_152012168" src="https://github.com/user-attachments/assets/74944378-9ac0-4fe7-b3af-3b879128d567">


2. 준비한 자료를 바탕으로 ollama의 openai를 이용하여 크롤링 해온 csv 파일을 로드하여 phi3.5에게 주었으나 자료가 없다는 대답만 돌아옴

3. 이후 모델의 messages 응답 구문에 읽어온 csv파일의 데이터가 포함되지 않는다는 걸 알고 수정

4. 그럼에도 답변이 이상하길래, phi3.5에게 넘어간 CSV파일의 데이터를 보았는데 데이터가 깨지고 사람이 봐도 판단하기 어려운 형태로 전달함

5. 데이터의 양이 gpt가 보고도 판단이 어려운 양이어서 하나의 파일만 불러와 읽혀 보도록 함

6. 하나의 데이터 파일 또한 phi3.5가 읽기 난해한 형태로 전달됨

7. 이를 보고 파일의 형태를 csv가 아니라 json으로 변환하기로 함

8. json 파일로 변환하는 과정에서 한글을 고려하지 않아 파일이 깨짐

9. 이를 수정하여 정상적인 형태로 json 파일로 변환에 성공

10. json 파일을 다시 넣어보았는데 이번엔 답변이 깨짐

11. 8번 과정에서 겪었던 문제와 같음을 깨닫고 json 파일을 답변에 dump 해주는 과정에서 한글 또한 허용해 주도록 변환하였더니, 이후 나름 정상적인 답변이 돌아옴


## Consequence


        import os
        import json
        import openai
        from openai import OpenAI
        
        # Ollama 서버의 API 엔드포인트 및 API 키 설정
        client = OpenAI(
            base_url='http://localhost:11434/v1',  # Ollama의 로컬 서버 주소
            api_key='ollama'
        )
        
        # JSON 파일을 읽어 데이터를 로드하는 함수
        def load_single_json_file(file_path):
            file_name = os.path.basename(file_path)  # 파일 이름 추출
            data_content = None
            
            # JSON 파일 읽기
            with open(file_path, 'r', encoding='utf-8', ) as file:
                data_content = json.load(file)
            
            print("JSON 파일이 성공적으로 읽혔습니다!")
            return data_content, file_name
        
        # 디렉토리에서 데이터와 파일 이름 로드
        json_file_path = './JPitchers_relative_record/Alcantara.json'  # 다운 받은 json파일의 주소를 여기에 입력
        context_data, file_name = load_single_json_file(json_file_path)
        
        # 사용자의 질문에 데이터를 참고하여 답변 생성
        def ask_question_with_context(question):
            # 데이터와 파일 이름을 직접 사용
            response = client.chat.completions.create(
                model="phi3.5",     
                messages=[
                    {"role": "system", "content": "You are a helpful assistant."},
                    {"role": "user", "content": f"The following file was loaded: {file_name}. The following context is provided for reference: {json.dumps(context_data, ensure_ascii=False)}. {question}"}
                ]
            )
            return response.choices[0].message.content  # 응답의 내용을 반환
        
        # 예시 질문
        # 아래의 question란에 원하는 질문 입력
        question = "This data is data that records Alcantara's record against each batter. Using this data, tell me the batter who is the strongest against Alcantara."
        answer = ask_question_with_context(question)
        
        print(f"Question: {question}")
        print(f"Answer: {answer}")


![image](https://github.com/user-attachments/assets/450273bf-0a67-4c6b-b770-2777996fe135)
=> 위의 코드가 자료를 분석해서 적절한 답을 해준 것을 확인했음



## regret point
하나 이상의 파일을 읽는데 시간이 너무 많이 소요됨

그리고 무엇보다 이 데이터들을 가지고 고차원적인 질문(ex. 승률 계산)을 했을 때, 답변의 질이 너무 떨어짐

사용 모델인 phi3.5만 그런 건가 싶어 llama3.1 또한 사용해 보았는데 gpt 만큼은 만족스러운 답변을 내어 주진 못함

이처럼 모델적 한계를 가장 많이 느낌
