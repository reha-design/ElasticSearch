# 🔍 ElasticSearch 학습 사이드 프로젝트 계획서

> **프로젝트명**: SearchHub — 실시간 통합 검색 플랫폼  
> **목적**: Elasticsearch 핵심 개념 학습 및 실무 적용 능력 함양  
> **기간**: 약 8주  
> **작성일**: 2026-04-03

---

## 1. 프로젝트 개요

### 1.1 프로젝트 소개

**SearchHub**는 상품, 게시글, 사용자 등 다양한 도메인 데이터를 Elasticsearch로 색인하고, 복잡한 검색 쿼리, 필터, 집계, 자동완성 등 Elasticsearch의 핵심 기능을 직접 구현하며 학습하는 풀스택 사이드 프로젝트입니다.

### 1.2 핵심 학습 목표

| # | 학습 주제 | 내용 |
|---|-----------|------|
| 1 | **인덱스 설계** | Mapping, Analyzer, Tokenizer 설정 |
| 2 | **검색 쿼리** | match, bool, filter, term, range, fuzzy |
| 3 | **집계(Aggregation)** | terms, histogram, date_histogram, avg, sum |
| 4 | **자동완성** | completion suggester, edge n-gram analyzer |
| 5 | **데이터 동기화** | RDB → Elasticsearch 동기화 전략 |
| 6 | **성능 최적화** | Pagination, Caching, Index 전략 |
| 7 | **운영/모니터링** | Kibana Dashboard, 클러스터 상태 관리 |

---

## 2. 기술 스택

### 2.1 Backend

```
Language   : Java 17
Framework  : Spring Boot 3.x
ORM        : Spring Data JPA + QueryDSL
ES Client  : Spring Data Elasticsearch (Elasticsearch 8.x)
DB         : MySQL 8.0 (원천 데이터)
Cache      : Redis (검색 결과 캐싱)
```

### 2.2 Elasticsearch 구성

```
Elasticsearch : 8.x (Docker)
Kibana        : 8.x (모니터링 및 Dev Tools)
Logstash      : RDB → ES 파이프라인 (선택)
```

### 2.3 Frontend

```
Framework  : React + Vite
UI Library : shadcn/ui
검색 UX    : 실시간 자동완성, 패싯 필터, 하이라이팅
```

### 2.4 인프라

```
Docker Compose : ES, Kibana, MySQL, Redis, App
```

---

## 3. 도메인 설계

### 3.1 핵심 도메인

프로젝트는 **전자상거래 검색 플랫폼**을 모티브로 합니다.

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
// products 인덱스 Mapping 설계 예시
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

- [ ] Docker Compose로 ES + Kibana 구동
- [ ] Spring Boot 프로젝트 생성 및 `spring-data-elasticsearch` 연동
- [ ] MySQL에 샘플 상품 데이터 삽입 (Faker or 오픈 데이터셋)
- [ ] **인덱스 생성**: Mapping 수동 설계 (`@Mapping`, `@Setting` 어노테이션)
- [ ] **한국어 Analyzer 설정**: Nori 플러그인 설치 및 커스텀 Analyzer 구성
- [ ] **데이터 벌크 색인**: JPA → ES Repository 동기화 배치 구현
- [ ] Kibana Dev Tools로 기본 쿼리 실습

**핵심 학습 포인트**
```
✅ Index vs Type 개념
✅ Inverted Index 동작 원리
✅ Analyzer = CharFilter + Tokenizer + TokenFilter
✅ Nori 한국어 형태소 분석기
```

---

### Phase 2: 검색 기능 구현 (3~4주)

- [ ] **기본 검색 API** (`/api/search?q=키워드`)
  - `multi_match` 쿼리 (name, description, tags 멀티필드)
  - `bool` 쿼리로 must / filter / should 조합
- [ ] **패싯 필터** (카테고리, 브랜드, 가격대, 평점)
  - `term`, `range` 필터 적용
- [ ] **정렬** (관련도순, 최신순, 가격순, 평점순)
- [ ] **페이지네이션** (from/size vs search_after)
- [ ] **검색 결과 하이라이팅** (`highlight` 옵션)
- [ ] **오타 교정(Fuzzy 검색)** (`fuzziness` 파라미터)

**핵심 학습 포인트**
```
✅ 쿼리 컨텍스트 vs 필터 컨텍스트 (스코어링 차이)
✅ TF-IDF / BM25 relevance scoring
✅ from/size의 Deep Pagination 성능 문제
✅ search_after를 이용한 안전한 페이지네이션
```

---

### Phase 3: 자동완성 및 추천 (5주차)

- [ ] **자동완성 API** (`/api/suggest?q=삼성`)
  - `completion` Suggester 구현
  - `edge_ngram` Analyzer 기반 구현 (비교 실습)
- [ ] **연관 검색어 추천** (검색 로그 기반 간단 구현)
- [ ] **인기 검색어 Top 10** (Redis Sorted Set 활용)

**핵심 학습 포인트**
```
✅ Completion Suggester vs Edge N-gram 방식 차이
✅ 검색어 저장 전략 (Redis vs ES)
```

---

### Phase 4: 집계(Aggregation) 및 분석 (6주차)

