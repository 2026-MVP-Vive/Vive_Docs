# 설스터디 MVP Q&A 검토 보고서

> **작성일**: 2026-02-05
> **참고 자료**: 설스터디_MVP_기획_Q&A.pdf, MVP_PLAN.md, API_SPEC.md

---

## 1. Q&A vs MVP_PLAN.md 비교 분석

### 1.1 수정이 필요한 부분

| 항목 | MVP_PLAN.md (현재) | Q&A 답변 (윤서영) | 조치 필요 |
|------|-------------------|------------------|----------|
| **리포트 주기** | "주간 학습리포트" | "월 단위 리포트를 학생 및 학부모님께 발송" | 주간 → **월간**으로 수정 |
| **주 단위 미니 캘린더** | "7일 가로 스크롤, 선택된 날짜 하이라이트" (네비게이션 용도) | "해당 주차의 모든 할 일 목록을 요약해서 보여주는 뷰(View)" | 기능 설명 **업데이트** |
| **상담받아보기 버튼** | 마이페이지에 "상담받아보기" 플로팅 버튼 존재 | "그 부분은 잘못 작성된 것 같습니다" | **삭제 또는 재검토** |
| **마이페이지 중요도** | 프로필, 달성률, 주간리포트 등 상세 기능 명시 | "중요한 페이지가 아님, 간단한 정보만 표시하면 충분" | **간소화 검토** |

---

### 1.2 추가해야 할 내용

| 항목 | 상세 내용 | 출처 |
|------|----------|------|
| **플래너 마감/피드백 요청 버튼** | 학생이 모든 할 일을 마치고 직접 [플래너 마감/피드백 요청] 버튼을 눌러야 멘토에게 넘어가는 **수동 완료 방식** | 후드라이언 질문 → 윤서영 답변 |
| **과제 업로드 필수/선택 정책** | - 멘토 생성 과제: 업로드 **필수**<br>- 멘티 생성 과제: 업로드 **선택** | 최은지 질문 → 윤서영 답변 |
| **미래 할 일 앞당겨 완료 불가** | 그날그날 루틴이므로 해당 날짜가 아니면 미리 완료 처리 불가능 | 손정민 질문 → 윤서영 답변 |
| **멘토-멘티 매칭 구조** | - 1명의 멘토 → 여러 멘티 담당<br>- 멘티는 1명의 멘토와만 매칭<br>- 한 멘토가 국/영/수 전 과목 피드백 | 최은지 질문 → 윤서영 답변 |
| **학습지/과제 업로드 UI** | 과제 상세 화면에서 학습지 영역과 업로드 영역을 **탭 형태로 구분** 가능 | 최상협 질문 → 윤서영 답변 |

---

### 1.3 명확해진 내용 (기존 기획과 일치 확인)

| 항목 | 확인된 내용 | 비고 |
|------|----------|------|
| 로그인 시스템 | 멘토-PC / 멘티-모바일 앱 분리 구현 OK | 김윤주 질문 → 윤서영 답변 |
| 코멘트 입력란 | 오늘의 각오, 멘토 질문, 회고 등 **다목적 사용** | 후드라이언 질문 → 윤서영 답변 |
| 목표 설정 | **자유롭게 텍스트로 입력**하는 구조 | 후드라이언 질문 → 윤서영 답변 |
| 피드백 시점 | 전날 과제 → 다음날 오전 11시까지 (권장, 필수 아님) | 최은지 질문 → 윤서영 답변 |
| 핵심 기능 | **'일일 플래너'와 '피드백' 기능만 있으면 충분**, 나머지는 자유 구성 | 후드라이언 질문 → 윤서영 답변 |
| 학습지 형식 | PDF 형식으로 첨부 | 이승민 질문 → 윤서영 답변 |
| 유저플로우 | **엑셀 문서를 중점적으로** 따를 것 | 승호 질문 → 윤서영 답변 |
| 시간 기록 방식 | 완료 후 몇시~몇시 직접 표시 또는 실시간 기록 버튼 둘 다 OK | 박지윤 질문 → 윤서영 답변 |
| 피드백 작성 규칙 | 특별한 규칙 없음, 확인하는 대로 작성, 순서 무관 | 김한빈 질문 → 윤서영 답변 |

---

### 1.4 MVP 범위에서 제외 확인

| 항목 | 비고 |
|------|------|
| OT 및 예비문항 시스템화 | 해커톤 범위에서 제외 |
| 학생 수준 파악 기능 (RAG) | 개발 중, 해커톤에서 고려 불필요 |
| 회원가입 페이지 | 테스트 계정 3개 사전 세팅으로 대체 |
| 멘토-멘티 매칭 기능 | 이미 연결된 테스트 계정 사용 |

---

### 1.5 Nice to Have 추가 고려

| 항목 | 상세 | 근거 |
|------|------|------|
| **할 일 반복 배치** | 요일(월~금, 화/목 등) 설정으로 자동 배치 기능 | 윤서영: "현재 노션에서 가장 불편한 점" |

---

## 2. API 명세서 업데이트 필요 사항

### 2.1 엔드포인트 수정

| 현재 | 변경 | 이유 |
|------|------|------|
| `GET /api/v1/mentee/weekly-reports` | `GET /api/v1/mentee/monthly-reports` | Q&A: "월 단위 리포트" |
| `GET /api/v1/mentee/weekly-reports/{reportId}` | `GET /api/v1/mentee/monthly-reports/{reportId}` | 동일 |
| `POST /api/v1/mentor/students/{studentId}/weekly-reports` | `POST /api/v1/mentor/students/{studentId}/monthly-reports` | 동일 |

---

### 2.2 엔드포인트 추가

