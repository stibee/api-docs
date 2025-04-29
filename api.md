# 스티비 API 사용 가이드

## 개요

스티비 API를 사용하면 스티비의 기능을 프로그래밍 방식으로 사용할 수 있습니다. 이 가이드에서는 API 사용 방법과 주요 엔드포인트에 대해 설명합니다.

## API 키 발급

### 1. API 키 생성

1. 스티비 서비스의 [워크스페이스 설정] > [API 키] 메뉴로 이동합니다.
2. [API 키 만들기] 버튼을 클릭합니다.
3. API 키 이름을 입력하고 필요한 권한을 선택합니다.
4. [저장] 버튼을 클릭하여 API 키를 생성합니다.

### 2. API 키 관리

- 생성된 API 키는 안전하게 보관해야 합니다.
- API 키는 발급 후 한 번만's 확인할 수 있으므로, 안전한 곳에 저장하세요.
- 더 이상 사용하지 않는 API 키는 즉시 삭제하세요.

## API 인증

모든 API 요청에는 인증을 위해 `AccessToken` 헤더가 필요합니다:

```
GET /lists
AccessToken: 발급받은_API_키
```

## API 요청 속도 제한

| API | 속도 제한 |
|-----|-----------|
| 주소록 구독자 대량 추가 | 최대 10회/분 |
| 구독자 목록 조회 | 최대 100회/분 |
| 그 외 모든 API | 최대 1000회/분 |

## 주요 API 기능

### 1. 구독자 관리 API

#### 구독자 추가

```
POST /lists/{id}/subscribers
```

**요청 예시:**
```json
{
  "subscriber": {
    "email": "example@example.com",
    "fields": {
      "name": "홍길동",
      "age": 30
    }
  }
}
```

**응답 예시:**
```json
{
  "success": true
}
```

#### 구독자 일괄 추가

```
POST /lists/{id}/subscribers/batch
```

**요청 예시:**
```json
{
  "subscribers": [
    {
      "email": "example1@example.com",
      "fields": {
        "name": "홍길동",
        "age": 30
      }
    },
    {
      "email": "example2@example.com",
      "fields": {
        "name": "고길동",
        "age": 40
      }
    }
  ]
}
```

**응답 예시:**
```json
{
  "createdSubscribers": ["example1@example.com", "example2@example.com"],
  "updatedSubscribers": [],
  "failNoEmails": null,
  "failInvalidEmails": null,
  "failDuplicatedEmails": null,
  "failInvalidFields": null,
  "failInvalidSubscriberStatus": null
}
```

#### 구독자 목록 조회

```
GET /lists/{id}/subscribers
```

**쿼리 파라미터:**
- `offset`: 조회 시작 위치 (기본값: 0)
- `limit`: 한 번에 조회할 구독자 수 (기본값: 20, 최대: 100)

**응답 예시:**
```json
{
  "items": [
    {
      "email": "example@example.com",
      "status": "subscribed",
      "fields": {
        "name": "홍길동",
        "age": "30"
      },
      "createdTime": "2023-06-01 14:30:00 +0900 KST",
      "modifiedTime": "2023-06-01 14:30:00 +0900 KST"
    }
  ],
  "totalCount": 1
}
```

#### 구독자 정보 수정

```
PUT /lists/{id}/subscribers/{email}
```

**요청 예시:**
```json
{
  "fields": {
    "name": "홍길동",
    "age": 31
  }
}
```

**응답 예시:**
```json
{
  "success": true
}
```

#### 구독자 삭제

```
DELETE /lists/{id}/subscribers
```

**요청 예시:**
```json
{
  "subscribers": [
    "example1@example.com",
    "example2@example.com"
  ]
}
```

**응답 예시:**
```json
{
  "success": true
}
```

### 2. 이메일 관리 API

#### 이메일 생성

```
POST /emails
```

**요청 예시:**
```json
{
  "subject": "이메일 제목",
  "senderEmail": "sender@example.com",
  "senderName": "발신자 이름",
  "listId": 1234
}
```

**응답 예시:**
```json
{
  "id": 5678
}
```

#### 이메일 목록 조회

