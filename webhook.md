# 스티비 웹훅(Webhook) 가이드

## 개요

웹훅은 특정 이벤트가 발생했을 때 외부 시스템에 실시간으로 알림을 보내는 기능입니다. 스티비의 웹훅을 사용하면 구독자 정보 변경, 수신거부 등의 이벤트가 발생할 때마다 자동으로 설정한 URL로 데이터를 전송합니다.

이를 통해 스티비 주소록과 외부 데이터베이스를 실시간으로 동기화하거나, 특정 이벤트에 따른 자동화된 작업을 수행할 수 있습니다.

## 웹훅 사용 사례

- **DB 동기화**: 스티비 주소록의 구독자 정보가 변경될 때 외부 DB에도 자동으로 반영
- **마케팅 자동화**: 새 구독자가 추가되면 자동 환영 메시지 발송
- **분석 및 보고**: 구독 취소 이벤트 데이터를 수집하여 이탈률 분석
- **통합 워크플로우**: 수신거부 이벤트 발생 시 다른 마케팅 채널에서도 해당 정보 업데이트

## 웹훅 설정 방법

### 1. 웹훅 생성하기

1. 스티비 서비스의 [워크스페이스 설정] > [API 및 웹훅] 메뉴로 이동합니다.
2. [웹훅 설정] 탭을 선택하고 [웹훅 생성하기] 버튼을 클릭합니다.
3. 웹훅 이름과 알림을 받을 URL을 입력합니다.
4. 알림을 받을 이벤트 유형을 선택합니다.
5. [저장] 버튼을 클릭하여 웹훅을 생성합니다.

### 2. 웹훅 이벤트 유형

스티비는 다음과 같은 웹훅 이벤트 유형을 제공합니다:

| 이벤트 유형 | 설명 |
|------------|------|
| `subscriber_created` | 구독자가 주소록에 추가되었을 때 발생 |
| `subscriber_updated` | 구독자 정보가 수정되었을 때 발생 |
| `subscriber_deleted` | 구독자가 삭제되었을 때 발생 |
| `subscriber_unsubscribed` | 구독자가 수신거부했을 때 발생 |

## 웹훅 데이터 형식

웹훅은 HTTP POST 요청으로 전송되며, 요청 본문은 JSON 형식입니다. 예시는 다음과 같습니다:

### 구독자 추가 이벤트 (subscriber_created)

```json
{
  "event": "subscriber_created",
  "listId": 1234,
  "data": {
    "email": "example@example.com",
    "status": "subscribed",
    "fields": {
      "name": "홍길동",
      "age": "30"
    },
    "createdTime": "2023-06-01 14:30:00 +0900 KST"
  }
}
```

### 구독자 정보 수정 이벤트 (subscriber_updated)

```json
{
  "event": "subscriber_updated",
  "listId": 1234,
  "data": {
    "email": "example@example.com",
    "status": "subscribed",
    "fields": {
      "name": "홍길동",
      "age": "31"
    },
    "createdTime": "2023-06-01 14:30:00 +0900 KST",
    "modifiedTime": "2023-06-15 09:45:00 +0900 KST"
  }
}
```

### 구독자 삭제 이벤트 (subscriber_deleted)

```json
{
  "event": "subscriber_deleted",
  "listId": 1234,
  "data": {
    "email": "example@example.com"
  }
}
```

### 수신거부 이벤트 (subscriber_unsubscribed)

```json
{
  "event": "subscriber_unsubscribed",
  "listId": 1234,
  "data": {
    "email": "example@example.com",
    "status": "unsubscribed",
    "fields": {
      "name": "홍길동",
      "age": "30"
    },
    "createdTime": "2023-06-01 14:30:00 +0900 KST",
    "modifiedTime": "2023-06-20 16:20:00 +0900 KST"
  }
}
```

## 웹훅 응답 처리

웹훅을 수신하는 서버는 다음과 같은 요구사항을 충족해야 합니다:

1. **HTTP 상태 코드**: 웹훅 처리에 성공한 경우 HTTP 상태 코드 200을 반환해야 합니다.
2. **응답 시간**: 웹훅 수신 후 10초 이내에 응답해야 합니다.
3. **재시도 정책**: 응답이 실패하거나 시간 초과가 발생하면 스티비는 자동으로 최대 3회까지 재시도합니다.

