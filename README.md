
# 🎬 하이브리드 영화 추천 시스템

## 📌 프로젝트 개요

본 프로젝트는 영화 메타데이터와 사용자 평가 데이터를 바탕으로 **콘텐츠 기반 필터링**과 **협업 필터링**을 결합한 **하이브리드 추천 시스템**을 구현하는 것을 목표로 합니다.  
영화 줄거리(LDA), 장르, 개봉 연도, 제작사, 언어 등 다양한 콘텐츠 특성과 사용자-아이템 평점 행렬을 함께 활용하여 사용자 맞춤형 추천을 제공합니다.

---

## 🧩 프로젝트 흐름

### 1. 데이터 수집 및 전처리
- `movies_metadata.csv`에서 영화 메타데이터 추출
- `ratings_small.csv`에서 사용자 평점 데이터 수집
- `links.csv`를 이용하여 영화 간 ID 연결
- 전처리 내용:
  - JSON 형식 파싱 (장르, 제작사, 컬렉션 등)
  - 개봉일(datetime), 영화 줄거리 전처리
  - 불용어, 특수문자, 숫자 제거
  - 한글/한자 제목 영화 제거 (영문만 사용)
  - 중복 줄거리 제거

---

### 2. 콘텐츠 기반 필터링

#### Feature 1: 시놉시스 (줄거리) - LDA 토픽 모델링
- Gensim의 LDA(Latent Dirichlet Allocation)를 사용하여 영화 개요에서 주제 추출
- 각 영화는 하나의 주제 벡터로 표현됨
- LDA 기반 토픽 유사도 계산 → **코사인 유사도 사용**

#### Feature 2~5: 장르, 개봉 연도, 제작사, 언어
| 항목 | 설명 | 유사도 계산 방식 |
|------|------|------------------|
| 장르 | CountVectorizer로 장르 벡터화 | 코사인 유사도 |
| 개봉 연도 | 연도 차이 반영 | `1 / (1 + 연도 차이)` |
| 제작사 | CountVectorizer로 벡터화 | 코사인 유사도 |
| 언어 | 동일 언어 여부 | 이진 유사도 (같으면 1, 다르면 0) |

#### ✅ 콘텐츠 기반 유사도 총합
최적화된 가중치(정규화 후)를 통해 각 유사도를 조합  
```text
content_similarity = 
    0.61 * LDA유사도 + 0.19 * 장르 + 0.02 * 연도 + 0.12 * 제작사 + 0.06 * 언어
```

---

### 3. 협업 필터링 (Collaborative Filtering)

- 사용자-아이템 평점 행렬 생성 (pivot table)
- 평점 시점에 따른 **시간 가중치 적용**
- TruncatedSVD를 사용하여 **잠재 요인(latent factor) 행렬** 생성
- 사용자 벡터 × 아이템 벡터의 내적으로 추천 점수 계산

---

### 4. 하이브리드 추천 시스템

- 콘텐츠 기반 추천 점수와 협업 필터링 추천 점수를 **단순 가중합 방식**으로 결합
- 최적화된 가중치:
```text
hybrid_score = 
    0.71 * 콘텐츠 기반 점수 + 0.29 * 협업 필터링 점수
```

- 추천 제외 처리:
  - 사용자가 이미 본 영화
  - 타겟 영화(입력 영화)

---

### 5. 추천 결과 예시

```python
hybrid_recommendations_with_optimized_weights(user_id=2, target_movie_title="Dumbo", top_n=5)
```

→ 사용자 ID 2가 "Dumbo"를 봤다고 했을 때, 그에 유사한 영화 5편 추천

| 제목 | 장르 | 개봉일 | 가중평균 평점 | 콘텐츠 점수 | 협업 점수 | 최종 점수 |
|------|------|--------|----------------|--------------|------------|------------|
| ...  | ...  | ...    | ...            | ...          | ...        | ...        |

---

## ⚙️ 사용된 주요 기술

- **LDA (Latent Dirichlet Allocation)**: 시놉시스를 토픽 벡터로 변환
- **SVD (TruncatedSVD)**: 협업 필터링 기반 추천 점수 계산
- **코사인 유사도**: 벡터 기반 유사도 측정
- **Hyperopt (베이지안 최적화)**: 가중치 최적화
- **Scikit-Learn, Pandas, Gensim, Numpy, Matplotlib** 등

---

## 🧠 정규화된 가중치 (최종 적용)

| 콘텐츠 기반 항목 | 가중치 |
|------------------|--------|
| LDA (시놉시스)   | 0.61   |
| 장르              | 0.19   |
| 개봉 연도         | 0.02   |
| 제작사            | 0.12   |
| 언어              | 0.06   |

| 하이브리드 결합 | 가중치 |
|------------------|--------|
| 콘텐츠 점수       | 0.71   |
| 협업 점수         | 0.29   |

---
