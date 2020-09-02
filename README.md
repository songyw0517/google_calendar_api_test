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