```
GET /emails
```

**쿼리 파라미터:**
- `listId`: 주소록 ID (선택 사항)
- `tagIds`: 태그 ID (선택 사항)
- `offset`: 조회 시작 위치 (기본값: 0)
- `limit`: 한 번에 조회할 이메일 수 (기본값: 20)

**응답 예시:**
```json
{
  "items": [
    {
      "id": 5678,
      "subject": "이메일 제목",
      "status": "CREATED",
      "createdTime": "2023-06-01T14:30:00+09:00",
      "senderName": "발신자 이름",
      "senderEmail": "sender@example.com"
    }
  ],
  "totalCount": 1
}
```

#### 이메일 내용 수정

```
POST /emails/{id}/content
```

**요청 예시:**
```html
<html>
  <head>
  </head>
  <body>
    <h1>이메일 내용입니다</h1>
    <p>안녕하세요, {{name}}님!</p>
  </body>
</html>
```

**응답 예시:**
```json
{
  "success": true
}
```

#### 이메일 발송

```
POST /emails/{id}/send
```

**응답 예시:**
```json
{
  "success": true
}
```

#### 이메일 예약 발송

```
POST /emails/{id}/reserve
```

**쿼리 파라미터:**
- `reserveTime`: 예약 발송 시간 (YYYYMMDDhhmmss 형식, 대한민국 표준시 기준)

**응답 예시:**
```json
{
  "success": true
}
```

### 3. 주소록 관리 API

#### 주소록 생성

```
POST /lists
```

**요청 예시:**
```json
{
  "name": "주소록 이름",
  "senderName": "발신자 이름",
  "companyName": "회사명",
  "companyContact": "02-123-4567",
  "companyAddress": "서울시 강남구",
  "useSubscriberAutoDelete": true,
  "canSendToUnsubscribed": false
}
```

**응답 예시:**
```json
{
  "id": 1234
}
```

#### 주소록 목록 조회

```
GET /lists
```

**쿼리 파라미터:**
- `offset`: 조회 시작 위치 (기본값: 0)
- `limit`: 한 번에 조회할 주소록 수 (기본값: 50)

**응답 예시:**
```json
{
  "items": [
    {
      "id": 1234,
      "name": "주소록 이름",
      "createdTime": "2023-06-01T14:30:00+09:00",
      "subscriberCount": 100
    }
  ],
  "totalCount": 1
}
```

### 4. 그룹 관리 API

#### 그룹 생성

```
POST /lists/{id}/groups
```

**요청 예시:**
```json
{
  "name": "그룹 이름"
}
```

**응답 예시:**
```json
{
  "id": 9876
}
```

#### 그룹 목록 조회

```
GET /lists/{id}/groups
```

**응답 예시:**
```json
[
  {
    "id": 9876,
    "name": "그룹 이름",
    "createdTime": "2023-06-01T14:30:00+09:00"
  }
]
```

#### 구독자를 그룹에 할당

```
POST /lists/{id}/groups/{groupId}/assign
```

**요청 예시:**
```json
{
  "subscriber": "example@example.com"
}
```

**응답 예시:**
```json
{
  "success": true
}
```

## 오류 처리

API 요청 실패 시 다음과 같은 형식의 오류 응답이 반환됩니다:

```json
{
  "code": "Errors.Data.InvalidRequest",
  "httpStatusCode": 400,
  "message": "잘못된 요청입니다."
}
```

주요 오류 코드는 다음과 같습니다:

| HTTP 상태 코드 | 오류 코드 | 설명 |
|--------------|------------|------|
| 400 | Errors.Data.InvalidRequest | 요청 형식이 잘못되었습니다. |
| 400 | Errors.Data.NotFound | 요청한 리소스를 찾을 수 없습니다. |
| 400 | Errors.Data.AlreadyExists | 이미 존재하는 리소스입니다. |
| 401 | Errors.Auth.InvalidToken | 유효하지 않은 API 키입니다. |
| 403 | Errors.Auth.InsufficientPermissions | 필요한 권한이 없습니다. |
| 429 | Errors.RateLimit.Exceeded | API 요청 속도 제한을 초과했습니다. |