- [ ] **카테고리별 상품 수** (`terms` aggregation)
- [ ] **가격대별 분포** (`histogram` aggregation)
- [ ] **월별 신규 상품** (`date_histogram` aggregation)
- [ ] **평균 평점 / 최대·최소 가격** (`avg`, `min`, `max` aggregation)
- [ ] **Kibana 대시보드** 구성: 집계 결과 시각화

**핵심 학습 포인트**
```
✅ Bucket / Metric / Pipeline Aggregation 분류
✅ Sub-aggregation (중첩 집계)
✅ Kibana Index Pattern 연동
```

---

### Phase 5: 데이터 동기화 전략 (7주차)

- [ ] **이벤트 기반 동기화**: Spring ApplicationEvent → ES 색인
- [ ] **배치 재색인**: 전체 재색인 배치 (zero-downtime 전략)
  - Alias 활용: `products_v1` → `products` alias
- [ ] **Logstash JDBC Input** 실습 (선택)

**핵심 학습 포인트**
```
✅ Index Alias를 이용한 무중단 재색인
✅ RDB와 ES 간 Eventually Consistent 동기화
✅ Bulk API 성능 최적화 (배치 사이즈)
```

---

### Phase 6: 성능 최적화 및 마무리 (8주차)

- [ ] **Redis 캐싱**: 자주 검색되는 쿼리 결과 캐싱
- [ ] **쿼리 프로파일링**: `_profile` API로 쿼리 성능 분석
- [ ] **샤드/레플리카 설정** 이해 및 실습
- [ ] **프로젝트 문서화**: README, API 명세, 아키텍처 다이어그램
- [ ] (선택) 포트폴리오 페이지 추가

---

## 5. API 명세 (주요)

| Method | Endpoint | 설명 |
|--------|----------|------|
| `GET` | `/api/products/search` | 통합 검색 (q, category, brand, minPrice, maxPrice, sort, page) |
| `GET` | `/api/products/suggest` | 자동완성 (q) |
| `GET` | `/api/products/popular` | 인기 검색어 Top 10 |
| `GET` | `/api/products/aggregations` | 카테고리/가격대 집계 |
| `POST` | `/api/admin/index/reindex` | 전체 재색인 (관리자) |
| `GET` | `/api/admin/index/status` | 인덱스 상태 조회 |

---

## 6. 디렉토리 구조

```
ElasticSearch/
├── backend/
│   ├── src/main/java/com/searchhub/
│   │   ├── config/
│   │   │   └── ElasticsearchConfig.java
│   │   ├── product/
│   │   │   ├── entity/          # JPA Entity
│   │   │   ├── document/        # ES Document
│   │   │   ├── repository/      # JPA + ES Repository
│   │   │   ├── service/
│   │   │   └── controller/
│   │   ├── search/
│   │   │   ├── dto/             # 검색 요청/응답 DTO
│   │   │   └── service/         # 복합 검색 로직
│   │   └── admin/
│   │       └── IndexController.java
│   └── src/main/resources/
│       ├── elasticsearch/
│       │   ├── products-mapping.json
│       │   └── products-settings.json
│       └── application.yml
├── frontend/
│   └── src/
│       ├── components/
│       │   ├── SearchBar.jsx     # 자동완성 포함
│       │   ├── FilterPanel.jsx   # 패싯 필터
│       │   ├── ProductCard.jsx   # 하이라이팅 표시
│       │   └── AggregationChart.jsx
│       └── pages/
│           ├── SearchPage.jsx
│           └── DashboardPage.jsx
├── docker-compose.yml
└── README.md
```

---

## 7. 학습 체크리스트

### Elasticsearch 개념

- [ ] Index, Shard, Replica 개념
- [ ] Document, Field, Mapping 구조
- [ ] Inverted Index 동작 원리
- [ ] Analyzer (Tokenizer, Filter) 이해
- [ ] Query DSL 전반 (match, term, bool, range, fuzzy, wildcard)
- [ ] Aggregation (bucket, metric, pipeline)
- [ ] Completion Suggester / Edge N-gram
- [ ] Alias / Reindex API
- [ ] `_explain`, `_profile` API

### Spring Data Elasticsearch

- [ ] `@Document`, `@Field`, `@Mapping` 어노테이션
- [ ] `ElasticsearchRepository` vs `ElasticsearchOperations`
- [ ] `NativeQuery` / `CriteriaQuery` 비교
- [ ] Index 생성 / 매핑 자동화

---

## 8. 참고 자료

| 분류 | 링크 |
|------|------|
| 공식 문서 | [Elasticsearch Guide 8.x](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html) |
| Spring 연동 | [Spring Data Elasticsearch Docs](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/) |
| 한국어 분석기 | [Nori Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori.html) |
| 샘플 데이터 | [Kaggle E-Commerce Dataset](https://www.kaggle.com/datasets) |
| 강의 참고 | 인프런 — "엘라스틱서치 완전 정복" |

---

> **시작 팁**: Phase 1에서 Kibana Dev Tools를 적극 활용하세요. 코드 작성 전에 Dev Tools에서 직접 쿼리를 실험하면 이해 속도가 훨씬 빨라집니다.
> 
> 각 Phase 완료 후 해당 기능의 블로그 포스팅 또는 README 업데이트를 병행하면 포트폴리오 퀄리티가 높아집니다.
