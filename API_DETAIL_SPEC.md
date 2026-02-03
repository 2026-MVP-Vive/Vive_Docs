# 설스터디 MVP API 명세서

> Backend: Java 17 + Spring Boot 3.x
> 인증: JWT Bearer Token
> 응답 형식: JSON
> Base URL: `/api/v1`

---

## 목차

1. [공통 사항](#1-공통-사항)
2. [인증 API](#2-인증-api)
3. [멘티 API](#3-멘티-api)
4. [멘토 API](#4-멘토-api)
5. [파일 API](#5-파일-api)

---

## 1. 공통 사항

### 1.1 인증 헤더

```
Authorization: Bearer {accessToken}
```

### 1.2 공통 응답 형식

**성공 응답**

```json
{
  "success": true,
  "data": { ... },
  "message": null
}
```

**에러 응답**

```json
{
  "success": false,
  "data": null,
  "message": "에러 메시지",
  "errorCode": "ERROR_CODE"
}
```

### 1.3 공통 에러 코드

| HTTP Status | Error Code     | 설명        |
| ----------- | -------------- | ----------- |
| 400         | BAD_REQUEST    | 잘못된 요청 |
| 401         | UNAUTHORIZED   | 인증 실패   |
| 403         | FORBIDDEN      | 권한 없음   |
| 404         | NOT_FOUND      | 리소스 없음 |
| 500         | INTERNAL_ERROR | 서버 오류   |

### 1.4 과목 코드

| 코드    | 과목명 |
| ------- | ------ |
| KOREAN  | 국어   |
| ENGLISH | 영어   |
| MATH    | 수학   |

### 1.5 사용자 역할

| 코드   | 역할 |
| ------ | ---- |
| MENTOR | 멘토 |
| MENTEE | 멘티 |

---

## 2. 인증 API

### 2.1 로그인

사전 세팅된 테스트 계정으로 로그인합니다.

**Endpoint**

```
POST /api/v1/auth/login
```

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| loginId | String | O | 로그인 아이디 |
| password | String | O | 비밀번호 |

**Response**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "tokenType": "Bearer",
    "expiresIn": 3600,
    "user": {
      "id": 1,
      "name": "민유진",
      "role": "MENTEE",
      "profileImageUrl": "/files/profile/1.jpg"
    }
  }
}
```

---

### 2.2 토큰 갱신

**Endpoint**

```
POST /api/v1/auth/refresh
```

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| refreshToken | String | O | 리프레시 토큰 |

**Response**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 3600
  }
}
```

---

### 2.3 로그아웃

**Endpoint**

```
POST /api/v1/auth/logout
```

**Headers**

- Authorization: Bearer {accessToken}

**Response**

```json
{
  "success": true,
  "data": null,
  "message": "로그아웃 되었습니다."
}
```

---

## 3. 멘티 API

### 3.1 할 일 목록 조회 (일일 플래너)

특정 날짜의 할 일 목록을 조회합니다.

**Endpoint**

```
GET /api/v1/mentee/tasks
```

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| date | String | O | 조회 날짜 (YYYY-MM-DD) |

**Response**

```json
{
  "success": true,
  "data": {
    "date": "2025-01-27",
    "tasks": [
      {
        "id": 1,
        "title": "수능완성 5회 1~11번",
        "subject": "KOREAN",
        "subjectName": "국어",
        "goalId": 10,
        "goalTitle": "문학 개념어 정리",
        "studyTime": 45,
        "isCompleted": false,
        "isMentorAssigned": true,
        "isMentorConfirmed": false,
        "hasSubmission": true,
        "hasFeedback": false,
        "materialCount": 1
      },
      {
        "id": 2,
        "title": "EBS 올림포스 Unit 5",
        "subject": "ENGLISH",
        "subjectName": "영어",
        "goalId": 11,
        "goalTitle": "빈칸 추론 유형 정복",
        "studyTime": null,
        "isCompleted": false,
        "isMentorAssigned": true,
        "isMentorConfirmed": false,
        "hasSubmission": false,
        "hasFeedback": false,
        "materialCount": 2
      }
    ],
    "summary": {
      "total": 5,
      "completed": 2,
      "totalStudyTime": 120
    }
  }
}
```

---

### 3.2 할 일 상세 조회 (과제 상세)

**Endpoint**

```
GET /api/v1/mentee/tasks/{taskId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| taskId | Long | O | 할 일 ID |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "title": "수능완성 5회 1~11번",
    "date": "2025-01-27",
    "subject": "KOREAN",
    "subjectName": "국어",
    "goal": {
      "id": 10,
      "title": "문학 개념어 정리",
      "subject": "KOREAN"
    },
    "studyTime": 45,
    "isMentorAssigned": true,
    "isMentorConfirmed": false,
    "materials": [
      {
        "id": 101,
        "fileName": "수능완성_국어_5회.pdf",
        "fileType": "PDF",
        "fileSize": 2048576,
        "downloadUrl": "/api/v1/files/101/download"
      }
    ],
    "submission": {
      "id": 201,
      "imageUrl": "/api/v1/files/201",
      "submittedAt": "2025-01-27T14:30:00"
    },
    "feedback": null,
    "createdAt": "2025-01-26T09:00:00"
  }
}
```

---

### 3.3 멘티 할 일 추가

멘티가 직접 추가하는 할 일 (멘토 지정 X)

**Endpoint**

```
POST /api/v1/mentee/tasks
```

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | O | 할 일 제목 |
| date | String | O | 수행 날짜 (YYYY-MM-DD) |
| subject | String | X | 과목 코드 (KOREAN/ENGLISH/MATH) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 15,
    "title": "오답노트 정리",
    "date": "2025-01-27",
    "subject": "MATH",
    "isMentorAssigned": false
  }
}
```

---

### 3.4 공부 시간 기록

**Endpoint**

```
PATCH /api/v1/mentee/tasks/{taskId}/study-time
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| taskId | Long | O | 할 일 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| studyTime | Integer | O | 공부 시간 (분 단위) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "studyTime": 60
  }
}
```

---

### 3.5 과제 제출 (인증 사진 업로드)

**Endpoint**

```
POST /api/v1/mentee/tasks/{taskId}/submission
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| taskId | Long | O | 할 일 ID |

