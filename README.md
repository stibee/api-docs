# 스티비 API 가이드

## 개요

스티비 API를 사용하면 외부 시스템과 스티비를 연동하여 이메일 마케팅을 자동화할 수 있습니다. 이 가이드는 스티비 API 사용 방법을 설명합니다.

API를 통해 다음과 같은 작업을 수행할 수 있습니다:
- 구독자 데이터 관리 (추가, 수정, 삭제)
- 이메일 캠페인 생성 및 발송
- 통계 데이터 조회
- 주소록 및 그룹 관리

## 시작하기

### 요금제 및 API 권한

| 요금제 | 구독자 API | 이메일, 그룹 API | 주소록, 발신자 주소 API |
|--------|------------|-----------------|------------------------|
| 스탠다드 | ✅ | ❌ | ❌ |
| 프로 | ✅ | ✅ | ❌ |
| 엔터프라이즈 | ✅ | ✅ | ✅ |

### API 키 발급

1. 스티비 서비스의 [워크스페이스 설정] > [API 키] 메뉴로 이동합니다.
2. [API 키 만들기] 버튼을 클릭하여 새로운 API 키를 생성합니다.
3. 생성된 API 키를 안전하게 보관하세요. 이 키는 모든 API 요청에 필요합니다.

### 기본 URL

모든 API 요청의 기본 URL은 다음과 같습니다:
```
https://api.stibee.com/v2
```

### 인증

모든 API 요청에는 인증을 위해 `AccessToken` 헤더가 필요합니다.

```
GET /emails
AccessToken: 발급받은_API_키
```

## API 요청 속도 제한

| API | 속도 제한 |
|-----|-----------|
| 주소록 구독자 대량 추가 | 최대 10회/분 |
| 구독자 목록 조회 | 최대 100회/분 |
| 그 외 모든 API | 최대 1000회/분 |

## 오류 응답

API 요청 실패 시 다음과 같은 형식의 오류 응답이 반환됩니다:

```json
{
  "code": "Errors.Data.InvalidRequest",
  "httpStatusCode": 400,
  "message": "잘못된 요청입니다."
}
```

## 주요 API 기능

### 구독자 관리 API

#### 구독자 추가
```
POST /lists/{id}/subscribers
```

구독자를 한 명씩 또는 여러 명 일괄 추가할 수 있습니다.

#### 구독자 조회
```
GET /lists/{id}/subscribers
```

구독자 목록을 조회할 수 있습니다.

#### 구독자 정보 수정
```
PUT /lists/{id}/subscribers/{email}
```

특정 구독자의 정보를 수정할 수 있습니다.

#### 구독자 삭제
```
DELETE /lists/{id}/subscribers
```

구독자를 주소록에서 삭제할 수 있습니다.

### 이메일 관리 API

#### 이메일 생성
```
POST /emails
```

새로운 이메일을 생성할 수 있습니다.

#### 이메일 목록 조회
```
GET /emails
```

이메일 목록을 조회할 수 있습니다.

#### 이메일 발송
```
POST /emails/{id}/send
```

이메일을 즉시 발송할 수 있습니다.

#### 이메일 예약 발송
```
POST /emails/{id}/reserve
```

이메일 발송을 예약할 수 있습니다.

## 웹훅(Webhook) 기능

웹훅을 통해 스티비에서 발생하는 이벤트(구독자 추가, 구독 취소 등)를 실시간으로 알림 받을 수 있습니다.

### 웹훅 설정 방법

1. 스티비 서비스의 [설정] > [API 및 웹훅] 메뉴로 이동합니다.
2. [웹훅 생성하기] 버튼을 클릭합니다.
3. 웹훅 이름과 알림을 받을 URL을 입력합니다.
4. 알림을 받을 이벤트 유형을 선택합니다.

### 웹훅 이벤트 유형

- `subscriber_created`: 구독자 추가
- `subscriber_updated`: 구독자 정보 수정
- `subscriber_deleted`: 구독자 삭제
- `subscriber_unsubscribed`: 구독 취소

### 웹훅 응답 형식

웹훅으로 전달되는 데이터는 다음과 같은 형식을 갖습니다:

```json
{
  "event": "subscriber_created",
  "listId": 1234,
  "data": {
    "email": "example@example.com",
    "fields": {
      "name": "홍길동",
      "age": "30"
    }
  }
}
```

## 코드 예제

### API 키를 사용한 인증 예제 (Node.js)

```javascript
const axios = require('axios');

async function fetchEmails() {
  try {
    const response = await axios.get('https://api.stibee.com/v2/emails', {
      headers: {
        'AccessToken': '발급받은_API_키'
      }
    });
    console.log(response.data);
  } catch (error) {
    console.error('API 요청 실패:', error.response.data);
  }
}

fetchEmails();
```

### 구독자 추가 예제 (Python)

```python
import requests

url = "https://api.stibee.com/v2/lists/1234/subscribers"
headers = {
    "AccessToken": "발급받은_API_키",
    "Content-Type": "application/json"
}
data = {
    "subscriber": {
        "email": "example@example.com",
        "fields": {
            "name": "홍길동",
            "age": 30
        }
    }
}

response = requests.post(url, headers=headers, json=data)
print(response.json())
```

## 추가 자료

더 자세한 API 사용 방법은 다음 링크를 참고하세요:
- [스티비 API 문서](https://api.stibee.com/docs/)
- [스티비 도움말: API 사용하기](https://help.stibee.com/api-webhook/api)
- [스티비 도움말: 웹훅 사용하기](https://help.stibee.com/api-webhook/list-webhook)