## 웹훅 보안

웹훅 URL은 외부에서 접근할 수 있는 공개 URL이므로, 보안에 유의해야 합니다:

1. **HTTPS 사용**: 웹훅 URL은 HTTPS 프로토콜을 사용하는 것이 좋습니다.
2. **토큰 검증**: 웹훅 수신 서버에서는 요청 헤더의 인증 토큰을 검증하여 정당한 요청인지 확인해야 합니다.
3. **IP 접근 제한**: 가능한 경우 스티비의 IP 주소에서만 요청을 수락하도록 설정하세요.

## 코드 예제: 웹훅 처리 서버 구현

### Node.js (Express)

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());

// 웹훅 엔드포인트
app.post('/webhook', (req, res) => {
  const { event, listId, data } = req.body;
  
  // 이벤트 유형에 따른 처리
  switch (event) {
    case 'subscriber_created':
      console.log(`새 구독자 추가: ${data.email}`);
      // 구독자 추가 처리 로직
      break;
    case 'subscriber_updated':
      console.log(`구독자 정보 수정: ${data.email}`);
      // 구독자 정보 수정 처리 로직
      break;
    case 'subscriber_deleted':
      console.log(`구독자 삭제: ${data.email}`);
      // 구독자 삭제 처리 로직
      break;
    case 'subscriber_unsubscribed':
      console.log(`구독 취소: ${data.email}`);
      // 구독 취소 처리 로직
      break;
    default:
      console.log(`알 수 없는 이벤트: ${event}`);
  }
  
  // 성공 응답 반환
  res.status(200).send('Webhook received');
});

app.listen(3000, () => {
  console.log('Webhook server listening on port 3000');
});
```

### Python (Flask)

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    event = data.get('event')
    list_id = data.get('listId')
    subscriber_data = data.get('data')
    
    # 이벤트 유형에 따른 처리
    if event == 'subscriber_created':
        print(f"새 구독자 추가: {subscriber_data.get('email')}")
        # 구독자 추가 처리 로직
    elif event == 'subscriber_updated':
        print(f"구독자 정보 수정: {subscriber_data.get('email')}")
        # 구독자 정보 수정 처리 로직
    elif event == 'subscriber_deleted':
        print(f"구독자 삭제: {subscriber_data.get('email')}")
        # 구독자 삭제 처리 로직
    elif event == 'subscriber_unsubscribed':
        print(f"구독 취소: {subscriber_data.get('email')}")
        # 구독 취소 처리 로직
    else:
        print(f"알 수 없는 이벤트: {event}")
    
    # 성공 응답 반환
    return jsonify({'status': 'success'}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## 웹훅 관리

웹훅 관리는 [워크스페이스 설정] > [API 및 웹훅] > [웹훅 설정] 메뉴에서 수행할 수 있습니다:

- **웹훅 수정**: 기존 웹훅의 URL, 이벤트 유형 등 설정 변경
- **웹훅 비활성화**: 필요에 따라 웹훅을 일시적으로 비활성화
- **웹훅 삭제**: 더 이상 필요하지 않은 웹훅 삭제
- **웹훅 이력 확인**: 웹훅 발송 이력 및 성공/실패 여부 확인

## 문제 해결

웹훅 작동 시 다음과 같은 문제가 발생할 수 있습니다:

1. **웹훅 전송 실패**
   - 웹훅 URL이 올바른지 확인하세요.
   - 웹훅 수신 서버가 정상 작동 중인지 확인하세요.
   - 방화벽 설정으로 인해 요청이 차단되지 않는지 확인하세요.

2. **이벤트 누락**
   - 웹훅 설정에서 해당 이벤트 유형이 활성화되어 있는지 확인하세요.
   - 웹훅 이력에서 발송 상태를 확인하세요.

3. **중복 이벤트 수신**
   - 웹훅 수신 처리 로직에서 중복 이벤트를 처리할 수 있도록 설계하세요.
   - 이벤트 ID를 기록하여 중복 처리를 방지하세요.

## 추가 자료

더 자세한 웹훅 사용 방법은 다음 링크를 참고하세요:
- [스티비 도움말: 웹훅 사용하기](https://help.stibee.com/api-webhook/list-webhook)
- [스티비 API 문서](https://api.stibee.com/docs/) 