**Request Body (multipart/form-data)**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| image | File | O | 인증 사진 (JPG, PNG) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 201,
    "taskId": 1,
    "imageUrl": "/api/v1/files/201",
    "submittedAt": "2025-01-27T14:30:00"
  }
}
```

---

### 3.6 어제자 피드백 목록 조회

**Endpoint**

```
GET /api/v1/mentee/feedbacks/yesterday
```

**Response**

```json
{
  "success": true,
  "data": {
    "date": "2025-01-26",
    "feedbacks": [
      {
        "id": 301,
        "taskId": 5,
        "taskTitle": "수능완성 4회 전범위",
        "subject": "KOREAN",
        "subjectName": "국어",
        "isImportant": true,
        "summary": "접속부사 활용 부분 다시 복습 필요",
        "createdAt": "2025-01-26T22:00:00"
      }
    ],
    "overallComment": "전체적으로 집중력이 좋았습니다. 국어 문학 부분 조금 더 신경 써주세요."
  }
}
```

---

### 3.7 날짜별 피드백 상세 조회

**Endpoint**

```
GET /api/v1/mentee/feedbacks
```

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| date | String | O | 조회 날짜 (YYYY-MM-DD) |

**Response**

```json
{
  "success": true,
  "data": {
    "date": "2025-01-26",
    "feedbacks": [
      {
        "id": 301,
        "taskId": 5,
        "taskTitle": "수능완성 4회 전범위",
        "subject": "KOREAN",
        "subjectName": "국어",
        "isImportant": true,
        "summary": "접속부사 활용 부분 다시 복습 필요",
        "content": "오늘 풀이한 문학 작품 해석은 전반적으로 잘 했습니다. 다만 접속부사 활용 부분에서 아직 개념이 헷갈리는 것 같아요. 내일 추가 자료 올려드릴게요.",
        "createdAt": "2025-01-26T22:00:00"
      },
      {
        "id": 302,
        "taskId": 6,
        "taskTitle": "EBS 올림포스 Unit 4",
        "subject": "ENGLISH",
        "subjectName": "영어",
        "isImportant": false,
        "summary": null,
        "content": "빈칸 추론 정답률이 많이 올랐어요! 이 페이스 유지해주세요.",
        "createdAt": "2025-01-26T22:05:00"
      }
    ],
    "overallComment": "전체적으로 집중력이 좋았습니다. 국어 문학 부분 조금 더 신경 써주세요.",
    "mentorName": "김멘토"
  }
}
```

---

### 3.8 약점 맞춤 솔루션 목록 조회

**Endpoint**

```
GET /api/v1/mentee/solutions
```

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| subject | String | X | 과목 필터 (KOREAN/ENGLISH/MATH, 미입력 시 전체) |

**Response**

```json
{
  "success": true,
  "data": {
    "solutions": [
      {
        "id": 10,
        "title": "문학 개념어 정리",
        "subject": "KOREAN",
        "subjectName": "국어",
        "materials": [
          {
            "id": 501,
            "fileName": "문학개념어_총정리.pdf",
            "fileType": "PDF",
            "downloadUrl": "/api/v1/files/501/download"
          }
        ]
      },
      {
        "id": 11,
        "title": "빈칸 추론 유형 정복",
        "subject": "ENGLISH",
        "subjectName": "영어",
        "materials": [
          {
            "id": 502,
            "fileName": "빈칸추론_공략법.pdf",
            "fileType": "PDF",
            "downloadUrl": "/api/v1/files/502/download"
          }
        ]
      }
    ]
  }
}
```

---

### 3.9 월간 계획표 조회

**Endpoint**

```
GET /api/v1/mentee/monthly-plan
```

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| year | Integer | O | 연도 |
| month | Integer | O | 월 (1-12) |

**Response**

```json
{
  "success": true,
  "data": {
    "year": 2025,
    "month": 1,
    "plans": [
      {
        "date": "2025-01-27",
        "dayOfWeek": "MON",
        "taskCount": 5,
        "completedCount": 2,
        "hasTask": true
      },
      {
        "date": "2025-01-28",
        "dayOfWeek": "TUE",
        "taskCount": 4,
        "completedCount": 0,
        "hasTask": true
      }
    ]
  }
}
```

---

### 3.10 서울대쌤 칼럼 목록 조회

**Endpoint**

```
GET /api/v1/mentee/columns
```

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| page | Integer | X | 페이지 번호 (기본값: 0) |
| size | Integer | X | 페이지 크기 (기본값: 10) |

**Response**

```json
{
  "success": true,
  "data": {
    "columns": [
      {
        "id": 1,
        "title": "수능 D-100 학습 전략",
        "summary": "수능까지 100일, 효과적인 마무리 전략을 알려드립니다.",
        "thumbnailUrl": "/api/v1/files/column/1/thumbnail",
        "createdAt": "2025-01-15"
      },
      {
        "id": 2,
        "title": "국어 비문학 독해 꿀팁",
        "summary": "비문학 지문을 빠르고 정확하게 읽는 방법",
        "thumbnailUrl": "/api/v1/files/column/2/thumbnail",
        "createdAt": "2025-01-10"
      }
    ],
    "pagination": {
      "page": 0,
      "size": 10,
      "totalElements": 11,
      "totalPages": 2
    }
  }
}
```

---

### 3.11 칼럼 상세 조회

**Endpoint**

```
GET /api/v1/mentee/columns/{columnId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| columnId | Long | O | 칼럼 ID |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "title": "수능 D-100 학습 전략",
    "content": "수능까지 100일, 효과적인 마무리 전략을 알려드립니다...(본문 내용)",
    "author": "서울대 국어국문학과",
    "createdAt": "2025-01-15",
    "attachments": [
      {
        "id": 601,
        "fileName": "D-100_체크리스트.pdf",
        "downloadUrl": "/api/v1/files/601/download"
      }
    ]
  }
}
```

---

### 3.12 마이페이지 프로필 조회

**Endpoint**

```
GET /api/v1/mentee/profile
```

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "민유진",
    "profileImageUrl": "/api/v1/files/profile/1",
    "mentorName": "김멘토",
    "startDate": "2024-12-01",
    "totalStudyDays": 58
  }
}
```