## 코드 예제

### API 키를 사용한 구독자 추가 (Node.js)

```javascript
const axios = require('axios');

async function addSubscriber() {
  try {
    const response = await axios.post(
      'https://api.stibee.com/v2/lists/1234/subscribers',
      {
        subscriber: {
          email: 'example@example.com',
          fields: {
            name: '홍길동',
            age: 30
          }
        }
      },
      {
        headers: {
          'AccessToken': '발급받은_API_키',
          'Content-Type': 'application/json'
        }
      }
    );
    
    console.log('구독자 추가 성공:', response.data);
  } catch (error) {
    console.error('구독자 추가 실패:', error.response?.data || error.message);
  }
}

addSubscriber();
```

### 이메일 발송 (Python)

```python
import requests

def send_email(email_id):
    url = f"https://api.stibee.com/v2/emails/{email_id}/send"
    headers = {
        "AccessToken": "발급받은_API_키"
    }
    
    response = requests.post(url, headers=headers)
    
    if response.status_code == 200:
        print("이메일 발송 성공:", response.json())
    else:
        print("이메일 발송 실패:", response.json())

# 이메일 ID 5678 발송
send_email(5678)
```

### 구독자 목록 조회 (PHP)

```php
<?php
$listId = 1234;
$apiKey = '발급받은_API_키';

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "https://api.stibee.com/v2/lists/{$listId}/subscribers?offset=0&limit=50");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    'AccessToken: ' . $apiKey
));

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

if ($httpCode == 200) {
    $subscribers = json_decode($response, true);
    echo "구독자 목록:\n";
    foreach ($subscribers['items'] as $subscriber) {
        echo $subscriber['email'] . " - " . $subscriber['fields']['name'] . "\n";
    }
} else {
    echo "오류 발생: " . $response . "\n";
}
?>
```

## 모범 사례

### 1. 오류 처리

모든 API 요청에 대해 오류 처리를 구현하여 예기치 않은 상황에 대응할 수 있도록 합니다.

```javascript
try {
  // API 요청 코드
} catch (error) {
  if (error.response) {
    // 서버가 응답한 오류 처리
    console.error('API 오류:', error.response.data);
    
    if (error.response.status === 429) {
      // 속도 제한 초과 처리
      setTimeout(retryRequest, 60000); // 1분 후 재시도
    }
  } else if (error.request) {
    // 요청은 전송되었으나 응답이 없는 경우
    console.error('응답 없음:', error.request);
  } else {
    // 요청 설정 중 오류 발생
    console.error('요청 오류:', error.message);
  }
}
```

### 2. 배치 처리

많은 구독자를 추가할 때는 개별 API 호출보다 배치 API를 사용하는 것이 효율적입니다.

```javascript
// 개별 추가 (비효율적)
for (const subscriber of subscribers) {
  await addSubscriber(subscriber); // API 호출 100번
}

// 배치 추가 (효율적)
const chunks = chunkArray(subscribers, 100); // 100명씩 그룹화
for (const chunk of chunks) {
  await addSubscribersBatch(chunk); // API 호출 1번으로 100명 추가
}
```

### 3. 속도 제한 관리

API 요청 속도 제한을 초과하지 않도록 요청 간격을 조절합니다.

```javascript
async function callApiWithRateLimit(apiFunction) {
  return new Promise((resolve, reject) => {
    setTimeout(async () => {
      try {
        const result = await apiFunction();
        resolve(result);
      } catch (error) {
        reject(error);
      }
    }, 1000); // 1초 간격으로 요청
  });
}

// 사용 예
const subscribers = ['user1@example.com', 'user2@example.com', ...];
for (const email of subscribers) {
  const result = await callApiWithRateLimit(() => getSubscriberInfo(email));
  console.log(result);
}
```

## 추가 자료

더 자세한 API 사용 방법은 다음 링크를 참고하세요:
- [스티비 API 문서](https://api.stibee.com/docs/)
- [스티비 도움말: API 사용하기](https://help.stibee.com/api-webhook/api)
- [스티비 도움말: 웹훅 사용하기](https://help.stibee.com/api-webhook/list-webhook) 