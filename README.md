# google_calendar_api_test
practice google calendar api

# Use google calendar api
1. google calendar api를 사용하는 방법 알아보기
2. Error report
3. 코드 리뷰
4. 시도해 볼만한 기능들

# google calendar api 사용방법
> flask에서 사용해보고 싶기 때문에 python으로 quick start를 시도했다.

1. google api를 검색하여
https://console.developers.google.com/apis/에 접속한다.

2. 왼쪽 메뉴의 라이브러리를 누르고, calendar를 검색하여 google Calendar API를 선택한다.

3. 사용버튼을 누르고 사용자 인증 정보를 만든다.
> flask에서 사용하기 위해서 API 호출하는 위치를 웹 서버로 설정하고,<br> 엑세스할 데이터를 사용자 데이터로 설정했다.

4. 승인된 자바스크립트 원본과 승인된 리디렉션 URI는 빈칸으로 하고 OAuth 클라이언트 ID만들기 버튼을 선택한다.

5. 사용자 인증 정보를 다운로드 한 뒤, 완료버튼을 누른다.


# Quick Start
1. https://developers.google.com/calendar/quickstart/python 로 이동하여 Step을 따라간다. (Step1은 하지 않아도 된다.)

2. Step3의 코드를 가져온다.
``` python
from __future__ import print_function
import datetime
import pickle
import os.path
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request

# If modifying these scopes, delete the file token.pickle.
SCOPES = ['https://www.googleapis.com/auth/calendar.readonly']


def main():
    """Shows basic usage of the Google Calendar API.
    Prints the start and name of the next 10 events on the user's calendar.
    """
    creds = None
    # The file token.pickle stores the user's access and refresh tokens, and is
    # created automatically when the authorization flow completes for the first
    # time.
    if os.path.exists('token.pickle'):
        with open('token.pickle', 'rb') as token:
            creds = pickle.load(token)
    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open('token.pickle', 'wb') as token:
            pickle.dump(creds, token)

    service = build('calendar', 'v3', credentials=creds)

    # Call the Calendar API
    now = datetime.datetime.utcnow().isoformat() + 'Z' # 'Z' indicates UTC time
    print('Getting the upcoming 10 events')
    events_result = service.events().list(calendarId='primary', timeMin=now,
                                        maxResults=10, singleEvents=True,
                                        orderBy='startTime').execute()
    events = events_result.get('items', [])

    if not events:
        print('No upcoming events found.')
    for event in events:
        start = event['start'].get('dateTime', event['start'].get('date'))
        print(start, event['summary'])


if __name__ == '__main__':
    main()
```

3. 실행시켜 본다.
> credential.json 파일이 없다는 오류가 뜰 것이다.

> 아까 다운받았던 사용자 인증 정보를 디렉토리에 넣고 이름을 credentials.json으로 바꿔준다.

4. 실행시켜 본다.
> 이번에는 잘 실행이 되지만, 승인 오류가 발생한다.
<pre>
400 오류: redirect_uri_mismatch
The redirect URI in the request,~~~
</pre>

5. 뒤에 나오는 주소를 복사한다.

6. 다시 google api 서비스에 들어가서 아까 설정하지 않은 리디렉션을 설정해주어야 한다.
> 사용자 인증 정보에 들어가서 생성된 API의 오른쪽의 수정버튼을 누른다.<br>
> 승인된 리디렉션 URI에 아까 복사해둔 주소를 넣어주고 저장을 한다.

7. 다시 실행하면 구글 계정을 선택할 수 있게 된다.

# Error Report
1. quickstart의 enable the google calendar api로 했을 때 리디렉션 URI 에러에서 어떻게 해야하는지 막혔었다. 그래서 google api 콘솔에서 설정하는 방법으로 수행한 것이다.
2. 리디렉션 문제도 처음에 보고 어떻게 처리해야하는지 막막했었다. 이것저것 방법들을 시도하다가 알아내었다.
3. 구글 계정을 선택하면 '확인되지 않은 앱'이라는 에러가 뜬다. 
> 고급 버튼을 누른 뒤에 만들어준 api 프로젝트 이름으로 이동버튼을 누르고 허용 버튼을 눌러주면 된다.

