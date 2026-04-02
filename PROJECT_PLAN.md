# 🔍 ElasticSearch 학습 사이드 프로젝트 계획서

> **프로젝트명**: SearchHub — 실시간 통합 검색 플랫폼  
> **목적**: Elasticsearch 핵심 개념 학습 + 멀티 프레임워크 벤치마크  
> **기간**: 약 11주  
> **작성일**: 2026-04-03

---

## 1. 프로젝트 개요

### 1.1 프로젝트 소개

**SearchHub**는 상품 데이터를 Elasticsearch로 색인하고, 검색 쿼리·집계·자동완성 등 ES 핵심 기능을 학습하는 프로젝트입니다.  
추가로 **동일한 API를 3개 언어/프레임워크로 구현**하고 k6 부하 테스트를 통해 프레임워크별 성능을 비교합니다.

### 1.2 프로젝트 구성

```
Phase 1~4  : Kotlin (Spring Boot 4.x) 메인 구현 — ES 핵심 학습
Phase 5    : Python (Robyn) / TypeScript (Elysia.js) 포팅
Phase 6    : k6 벤치마크 — 프레임워크별 성능 비교
Phase 7    : 문서화 및 블로그 포스팅
```

### 1.3 핵심 학습 목표

| # | 학습 주제 | 내용 |
|---|-----------|------|
| 1 | **인덱스 설계** | Mapping, Analyzer, Tokenizer 설정 |
| 2 | **검색 쿼리** | match, bool, filter, term, range, fuzzy |
| 3 | **집계(Aggregation)** | terms, histogram, date_histogram, avg, sum |
| 4 | **자동완성** | completion suggester, edge n-gram analyzer |
| 5 | **데이터 동기화** | RDB → Elasticsearch 동기화 전략 |
| 6 | **프레임워크 벤치마크** | Kotlin vs Python vs TypeScript 성능 비교 |

---

## 2. 기술 스택

### 2.1 공유 인프라 (Docker Compose)

```
Elasticsearch : 9.x
Kibana        : 9.x  (모니터링 / Dev Tools)
MySQL         : 8.0  (원천 데이터)
Redis         : 7.x  (인기 검색어 / 캐시)
```

### 2.2 Backend — 3개 구현체 (동일 API)

| 구분 | 언어 | 프레임워크 | ES 클라이언트 | 런타임 |
|------|------|-----------|--------------|--------|
| **메인** | Kotlin | Spring Boot 4.x | Spring Data Elasticsearch | JVM (Java 25) |
| **포팅 A** | Python | Robyn | elasticsearch-py | CPython 3.13 |
| **포팅 B** | TypeScript | Elysia.js | @elastic/elasticsearch | Bun |

### 2.3 Frontend

```
Framework  : React + Vite (TypeScript)
UI Library : shadcn/ui
검색 UX    : 실시간 자동완성, 패싯 필터, 하이라이팅
```

### 2.4 벤치마크 도구

```
부하 테스트 : k6
시나리오    : 검색 / 자동완성 / 집계 API 각각 독립 테스트
지표        : RPS, P50/P95/P99 레이턴시, 에러율
```

---

## 3. 도메인 설계

### 3.1 핵심 도메인 (전자상거래 검색 플랫폼)

```
Product (상품)
  ├── id, name, description, category, price, brand
  ├── tags[], stock, rating, reviewCount
  └── createdAt, updatedAt

Review (리뷰)
  ├── id, productId, userId, content, score
  └── createdAt

User (사용자)
  ├── id, username, nickname
  └── createdAt
```

### 3.2 Elasticsearch 인덱스 구조

```json
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "korean_analyzer",
        "fields": {
          "keyword": { "type": "keyword" },
          "suggest": { "type": "completion" }
        }
      },
      "description": { "type": "text", "analyzer": "korean_analyzer" },
      "category":    { "type": "keyword" },
      "brand":       { "type": "keyword" },
      "price":       { "type": "double" },
      "rating":      { "type": "float" },
      "tags":        { "type": "keyword" },
      "createdAt":   { "type": "date" }
    }
  }
}
```

---

## 4. 단계별 개발 로드맵

### Phase 1: 환경 구성 및 기초 색인 (1~2주)

> 메인 구현: **Kotlin + Spring Boot 4.x**

- [ ] Docker Compose로 ES 9.x + Kibana 구동
- [ ] Kotlin + Spring Boot 4.x 프로젝트 생성
- [ ] `spring-data-elasticsearch` 연동 (ElasticsearchClient 기반)
- [ ] MySQL 샘플 상품 데이터 삽입 (Faker)
- [ ] **인덱스 생성**: Mapping 수동 설계 (`@Mapping`, `@Setting`)
- [ ] **한국어 Analyzer**: Nori 플러그인 설치 및 커스텀 Analyzer 구성
- [ ] **벌크 색인**: JPA → ES 동기화 배치 구현
- [ ] Kibana Dev Tools로 기본 쿼리 실습

