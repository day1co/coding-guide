# UNIVERSAL-API 코딩 가이드

본 문서는 Fastify 기반의 TypeScript 프로젝트 개발 시 Working(동작), Right(정확성), Fast(최적화)의 순서로 단계별 세부 사항을 기술한 내부 가이드입니다.

## 1. Working (작동)

이 단계의 목표는 코드를 정상적으로 동작시키는 것입니다.

### 폴더 구조
```
├── @types/          # TypeScript 타입 정의
├── common/          # 공통 유틸리티 및 상수
├── controllers/     # API 컨트롤러
├── facades/         # 외부 서비스 연동 계층
├── middlewares/     # 미들웨어
├── routes/          # API 라우트 정의
├── schema/          # API 스키마 정의
├── services/        # 비즈니스 로직
├── utils/          # 유틸리티 함수
├── server.ts       # 서버 설정
└── index.ts        # 진입점
```

### 세부 지침:

* **라우팅 구성**
  * 모든 라우트는 `routes` 디렉토리에 도메인별로 구성
  * 라우트 등록은 `routes/register-routes.ts`에서 중앙 관리
  * URL 패턴: `/{도메인}/{리소스}/{동작}`

* **컨트롤러 구성**
  * 도메인별로 디렉토리 분리
  * 컨트롤러는 요청/응답 처리에만 집중
  * Fastify 플러그인 형태로 구현

* **미들웨어 활용**
  * 인증/인가: `auth.middleware.ts`
  * 캐시: `cache.middleware.ts`
  * 파라미터 검증: `param-validator.middleware.ts`
  * 에러 처리: `error.handler.ts`

## 2. Right (정확성)

이 단계는 코드가 올바르게 동작하고 유지보수가 쉽도록 작성하는 단계입니다.

### 세부 지침:

* **타입스크립트(TypeScript) 사용**
  * 모든 함수와 변수에 타입 명시 필수
  * 공통 타입은 `@types` 디렉토리에 정의
  * 도메인별 타입은 해당 도메인 디렉토리 내 `types.ts`에 정의

* **스키마 검증(Schema Validation)**
  * Fastify는 내부적으로 ajv(Another JSON Schema Validator) 라이브러리를 사용하여 요청/응답에 대한 유효성을 검사
  * 모든 API 응답은 JSON Schema로 정의하여 검증
    - 응답 스키마를 정의할 경우, 스키마에 정의되지 않은 프로퍼티는 response에서 제외됨
  * `schema` 디렉토리에 도메인별로 스키마 정의
  * $id를 사용하여, 재사용성과 참조를 위한 식별자 지정 가능

  ```typescript
  // 스키마 정의 예시
  // src/schema/page.ts
  export const PageWithCuration = {
    $id: 'pageWithCuration',
    type: 'object',
    properties: {
      ...ObjectUtil.omit(Page.properties, ['headJson', 'mainJson']),
      currentPageState: { type: 'string' },
      curationGroups: {
        type: 'array',
        items: { $ref: 'curationGroup#' },
      },
    },
  };

  // 스키마 등록 예시
  // src/schema/index.ts
  export const registerSchema = (server: any) => {
    server.addSchema(PageWithCuration);
  };

  // 스키마 참조 예시
  // src/routes/page-v2/page.schema.ts
  export const getPreviewPageSchema = {
    tags: ['page', 'v2'],
    summary: '페이지 미리보기',
    description: 'id로 페이지 미리보기를 지원합니다. curation 데이터도 포함됩니다.',
    params: {
      type: 'object',
      properties: {
        id: { type: 'integer', description: 'page id' },
      },
      required: ['id'],
    },
    querystring: {
      type: 'object',
      properties: {
        language: { type: 'string', description: `${Object.values(CustomerLanguageCode)}` },
      },
    },
    response: {
      200: {
        type: 'object',
        properties: {
          data: { $ref: 'pageWithCuration#' },
        },
      },
    },
  };
  ```

* **코드 포매팅**
  ```javascript
  // .prettierrc.js
  module.exports = {
    ...require('@day1co/prettier-config'),
  };
  ```

* **테스트 코드 작성**
  * 테스트 파일 위치: `tests/{도메인}.spec.ts`
  * 테스트 헬퍼: `tests/helper/`
  * 테스트 데이터: `tests/fixtures/`
  * Jest와 Supertest 활용

```typescript
// 테스트 코드 예시
describe('Domain API Test', () => {
  let server: ApiHandler;

  beforeAll(async () => {
    server = await setupServer(ServiceName.COLOSO);
  });

  afterAll(async () => {
    await server.stop();
  });

  it('should handle request correctly', async () => {
    const response = await server.use().inject({
      method: 'GET',
      url: '/domain/resource',
    });
    expect(response.statusCode).toBe(200);
  });
});
```

## 3. Fast (최적화)

성능을 고려하여 빠르고 효율적인 코드 작성이 목표입니다.

### 세부 지침:

* **캐싱 전략**
  * Redis 캐싱 활용
  * 응답 압축 (@fastify/compress)
  * 적절한 캐시 TTL 설정

* **데이터베이스 최적화**
  * TypeORM 인덱스 활용
  * 적절한 커넥션 풀 설정
  * 쿼리 최적화

* **로깅 전략**
  * 로그 레벨 적절히 설정
  * 운영 환경에서는 debug 로그 비활성화
  * 중요 API는 요청/응답 로깅 필수

## 4. API 설계 가이드

### 기본 원칙:

* **RESTful API 설계**
  ```typescript
  // routes/example/index.ts
  export const ExampleRoute = (server: FastifyInstance) => {
    server.get('/examples', listHandler);           // 목록 조회
    server.get('/examples/:id', getHandler);        // 단일 조회
    server.post('/examples', createHandler);        // 생성
    server.put('/examples/:id', updateHandler);     // 수정
    server.delete('/examples/:id', deleteHandler);  // 삭제
  };
  ```

* **응답 형식 통일**
  ```typescript
  interface ApiResponse<T> {
    success: boolean;
    data?: T;
    error?: {
      code: string;
      message: string;
    };
  }
  ```

### 에러 처리:

* **표준 에러 코드 사용**
  ```typescript
  // common/errors.ts
  export class CustomException extends Error {
    constructor(
      public code: string,
      public status: number,
      message: string
    ) {
      super(message);
    }
  }
  ```

## 5. 배포 및 운영

* **도커라이즈**
  * 개발용: `dockerfile`
  * 운영용: `production.dockerfile`

* **환경 설정**
  * Spring Cloud Config 사용
  * 환경별 설정 분리
  * 민감 정보는 환경 변수로 관리

## 6. 의존성 관리

* **패키지 버전 관리**
  * package.json의 의존성 버전은 구체적으로 명시
  * 내부 패키지(@day1co/)는 호환성 보장된 버전 사용
  * npm workspace 활용

## 추가 지침

* **코드 리뷰 프로세스**
  * PR 전 로컬 테스트 수행
  * 테스트 커버리지 확인
  * 코드 품질 검사 (ESLint)

* **문서화**
  * Swagger를 통한 API 문서 자동화
  * README.md 최신화
  * 주요 설정 및 스크립트 문서화

이 가이드라인은 UNIVERSAL-API 프로젝트의 구조와 패턴을 기반으로 작성되었으며, 팀의 생산성과 코드 품질 향상을 위한 참고 자료로 활용됩니다. 
