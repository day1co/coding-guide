# React + Next.js 14.x 세부 코딩 가이드 (AI Agent 활용 고려)

본 문서는 Next.js 최신 안정 버전(14.x)을 기준으로 React 프로젝트 개발 시 Working(동작), Right(정확성), Fast(최적화)의 순서로 단계별 세부 사항을 기술한 내부 가이드입니다. 특히 AI Agent의 활용 환경을 고려하고 있습니다.

## 1. Working (작동)

이 단계의 목표는 코드를 정상적으로 동작시키는 것입니다.

### 세부 지침:

- **폴더 구조 (App Router 필수 사용)**
  ```
  app/
  ├── layout.tsx
  ├── page.tsx
  ├── api/ (API Routes)
  └── components/
  ```

- **페이지 및 라우팅**
  - 파일명은 항상 `page.tsx` 사용 (App Router 기준)
  - [Next.js Routing 공식 문서](https://nextjs.org/docs/app/building-your-application/routing)

- **컴포넌트 구성**
  - 최소한의 단위로 컴포넌트를 분리하여 재사용성 제고
  - 상태관리 간결화(useState/useReducer)

- **서버 및 클라이언트 컴포넌트 구분**
  - 기본은 서버 컴포넌트이며, 필요한 경우에만 `'use client'` 선언
  - [서버 vs 클라이언트 컴포넌트 공식 문서](https://nextjs.org/docs/app/building-your-application/rendering/server-components)

## 2. Right (정확성)

이 단계는 코드가 올바르게 동작하고 유지보수가 쉽도록 작성하는 단계입니다.

### 세부 지침:

- **타입스크립트(TypeScript) 사용**
  - 모든 컴포넌트 및 데이터 타입 명시 필수
  - 타입 정의는 별도의 파일 또는 컴포넌트 상단에 선언

- **책임 명확한 컴포넌트 설계**
  - 하나의 컴포넌트는 하나의 명확한 역할만 수행

- **명확한 상태관리 전략**
  - Zustand, Tanstack Query 권장
  - [Tanstack Query Next.js 예제](https://tanstack.com/query/latest/docs/framework/react/guides/ssr#nextjs)

- **코드 포매팅 (Prettier & ESLint)**
  - ESLint는 Next.js 기본 설정 권장 (`next lint`)
  - [Vercel 권장 설정](https://vercel.com/docs/concepts/projects/code-linting)

- **테스트 코드 작성 (중요)**
  - 단위 테스트: Jest 및 React Testing Library를 사용하여 필수 컴포넌트 및 로직을 테스트합니다.
  - E2E 테스트: Playwright를 사용하여 주요 사용자 흐름을 테스트합니다.
  - 테스트 코드는 명확한 시나리오 중심으로 작성하고, 유지보수가 용이하도록 별도 디렉토리 관리(`__tests__/`)
  - [Jest 공식 문서](https://jestjs.io/)
  - [React Testing Library 공식 문서](https://testing-library.com/docs/react-testing-library/intro/)
  - StackOverflow 및 커뮤니티에서 권장하는 Best Practice와 코드 예제를 적극 참조하여 최신 테스트 전략 적용

## 3. Fast (최적화)

성능을 고려하여 빠르고 효율적인 코드 작성이 목표입니다.

### 세부 지침:

- **Next.js 내장 최적화 적극 활용**
  - 이미지 및 폰트 최적화 필수 적용
  - [Vercel 성능 최적화 권장 사항](https://vercel.com/docs/concepts/performance/overview)

- **코드 분할 및 지연 로딩**
  - `next/dynamic` 적극 활용

- **명시적 캐싱 전략**
  - fetch 요청 시 명확한 캐싱 전략(`force-cache`, `no-store`) 사용

- **Bundle 최적화**
  - Bundle analyzer를 통해 지속적으로 모니터링

## 추가 지침

- **실험적 모듈 관리 전략 (Switch On/Off)**
  - 새롭게 도입하거나 실험적인 모듈은 언제든지 기능을 쉽게 비활성화할 수 있도록 명확한 스위칭 로직 적용 (Feature Toggle 기법)

### AI Agent 활용 지침

- **명확한 코드 구조 및 주석**
  - AI Agent가 코드 생성 및 분석 시 혼란을 방지하기 위해, 코드 흐름에 대한 명확한 주석 작성

- **코드 리뷰 프로세스 의무화**
  - AI Agent 생성 코드는 사람이 최종 검증 필수

## 공식 문서 및 커뮤니티 권장 사항 참조

- [Next.js 14 Documentation](https://nextjs.org/docs)
- [Next.js Best Practices](https://nextjs.org/docs/app/building-your-application/best-practices)
- [Vercel 플랫폼 Best Practices](https://vercel.com/docs/concepts)
- StackOverflow, GitHub Discussions 등 커뮤니티에서 제안된 최신 패턴 및 권장 사항을 적극적으로 참고하여 지속적으로 업데이트

본 문서는 팀의 공통 이해를 돕고 코드의 품질과 생산성을 지속적으로 향상시키기 위해 존재합니다.