**핵심 학습 포인트**
```
✅ Index vs Type 개념
✅ Inverted Index 동작 원리
✅ Analyzer = CharFilter + Tokenizer + TokenFilter
✅ Nori 한국어 형태소 분석기
✅ RestHighLevelClient 제거 → Java API Client (ElasticsearchClient)
```

---

### Phase 2: 검색 기능 구현 (3~4주)

- [ ] **기본 검색 API** (`GET /api/products/search?q=키워드`)
  - `multi_match` 쿼리 (name, description, tags)
  - `bool` 쿼리로 must / filter / should 조합
- [ ] **패싯 필터** (카테고리, 브랜드, 가격대, 평점)
- [ ] **정렬** (관련도순, 최신순, 가격순, 평점순)
- [ ] **페이지네이션** (from/size vs search_after)
- [ ] **검색 결과 하이라이팅** (`highlight` 옵션)
- [ ] **Fuzzy 검색** (`fuzziness` 파라미터)

**핵심 학습 포인트**
```
✅ 쿼리 컨텍스트 vs 필터 컨텍스트 (스코어링 차이)
✅ TF-IDF / BM25 relevance scoring
✅ from/size Deep Pagination 성능 문제
✅ search_after를 이용한 안전한 페이지네이션
```

---

### Phase 3: 자동완성 및 추천 (5주차)

- [ ] **자동완성 API** (`GET /api/products/suggest?q=삼성`)
  - `completion` Suggester 구현
  - `edge_ngram` Analyzer 기반 구현 (비교 실습)
- [ ] **인기 검색어 Top 10** (Redis Sorted Set 활용)
- [ ] **연관 검색어 추천** (검색 로그 기반)

**핵심 학습 포인트**
```
✅ Completion Suggester vs Edge N-gram 방식 차이
✅ 검색어 저장 전략 (Redis vs ES)
```

---

### Phase 4: 집계(Aggregation) 및 데이터 동기화 (6~7주차)

**집계**
- [ ] 카테고리별 상품 수 (`terms` aggregation)
- [ ] 가격대별 분포 (`histogram` aggregation)
- [ ] 월별 신규 상품 (`date_histogram` aggregation)
- [ ] 평균 평점 / 최대·최소 가격 (`avg`, `min`, `max`)
- [ ] Kibana 대시보드 구성

**데이터 동기화**
- [ ] 이벤트 기반 동기화: ApplicationEvent → ES 색인
- [ ] Index Alias 활용 무중단 재색인 (`products_v1` → `products`)
- [ ] Logstash JDBC Input 실습 (선택)

**핵심 학습 포인트**
```
✅ Bucket / Metric / Pipeline Aggregation 분류
✅ Sub-aggregation (중첩 집계)
✅ Index Alias를 이용한 zero-downtime 재색인
✅ Bulk API 성능 최적화
```

---

### Phase 5: Python / TypeScript 포팅 (8~9주차)

> Kotlin 구현이 완성된 동일한 API를 두 언어로 포팅합니다.  
> ES 쿼리 로직은 동일하게 유지하여 순수 프레임워크 성능만 비교합니다.

**Python — Robyn**
- [ ] Robyn 프로젝트 생성 (Python 3.13)
- [ ] `elasticsearch-py` 연동
- [ ] 검색 / 자동완성 / 집계 API 포팅
- [ ] MySQL 연동 (SQLAlchemy)

**TypeScript — Elysia.js (Bun)**
- [ ] Elysia.js 프로젝트 생성 (Bun 런타임)
- [ ] `@elastic/elasticsearch` 연동
- [ ] 검색 / 자동완성 / 집계 API 포팅
- [ ] Prisma or Bun SQLite → MySQL 연동

**공통 포인트**
```
✅ 각 언어별 ES 클라이언트 차이점 파악
✅ 비동기 처리 방식 비교 (Kotlin Coroutines vs Python async vs Bun)
```

---

### Phase 6: 프레임워크 벤치마크 (10주차)

> **공정한 비교를 위한 조건 통일**

```
- 동일한 Docker 네트워크 내 실행
- 동일한 ES 9.x 클러스터 및 MySQL 8.0 사용
- JVM 워밍업: Kotlin은 30초 워밍업 후 측정
- 각 서버 단일 인스턴스, 기본 설정
- Virtual Users: 50 / Duration: 30s / Ramp-up: 10s
```

**벤치마크 시나리오**

| 시나리오 | 엔드포인트 | 설명 |
|---------|-----------|------|
| 검색 부하 | `GET /api/products/search?q=노트북` | 가장 복잡한 bool 쿼리 |
| 자동완성 | `GET /api/products/suggest?q=삼` | 짧은 레이턴시 중요 |
| 집계 조회 | `GET /api/products/aggregations` | 무거운 집계 쿼리 |

**측정 지표**
- RPS (Requests Per Second)
- P50 / P95 / P99 응답 시간
- 에러율 (%)
- CPU / Memory 사용량

**결과 예시 포맷**

| 지표 | Kotlin (Spring) | Python (Robyn) | TypeScript (Elysia) |
|------|:--------------:|:--------------:|:-------------------:|
| RPS  | -              | -              | -                   |
| P50  | -              | -              | -                   |
| P95  | -              | -              | -                   |
| P99  | -              | -              | -                   |