# Code Review
> 다른 개발자가 작성한 코드를 가지고 추측을 해본다.


참고) https://ai-creator.tistory.com/19?category=759438
``` python
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build # 구글 캘린더 API 서비스 객체를 생성하기 위함

########################################################
# 구글 클라우드 콘솔에서 다운 받은 클라이언트 파일 경로
creds_filename = 'credentials.json'

# 사용 권한 지정
# https://www.googleapis.com/auth/calendar              캘린더 읽기/쓰기 권한
# https://www.googleapis.com/auth/calendar.readonly     캘린더 읽기 권한
SCOPES = ['https://www.googleapis.com/auth/calendar']

# 파일에 담긴 인증 정보로 구글 서버에 인증하기
# 새 창이 열리면서 구글 로그인 및 정보 제공 동의 후 최종 인증이 완료됩니다.
flow = InstalledAppFlow.from_client_secrets_file(creds_filename, SCOPES)
creds = flow.run_local_server(port=0) # ? 왜 포트는 0 일까

##################################################### 
service = build('calendar', 'v3', credentials=creds)

########################일정 생성######################
event = {
        'summary': 'itsplay의 OpenAPI 수업', # 일정 제목
        'location': '서울특별시 성북구 정릉동 정릉로 77', # 일정 장소
        'description': 'itsplay와 OpenAPI 수업에 대한 설명입니다.', # 일정 설명
        'start': { # 시작 날짜
            'dateTime': '2020-02-13T09:00:00+09:00', 
            'timeZone': 'Asia/Seoul',
        },
        'end': { # 종료 날짜
            'dateTime': '2020-02-13T10:00:00+09:00', 
            'timeZone': 'Asia/Seoul',
        },
        'recurrence': [ # 반복 지정
            'RRULE:FREQ=DAILY;COUNT=2' # 일단위; 총 2번 반복
        ],
        'attendees': [ # 참석자
            {'email': 'lpage@example.com'},
            {'email': 'sbrin@example.com'},
        ],
        'reminders': { # 알림 설정
            'useDefault': False,
            'overrides': [
                {'method': 'email', 'minutes': 24 * 60}, # 24 * 60분 = 하루 전 알림
                {'method': 'popup', 'minutes': 10}, # 10분 전 알림
            ],
        },
    }

# calendarId : 캘린더 ID. primary이 기본 값입니다.
event = service.events().insert(calendarId='primary', body=event).execute()
print('Event created: %s' % (event.get('htmlLink')))
```
다음 코드는 캘린더에 접근하여 일정을 추가하는 코드이다. 일정을 추가한 뒤, 성공하면 맨 뒤의 print()문으로 출력이 되는 것을 확인 할 수 있다.
각 코드마다 주석을 붙여 리뷰를 했다.

> 이를 위해서는 받은 API의 OAuth 동의화면에서 앱 수정을 들어가 'Google API의 범위'를 calendar.readonly 뿐만 아니라 calendar도 설정을 해주어야 하는데, 이를 위해서는 애플리케이션 홈페이지 링크와 애플리케이션 개인정보처리방침 링크를 설정해주어야 한다. 필자는 둘다 google.com으로 처리했다.

> 그리고 나서 저장 버튼이 아닌 제출하여 확인 받기 버튼을 누르고 필요한 정보를 작성하여 인증한다.


