## 부록: API 엔드포인트 요약

### 인증 (Auth)
| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | /api/v1/auth/login | 로그인 |
| POST | /api/v1/auth/refresh | 토큰 갱신 |
| POST | /api/v1/auth/logout | 로그아웃 |

### 멘티 (Mentee)
| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | /api/v1/mentee/tasks | 할 일 목록 조회 |
| GET | /api/v1/mentee/tasks/{taskId} | 할 일 상세 조회 |
| POST | /api/v1/mentee/tasks | 할 일 추가 (멘티) |
| PATCH | /api/v1/mentee/tasks/{taskId}/study-time | 공부 시간 기록 |
| POST | /api/v1/mentee/tasks/{taskId}/submission | 과제 제출 |
| GET | /api/v1/mentee/feedbacks/yesterday | 어제자 피드백 |
| GET | /api/v1/mentee/feedbacks | 날짜별 피드백 |
| GET | /api/v1/mentee/solutions | 약점 맞춤 솔루션 |
| GET | /api/v1/mentee/monthly-plan | 월간 계획표 |
| GET | /api/v1/mentee/columns | 칼럼 목록 |
| GET | /api/v1/mentee/columns/{columnId} | 칼럼 상세 |
| GET | /api/v1/mentee/profile | 프로필 조회 |
| GET | /api/v1/mentee/achievement | 과목별 달성률 |
| GET | /api/v1/mentee/weekly-reports | 주간 리포트 목록 |
| GET | /api/v1/mentee/weekly-reports/{reportId} | 주간 리포트 상세 |
| POST | /api/v1/mentee/comments | 코멘트/질문 등록 |

### 멘토 (Mentor)
| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | /api/v1/mentor/students | 담당 멘티 목록 |
| GET | /api/v1/mentor/students/{studentId}/tasks | 멘티 할 일 목록 |
| POST | /api/v1/mentor/students/{studentId}/tasks | 할 일 등록 |
| PUT | /api/v1/mentor/students/{studentId}/tasks/{taskId} | 할 일 수정 |
| DELETE | /api/v1/mentor/students/{studentId}/tasks/{taskId} | 할 일 삭제 |
| PATCH | /api/v1/mentor/students/{studentId}/tasks/{taskId}/confirm | 멘토 확인 |
| POST | /api/v1/mentor/students/{studentId}/feedbacks | 피드백 작성 |
| PUT | /api/v1/mentor/students/{studentId}/feedbacks/overall | 총평 작성 |
| PUT | /api/v1/mentor/feedbacks/{feedbackId} | 피드백 수정 |
| GET | /api/v1/mentor/students/{studentId}/solutions | 솔루션 목록 |
| POST | /api/v1/mentor/students/{studentId}/solutions | 솔루션 등록 |
| PUT | /api/v1/mentor/students/{studentId}/solutions/{solutionId} | 솔루션 수정 |
| DELETE | /api/v1/mentor/students/{studentId}/solutions/{solutionId} | 솔루션 삭제 |
| POST | /api/v1/mentor/students/{studentId}/weekly-reports | 주간 리포트 작성 |

### 파일 (File)
| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | /api/v1/files/upload | 파일 업로드 |
| GET | /api/v1/files/{fileId}/download | 파일 다운로드 |
| GET | /api/v1/files/{fileId} | 파일 미리보기 |
| GET | /api/v1/files/{fileId}/thumbnail | 썸네일 조회 |
| DELETE | /api/v1/files/{fileId} | 파일 삭제 |

---