---

### 3.13 과목별 달성률 조회

**Endpoint**

```
GET /api/v1/mentee/achievement
```

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| startDate | String | X | 시작 날짜 (YYYY-MM-DD, 기본값: 이번 주 시작) |
| endDate | String | X | 종료 날짜 (YYYY-MM-DD, 기본값: 오늘) |

**Response**

```json
{
  "success": true,
  "data": {
    "period": {
      "startDate": "2025-01-20",
      "endDate": "2025-01-27"
    },
    "achievements": [
      {
        "subject": "KOREAN",
        "subjectName": "국어",
        "totalTasks": 10,
        "completedTasks": 8,
        "completionRate": 80,
        "totalStudyTime": 300
      },
      {
        "subject": "ENGLISH",
        "subjectName": "영어",
        "totalTasks": 8,
        "completedTasks": 7,
        "completionRate": 87,
        "totalStudyTime": 240
      },
      {
        "subject": "MATH",
        "subjectName": "수학",
        "totalTasks": 12,
        "completedTasks": 10,
        "completionRate": 83,
        "totalStudyTime": 360
      }
    ],
    "overall": {
      "totalTasks": 30,
      "completedTasks": 25,
      "completionRate": 83,
      "totalStudyTime": 900
    }
  }
}
```

---

### 3.14 주간 학습리포트 목록 조회

