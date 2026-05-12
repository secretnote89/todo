# Todo List 애플리케이션 (Supabase 연동)

Supabase PostgreSQL과 이메일 인증을 통합한 머터리얼 디자인 반응형 Todo List 웹 애플리케이션입니다.

## 📋 프로젝트 개요

HTML, CSS, JavaScript와 Supabase를 사용하여 구현한 멀티 유저 Todo List 앱입니다. 프레임워크 없이 순수 웹 기술과 Supabase BaaS로 구현되었습니다.

## ✨ 주요 기능

### 인증 시스템
1. **이메일 회원가입**
   - Supabase Auth 기반 회원가입
   - 이메일 인증 링크 전송
   - 안전한 비밀번호 암호화

2. **로그인/로그아웃**
   - 이메일/비밀번호 로그인
   - 세션 기반 인증 유지
   - 안전한 로그아웃

### Todo 관리
3. **할 일 추가**
   - 텍스트 입력을 통한 할 일 추가
   - Enter 키 또는 버튼 클릭으로 추가
   - 빈 입력 방지 검증

4. **할 일 완료 체크**
   - 체크박스로 완료/미완료 상태 토글
   - 완료된 항목은 취소선 및 회색 처리
   - 실시간 데이터베이스 동기화

5. **할 일 삭제**
   - 개별 항목마다 삭제 버튼
   - 아이콘 기반 직관적인 UI

6. **Supabase 저장**
   - PostgreSQL 데이터베이스에 저장
   - 사용자별 데이터 격리 (RLS)
   - 실시간 데이터 동기화

### 고급 기능