---

### Phase 7: 문서화 및 마무리 (11주차)

- [ ] README.md — 프로젝트 소개, 아키텍처 다이어그램, 실행 방법
- [ ] 벤치마크 결과 분석 및 인사이트 정리
- [ ] 블로그 포스팅 (Tistory / 개인 블로그)
  - ES 핵심 개념 정리
  - 프레임워크 벤치마크 결과 분석
- [ ] (선택) 포트폴리오 페이지 추가

---

## 5. API 명세 (공통)

> 3개 백엔드 모두 동일한 스펙으로 구현합니다.

| Method | Endpoint | 설명 |
|--------|----------|------|
| `GET` | `/api/products/search` | 통합 검색 (q, category, brand, minPrice, maxPrice, sort, page) |
| `GET` | `/api/products/suggest` | 자동완성 (q) |
| `GET` | `/api/products/popular` | 인기 검색어 Top 10 |
| `GET` | `/api/products/aggregations` | 카테고리/가격대 집계 |
| `POST` | `/api/admin/index/reindex` | 전체 재색인 |
| `GET` | `/api/admin/index/status` | 인덱스 상태 조회 |

---

## 6. 디렉토리 구조

```
ElasticSearch/
├── core/                        # 공유 ES 설정 및 매핑
│   ├── mappings/
│   │   ├── products-mapping.json
│   │   └── products-settings.json
│   └── seed/                   # 샘플 데이터 스크립트
│
├── backend-kotlin/              # 메인 구현 (Spring Boot 4 + Kotlin)
│   └── src/main/kotlin/com/searchhub/
│       ├── config/
│       ├── product/
│       │   ├── entity/
│       │   ├── document/
│       │   ├── repository/
│       │   ├── service/
│       │   └── controller/
│       └── search/
│           ├── dto/
│           └── service/
│
├── backend-python/              # 포팅 A (Robyn + elasticsearch-py)
│   ├── app/
│   │   ├── routers/
│   │   ├── services/
│   │   └── models/
│   └── requirements.txt
│
├── backend-ts/                  # 포팅 B (Elysia.js + Bun)
│   ├── src/
│   │   ├── routes/
│   │   ├── services/
│   │   └── types/
│   └── package.json
│
├── frontend/                    # React + Vite
│   └── src/
│       ├── components/
│       │   ├── SearchBar.tsx
│       │   ├── FilterPanel.tsx
│       │   ├── ProductCard.tsx
│       │   └── AggregationChart.tsx
│       └── pages/
│           ├── SearchPage.tsx
│           └── DashboardPage.tsx
│
├── benchmark/                   # k6 벤치마크 스크립트
│   ├── search_load.js
│   ├── suggest_load.js
│   ├── aggregation_load.js
│   └── results/
│       ├── kotlin/
│       ├── python/
│       └── typescript/
│
├── docker-compose.yml           # ES, Kibana, MySQL, Redis
└── README.md
```

---

## 7. 학습 체크리스트

### Elasticsearch 개념

- [ ] Index, Shard, Replica 개념
- [ ] Document, Field, Mapping 구조
- [ ] Inverted Index 동작 원리
- [ ] Analyzer (Tokenizer, Filter) 이해
- [ ] Query DSL 전반 (match, term, bool, range, fuzzy)
- [ ] Aggregation (bucket, metric, pipeline)
- [ ] Completion Suggester / Edge N-gram
- [ ] Alias / Reindex API
- [ ] `_explain`, `_profile` API

### Spring Data Elasticsearch (Kotlin)

- [ ] `@Document`, `@Field`, `@Mapping` 어노테이션
- [ ] `ElasticsearchRepository` vs `ElasticsearchOperations`
- [ ] `NativeQuery` 사용
- [ ] Kotlin Coroutines + ES 비동기 클라이언트

### 벤치마크

- [ ] k6 스크립트 작성법 이해
- [ ] 공정한 비교 조건 설계
- [ ] 결과 해석 및 병목 원인 분석

---

## 8. 참고 자료

| 분류 | 링크 |
|------|------|
| ES 공식 문서 | [Elasticsearch Guide 9.x](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html) |
| Spring Data ES | [Spring Data Elasticsearch Docs](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/) |
| Nori 분석기 | [Nori Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori.html) |
| Robyn | [robyn.rs](https://robyn.rs) |
| Elysia.js | [elysiajs.com](https://elysiajs.com) |
| k6 | [k6.io/docs](https://k6.io/docs) |
| 샘플 데이터 | [Kaggle E-Commerce Dataset](https://www.kaggle.com/datasets) |

---

> **시작 팁**: Phase 1에서 Kibana Dev Tools를 적극 활용하세요. 코드 작성 전에 Dev Tools에서 직접 쿼리를 실험하면 이해 속도가 훨씬 빨라집니다.
>
> **벤치마크 팁**: 성능 비교보다 "왜 이런 차이가 나는가"를 분석하는 과정이 핵심입니다. JVM 워밍업, GC, 비동기 모델 차이를 함께 기록하세요.