**Endpoint**

```
GET /api/v1/mentee/weekly-reports
```

**Response**

```json
{
  "success": true,
  "data": {
    "reports": [
      {
        "id": 1,
        "week": 4,
        "title": "4주차 (25.12.21~27)",
        "startDate": "2024-12-21",
        "endDate": "2024-12-27",
        "isAvailable": true
      },
      {
        "id": 2,
        "week": 3,
        "title": "3주차 (25.12.14~20)",
        "startDate": "2024-12-14",
        "endDate": "2024-12-20",
        "isAvailable": true
      }
    ]
  }
}
```

---

### 3.15 주간 학습리포트 상세 조회

**Endpoint**

```
GET /api/v1/mentee/weekly-reports/{reportId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| reportId | Long | O | 리포트 ID |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "week": 4,
    "title": "4주차 (25.12.21~27)",
    "startDate": "2024-12-21",
    "endDate": "2024-12-27",
    "summary": "이번 주 전체적으로 잘 수행했습니다.",
    "subjectReports": [
      {
        "subject": "KOREAN",
        "subjectName": "국어",
        "completionRate": 90,
        "totalStudyTime": 180,
        "feedback": "문학 영역 집중 학습 잘 진행됨"
      },
      {
        "subject": "ENGLISH",
        "subjectName": "영어",
        "completionRate": 85,
        "totalStudyTime": 150,
        "feedback": "빈칸 추론 정답률 향상"
      },
      {
        "subject": "MATH",
        "subjectName": "수학",
        "completionRate": 80,
        "totalStudyTime": 200,
        "feedback": "미적분 개념 보완 필요"
      }
    ],
    "overallFeedback": "전체적으로 학습 루틴이 잘 잡혀가고 있습니다. 다음 주에는 수학 미적분 부분에 조금 더 시간을 투자해주세요.",
    "createdAt": "2024-12-28"
  }
}
```

---

### 3.16 코멘트/질문 등록

**Endpoint**

