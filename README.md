# [현대오토에버 사전과제전형_문형규] 
### Springboot 기반 프로젝트 회원가입 및 시스템 관리자 API 서비스


## 🖥️ 초기환경설정 

```
$ git clone --recursive https://github.com/bbangssi91/hyundai-autoever-moon-pretest.git

$ git submodule foreach git pull origin main 

$ mvn clean package

$ docker compose up -d
```

## 🏛️ Project 구조

```
hyundai-autoever-moon-pretest (project-root) /
│
├── docker-compose.yml
├── pom.xml               
│
├── UserAdminApplication/
│   └── Dockerfile
│   └── pom.xml
│
├── MessageConsumerApp/
│   └── Dockerfile
│   └── pom.xml
│
├── KakaoMessageMockingApp/
│   └── Dockerfile
│   └── pom.xml
│
└── SmsMessageMockingApp/
    └── Dockerfile
    └── pom.xml
```

## ✅ API Endpoint 설계

### 1. [관리자] 회원정보 조회 API

`GET /admin/users?page=1&size=100`

- Default 페이지 사이즈 100
- pageSize 최대크기는 300으로 제한

### Sample Call API
```bash

curl -u admin:1212 "http://localhost:80/admin/users?page=1&size=30"
```

### 응답 Sample
```
{
    "status": 200,
    "message": "SUCCESS",
    "timestamp": "2025-08-07T04:58:27.585293096",
    "data": [
        {
            "id": 1,
            "userName": "관리자",
            "accountId": "admin",
            "phoneNumber": "010-0000-0000",
            "roleType": "ADMIN"
        },
        {
            "id": 2,
            "userName": "유저1",
            "accountId": "user01",
            "phoneNumber": "010-0000-0001",
            "roleType": "USER"
        },
        {
            "id": 3,
            "userName": "유저2",
            "accountId": "user02",
            "phoneNumber": "010-0000-0002",
            "roleType": "USER"
        },
        {
            "id": 4,
            "userName": "유저3",
            "accountId": "user03",
            "phoneNumber": "010-0000-0003",
            "roleType": "USER"
        }
    ]
}
```

---

### 2. [관리자] 회원정보 수정 API

`PUT /admin/users`

* 요청 Header
  * accept :: application/json
  * Content-Type :: application/json
* 요청 Body
  * 필드명
    * id (long) (* 필수)
    * password (string)
    * city (string)
    * address (string)

` * 비밀번호, 주소 정보 중 null로 Parameter를 넘길 시, 수정안함`

### Sample Call API
```bash

curl -X PUT "http://localhost:80/admin/users" \
  -H "Content-Type: application/json" \
  -u admin:1212 \
  -d '{
    "id": 2,
    "password": "test",
    "city": "서울특별시",
    "address": "송파구"
}'
```

### 응답 Sample
```
{
    "status": 200,
    "message": "SUCCESS",
    "timestamp": "2025-08-07T04:59:53.165179136",
    "data": {
        "id": 4,
        "before": {
            "password": "$2a$10$yadjiW48DgiEjjGYfPQZZu3T.Y9yOpDWRsEtHR/94n9/PwWQOpZcO",
            "city": "부산광역시",
            "address": "해운대구"
        },
        "after": {
            "password": "$2a$10$LKAly85ivNK8UVtYs2Ezoep8WQFglDXZeR32N2PuK45WArVnQfBHq",
            "city": "서울특별시",
            "address": "송파구"
        }
    }
}
```

---

### 3. [관리자] 전체회원 메시지 발송

`POST /admin/send-message-all-user`

* 요청 Header
  * accept :: application/json
  * Content-Type :: application/json
* 요청 Body
  * 필드명
    * message (string) (* 필수)

`- Redis 큐에 이벤트를 발행한뒤, Consumer App 에서 사용자의 전화번호와 메시지를 전달하여 API 호출`

### Sample Call API

```bash

curl -i -X POST "http://localhost:80/admin/send-message-all-user" \
  -H "Content-Type: application/json" \
  -u admin:1212 \
  -d '{
    "message": "테스트 메시지입니다"
}'
```

### 응답 Sample

```
200 OK
empty body
```

---

### 4. [사용자] 회원가입 API

`POST /api/sign-up`

* 요청 Header
  * accept :: application/json
  * Content-Type :: application/json
* 요청 Body
  * 필드명 (모두 필수)
    * accountId (string)
    * password (string)
    * userName (string)
    * residentRegistrationNumber (string)
    * phoneNumber (string)
    * city (string)
    * address (string)

### Sample Call API

```bash

curl -i -X POST "http://localhost:80/api/sign-up" \
  -H "Content-Type: application/json" \
  -d '{
        "accountId" : "user02",
        "password" : "user02",
        "userName" : "유저02",
        "residentRegistrationNumber" : "111111-2222222",
        "phoneNumber" : "010-1234-1234",
        "city" : "서울특별시",
        "address" : "강남구"
}'
```

### 응답 Sample
```
{
    "status": 200,
    "message": "SUCCESS",
    "timestamp": "2025-08-07T05:27:48.1725163",
    "data": {
        "id": 9,
        "accountId": "user02",
        "userName": "유저02",
        "phoneNumber": "010-1234-1234",
        "city": "서울특별시",
        "address": "강남구"
    }
}

```

---

### 5. [사용자] 사용자 로그인 API

`POST /api/login`

* 요청 Header
  * accept :: application/json
  * Content-Type :: application/json
* 요청 Body
  * 필드명 (모두 필수)
    * accountId (string)
    * password (string)

### Sample Call API

```bash

curl -i -X POST "http://localhost:80/api/login" \
  -H "Content-Type: application/json" \
  -d '{
        "accountId" : "user02",
        "password" : "user02"
}'
```

### 응답 Sample
```
{
    "status": 200,
    "message": "SUCCESS",
    "timestamp": "2025-08-07T10:30:14.45166434",
    "data": {
        "accountId": "user02",
        "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMDIiLCJpZCI6OSwicm9sZSI6IlVTRVIiLCJpYXQiOjE3NTQ1MTIyMTQsImV4cCI6MTc1NDUxNTgxNH0.sOhdnUvIRMeKXW8-oi_baXsrClF9Wfx3sWxlJY-be74"
    }
}

```

---

### 6. [사용자] 사용자 정보 조회 API

`GET /api/users/{id}`

* 요청 Header
  * accept :: application/json
  * Content-Type :: application/json
  * Authorization :: Bearer <ACCESS_TOKEN>

- `5번의 사용자 로그인 API에서 발급받은 ACCESS_TOKEN 을 넣어주어야함`
- `현재 사용자의 ID가 9라 가정할 때, 다른 사람의 정보의 요청은 FORBIDDEN`

### Sample Call API

```bash

curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMDIiLCJpZCI6OSwicm9sZSI6IlVTRVIiLCJpYXQiOjE3NTQ1MTIyMTQsImV4cCI6MTc1NDUxNTgxNH0.sOhdnUvIRMeKXW8-oi_baXsrClF9Wfx3sWxlJY-be74" \
     -H "Content-Type: application/json" \
     http://localhost:80/api/users/9
```

### 응답 Sample
```
{
    "status": 200,
    "message": "SUCCESS",
    "timestamp": "2025-08-07T05:37:12.339422297",
    "data": {
        "id": 9,
        "accountId": "user02",
        "userName": "유저02",
        "city": "서울특별시"
    }
}

```