| Method | Endpoint | 설명 | 근거 |
|--------|----------|------|------|
| **POST** | `/api/v1/mentee/planner/{date}/complete` | 플래너 마감/피드백 요청 | Q&A: 수동 완료 방식 채택 |
| **GET** | `/api/v1/mentee/tasks/weekly-summary` | 주간 할 일 요약 조회 | Q&A: 주 단위 캘린더가 할 일 목록 요약 뷰 |
| **PATCH** | `/api/v1/mentee/tasks/{taskId}/comment` | 할 일별 코멘트/질문 수정 | 기존 POST만 존재 |

---

### 2.3 응답 필드 추가 검토

#### `GET /api/v1/mentee/tasks/{taskId}` 응답

```json
{
  "taskId": "string",
  "title": "string",
  "subject": "국어|영어|수학",
  "date": "2026-02-05",
  "isCreatedByMentor": true,       // [추가] 멘토 생성 여부
  "isUploadRequired": true,        // [추가] 업로드 필수 여부
  "isCompleted": false,            // [추가] 완료 여부
  "canComplete": true,             // [추가] 완료 가능 여부 (해당 날짜만 가능)
  "studyTime": {
    "startTime": "14:00",
    "endTime": "16:00"
  },
  "submission": { ... },
  "feedback": { ... }
}
```

#### `GET /api/v1/mentee/tasks` 쿼리 파라미터 추가

```
?date=2026-02-05           // 특정 날짜 조회
?weekOf=2026-02-03         // 주간 조회 (해당 주 월요일 기준)
?includeCompleted=true     // 완료된 항목 포함 여부
```

---

### 2.4 API 우선순위 재분류

#### Must Have (핵심 - 일일 플래너 + 피드백)

**멘티 API:**
| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/v1/mentee/tasks` | 할 일 목록 조회 |
| GET | `/api/v1/mentee/tasks/{taskId}` | 할 일 상세 조회 |
| POST | `/api/v1/mentee/tasks` | 할 일 추가 (멘티) |
| PATCH | `/api/v1/mentee/tasks/{taskId}/study-time` | 공부 시간 기록 |
| POST | `/api/v1/mentee/tasks/{taskId}/submission` | 과제 제출 |
| **POST** | `/api/v1/mentee/planner/{date}/complete` | **플래너 마감 (신규)** |
| GET | `/api/v1/mentee/feedbacks` | 피드백 조회 |
| GET | `/api/v1/mentee/feedbacks/yesterday` | 어제자 피드백 |
| POST | `/api/v1/mentee/comments` | 코멘트/질문 등록 |

**멘토 API:**
| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/v1/mentor/students` | 담당 멘티 목록 |
| GET | `/api/v1/mentor/students/{studentId}/tasks` | 멘티 할 일 목록 |
| POST | `/api/v1/mentor/students/{studentId}/tasks` | 할 일 등록 |
| PUT | `/api/v1/mentor/students/{studentId}/tasks/{taskId}` | 할 일 수정 |
| DELETE | `/api/v1/mentor/students/{studentId}/tasks/{taskId}` | 할 일 삭제 |
| PATCH | `/api/v1/mentor/students/{studentId}/tasks/{taskId}/confirm` | 멘토 확인 |
| POST | `/api/v1/mentor/students/{studentId}/feedbacks` | 피드백 작성 |
| PUT | `/api/v1/mentor/students/{studentId}/feedbacks/overall` | 총평 작성 |
| PUT | `/api/v1/mentor/feedbacks/{feedbackId}` | 피드백 수정 |

**공통 API:**
| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/v1/auth/login` | 로그인 |
| POST | `/api/v1/auth/logout` | 로그아웃 |
| POST | `/api/v1/files/upload` | 파일 업로드 |
| GET | `/api/v1/files/{fileId}/download` | 파일 다운로드 |

#### Nice to Have (마이페이지, 학습자료 등)

| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/v1/mentee/profile` | 프로필 조회 |
| GET | `/api/v1/mentee/achievement` | 과목별 달성률 |
| GET | `/api/v1/mentee/monthly-reports` | 월간 리포트 목록 |
| GET | `/api/v1/mentee/monthly-reports/{reportId}` | 월간 리포트 상세 |
| GET | `/api/v1/mentee/solutions` | 약점 맞춤 솔루션 |
| GET | `/api/v1/mentee/monthly-plan` | 월간 계획표 |
| GET | `/api/v1/mentee/columns` | 칼럼 목록 |
| GET | `/api/v1/mentee/columns/{columnId}` | 칼럼 상세 |
| GET | `/api/v1/mentor/students/{studentId}/solutions` | 솔루션 목록 |
| POST | `/api/v1/mentor/students/{studentId}/solutions` | 솔루션 등록 |
| PUT | `/api/v1/mentor/students/{studentId}/solutions/{solutionId}` | 솔루션 수정 |
| DELETE | `/api/v1/mentor/students/{studentId}/solutions/{solutionId}` | 솔루션 삭제 |
| POST | `/api/v1/mentor/students/{studentId}/monthly-reports` | 월간 리포트 작성 |

---

## 3. 요약

### 핵심 변경 사항
1. **주간 리포트 → 월간 리포트** (전체 적용)
2. **플래너 마감/피드백 요청 기능 추가** (수동 완료 방식)
3. **주간 캘린더 = 할 일 요약 뷰** (단순 네비게이션 아님)
4. **마이페이지 간소화** (핵심 기능 아님)
5. **과제 업로드 정책 구분** (멘토 생성=필수, 멘티 생성=선택)

### 핵심 기능 (윤서영 대표 확인)
> **"일일 플래너와 피드백 기능만 있으면 충분하고, 그 외 부분들은 자유롭게 구성해주셔도 됩니다"**