```
POST /api/v1/mentee/comments
```

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| content | String | O | 코멘트/질문 내용 |
| date | String | O | 관련 날짜 (YYYY-MM-DD) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 401,
    "content": "오늘 국어 문제 풀다가 막히는 부분이 있었어요",
    "date": "2025-01-27",
    "createdAt": "2025-01-27T15:00:00"
  }
}
```

---

## 4. 멘토 API

### 4.1 담당 멘티 목록 조회

**Endpoint**

```
GET /api/v1/mentor/students
```

**Response**

```json
{
  "success": true,
  "data": {
    "students": [
      {
        "id": 1,
        "name": "민유진",
        "profileImageUrl": "/api/v1/files/profile/1",
        "todayTaskSummary": {
          "total": 5,
          "completed": 2,
          "pendingConfirmation": 2
        },
        "lastFeedbackDate": "2025-01-26",
        "hasPendingFeedback": true
      },
      {
        "id": 2,
        "name": "김철수",
        "profileImageUrl": "/api/v1/files/profile/2",
        "todayTaskSummary": {
          "total": 4,
          "completed": 4,
          "pendingConfirmation": 1
        },
        "lastFeedbackDate": "2025-01-27",
        "hasPendingFeedback": false
      }
    ]
  }
}
```

---

### 4.2 멘티 할 일 목록 조회 (상세 관리)

**Endpoint**

```
GET /api/v1/mentor/students/{studentId}/tasks
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| date | String | O | 조회 날짜 (YYYY-MM-DD) |

**Response**

```json
{
  "success": true,
  "data": {
    "studentId": 1,
    "studentName": "민유진",
    "date": "2025-01-27",
    "tasks": [
      {
        "id": 1,
        "title": "수능완성 5회 1~11번",
        "subject": "KOREAN",
        "subjectName": "국어",
        "goal": {
          "id": 10,
          "title": "문학 개념어 정리"
        },
        "materials": [
          {
            "id": 101,
            "fileName": "수능완성_국어_5회.pdf",
            "downloadUrl": "/api/v1/files/101/download"
          }
        ],
        "studyTime": 45,
        "isMentorConfirmed": false,
        "submission": {
          "id": 201,
          "imageUrl": "/api/v1/files/201",
          "thumbnailUrl": "/api/v1/files/201/thumbnail",
          "submittedAt": "2025-01-27T14:30:00"
        },
        "feedback": null
      }
    ],
    "comments": [
      {
        "id": 401,
        "content": "오늘 국어 문제 풀다가 막히는 부분이 있었어요",
        "createdAt": "2025-01-27T15:00:00"
      }
    ]
  }
}
```

---

### 4.3 할 일 등록

멘티에게 할 일을 부여합니다.

**Endpoint**

```
POST /api/v1/mentor/students/{studentId}/tasks
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | O | 할 일 제목 |
| date | String | O | 수행 날짜 (YYYY-MM-DD) |
| goalId | Long | X | 목표(약점 맞춤 솔루션) ID |
| materialIds | List<Long> | X | 학습지 파일 ID 목록 |

**Request Body (multipart/form-data - 파일 직접 업로드 시)**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | O | 할 일 제목 |
| date | String | O | 수행 날짜 (YYYY-MM-DD) |
| goalId | Long | X | 목표 ID |
| materials | List<File> | X | 학습지 파일 (PDF 등) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 20,
    "title": "수능완성 6회 1~15번",
    "date": "2025-01-28",
    "subject": "KOREAN",
    "goal": {
      "id": 10,
      "title": "문학 개념어 정리"
    },
    "materials": [
      {
        "id": 105,
        "fileName": "수능완성_국어_6회.pdf"
      }
    ],
    "createdAt": "2025-01-27T16:00:00"
  }
}
```

---

### 4.4 할 일 수정

**Endpoint**

