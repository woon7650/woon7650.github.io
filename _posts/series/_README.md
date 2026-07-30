# 대용량 트래픽 시리즈 — 포스트 스켈레톤 34건

## 배치 방법

이 `series` 폴더를 아래 위치에 그대로 넣으십시오.

```
C:\Users\User\gitBlog\_posts\series\
```

`_posts` 안쪽 서브폴더는 카테고리와 URL에 영향을 주지 않습니다.
카테고리는 `_posts` 위쪽 디렉터리에서만 파생되기 때문입니다.

## 파일 목록

| PART | 날짜 | 파일명 |
|---|---|---|
| 1-1 | 2026-07-30 | `2026-07-30-index-execution-plan.md` |
| 1-2 | 2026-07-31 | `2026-07-31-transaction-mvcc.md` |
| 1-3 | 2026-08-01 | `2026-08-01-innodb-lock-mechanism.md` |
| 1-4 | 2026-08-02 | `2026-08-02-concurrency-control-strategy.md` |
| 1-5 | 2026-08-03 | `2026-08-03-jpa-n-plus-one.md` |
| 2-1 | 2026-08-04 | `2026-08-04-concurrency-model.md` |
| 2-2 | 2026-08-05 | `2026-08-05-pool-and-timeout-layers.md` |
| 2-3 | 2026-08-06 | `2026-08-06-jvm-gc-tuning.md` |
| 3-1 | 2026-08-07 | `2026-08-07-cap-pacelc.md` |
| 3-2 | 2026-08-08 | `2026-08-08-distributed-transaction.md` |
| 3-3 | 2026-08-09 | `2026-08-09-idempotency-design.md` |
| 3-4 | 2026-08-10 | `2026-08-10-distributed-lock-limits.md` |
| 4-1 | 2026-08-11 | `2026-08-11-redis-internals-operations.md` |
| 4-2 | 2026-08-12 | `2026-08-12-caching-strategy.md` |
| 4-3 | 2026-08-13 | `2026-08-13-kafka-architecture.md` |
| 4-4 | 2026-08-14 | `2026-08-14-kafka-reliability-patterns.md` |
| 5-1 | 2026-08-15 | `2026-08-15-load-balancing-api-gateway.md` |
| 5-2 | 2026-08-16 | `2026-08-16-service-discovery-http-client.md` |
| 5-3 | 2026-08-17 | `2026-08-17-circuit-breaker-rate-limiting.md` |
| 5-4 | 2026-08-18 | `2026-08-18-distributed-tracing.md` |
| 6-1 | 2026-08-19 | `2026-08-19-api-design-style.md` |
| 6-2 | 2026-08-20 | `2026-08-20-authentication-authorization.md` |
| 6-3 | 2026-08-21 | `2026-08-21-realtime-communication.md` |
| 7-1 | 2026-08-22 | `2026-08-22-replication-read-write-split.md` |
| 7-2 | 2026-08-23 | `2026-08-23-sharding-distributed-id.md` |
| 7-3 | 2026-08-24 | `2026-08-24-consistent-hashing.md` |
| 7-4 | 2026-08-25 | `2026-08-25-nosql-data-modeling.md` |
| 7-5 | 2026-08-26 | `2026-08-26-search-infrastructure.md` |
| 8-1 | 2026-08-27 | `2026-08-27-cdn-static-assets.md` |
| 8-2 | 2026-08-28 | `2026-08-28-load-testing.md` |
| 8-3 | 2026-08-29 | `2026-08-29-observability.md` |
| 8-4 | 2026-08-30 | `2026-08-30-slo-error-budget.md` |
| 8-5 | 2026-08-31 | `2026-08-31-deployment-strategy.md` |
| 8-6 | 2026-09-01 | `2026-09-01-auto-scaling.md` |

## 각 파일에 들어있는 것

- `title` — `[PART 1-1]` 형태. **파일명·URL에는 번호 없음**
- `date`, `last_modified_at` — 하루 1건 배정
- `tags` — 주제별로 미리 채움
- `published: false` — **발행 시 이 줄을 삭제**
- 검증 환경 블록 — 버전은 `(버전)` 플레이스홀더
- `{% include series-nav.html %}` — 본문 맨 아래

## 들어있지 않은 것 (`_config.yml` defaults가 처리)

`layout`, `author_profile`, `categories`, `toc`, `toc_sticky`, `toc_label`, `sidebar`

> `_config.yml`에 series 전용 defaults 블록을 넣기 **전에는 목차가 나오지 않습니다.**
> 설정을 먼저 적용하고 확인하십시오.

## 남은 작업

1. Intro 파일을 `_posts/series/`로 이동 (파일명 날짜만 변경, URL은 유지됨)
2. Intro의 front matter `date:` 수정 — front matter가 파일명보다 우선
3. Intro 본문 맨 아래에 `{% include series-nav.html %}` 추가
4. Intro 로드맵을 34개 / PART 표기로 교체
5. `_data/navigation.yml`에 `massive_traffic` 키 추가
6. `_config.yml`의 `defaults`에 series 블록 추가
7. `_includes/series-nav.html` 생성
8. `jekyll serve` 재시작 — `_config.yml`은 자동 반영되지 않음
