# Google OAuth 인증 확장 가이드

## 개요
현재 이메일/비밀번호 인증만 지원하는 Todo 앱에 Google OAuth 로그인을 추가하는 작업 가이드입니다.

## 필요한 작업

### 1. Google Cloud Console 설정

#### 1.1 프로젝트 생성 및 OAuth 클라이언트 설정
1. [Google Cloud Console](https://console.cloud.google.com/) 접속
2. 새 프로젝트 생성 또는 기존 프로젝트 선택
3. **APIs & Services** > **OAuth consent screen** 이동
   - User Type: External 선택
   - 앱 정보 입력 (앱 이름, 사용자 지원 이메일 등)
   - 범위(Scopes) 추가: `email`, `profile`, `openid`
   - 테스트 사용자 추가 (개발 단계)

#### 1.2 OAuth 클라이언트 ID 생성
1. **APIs & Services** > **Credentials** 이동
2. **Create Credentials** > **OAuth client ID** 클릭
3. Application type: **Web application** 선택
4. **Authorized redirect URIs** 추가:
   ```
   https://ivessibhwguttctcgirb.supabase.co/auth/v1/callback
   ```
5. Client ID와 Client Secret 복사 (Supabase 설정에 필요)

### 2. Supabase 설정

#### 2.1 Google Provider 활성화
1. [Supabase Dashboard](https://supabase.com/dashboard) 접속
2. 프로젝트 선택: `ivessibhwguttctcgirb`
3. **Authentication** > **Providers** 이동
4. **Google** 찾아서 활성화
5. Google Cloud Console에서 복사한 정보 입력:
   - **Client ID**: Google OAuth 클라이언트 ID
   - **Client Secret**: Google OAuth 클라이언트 Secret
6. **Save** 클릭

#### 2.2 Redirect URL 확인
- Supabase에서 제공하는 Redirect URL 확인:
  ```
  https://ivessibhwguttctcgirb.supabase.co/auth/v1/callback
  ```
- 이 URL이 Google Cloud Console의 Authorized redirect URIs에 등록되어 있는지 확인

### 3. 프론트엔드 코드 수정

#### 3.1 auth.html 수정

##### Google 로그인 버튼 추가
```html
<!-- 기존 로그인/회원가입 폼 위에 추가 -->
<div class="social-login">
    <button class="btn btn-google" id="googleLoginBtn">
        <svg width="18" height="18" viewBox="0 0 18 18">
            <path fill="#4285F4" d="M17.64,9.2..."/> <!-- Google 로고 SVG -->
        </svg>
        Google로 계속하기
    </button>
</div>
<div class="divider">또는</div>
```

##### 스타일 추가
```css
.social-login {
    margin-bottom: 24px;
}

.btn-google {
    background: white;
    color: #757575;
    border: 1px solid #dadce0;
}

.btn-google:hover {
    background: #f8f9fa;
    border-color: #dadce0;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.divider {
    text-align: center;
    margin: 24px 0;
    color: #757575;
    font-size: 14px;
    position: relative;
}

.divider::before,
.divider::after {
    content: '';
    position: absolute;
    top: 50%;
    width: 40%;
    height: 1px;
    background: #e0e0e0;
}

.divider::before {
    left: 0;
}

.divider::after {
    right: 0;
}
```

##### JavaScript 코드 추가
```javascript
// Google 로그인 핸들러
document.getElementById('googleLoginBtn').addEventListener('click', async () => {
    try {
        const { data, error } = await supabaseClient.auth.signInWithOAuth({
            provider: 'google',
            options: {
                redirectTo: window.location.origin + '/todo/docs/index.html'
            }
        });

        if (error) throw error;

        // OAuth 플로우가 시작되면 자동으로 Google 로그인 페이지로 리다이렉트됨
    } catch (error) {
        console.error('Google login error:', error);
        showMessage('Google 로그인에 실패했습니다.', 'error');
    }
});
```

#### 3.2 docs/index.html (메인 앱) 수정

##### OAuth 콜백 처리 추가
```javascript
// 초기화 부분에 추가
(async () => {
    // OAuth 콜백 처리 (Google 로그인 후 리다이렉트됨)
    const { data: { session }, error } = await supabaseClient.auth.getSession();
    
    if (error) {
        console.error('Auth error:', error);
        window.location.href = '../auth.html';
        return;
    }

    if (session) {
        currentUser = session.user;
        document.getElementById('userEmail').textContent = currentUser.email;
        await loadTodos();
    } else {
        window.location.href = '../auth.html';
    }
})();
```

### 4. 로컬 개발 환경 설정

#### 4.1 Redirect URL 설정
개발 환경에서 테스트하려면 Google Cloud Console에 로컬 URL도 추가:
```
http://localhost:3000/auth/v1/callback
http://localhost:8000/todo/docs/index.html
```

#### 4.2 Supabase Redirect URL 설정
Supabase Dashboard > Authentication > URL Configuration:
- **Site URL** 설정
- **Redirect URLs** 추가:
  - `http://localhost:8000/todo/docs/index.html`
  - `https://{your-github-username}.github.io/kosa-vibecoding-2026-1st/src/exercise/secretnote89/day02/todo/docs/index.html`

### 5. 데이터베이스 고려사항

#### 5.1 사용자 프로필 확장 (선택사항)
Google OAuth를 통해 가입한 사용자의 추가 정보를 저장하려면:

```sql
-- user_profiles 테이블 생성 (선택사항)
CREATE TABLE IF NOT EXISTS user_profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    display_name TEXT,
    avatar_url TEXT,
    provider TEXT, -- 'email' 또는 'google'
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS 활성화
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;

-- 사용자는 자신의 프로필만 조회/수정 가능
CREATE POLICY "Users can view own profile"
    ON user_profiles FOR SELECT
    USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
    ON user_profiles FOR UPDATE
    USING (auth.uid() = id);

-- 프로필 자동 생성 트리거
CREATE OR REPLACE FUNCTION create_user_profile()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO user_profiles (id, display_name, avatar_url, provider)
    VALUES (
        NEW.id,
        NEW.raw_user_meta_data->>'full_name',
        NEW.raw_user_meta_data->>'avatar_url',
        COALESCE(NEW.raw_app_meta_data->>'provider', 'email')
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
    AFTER INSERT ON auth.users
    FOR EACH ROW
    EXECUTE FUNCTION create_user_profile();
```

### 6. 테스트 체크리스트

- [ ] Google Cloud Console OAuth 클라이언트 설정 완료
- [ ] Supabase Google Provider 활성화 및 설정 완료
- [ ] auth.html에 Google 로그인 버튼 추가
- [ ] Google 로그인 버튼 클릭 시 OAuth 플로우 시작
- [ ] Google 로그인 성공 후 메인 앱으로 리다이렉트
- [ ] 사용자 정보(이메일) 정상 표시
- [ ] 로그아웃 기능 정상 동작
- [ ] 이메일/비밀번호 로그인과 Google 로그인 모두 정상 동작
- [ ] 동일 이메일로 이메일 가입 후 Google 로그인 시 처리 (Supabase가 자동 처리)

### 7. 보안 고려사항

#### 7.1 PKCE 플로우 사용
Supabase는 기본적으로 PKCE (Proof Key for Code Exchange) 플로우를 사용하여 보안을 강화합니다.

#### 7.2 HTTPS 사용
프로덕션 환경에서는 반드시 HTTPS를 사용해야 합니다.
- GitHub Pages는 기본적으로 HTTPS 지원

#### 7.3 환경 변수 관리
민감한 정보는 환경 변수로 관리:
```javascript
// 프로덕션에서는 환경 변수 사용 권장
const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL;
const SUPABASE_ANON_KEY = import.meta.env.VITE_SUPABASE_ANON_KEY;
```

### 8. 추가 개선사항

#### 8.1 여러 OAuth Provider 지원
- GitHub OAuth
- Facebook OAuth
- Apple Sign In

#### 8.2 프로필 이미지 표시
Google OAuth를 통해 받은 프로필 이미지를 헤더에 표시:
```javascript
// user_metadata에서 프로필 이미지 가져오기
const avatarUrl = currentUser.user_metadata?.avatar_url;
if (avatarUrl) {
    // 프로필 이미지 표시
}
```

#### 8.3 계정 연결 기능
같은 이메일로 여러 Provider 연결 가능:
```javascript
const { data, error } = await supabaseClient.auth.linkIdentity({
    provider: 'google'
});
```

## 참고 자료

- [Supabase Google OAuth 공식 문서](https://supabase.com/docs/guides/auth/social-login/auth-google)
- [Google OAuth 2.0 가이드](https://developers.google.com/identity/protocols/oauth2)
- [Supabase Auth 헬퍼 함수](https://supabase.com/docs/reference/javascript/auth-signinwithoauth)

## 예상 소요 시간

- Google Cloud Console 설정: 15분
- Supabase 설정: 5분
- 프론트엔드 코드 수정: 30분
- 테스트 및 디버깅: 30분

**총 예상 시간: 약 1.5시간**