```
PUT /api/v1/mentor/students/{studentId}/tasks/{taskId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |
| taskId | Long | O | 할 일 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | X | 할 일 제목 |
| date | String | X | 수행 날짜 (YYYY-MM-DD) |
| goalId | Long | X | 목표 ID |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 20,
    "title": "수능완성 6회 전범위",
    "date": "2025-01-28",
    "updatedAt": "2025-01-27T16:30:00"
  }
}
```

---

### 4.5 할 일 삭제

**Endpoint**

```
DELETE /api/v1/mentor/students/{studentId}/tasks/{taskId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |
| taskId | Long | O | 할 일 ID |

**Response**

```json
{
  "success": true,
  "data": null,
  "message": "할 일이 삭제되었습니다."
}
```

---

### 4.6 멘토 확인 토글

제출물 확인 체크박스를 토글합니다.

**Endpoint**

```
PATCH /api/v1/mentor/students/{studentId}/tasks/{taskId}/confirm
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |
| taskId | Long | O | 할 일 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| confirmed | Boolean | O | 확인 여부 |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "isMentorConfirmed": true,
    "confirmedAt": "2025-01-27T17:00:00"
  }
}
```

---

### 4.7 피드백 작성

**Endpoint**

```
POST /api/v1/mentor/students/{studentId}/feedbacks
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| taskId | Long | O | 할 일 ID |
| content | String | O | 피드백 내용 |
| summary | String | X | 요약 (중요 표시 시 사용) |
| isImportant | Boolean | X | 중요 표시 여부 (기본값: false) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 310,
    "taskId": 1,
    "content": "오늘 풀이한 문학 작품 해석은 전반적으로 잘 했습니다.",
    "summary": null,
    "isImportant": false,
    "createdAt": "2025-01-27T22:00:00"
  }
}
```

---

### 4.8 총평 작성/수정

해당 날짜 전체에 대한 총평을 작성합니다.

**Endpoint**

```
PUT /api/v1/mentor/students/{studentId}/feedbacks/overall
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| date | String | O | 대상 날짜 (YYYY-MM-DD) |
| content | String | O | 총평 내용 |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 50,
    "date": "2025-01-27",
    "content": "전체적으로 집중력이 좋았습니다. 국어 문학 부분 조금 더 신경 써주세요.",
    "updatedAt": "2025-01-27T22:30:00"
  }
}
```

---

### 4.9 피드백 수정

**Endpoint**

```
PUT /api/v1/mentor/feedbacks/{feedbackId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| feedbackId | Long | O | 피드백 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| content | String | X | 피드백 내용 |
| summary | String | X | 요약 |
| isImportant | Boolean | X | 중요 표시 여부 |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 310,
    "content": "수정된 피드백 내용",
    "summary": "핵심 요약",
    "isImportant": true,
    "updatedAt": "2025-01-27T23:00:00"
  }
}
```

---

### 4.10 약점 맞춤 솔루션 목록 조회

**Endpoint**

```
GET /api/v1/mentor/students/{studentId}/solutions
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| subject | String | X | 과목 필터 |

**Response**

```json
{
  "success": true,
  "data": {
    "studentId": 1,
    "studentName": "민유진",
    "solutions": [
      {
        "id": 10,
        "title": "문학 개념어 정리",
        "subject": "KOREAN",
        "subjectName": "국어",
        "materials": [
          {
            "id": 501,
            "fileName": "문학개념어_총정리.pdf",
            "downloadUrl": "/api/v1/files/501/download"
          }
        ],
        "linkedTaskCount": 5,
        "createdAt": "2024-12-01"
      }
    ]
  }
}
```

---

### 4.11 약점 맞춤 솔루션 등록

**Endpoint**

```
POST /api/v1/mentor/students/{studentId}/solutions
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Request Body (multipart/form-data)**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | O | 보완점 제목 |
| subject | String | O | 과목 (KOREAN/ENGLISH/MATH) |
| materials | List<File> | X | 추가자료 파일 |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 15,
    "title": "수열 점화식 유형",
    "subject": "MATH",
    "subjectName": "수학",
    "materials": [
      {
        "id": 510,
        "fileName": "점화식_유형정리.pdf"
      }
    ],
    "createdAt": "2025-01-27T16:00:00"
  }
}
```