# quickstart.py_Code_Review
```python
from __future__ import print_function                   # ?
import datetime                                         # ?
import pickle                                           # 아마 토큰을 다루기 위한 모듈인 것 같다.
import os.path                                          # 경로 지정
from googleapiclient.discovery import build             # 구글 캘린더 API service 객체를 생성하기 위함
from google_auth_oauthlib.flow import InstalledAppFlow  # 인증을 위한 모듈인 것 같다.
from google.auth.transport.requests import Request      # 얘도 인증을 위한 것?

# If modifying these scopes, delete the file token.pickle.
# 사용 권한 지정
# https://www.googleapis.com/auth/calendar              캘린더 읽기/쓰기 권한
# https://www.googleapis.com/auth/calendar.readonly     캘린더 읽기만 권한
SCOPES = ['https://www.googleapis.com/auth/calendar']


def main():
    """Shows basic usage of the Google Calendar API.
    Prints the start and name of the next 10 events on the user's calendar.
    """
    ''' 이 부분은 토큰의 유무에 따른 인증 과정을 나눠놓은 것 같다.'''
    creds = None
    # The file token.pickle stores the user's access and refresh tokens, and is
    # created automatically when the authorization flow completes for the first
    # time.
    
    # 토큰이 존재한다면 -> creds를 토큰 값을 넣는다.
    if os.path.exists('token.pickle'):
        with open('token.pickle', 'rb') as token:
            creds = pickle.load(token)

    # If there are no (valid) credentials available, let the user log in.
    # 토큰이 없거나 유효하지 않는다면 -> 따로 인증을 하고나서 토큰을 생성한다.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else: # 토큰이 없다면 'credential.json'파일을 읽어서 creds에 넣어준다.
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open('token.pickle', 'wb') as token:
            pickle.dump(creds, token)

    service = build('calendar', 'v3', credentials=creds) # 서비스 객체를 생성한다.

    # Call the Calendar API
    now = datetime.datetime.utcnow().isoformat() + 'Z' # 'Z' indicates UTC time // 현재 시간을 가져온다.
    print('Getting the upcoming 10 events')
    events_result = service.events().list(calendarId='primary', timeMin=now,
                                        maxResults=10, singleEvents=True,
                                        orderBy='startTime').execute() # .list 메소드로 이벤트를 조회한다.
    events = events_result.get('items', []) #events_result에서 아이템을 가져온다.

    if not events: # 이벤트가 없으면 다음을 출력한다.
        print('No upcoming events found.')
    for event in events: # 이벤트가 있으면 start에 날짜와 시간을 넣는다.
        start = event['start'].get('dateTime', event['start'].get('date'))
        print(start, event['summary']) # start와 이벤트 내용을 출력한다.

    ########################일정 생성######################
    event = {
        'summary': 'itsplay의 OpenAPI 수업', # 일정 제목
        'location': '서울특별시 성북구 정릉동 정릉로 77', # 일정 장소
        'description': 'itsplay와 OpenAPI 수업에 대한 설명입니다.', # 일정 설명
        'start': { # 시작 날짜
            'dateTime': '2020-02-13T09:00:00+09:00', 
            'timeZone': 'Asia/Seoul',
        },
        'end': { # 종료 날짜
            'dateTime': '2020-02-13T10:00:00+09:00', 
            'timeZone': 'Asia/Seoul',
        },
        'recurrence': [ # 반복 지정
            'RRULE:FREQ=DAILY;COUNT=2' # 일단위; 총 2번 반복
        ],
        'attendees': [ # 참석자
            {'email': 'lpage@example.com'},
            {'email': 'sbrin@example.com'},
        ],
        'reminders': { # 알림 설정
            'useDefault': False,
            'overrides': [
                {'method': 'email', 'minutes': 24 * 60}, # 24 * 60분 = 하루 전 알림
                {'method': 'popup', 'minutes': 10}, # 10분 전 알림
            ],
        },
    }

    # calendarId : 캘린더 ID. primary이 기본 값입니다.
    event = service.events().insert(calendarId='primary', body=event).execute()
    print('Event created: %s' % (event.get('htmlLink')))
if __name__ == '__main__':
    main()
```