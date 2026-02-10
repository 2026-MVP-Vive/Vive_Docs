# 설스터디 MVP 데이터베이스 스키마

> DBMS: MySQL 8.4 (RDS)
> ORM: Spring Data JPA (Hibernate)
> 문자셋: utf8mb4

---

## 목차

1. [ERD 다이어그램](#1-erd-다이어그램)
2. [테이블 정의](#2-테이블-정의)
3. [인덱스 설계](#3-인덱스-설계)
4. [초기 데이터](#4-초기-데이터)

---

## 1. ERD 다이어그램

```
┌─────────────────────┐     ┌─────────────────┐
│        users        │     │   mentor_mentee │
├─────────────────────┤     ├─────────────────┤
│ id (PK)             │◄────│ mentor_id (FK)  │
│ login_id            │     │ mentee_id (FK)  │
│ password            │◄────│ created_at      │
│ name                │     └─────────────────┘
│ role                │
│ profile_image_id(FK)│──────┐
│ fcm_token           │      │
│ created_at          │      │
└──┬──────────────────┘      │
   │                         │
   │    ┌────────────────────▼────┐
   │    │         files           │
   │    ├─────────────────────────┤
   │    │ id (PK)                 │
   │    │ original_name           │
   │    │ stored_name             │
   │    │ file_path               │
   │    │ file_type / file_size   │
   │    │ category                │
   │    │ uploader_id (FK)        │
   │    └─────────────────────────┘
   │                   ▲
   ▼                   │
┌─────────────────┐    │    ┌──────────────────────┐
│    solutions    │    │    │  solution_materials  │
├─────────────────┤    │    ├──────────────────────┤
│ id (PK)         │◄───┼────│ solution_id (FK)     │
│ mentee_id (FK)  │    │    │ file_id (FK)         │
│ title / subject │    │    └──────────────────────┘
└────────┬────────┘    │
         │             │
         ▼             │
┌──────────────────────┐    ┌─────────────────────┐
│       tasks          │    │   task_materials    │
├──────────────────────┤    ├─────────────────────┤
│ id (PK)              │    │ task_id (FK)        │
│ mentee_id (FK)       ├────│ file_id (FK)        │
│ solution_id (FK)     │    └─────────────────────┘
│ title / task_date    │
│ subject / study_time │
│ is_upload_required   │
│ is_mentor_assigned   │
│ is_mentor_confirmed  │
│ is_mentee_completed  │
│ mentee_completed_at  │
│ confirmed_at         │
│ created_by (FK)      │
└──┬───────────────────┘
   │
   ▼
┌─────────────────┐    ┌─────────────────┐
│   submissions   │    │    feedbacks    │
├─────────────────┤    ├─────────────────┤
│ task_id (FK)    │    │ task_id (FK)    │
│ file_id (FK)    │    │ mentor_id (FK)  │
│ submitted_at    │    │ content/summary │
└─────────────────┘    │ is_important    │
                       └─────────────────┘

┌─────────────────────┐   ┌─────────────────────┐
│  overall_feedbacks  │   │   planner_completions│
├─────────────────────┤   ├─────────────────────┤
│ mentee_id (FK)      │   │ mentee_id (FK)      │
│ mentor_id (FK)      │   │ plan_date           │
│ feedback_date       │   │ completed_at        │
│ content             │   └─────────────────────┘
└─────────────────────┘

┌─────────────────┐       ┌─────────────────────┐
│     columns     │       │ column_attachments  │
├─────────────────┤       ├─────────────────────┤
│ id (PK)         │◄──────│ column_id (FK)      │
│ title / content │       │ file_id (FK)        │
│ summary / author│       └─────────────────────┘
│ thumbnail_id(FK)│
└─────────────────┘

┌─────────────────────┐   ┌─────────────────────────┐
│   weekly_reports    │   │ weekly_report_subjects  │
├─────────────────────┤   ├─────────────────────────┤
│ id (PK)             │◄──│ report_id (FK)          │
│ mentee_id / mentor_id   │ subject                 │
│ week_number / title │   │ completion_rate         │
│ start_date / end_date   │ total_study_time        │
│ summary             │   │ feedback                │
│ overall_feedback    │   └─────────────────────────┘
└─────────────────────┘

┌──────────────────────┐  ┌──────────────────────────┐
│   monthly_reports    │  │ monthly_report_subjects  │
├──────────────────────┤  ├──────────────────────────┤
│ id (PK)              │◄─│ monthly_report_id (FK)   │
│ mentee_id / mentor_id│  │ subject                  │
│ report_year / month  │  │ completion_rate           │
│ title / summary      │  │ total_study_time          │
│ overall_feedback     │  │ feedback                 │
└──────────────────────┘  └──────────────────────────┘

┌─────────────────┐  ┌──────────────────┐  ┌─────────────────┐
│    comments     │  │  notifications   │  │  zoom_meetings  │
├─────────────────┤  ├──────────────────┤  ├─────────────────┤
│ mentee_id (FK)  │  │ user_id          │  │ mentee_id (FK)  │
│ comment_date    │  │ type / title     │  │ preferred_date  │
│ content         │  │ message          │  │ preferred_time  │
└─────────────────┘  │ is_read / is_sent│  │ status          │
                     └──────────────────┘  └─────────────────┘

┌─────────────────┐  ┌──────────────────┐
│   fcm_tokens    │  │  refresh_tokens  │
├─────────────────┤  ├──────────────────┤
│ user_id         │  │ user_id (FK)     │
│ token (UNIQUE)  │  │ token (UNIQUE)   │
│ last_used_at    │  │ expires_at       │
└─────────────────┘  └──────────────────┘
```

---

## 2. 테이블 정의

### 2.1 users (사용자)

멘토와 멘티 정보를 통합 관리합니다.

| 컬럼명           | 타입                    | NULL | 기본값            | 설명                   |
| ---------------- | ----------------------- | ---- | ----------------- | ---------------------- |
| id               | BIGINT                  | NO   | AUTO_INCREMENT    | PK                     |
| login_id         | VARCHAR(50)             | NO   | -                 | 로그인 아이디 (UNIQUE) |
| password         | VARCHAR(255)            | NO   | -                 | 암호화된 비밀번호      |
| name             | VARCHAR(50)             | NO   | -                 | 이름                   |
| role             | ENUM('MENTOR','MENTEE') | NO   | -                 | 역할                   |
| profile_image_id | BIGINT                  | YES  | NULL              | 프로필 이미지 FK       |
| created_at       | DATETIME                | NO   | CURRENT_TIMESTAMP | 생성일시               |
| updated_at       | DATETIME                | NO   | CURRENT_TIMESTAMP | 수정일시               |
| fcm_token        | VARCHAR(255)            | YES  | NULL              | FCM 토큰               |

```sql
CREATE TABLE users (
    id BIGINT NOT NULL AUTO_INCREMENT,
    login_id VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(50) NOT NULL,
    role ENUM('MENTOR', 'MENTEE') NOT NULL,
    profile_image_id BIGINT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    fcm_token VARCHAR(255) NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uk_users_login_id (login_id),
    KEY idx_users_role (role),
    CONSTRAINT fk_users_profile_image FOREIGN KEY (profile_image_id) REFERENCES files(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.2 mentor_mentee (멘토-멘티 관계)

멘토와 멘티의 담당 관계를 관리합니다. (1:N 관계)

| 컬럼명     | 타입     | NULL | 기본값            | 설명        |
| ---------- | -------- | ---- | ----------------- | ----------- |
| id         | BIGINT   | NO   | AUTO_INCREMENT    | PK          |
| mentor_id  | BIGINT   | NO   | -                 | 멘토 FK     |
| mentee_id  | BIGINT   | NO   | -                 | 멘티 FK     |
| created_at | DATETIME | NO   | CURRENT_TIMESTAMP | 관계 생성일 |

```sql
CREATE TABLE mentor_mentee (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentor_id BIGINT NOT NULL,
    mentee_id BIGINT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_mentor_mentee (mentor_id, mentee_id),
    KEY idx_mentor_mentee_mentor (mentor_id),
    KEY idx_mentor_mentee_mentee (mentee_id),
    CONSTRAINT fk_mentor_mentee_mentor FOREIGN KEY (mentor_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT fk_mentor_mentee_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.3 files (파일)

모든 업로드 파일을 통합 관리합니다.

| 컬럼명        | 타입                                                     | NULL | 기본값            | 설명                         |
| ------------- | -------------------------------------------------------- | ---- | ----------------- | ---------------------------- |
| id            | BIGINT                                                   | NO   | AUTO_INCREMENT    | PK                           |
| original_name | VARCHAR(255)                                             | NO   | -                 | 원본 파일명                  |
| stored_name   | VARCHAR(255)                                             | NO   | -                 | 저장된 파일명 (UUID)         |
| file_path     | VARCHAR(500)                                             | NO   | -                 | 저장 경로                    |
| file_type     | VARCHAR(50)                                              | NO   | -                 | 파일 타입 (PDF, JPG, PNG 등) |
| file_size     | BIGINT                                                   | NO   | -                 | 파일 크기 (bytes)            |
| category      | ENUM('MATERIAL','SUBMISSION','PROFILE','COLUMN','OTHER') | NO   | 'OTHER'           | 파일 분류                    |
| uploader_id   | BIGINT                                                   | YES  | NULL              | 업로더 FK                    |
| created_at    | DATETIME                                                 | NO   | CURRENT_TIMESTAMP | 업로드일시                   |

```sql
CREATE TABLE files (
    id BIGINT NOT NULL AUTO_INCREMENT,
    original_name VARCHAR(255) NOT NULL,
    stored_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_type VARCHAR(50) NOT NULL,
    file_size BIGINT NOT NULL,
    category ENUM('MATERIAL', 'SUBMISSION', 'PROFILE', 'COLUMN', 'OTHER') NOT NULL DEFAULT 'OTHER',
    uploader_id BIGINT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_files_category (category),
    KEY idx_files_uploader (uploader_id),
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.4 solutions (약점 맞춤 솔루션)

노션의 "약점 맞춤 솔루션" DB에 해당합니다.

| 컬럼명     | 타입                            | NULL | 기본값            | 설명        |
| ---------- | ------------------------------- | ---- | ----------------- | ----------- |
| id         | BIGINT                          | NO   | AUTO_INCREMENT    | PK          |
| mentee_id  | BIGINT                          | NO   | -                 | 멘티 FK     |
| title      | VARCHAR(200)                    | NO   | -                 | 보완점 제목 |
| subject    | ENUM('KOREAN','ENGLISH','MATH') | NO   | -                 | 과목        |
| created_at | DATETIME                        | NO   | CURRENT_TIMESTAMP | 생성일시    |
| updated_at | DATETIME                        | NO   | CURRENT_TIMESTAMP | 수정일시    |

```sql
CREATE TABLE solutions (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL,
    title VARCHAR(200) NOT NULL,
    subject ENUM('KOREAN', 'ENGLISH', 'MATH') NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_solutions_mentee (mentee_id),
    KEY idx_solutions_subject (subject),
    KEY idx_solutions_mentee_subject (mentee_id, subject),
    CONSTRAINT fk_solutions_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.5 solution_materials (솔루션-파일 연결)

솔루션에 첨부된 학습자료 파일을 연결합니다.

| 컬럼명      | 타입   | NULL | 기본값         | 설명      |
| ----------- | ------ | ---- | -------------- | --------- |
| id          | BIGINT | NO   | AUTO_INCREMENT | PK        |
| solution_id | BIGINT | NO   | -              | 솔루션 FK |
| file_id     | BIGINT | NO   | -              | 파일 FK   |

```sql
CREATE TABLE solution_materials (
    id BIGINT NOT NULL AUTO_INCREMENT,
    solution_id BIGINT NOT NULL,
    file_id BIGINT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uk_solution_materials (solution_id, file_id),
    KEY idx_solution_materials_solution (solution_id),
    CONSTRAINT fk_solution_materials_solution FOREIGN KEY (solution_id) REFERENCES solutions(id) ON DELETE CASCADE,
    CONSTRAINT fk_solution_materials_file FOREIGN KEY (file_id) REFERENCES files(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.6 tasks (할 일)

노션의 "오늘 할 일" DB에 해당합니다.

| 컬럼명              | 타입                            | NULL | 기본값            | 설명                                  |
| ------------------- | ------------------------------- | ---- | ----------------- | ------------------------------------- |
| id                  | BIGINT                          | NO   | AUTO_INCREMENT    | PK                                    |
| mentee_id           | BIGINT                          | NO   | -                 | 멘티 FK                               |
| solution_id         | BIGINT                          | YES  | NULL              | 목표(솔루션) FK (relation)            |
| title               | VARCHAR(200)                    | NO   | -                 | 할 일 제목                            |
| task_date           | DATE                            | NO   | -                 | 수행 날짜                             |
| subject             | ENUM('KOREAN','ENGLISH','MATH') | YES  | NULL              | 과목 (솔루션에서 롤업 또는 직접 지정) |
| study_time          | INT                             | YES  | NULL              | 공부 시간 (분)                        |
| is_upload_required  | TINYINT(1)                      | NO   | 0                 | 인증 사진 업로드 필수 여부            |
| is_mentor_assigned  | TINYINT(1)                      | NO   | 0                 | 멘토 지정 여부                        |
| is_mentor_confirmed | TINYINT(1)                      | NO   | 0                 | 멘토 확인 여부                        |
| is_mentee_completed | TINYINT(1)                      | NO   | 0                 | 멘티 완료 체크 여부                   |
| mentee_completed_at | DATETIME                        | YES  | NULL              | 멘티 완료 체크 시간                   |
| confirmed_at        | DATETIME                        | YES  | NULL              | 멘토 확인 일시                        |
| created_by          | BIGINT                          | NO   | -                 | 생성자 FK (멘토 or 멘티)              |
| created_at          | DATETIME                        | NO   | CURRENT_TIMESTAMP | 생성일시                              |
| updated_at          | DATETIME                        | NO   | CURRENT_TIMESTAMP | 수정일시                              |

```sql
CREATE TABLE tasks (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL,
    solution_id BIGINT NULL,
    title VARCHAR(200) NOT NULL,
    task_date DATE NOT NULL,
    subject ENUM('KOREAN', 'ENGLISH', 'MATH') NULL,
    study_time INT NULL,
    is_upload_required TINYINT(1) NOT NULL DEFAULT 0 COMMENT '인증 사진 업로드 필수 여부',
    is_mentor_assigned TINYINT(1) NOT NULL DEFAULT 0,
    is_mentor_confirmed TINYINT(1) NOT NULL DEFAULT 0,
    is_mentee_completed TINYINT(1) NOT NULL DEFAULT 0 COMMENT '멘티가 완료 체크 여부',
    mentee_completed_at DATETIME NULL COMMENT '멘티 완료 체크 시간',
    confirmed_at DATETIME NULL,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_tasks_mentee (mentee_id),
    KEY idx_tasks_date (task_date),
    KEY idx_tasks_mentee_date (mentee_id, task_date),
    KEY idx_tasks_solution (solution_id),
    KEY idx_tasks_mentee_date_confirmed (mentee_id, task_date, is_mentor_confirmed),
    CONSTRAINT fk_tasks_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT fk_tasks_solution FOREIGN KEY (solution_id) REFERENCES solutions(id) ON DELETE SET NULL,
    CONSTRAINT fk_tasks_created_by FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.7 task_materials (할일-파일 연결)

할 일에 첨부된 학습지 파일을 연결합니다.

| 컬럼명  | 타입   | NULL | 기본값         | 설명     |
| ------- | ------ | ---- | -------------- | -------- |
| id      | BIGINT | NO   | AUTO_INCREMENT | PK       |
| task_id | BIGINT | NO   | -              | 할 일 FK |
| file_id | BIGINT | NO   | -              | 파일 FK  |

```sql
CREATE TABLE task_materials (
    id BIGINT NOT NULL AUTO_INCREMENT,
    task_id BIGINT NOT NULL,
    file_id BIGINT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uk_task_materials (task_id, file_id),
    KEY idx_task_materials_task (task_id),
    CONSTRAINT fk_task_materials_task FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE CASCADE,
    CONSTRAINT fk_task_materials_file FOREIGN KEY (file_id) REFERENCES files(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.8 submissions (과제 제출)

멘티가 업로드한 인증 사진을 관리합니다.

| 컬럼명       | 타입     | NULL | 기본값            | 설명         |
| ------------ | -------- | ---- | ----------------- | ------------ |
| id           | BIGINT   | NO   | AUTO_INCREMENT    | PK           |
| task_id      | BIGINT   | NO   | -                 | 할 일 FK     |
| file_id      | BIGINT   | NO   | -                 | 인증 사진 FK |
| submitted_at | DATETIME | NO   | CURRENT_TIMESTAMP | 제출일시     |

```sql
CREATE TABLE submissions (
    id BIGINT NOT NULL AUTO_INCREMENT,
    task_id BIGINT NOT NULL,
    file_id BIGINT NOT NULL,
    submitted_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_submissions_task (task_id),
    CONSTRAINT fk_submissions_task FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE CASCADE,
    CONSTRAINT fk_submissions_file FOREIGN KEY (file_id) REFERENCES files(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.9 feedbacks (피드백)

멘토가 작성하는 할 일별 피드백입니다.

| 컬럼명       | 타입         | NULL | 기본값            | 설명                |
| ------------ | ------------ | ---- | ----------------- | ------------------- |
| id           | BIGINT       | NO   | AUTO_INCREMENT    | PK                  |
| task_id      | BIGINT       | NO   | -                 | 할 일 FK            |
| mentor_id    | BIGINT       | NO   | -                 | 멘토 FK             |
| content      | TEXT         | NO   | -                 | 피드백 내용         |
| summary      | VARCHAR(500) | YES  | NULL              | 요약 (중요 표시 시) |
| is_important | TINYINT(1)   | NO   | 0                 | 중요 표시 여부      |
| created_at   | DATETIME     | NO   | CURRENT_TIMESTAMP | 생성일시            |
| updated_at   | DATETIME     | NO   | CURRENT_TIMESTAMP | 수정일시            |

```sql
CREATE TABLE feedbacks (
    id BIGINT NOT NULL AUTO_INCREMENT,
    task_id BIGINT NOT NULL,
    mentor_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    summary VARCHAR(500) NULL,
    is_important TINYINT(1) NOT NULL DEFAULT 0,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_feedbacks_task (task_id),
    KEY idx_feedbacks_mentor (mentor_id),
    CONSTRAINT fk_feedbacks_task FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE CASCADE,
    CONSTRAINT fk_feedbacks_mentor FOREIGN KEY (mentor_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.10 overall_feedbacks (총평)

멘토가 작성하는 일자별 총평입니다.

| 컬럼명        | 타입     | NULL | 기본값            | 설명             |
| ------------- | -------- | ---- | ----------------- | ---------------- |
| id            | BIGINT   | NO   | AUTO_INCREMENT    | PK               |
| mentee_id     | BIGINT   | NO   | -                 | 멘티 FK          |
| mentor_id     | BIGINT   | NO   | -                 | 멘토 FK          |
| feedback_date | DATE     | NO   | -                 | 피드백 대상 날짜 |
| content       | TEXT     | NO   | -                 | 총평 내용        |
| created_at    | DATETIME | NO   | CURRENT_TIMESTAMP | 생성일시         |
| updated_at    | DATETIME | NO   | CURRENT_TIMESTAMP | 수정일시         |

```sql
CREATE TABLE overall_feedbacks (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL,
    mentor_id BIGINT NOT NULL,
    feedback_date DATE NOT NULL,
    content TEXT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_overall_feedbacks_mentee_date (mentee_id, feedback_date),
    KEY idx_overall_feedbacks_mentee (mentee_id),
    KEY idx_overall_feedbacks_date (feedback_date),
    CONSTRAINT fk_overall_feedbacks_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT fk_overall_feedbacks_mentor FOREIGN KEY (mentor_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.11 comments (코멘트/질문)

멘티가 작성하는 코멘트/질문입니다.

| 컬럼명       | 타입     | NULL | 기본값            | 설명      |
| ------------ | -------- | ---- | ----------------- | --------- |
| id           | BIGINT   | NO   | AUTO_INCREMENT    | PK        |
| mentee_id    | BIGINT   | NO   | -                 | 멘티 FK   |
| comment_date | DATE     | NO   | -                 | 관련 날짜 |
| content      | TEXT     | NO   | -                 | 내용      |
| created_at   | DATETIME | NO   | CURRENT_TIMESTAMP | 생성일시  |

```sql
CREATE TABLE comments (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL,
    comment_date DATE NOT NULL,
    content TEXT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_comments_mentee (mentee_id),
    KEY idx_comments_date (comment_date),
    KEY idx_comments_mentee_date (mentee_id, comment_date),
    CONSTRAINT fk_comments_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.12 columns (서울대쌤 칼럼)

서울대쌤 칼럼 게시글입니다.

| 컬럼명       | 타입         | NULL | 기본값            | 설명             |
| ------------ | ------------ | ---- | ----------------- | ---------------- |
| id           | BIGINT       | NO   | AUTO_INCREMENT    | PK               |
| title        | VARCHAR(200) | NO   | -                 | 칼럼 제목        |
| content      | LONGTEXT     | NO   | -                 | 본문 내용        |
| summary      | VARCHAR(500) | YES  | NULL              | 요약             |
| author       | VARCHAR(100) | YES  | NULL              | 작성자           |
| thumbnail_id | BIGINT       | YES  | NULL              | 썸네일 이미지 FK |
| created_at   | DATETIME     | NO   | CURRENT_TIMESTAMP | 생성일시         |
| updated_at   | DATETIME     | NO   | CURRENT_TIMESTAMP | 수정일시         |

```sql
CREATE TABLE columns (
    id BIGINT NOT NULL AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    content LONGTEXT NOT NULL,
    summary VARCHAR(500) NULL,
    author VARCHAR(100) NULL,
    thumbnail_id BIGINT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_columns_created (created_at DESC),
    CONSTRAINT fk_columns_thumbnail FOREIGN KEY (thumbnail_id) REFERENCES files(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.13 column_attachments (칼럼 첨부파일)

칼럼에 첨부된 파일을 연결합니다.

| 컬럼명    | 타입   | NULL | 기본값         | 설명    |
| --------- | ------ | ---- | -------------- | ------- |
| id        | BIGINT | NO   | AUTO_INCREMENT | PK      |
| column_id | BIGINT | NO   | -              | 칼럼 FK |
| file_id   | BIGINT | NO   | -              | 파일 FK |

```sql
CREATE TABLE column_attachments (
    id BIGINT NOT NULL AUTO_INCREMENT,
    column_id BIGINT NOT NULL,
    file_id BIGINT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uk_column_attachments (column_id, file_id),
    KEY idx_column_attachments_column (column_id),
    CONSTRAINT fk_column_attachments_column FOREIGN KEY (column_id) REFERENCES `columns`(id) ON DELETE CASCADE,
    CONSTRAINT fk_column_attachments_file FOREIGN KEY (file_id) REFERENCES files(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.14 weekly_reports (주간 학습리포트)

멘토가 작성하는 주간 학습리포트입니다.

| 컬럼명           | 타입         | NULL | 기본값            | 설명        |
| ---------------- | ------------ | ---- | ----------------- | ----------- |
| id               | BIGINT       | NO   | AUTO_INCREMENT    | PK          |
| mentee_id        | BIGINT       | NO   | -                 | 멘티 FK     |
| mentor_id        | BIGINT       | NO   | -                 | 멘토 FK     |
| week_number      | INT          | NO   | -                 | 주차 번호   |
| title            | VARCHAR(100) | NO   | -                 | 리포트 제목 |
| start_date       | DATE         | NO   | -                 | 주차 시작일 |
| end_date         | DATE         | NO   | -                 | 주차 종료일 |
| summary          | TEXT         | YES  | NULL              | 요약        |
| overall_feedback | TEXT         | NO   | -                 | 전체 피드백 |
| created_at       | DATETIME     | NO   | CURRENT_TIMESTAMP | 생성일시    |
| updated_at       | DATETIME     | NO   | CURRENT_TIMESTAMP | 수정일시    |

```sql
CREATE TABLE weekly_reports (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL,
    mentor_id BIGINT NOT NULL,
    week_number INT NOT NULL,
    title VARCHAR(100) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    summary TEXT NULL,
    overall_feedback TEXT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_weekly_reports_mentee_week (mentee_id, week_number),
    KEY idx_weekly_reports_mentee (mentee_id),
    CONSTRAINT fk_weekly_reports_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT fk_weekly_reports_mentor FOREIGN KEY (mentor_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.15 weekly_report_subjects (주간 리포트 과목별 상세)

주간 리포트의 과목별 상세 피드백입니다.

| 컬럼명           | 타입                            | NULL | 기본값         | 설명             |
| ---------------- | ------------------------------- | ---- | -------------- | ---------------- |
| id               | BIGINT                          | NO   | AUTO_INCREMENT | PK               |
| report_id        | BIGINT                          | NO   | -              | 주간 리포트 FK   |
| subject          | ENUM('KOREAN','ENGLISH','MATH') | NO   | -              | 과목             |
| completion_rate  | INT                             | NO   | 0              | 완료율 (%)       |
| total_study_time | INT                             | NO   | 0              | 총 공부시간 (분) |
| feedback         | TEXT                            | YES  | NULL           | 과목별 피드백    |

```sql
CREATE TABLE weekly_report_subjects (
    id BIGINT NOT NULL AUTO_INCREMENT,
    report_id BIGINT NOT NULL,
    subject ENUM('KOREAN', 'ENGLISH', 'MATH') NOT NULL,
    completion_rate INT NOT NULL DEFAULT 0,
    total_study_time INT NOT NULL DEFAULT 0,
    feedback TEXT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uk_report_subjects (report_id, subject),
    CONSTRAINT fk_report_subjects_report FOREIGN KEY (report_id) REFERENCES weekly_reports(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.16 refresh_tokens (리프레시 토큰)

JWT 리프레시 토큰 관리용 테이블입니다.

| 컬럼명     | 타입         | NULL | 기본값            | 설명          |
| ---------- | ------------ | ---- | ----------------- | ------------- |
| id         | BIGINT       | NO   | AUTO_INCREMENT    | PK            |
| user_id    | BIGINT       | NO   | -                 | 사용자 FK     |
| token      | VARCHAR(500) | NO   | -                 | 리프레시 토큰 |
| expires_at | DATETIME     | NO   | -                 | 만료일시      |
| created_at | DATETIME     | NO   | CURRENT_TIMESTAMP | 생성일시      |

```sql
CREATE TABLE refresh_tokens (
    id BIGINT NOT NULL AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    token VARCHAR(500) NOT NULL,
    expires_at DATETIME NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_refresh_tokens_token (token),
    KEY idx_refresh_tokens_user (user_id),
    KEY idx_refresh_tokens_expires (expires_at),
    CONSTRAINT fk_refresh_tokens_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.17 fcm_tokens (FCM 토큰)

FCM 푸시 알림 토큰 관리용 테이블입니다.

| 컬럼명       | 타입         | NULL | 기본값         | 설명        |
| ------------ | ------------ | ---- | -------------- | ----------- |
| id           | BIGINT       | NO   | AUTO_INCREMENT | PK          |
| user_id      | BIGINT       | NO   | -              | 사용자 FK   |
| token        | VARCHAR(255) | NO   | -              | FCM 토큰    |
| last_used_at | DATETIME(6)  | YES  | NULL           | 마지막 사용 |

```sql
CREATE TABLE fcm_tokens (
    id BIGINT NOT NULL AUTO_INCREMENT,
    last_used_at DATETIME(6) NULL,
    token VARCHAR(255) NOT NULL,
    user_id BIGINT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY UKqopjyk0c1cho2ep0abxd9hi5q (token)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.18 planner_completions (일일 플래너 마감)

일일 플래너 마감/피드백 요청 기록입니다.

| 컬럼명       | 타입     | NULL | 기본값            | 설명               |
| ------------ | -------- | ---- | ----------------- | ------------------ |
| id           | BIGINT   | NO   | AUTO_INCREMENT    | PK                 |
| mentee_id    | BIGINT   | NO   | -                 | 멘티 FK            |
| plan_date    | DATE     | NO   | -                 | 대상 날짜          |
| completed_at | DATETIME | NO   | CURRENT_TIMESTAMP | 마감 버튼 누른 시간|

```sql
CREATE TABLE planner_completions (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL COMMENT '멘티 FK',
    plan_date DATE NOT NULL COMMENT '대상 날짜',
    completed_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '마감 버튼 누른 시간',
    PRIMARY KEY (id),
    UNIQUE KEY uk_planner_completions_date (mentee_id, plan_date),
    CONSTRAINT fk_planner_completions_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='일일 플래너 마감/피드백 요청 기록';
```

---

### 2.19 monthly_reports (월간 리포트)

멘토가 작성하는 월간 학습리포트입니다.

| 컬럼명           | 타입         | NULL | 기본값            | 설명        |
| ---------------- | ------------ | ---- | ----------------- | ----------- |
| id               | BIGINT       | NO   | AUTO_INCREMENT    | PK          |
| mentee_id        | BIGINT       | NO   | -                 | 멘티 FK     |
| mentor_id        | BIGINT       | NO   | -                 | 멘토 FK     |
| report_year      | INT          | NO   | 2025              | 리포트 연도 |
| report_month     | INT          | NO   | -                 | 리포트 월   |
| title            | VARCHAR(100) | NO   | -                 | 리포트 제목 |
| summary          | TEXT         | YES  | NULL              | 요약        |
| overall_feedback | TEXT         | NO   | -                 | 전체 피드백 |
| created_at       | DATETIME     | NO   | CURRENT_TIMESTAMP | 생성일시    |
| updated_at       | DATETIME     | NO   | CURRENT_TIMESTAMP | 수정일시    |

```sql
CREATE TABLE monthly_reports (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL,
    mentor_id BIGINT NOT NULL,
    report_year INT NOT NULL DEFAULT 2025 COMMENT '리포트 연도',
    report_month INT NOT NULL COMMENT '리포트 월',
    title VARCHAR(100) NOT NULL,
    summary TEXT NULL,
    overall_feedback TEXT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_monthly_reports_unique (mentee_id, report_year, report_month),
    KEY idx_weekly_reports_mentee (mentee_id),
    CONSTRAINT fk_weekly_reports_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT fk_weekly_reports_mentor FOREIGN KEY (mentor_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.20 monthly_report_subjects (월간 리포트 과목별 상세)

월간 리포트의 과목별 상세 피드백입니다.

| 컬럼명           | 타입                            | NULL | 기본값         | 설명             |
| ---------------- | ------------------------------- | ---- | -------------- | ---------------- |
| id               | BIGINT                          | NO   | AUTO_INCREMENT | PK               |
| monthly_report_id| BIGINT                          | NO   | -              | 월간 리포트 FK   |
| subject          | ENUM('KOREAN','ENGLISH','MATH') | NO   | -              | 과목             |
| completion_rate  | INT                             | NO   | 0              | 완료율 (%)       |
| total_study_time | INT                             | NO   | 0              | 총 공부시간 (분) |
| feedback         | TEXT                            | YES  | NULL           | 과목별 피드백    |

```sql
CREATE TABLE monthly_report_subjects (
    id BIGINT NOT NULL AUTO_INCREMENT,
    monthly_report_id BIGINT NOT NULL,
    subject ENUM('KOREAN', 'ENGLISH', 'MATH') NOT NULL,
    completion_rate INT NOT NULL DEFAULT 0,
    total_study_time INT NOT NULL DEFAULT 0,
    feedback TEXT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY uk_monthly_report_subjects_unique (monthly_report_id, subject),
    CONSTRAINT fk_monthly_report_subjects_report FOREIGN KEY (monthly_report_id) REFERENCES monthly_reports(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.21 notifications (알림)

푸시 알림 내역을 관리합니다.

| 컬럼명     | 타입                                                                       | NULL | 기본값 | 설명       |
| ---------- | -------------------------------------------------------------------------- | ---- | ------ | ---------- |
| id         | BIGINT                                                                     | NO   | AUTO_INCREMENT | PK   |
| user_id    | BIGINT                                                                     | NO   | -      | 사용자 FK  |
| type       | ENUM('FEEDBACK_RECEIVED','PLANNER_COMPLETED','TASK_CONFIRMED','ZOOM_REQUEST') | NO | - | 알림 유형 |
| title      | VARCHAR(255)                                                               | NO   | -      | 알림 제목  |
| message    | VARCHAR(255)                                                               | NO   | -      | 알림 내용  |
| related_id | BIGINT                                                                     | YES  | NULL   | 관련 ID    |
| is_read    | BIT(1)                                                                     | NO   | -      | 읽음 여부  |
| is_sent    | BIT(1)                                                                     | NO   | -      | 전송 여부  |
| created_at | DATETIME(6)                                                                | NO   | -      | 생성일시   |

```sql
CREATE TABLE notifications (
    id BIGINT NOT NULL AUTO_INCREMENT,
    created_at DATETIME(6) NOT NULL,
    is_read BIT(1) NOT NULL,
    is_sent BIT(1) NOT NULL,
    message VARCHAR(255) NOT NULL,
    related_id BIGINT NULL,
    title VARCHAR(255) NOT NULL,
    type ENUM('FEEDBACK_RECEIVED','PLANNER_COMPLETED','TASK_CONFIRMED','ZOOM_REQUEST') NOT NULL,
    user_id BIGINT NOT NULL,
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### 2.22 zoom_meetings (Zoom 미팅 신청)

Zoom 미팅 신청 내역을 관리합니다.

| 컬럼명         | 타입                                   | NULL | 기본값            | 설명       |
| -------------- | -------------------------------------- | ---- | ----------------- | ---------- |
| id             | BIGINT                                 | NO   | AUTO_INCREMENT    | PK         |
| mentee_id      | BIGINT                                 | NO   | -                 | 멘티 FK    |
| preferred_date | DATE                                   | NO   | -                 | 희망 날짜  |
| preferred_time | TIME                                   | NO   | -                 | 희망 시간  |
| status         | ENUM('PENDING','CONFIRMED','CANCELLED')| NO   | 'PENDING'         | 신청 상태  |
| created_at     | DATETIME                               | NO   | CURRENT_TIMESTAMP | 신청 일시  |
| confirmed_at   | DATETIME(6)                            | YES  | NULL              | 확인 일시  |

```sql
CREATE TABLE zoom_meetings (
    id BIGINT NOT NULL AUTO_INCREMENT,
    mentee_id BIGINT NOT NULL COMMENT '멘티 FK',
    preferred_date DATE NOT NULL COMMENT '희망 날짜',
    preferred_time TIME NOT NULL COMMENT '희망 시간',
    status ENUM('PENDING','CONFIRMED','CANCELLED') NOT NULL DEFAULT 'PENDING' COMMENT '신청 상태',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '신청 일시',
    confirmed_at DATETIME(6) NULL,
    PRIMARY KEY (id),
    KEY idx_zoom_meetings_mentee_created (mentee_id, created_at DESC),
    CONSTRAINT fk_zoom_meetings_mentee FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='Zoom 미팅 신청 내역';
```

---

## 3. 인덱스 설계

### 3.1 복합 인덱스 설명

| 테이블                 | 인덱스                              | 용도                           |
| ---------------------- | ----------------------------------- | ------------------------------ |
| tasks                  | idx_tasks_mentee_date               | 멘티별 날짜 조회 (일일 플래너) |
| tasks                  | idx_tasks_mentee_date_confirmed     | 멘티별 날짜+확인 상태 조회     |
| solutions              | idx_solutions_mentee_subject        | 멘티별 과목 필터링             |
| comments               | idx_comments_mentee_date            | 멘티별 날짜 코멘트 조회        |
| overall_feedbacks      | uk_overall_feedbacks_mentee_date    | 멘티별 날짜 총평 (UNIQUE)      |
| weekly_reports         | uk_weekly_reports_mentee_week       | 멘티별 주차 리포트 (UNIQUE)    |
| monthly_reports        | uk_monthly_reports_unique           | 멘티별 연/월 리포트 (UNIQUE)   |
| monthly_report_subjects| uk_monthly_report_subjects_unique   | 리포트별 과목 (UNIQUE)         |
| planner_completions    | uk_planner_completions_date         | 멘티별 날짜 마감 (UNIQUE)      |
| zoom_meetings          | idx_zoom_meetings_mentee_created    | 멘티별 신청 내역 조회          |

### 3.2 조회 최적화

```sql
-- 멘티의 오늘/어제 할 일 조회 최적화
CREATE INDEX idx_tasks_mentee_date_confirmed
    ON tasks(mentee_id, task_date, is_mentor_confirmed);

-- 피드백이 있는 할 일 조회 최적화
CREATE INDEX idx_feedbacks_created
    ON feedbacks(created_at DESC);

-- 만료된 리프레시 토큰 정리용
CREATE INDEX idx_refresh_tokens_expires
    ON refresh_tokens(expires_at);

-- Zoom 미팅 신청 내역 조회
CREATE INDEX idx_zoom_meetings_mentee_created
    ON zoom_meetings(mentee_id, created_at DESC);
```

---

## 4. 초기 데이터

MVP 테스트를 위한 초기 데이터입니다. (멘토 1명 + 멘티 2명)

```sql
-- 비밀번호는 'password123'을 BCrypt로 암호화한 값
SET @encrypted_password = '$2a$10$N9qo8uLOickgx2ZMRZoMy.MqrqoLrIdV8YExvqz1xPJBj4VGJzTAO';

-- 사용자 등록
INSERT INTO users (login_id, password, name, role) VALUES
('mentor01', @encrypted_password, '김멘토', 'MENTOR'),
('mentee01', @encrypted_password, '민유진', 'MENTEE'),
('mentee02', @encrypted_password, '김철수', 'MENTEE');

-- 멘토-멘티 관계 설정
INSERT INTO mentor_mentee (mentor_id, mentee_id) VALUES
(1, 2),  -- 김멘토 - 민유진
(1, 3);  -- 김멘토 - 김철수

-- 약점 맞춤 솔루션 (민유진)
INSERT INTO solutions (mentee_id, title, subject) VALUES
(2, '문학 개념어 정리', 'KOREAN'),
(2, '빈칸 추론 유형 정복', 'ENGLISH'),
(2, '수열 점화식 유형', 'MATH');

-- 약점 맞춤 솔루션 (김철수)
INSERT INTO solutions (mentee_id, title, subject) VALUES
(3, '비문학 독해 전략', 'KOREAN'),
(3, '어법 총정리', 'ENGLISH'),
(3, '미적분 개념 정리', 'MATH');

-- 오늘 할 일 샘플 (민유진 - 오늘 날짜)
INSERT INTO tasks (mentee_id, solution_id, title, task_date, subject, is_mentor_assigned, created_by) VALUES
(2, 1, '수능완성 5회 1~11번', CURDATE(), 'KOREAN', 1, 1),
(2, 2, 'EBS 올림포스 Unit 5', CURDATE(), 'ENGLISH', 1, 1),
(2, 3, '수학의 정석 연습문제 3-1', CURDATE(), 'MATH', 1, 1);

-- 어제 할 일 샘플 (민유진 - 어제 날짜, 피드백 완료)
INSERT INTO tasks (mentee_id, solution_id, title, task_date, subject, study_time, is_mentor_assigned, is_mentor_confirmed, created_by) VALUES
(2, 1, '수능완성 4회 전범위', DATE_SUB(CURDATE(), INTERVAL 1 DAY), 'KOREAN', 60, 1, 1, 1),
(2, 2, 'EBS 올림포스 Unit 4', DATE_SUB(CURDATE(), INTERVAL 1 DAY), 'ENGLISH', 45, 1, 1, 1);

-- 어제 할 일에 대한 피드백
INSERT INTO feedbacks (task_id, mentor_id, content, summary, is_important) VALUES
(4, 1, '오늘 풀이한 문학 작품 해석은 전반적으로 잘 했습니다. 다만 접속부사 활용 부분에서 아직 개념이 헷갈리는 것 같아요.', '접속부사 활용 부분 다시 복습 필요', 1),
(5, 1, '빈칸 추론 정답률이 많이 올랐어요! 이 페이스 유지해주세요.', NULL, 0);

-- 어제 총평
INSERT INTO overall_feedbacks (mentee_id, mentor_id, feedback_date, content) VALUES
(2, 1, DATE_SUB(CURDATE(), INTERVAL 1 DAY), '전체적으로 집중력이 좋았습니다. 국어 문학 부분 조금 더 신경 써주세요.');

-- 서울대쌤 칼럼 샘플
INSERT INTO `columns` (title, content, summary, author) VALUES
('수능 D-100 학습 전략', '수능까지 100일, 효과적인 마무리 전략을 알려드립니다.\n\n1. 취약 영역 집중 보완\n2. 기출문제 반복 학습\n3. 시간 관리 연습\n...', '수능까지 100일, 효과적인 마무리 전략을 알려드립니다.', '서울대 국어국문학과'),
('국어 비문학 독해 꿀팁', '비문학 지문을 빠르고 정확하게 읽는 방법을 소개합니다.\n\n핵심은 문단별 주제 파악과 논리 구조 분석입니다...', '비문학 지문을 빠르고 정확하게 읽는 방법', '서울대 국어국문학과');

-- 주간 학습리포트 샘플
INSERT INTO weekly_reports (mentee_id, mentor_id, week_number, title, start_date, end_date, summary, overall_feedback) VALUES
(2, 1, 4, '4주차 (24.12.21~27)', '2024-12-21', '2024-12-27', '이번 주 전체적으로 잘 수행했습니다.', '전체적으로 학습 루틴이 잘 잡혀가고 있습니다. 다음 주에는 수학 미적분 부분에 조금 더 시간을 투자해주세요.');

INSERT INTO weekly_report_subjects (report_id, subject, completion_rate, total_study_time, feedback) VALUES
(1, 'KOREAN', 90, 180, '문학 영역 집중 학습 잘 진행됨'),
(1, 'ENGLISH', 85, 150, '빈칸 추론 정답률 향상'),
(1, 'MATH', 80, 200, '미적분 개념 보완 필요');
```

---

## 5. 테이블 관계 요약

| 관계                              | 설명                                  |
| --------------------------------- | ------------------------------------- |
| users ↔ mentor_mentee             | 멘토-멘티 N:M 관계 (중간 테이블)      |
| users → solutions                 | 멘티별 약점 맞춤 솔루션 (1:N)         |
| solutions → tasks                 | 목표-할일 관계 (1:N, relation)        |
| tasks → submissions               | 할 일당 1개 제출 (1:1)                |
| tasks → feedbacks                 | 할 일당 1개 피드백 (1:1)              |
| users → overall_feedbacks         | 멘티별 날짜당 1개 총평                |
| users → weekly_reports            | 멘티별 주간 리포트 (1:N)              |
| users → monthly_reports           | 멘티별 월간 리포트 (1:N)              |
| monthly_reports → monthly_report_subjects | 월간 리포트 과목별 상세 (1:N) |
| users → planner_completions       | 멘티별 일일 플래너 마감 (1:N)         |
| users → notifications             | 사용자별 알림 (1:N)                   |
| users → zoom_meetings             | 멘티별 Zoom 미팅 신청 (1:N)           |
| users → fcm_tokens                | 사용자별 FCM 토큰 (1:N)               |
| files                             | 다목적 파일 저장 (중간 테이블로 연결) |

---

_문서 버전: 2.0_
_최종 수정일: 2026-02-10_