---

### 4.12 약점 맞춤 솔루션 수정

**Endpoint**

```
PUT /api/v1/mentor/students/{studentId}/solutions/{solutionId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |
| solutionId | Long | O | 솔루션 ID |

**Request Body (multipart/form-data)**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| title | String | X | 보완점 제목 |
| subject | String | X | 과목 |
| materials | List<File> | X | 추가할 파일 |
| deleteFileIds | List<Long> | X | 삭제할 파일 ID 목록 |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 15,
    "title": "수열 점화식 유형 (수정)",
    "subject": "MATH",
    "updatedAt": "2025-01-27T17:00:00"
  }
}
```

---

### 4.13 약점 맞춤 솔루션 삭제

**Endpoint**

```
DELETE /api/v1/mentor/students/{studentId}/solutions/{solutionId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |
| solutionId | Long | O | 솔루션 ID |

**Response**

```json
{
  "success": true,
  "data": null,
  "message": "솔루션이 삭제되었습니다."
}
```

> ⚠️ 연결된 할 일이 있는 경우 삭제 불가 (400 에러)

---

### 4.14 주간 학습리포트 작성

**Endpoint**

```
POST /api/v1/mentor/students/{studentId}/weekly-reports
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| studentId | Long | O | 멘티 ID |

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| startDate | String | O | 주차 시작일 (YYYY-MM-DD) |
| endDate | String | O | 주차 종료일 (YYYY-MM-DD) |
| title | String | O | 리포트 제목 |
| summary | String | X | 요약 |
| overallFeedback | String | O | 전체 피드백 |
| subjectFeedbacks | List | O | 과목별 피드백 |

**subjectFeedbacks 항목**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| subject | String | O | 과목 코드 |
| feedback | String | O | 과목별 피드백 |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 5,
    "title": "5주차 (25.12.28~26.01.03)",
    "startDate": "2024-12-28",
    "endDate": "2025-01-03",
    "createdAt": "2025-01-04T10:00:00"
  }
}
```

---

## 5. 파일 API

### 5.1 파일 업로드

**Endpoint**

```
POST /api/v1/files/upload
```

**Request Body (multipart/form-data)**
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| file | File | O | 업로드할 파일 |
| type | String | O | 파일 유형 (MATERIAL / SUBMISSION / PROFILE) |

**Response**

```json
{
  "success": true,
  "data": {
    "id": 701,
    "fileName": "학습자료.pdf",
    "fileType": "PDF",
    "fileSize": 1048576,
    "url": "/api/v1/files/701"
  }
}
```

---

### 5.2 파일 조회 (다운로드)

**Endpoint**

```
GET /api/v1/files/{fileId}/download
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| fileId | Long | O | 파일 ID |

**Response**

- Content-Type: application/octet-stream (또는 해당 파일 MIME 타입)
- Content-Disposition: attachment; filename="파일명"

---

### 5.3 파일 미리보기 (이미지)

**Endpoint**

```
GET /api/v1/files/{fileId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| fileId | Long | O | 파일 ID |

**Response**

- Content-Type: image/jpeg 또는 image/png

---

### 5.4 썸네일 조회

**Endpoint**

```
GET /api/v1/files/{fileId}/thumbnail
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| fileId | Long | O | 파일 ID |

**Query Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| width | Integer | X | 썸네일 너비 (기본값: 200) |
| height | Integer | X | 썸네일 높이 (기본값: 200) |

**Response**

- Content-Type: image/jpeg

---

### 5.5 파일 삭제

**Endpoint**

```
DELETE /api/v1/files/{fileId}
```

**Path Parameters**
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| fileId | Long | O | 파일 ID |

**Response**

```json
{
  "success": true,
  "data": null,
  "message": "파일이 삭제되었습니다."
}
```

---
