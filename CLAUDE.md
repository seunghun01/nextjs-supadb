# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

Next.js + Supabase 스타터킷 기반 프로젝트. 비밀번호 기반 인증(로그인, 회원가입, 비밀번호 재설정)이 구현되어 있으며, Supabase Auth를 쿠키 기반으로 사용한다.

## 개발 명령어

```bash
npm run dev      # 개발 서버 실행 (localhost:3000)
npm run build    # 프로덕션 빌드
npm run start    # 프로덕션 서버 실행
npm run lint     # ESLint 실행
```

shadcn/ui 컴포넌트 추가:
```bash
npx shadcn@latest add [component-name]
```

## 환경변수

`.env.local` 파일에 다음 두 변수 필요:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`

## 아키텍처

### 기술 스택
- **Next.js** (App Router) + **React 19** + **TypeScript**
- **Supabase** (@supabase/ssr, @supabase/supabase-js) - 인증 및 DB
- **Tailwind CSS 3** + **shadcn/ui** (new-york 스타일, lucide 아이콘)
- **next-themes** - 다크모드 지원

### Supabase 클라이언트 패턴 (중요)

세 가지 Supabase 클라이언트가 존재하며, 용도에 맞게 사용해야 한다:

| 파일 | 용도 | 주의사항 |
|---|---|---|
| `lib/supabase/server.ts` | Server Components, Route Handlers | 항상 함수 내에서 새로 생성. 절대 전역 변수로 캐시하지 않음 |
| `lib/supabase/client.ts` | Client Components (`"use client"`) | `createBrowserClient` 사용 |
| `lib/supabase/proxy.ts` | Middleware (세션 갱신) | `proxy.ts`에서 호출. `createServerClient` 후 즉시 `getClaims()` 호출 필수 |

### 인증 흐름

- `proxy.ts` (미들웨어): 모든 요청에서 세션을 갱신하고, 미인증 사용자를 `/auth/login`으로 리다이렉트 (`/`, `/login`, `/auth/*` 제외)
- `app/auth/*`: 인증 관련 페이지 (login, sign-up, forgot-password, update-password, confirm, error)
- `app/protected/*`: 인증 필요 영역. 서버 컴포넌트에서 `getClaims()`로 인증 확인 후, 실패 시 `/auth/login`으로 리다이렉트
- 인증 폼 컴포넌트들은 Client Component로 구현 (`login-form.tsx`, `sign-up-form.tsx` 등)

### 라우트 구조

- `/` - 홈 (공개)
- `/auth/*` - 인증 페이지들 (공개)
- `/protected/*` - 인증 필요 영역

### 경로 별칭

`@/*` → 프로젝트 루트 기준 (예: `@/components/ui/button`, `@/lib/utils`)

## 컨벤션

- **파일명**: kebab-case (`login-form.tsx`)
- **컴포넌트명**: PascalCase, named export 사용 (`export function LoginForm()`)
- **페이지 컴포넌트**: default export 사용
- **폴더명**: 소문자 kebab-case
- **import 순서**: 외부 라이브러리 → 내부 (`@/`) → 상대 경로
- shadcn/ui 컴포넌트는 `components/ui/`에 위치
- CSS 클래스 결합 시 `cn()` 유틸리티 사용 (`lib/utils.ts`)