7. **우선순위 설정**
   - 3단계 우선순위: 높음 / 중간 / 낮음
   - 우선순위별 색상 구분
     - 높음: 레드 (#c62828)
     - 중간: 오렌지 (#f57f17)
     - 낮음: 그린 (#2e7d32)
   - 우선순위 뱃지 표시

8. **드래그 앤 드롭**
   - 할 일 항목을 드래그하여 순서 변경
   - 다른 우선순위 영역으로 드래그 시 우선순위 자동 변경
   - 우선순위 변경 시 확인 알림창 표시
   - 드래그 핸들 아이콘 제공

9. **Position 기반 정렬**
   - 데이터베이스 position 컬럼으로 순서 관리
   - 드래그 앤 드롭 시 자동 position 업데이트

## 🎨 디자인

### 머터리얼 디자인 적용
- Google Material Design 가이드라인 준수
- Roboto 폰트 사용
- Material Icons 적용
- 파스텔 톤 컬러 스킴
- 그라데이션 배경 (핑크 → 블루)

### UI 컴포넌트
- **인증 페이지**: 탭 기반 로그인/회원가입 전환
- **헤더**: 사용자 정보 및 로그아웃 버튼
- **입력 필드**: 아이콘이 있는 플로팅 입력
- **버튼**: 머터리얼 Raised Button 스타일
- **카드**: 그림자 효과가 있는 카드 레이아웃
- **뱃지**: 도트 표시가 있는 우선순위 뱃지
- **아이콘**: Material Icons (드래그, 삭제, 체크 등)

### 반응형 디자인
- **데스크톱**: 최대 너비 600px
- **태블릿** (≤768px): 레이아웃 조정
- **모바일** (≤480px): 모바일 최적화

## 🛠️ 기술 스택

### Frontend
- **HTML5**: 시맨틱 마크업
- **CSS3**: 
  - Flexbox 레이아웃
  - CSS 그라데이션
  - CSS 트랜지션 & 애니메이션
  - 커스텀 스크롤바
  - 미디어 쿼리
- **JavaScript (Vanilla)**:
  - ES6+ 문법
  - Async/Await
  - DOM 조작
  - 이벤트 처리
  - Drag and Drop API

### Backend (Supabase)
- **Database**: PostgreSQL
- **Authentication**: Supabase Auth (이메일 인증)
- **Security**: Row Level Security (RLS)
- **SDK**: @supabase/supabase-js@2

### Deployment
- **GitHub Actions**: 자동 배포 CI/CD
- **GitHub Pages**: 정적 호스팅

## 📁 파일 구조

```
todo/
├── index.html              # 메인 Todo 앱 (인증된 사용자용)
├── auth.html              # 로그인/회원가입 페이지
├── supabase-setup.sql     # Supabase 데이터베이스 설정 스크립트
└── README.md              # 프로젝트 문서
```

## 🗄️ 데이터베이스 스키마

### todos 테이블
```sql
CREATE TABLE todos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  text TEXT NOT NULL,
  completed BOOLEAN DEFAULT FALSE,
  priority TEXT CHECK (priority IN ('high', 'medium', 'low')) DEFAULT 'medium',
  position INTEGER NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 인덱스
- `idx_todos_user_id`: 사용자별 조회 최적화
- `idx_todos_priority`: 우선순위별 필터링
- `idx_todos_position`: 정렬 성능 향상

### RLS 정책
- 사용자는 자신의 todos만 조회/삽입/수정/삭제 가능
- `auth.uid() = user_id` 조건으로 데이터 격리

## 🚀 설정 및 배포

### 1. Supabase 설정

#### 프로젝트 생성
1. [Supabase Dashboard](https://app.supabase.com) 접속
2. 새 프로젝트 생성
3. Project URL과 anon key 확인

#### 데이터베이스 설정
1. Supabase Dashboard → SQL Editor
2. `supabase-setup.sql` 파일 내용 복사
3. SQL 실행

#### 인증 설정
1. Authentication → Providers → Email 활성화
2. Confirm email 활성화 (권장)
3. Authentication → URL Configuration:
   - Site URL: `https://YOUR-USERNAME.github.io/kosa-vibecoding-2026-1st/src/exercise/secretnote89/day02/todo/`
   - Redirect URLs:
     - `http://localhost:8000/*`
     - `https://YOUR-USERNAME.github.io/*`

### 2. 환경 변수 설정

`auth.html`과 `index.html`의 Supabase 설정 확인:
```javascript
const SUPABASE_URL = 'https://ivessibhwguttctcgirb.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key';
```

### 3. GitHub Pages 배포

#### Repository 설정
1. GitHub Repository → Settings → Pages
2. Source: "GitHub Actions" 선택
3. `.github/workflows/deploy-todo.yml` 푸시

#### 자동 배포
- `main` 브랜치에 푸시하면 자동 배포
- `src/exercise/secretnote89/day02/todo/**` 경로 변경 감지

#### 배포 URL
`https://YOUR-USERNAME.github.io/kosa-vibecoding-2026-1st/src/exercise/secretnote89/day02/todo/`

## 🧪 로컬 테스트

### 로컬 서버 실행
```bash
cd src/exercise/secretnote89/day02/todo
python3 -m http.server 8000
```

### 테스트 시나리오
1. `http://localhost:8000/auth.html` 접속
2. 회원가입 → 이메일 인증
3. 로그인 → `index.html`로 리다이렉트
4. Todo CRUD 기능 테스트
5. 드래그 앤 드롭 테스트
6. 로그아웃 → 재로그인

### Supabase 확인
1. Table Editor → todos 테이블
2. Authentication → Users
3. Logs → 쿼리 실행 로그

## 💾 데이터 구조

### Todo 객체
```javascript
{
  id: UUID,               // 고유 ID
  user_id: UUID,          // 사용자 ID (FK)
  text: String,           // 할 일 내용
  completed: Boolean,     // 완료 상태
  priority: String,       // 우선순위 ('high', 'medium', 'low')
  position: Integer,      // 정렬 순서
  created_at: Timestamp,  // 생성 시간
  updated_at: Timestamp   // 수정 시간
}
```

## 🔒 보안

### Row Level Security (RLS)
- 모든 쿼리에 `user_id` 필터 자동 적용
- SQL Injection 방지
- 사용자간 데이터 격리

### 인증
- Supabase Auth로 안전한 인증
- JWT 토큰 기반 세션 관리
- 비밀번호 bcrypt 암호화

### 환경 변수
- `anon key`는 클라이언트 노출 가능 (RLS로 보호)
- `service_role key`는 절대 노출 금지

## 🎯 주요 특징

1. **멀티 유저 지원**: 사용자별 독립적인 Todo 관리
2. **실시간 동기화**: Supabase 실시간 데이터베이스
3. **안전한 인증**: 이메일 인증 및 RLS
4. **반응형 디자인**: 모든 디바이스 지원
5. **직관적 UX**: 드래그 앤 드롭, 시각적 피드백
6. **자동 배포**: GitHub Actions CI/CD
7. **무료 호스팅**: GitHub Pages

## 🔄 개발 히스토리

### v1.0 - 기본 기능
- 할 일 추가/완료/삭제
- 로컬스토리지 저장
- 파스텔 톤 색상 테마

### v2.0 - 우선순위 기능
- 3단계 우선순위 시스템
- 우선순위별 색상 구분
- 자동 정렬 기능

### v3.0 - 드래그 앤 드롭
- 드래그 앤 드롭으로 순서 변경
- 드래그로 우선순위 변경
- 우선순위 변경 확인 알림

### v4.0 - 머터리얼 디자인
- 머터리얼 디자인 가이드라인 적용
- Material Icons 통합
- 개선된 반응형 레이아웃

### v5.0 - Supabase 연동 (현재)
- PostgreSQL 데이터베이스 연동
- 이메일 인증 시스템
- 멀티 유저 지원
- GitHub Pages 자동 배포
- RLS 보안 적용

## 📱 브라우저 지원

- Chrome (권장)
- Firefox
- Safari
- Edge
- 모던 브라우저 (ES6+ 지원 필요)

## 🐛 트러블슈팅

### 이메일 인증이 안 됨
- Supabase Dashboard → Authentication → Email Templates 확인
- 스팸 메일함 확인
- SMTP 설정 확인

### RLS 정책 에러
- SQL Editor에서 RLS 정책 재생성
- `auth.uid()`가 올바른지 확인

### 배포 URL 접속 안 됨
- GitHub Actions 로그 확인
- Pages 설정 확인 (Source: GitHub Actions)
- 몇 분 대기 후 재시도

### 로컬 테스트 시 CORS 에러
- 로컬 서버(`python3 -m http.server`) 사용 필수
- `file://` 프로토콜은 지원 안 됨

## 📝 라이선스

본 프로젝트는 교육 목적으로 제작되었습니다.

## 👨‍💻 개발자

- KOSA 2026 1기
- secretnote89

## 🔗 링크

- [Supabase 공식 문서](https://supabase.com/docs)
- [Material Design](https://material.io/design)
- [GitHub Pages](https://pages.github.com/)
# todo
