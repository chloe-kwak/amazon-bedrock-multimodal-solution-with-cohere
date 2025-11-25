# 🎨 Multimodal Fashion Search with Cohere Embed v4

Amazon Bedrock의 Cohere embed-v4 모델과 OpenSearch Serverless를 활용한 멀티모달 패션 검색 시스템

## 🚀 Quick Start

### Prerequisites
- AWS 계정 및 적절한 권한 (Bedrock, OpenSearch Serverless)
- Python 3.8+
- Jupyter Notebook 환경

### Installation
```bash
pip install boto3>=1.34.0 opensearch-py>=2.4.0 requests-aws4auth>=1.2.3 pandas>=2.0.0 Pillow>=10.0.0 python-dotenv>=1.0.0
```

### Configuration
1. `.env.example`을 `.env`로 복사
2. 기존 OpenSearch 엔드포인트가 있으면 `.env`에 설정:
   ```
   OPENSEARCH_ENDPOINT=https://your-endpoint.us-east-1.aoss.amazonaws.com
   ```
3. 없으면 비워두면 자동으로 활성 컬렉션을 검색합니다

### Usage
1. `multimodal_solution.ipynb` 노트북 실행
2. AWS 자격증명 설정 확인
3. 셀 순서대로 실행 - OpenSearch 컬렉션이 자동으로 감지됩니다

## 🎯 Features

- **자동 인프라 생성**: OpenSearch Serverless 컬렉션 자동 생성/감지
- **멀티모달 임베딩**: 텍스트와 이미지를 1536차원 벡터로 통합
- **크로스 모달 검색**: 텍스트로 이미지 검색, 이미지로 텍스트 검색
- **하이브리드 검색**: 텍스트 + 이미지 조합 검색
- **실시간 벡터 검색**: HNSW 알고리즘 기반 고성능 검색
- **3가지 배치 처리 방식**: 순차, 동기 배치, 비동기 배치 지원

## 🛠 Tech Stack

- **Embedding Model**: Cohere embed-v4 (Amazon Bedrock)
- **Vector Database**: Amazon OpenSearch Serverless
- **Search Algorithm**: HNSW + Cosine Similarity
- **Image Processing**: PIL/Pillow
- **AWS SDK**: Boto3

## 📊 Architecture

```
텍스트 + 이미지 → Cohere embed-v4 → 1536D 벡터 → OpenSearch Serverless → 검색 결과
```

## 📚 Documentation

### 노트북

- **[multimodal_solution.ipynb](multimodal_solution.ipynb)**: Cohere Embed v4 구현 (권장)
  - 섹션 1-6: 환경 설정 및 인덱스 생성
  - 섹션 7: 순차 처리 (1개씩)
  - 섹션 7-2: 다중 입력 처리 (여러 개를 한 API 호출에 포함) ⭐
  - 섹션 7-2-1: 대용량 최적화 (큰 그룹 크기)
  - 섹션 7-2-2: 병렬 처리 (멀티스레드)
  - 섹션 8-9: 검색 기능

- **[multimodal_solution_nova.ipynb](multimodal_solution_nova.ipynb)**: Amazon Nova 구현 (참고용)
  - Nova 1024 차원 인덱스 생성
  - Nova 형식 입력 데이터 준비
  - Bedrock Batch Inference 사용
  - 대량 데이터 처리 (> 10,000개)

### 가이드

- **[QUICK_START.md](QUICK_START.md)**: 빠른 시작 가이드
- **[BATCH_PROCESSING_GUIDE.md](BATCH_PROCESSING_GUIDE.md)**: 배치 처리 상세 가이드
- **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)**: 문제 해결 가이드

## 📝 Sample Data

노트북에는 5개의 샘플 패션 아이템이 포함되어 있습니다:
- 블루 데님 재킷
- 블랙 이브닝 드레스  
- 화이트 스니커즈
- 레드 울 스웨터
- 브라운 가죽 핸드백

## 🔍 Search Types

### 1. 텍스트 검색
```python
results = search(query_text="블루 캐주얼 재킷")
```

### 2. 이미지 검색
```python
results = search(query_image_path="data/images/item_001.jpg")
```

### 3. 멀티모달 검색
```python
results = search(
    query_text="우아한 드레스",
    query_image_path="data/images/item_002.jpg"
)
```

## ⚡ Batch Processing

이 프로젝트는 3가지 배치 처리 방식을 제공합니다:

### 1. 순차 처리 (섹션 7)
```python
# 아이템을 하나씩 처리
for item in sample_items:
    embedding = generate_embedding(text, image_base64)
    time.sleep(2)  # API 제한 회피
```
- **적합한 경우**: 소량 데이터 (< 10개), 실시간 처리
- **처리 시간**: 아이템당 2초 이상
- **API 호출**: 아이템 수만큼

### 2. 다중 입력 처리 (섹션 7-2) ⭐ 권장
```python
# 한 번의 API 호출에 여러 입력 포함
payload = {
    "inputs": [
        {"content": [...]},  # 입력 1
        {"content": [...]}   # 입력 2
    ]
}
embeddings = generate_batch_embeddings(batch_data)
```
- **방식**: Cohere API의 다중 입력 기능 활용 (Bedrock Batch API 아님)
- **적합한 경우**: 중량 데이터 (10-1000개), 즉시 결과 필요
- **처리 시간**: API 호출당 1-2초
- **성능 향상**: 최대 90% 시간 단축
- **API 호출**: 그룹 수만큼 (5개 그룹 → 5회 호출, 각 호출에 여러 입력)

### 3. Bedrock Batch Inference (섹션 7-3) ⚠️ Cohere Embed v4 미지원
```python
# AWS Bedrock의 공식 배치 API (S3 기반)
bedrock.create_model_invocation_job(
    inputDataConfig={'s3InputDataConfig': {'s3Uri': input_s3_uri}},
    outputDataConfig={'s3OutputDataConfig': {'s3Uri': output_s3_uri}}
)
```
- **방식**: AWS Bedrock의 공식 Batch Inference API
- **⚠️ 중요**: Cohere Embed v4는 이 기능을 지원하지 않음
- **대안 1**: 섹션 7-2 (다중 입력 처리) 사용 - Cohere 유지
- **대안 2**: Amazon Nova Multimodal Embeddings로 모델 변경 ⭐ 신규
- **지원 모델**: Amazon Nova, Titan Embeddings, Claude, Jurassic 등
- **적합한 경우**: 대량 데이터 (> 10,000개), 비용 민감 (지원 모델 사용 시)
- **처리 시간**: 수분~수시간 (비동기)
- **비용 절감**: 최대 50% 할인
- **추가 설정**: S3 버킷, IAM 역할 필요

## 🆕 Amazon Nova Multimodal Embeddings

**배치 추론을 지원하는 차세대 멀티모달 임베딩 모델**

### Nova vs Cohere 비교

| 특징 | Cohere Embed v4 | Amazon Nova |
|------|----------------|-------------|
| 임베딩 차원 | 1536 (고정) | 768, 1024, 3072 (선택) |
| 배치 추론 | ❌ 미지원 | ✅ 지원 (비동기) |
| 언어 지원 | 100+ | 200+ (한국어 포함) |
| 컨텍스트 길이 | 512 토큰 | 8K 토큰 |
| 멀티모달 | 텍스트+이미지 | 텍스트+이미지+비디오+오디오 |
| 모델 ID | `cohere.embed-v4:0` | `amazon.nova-2-multimodal-embeddings-v1:0` |
| 세그먼테이션 | ❌ | ✅ (비디오/오디오 자동 분할) |
| API 형식 | Cohere 형식 | Nova 전용 형식 |
| OpenSearch 호환 | ✅ 기존 인덱스 사용 | ⚠️ 새 인덱스 필요 (차원 다름) |

### Nova 사용 시 장점 (텍스트+이미지)

- ✅ **Bedrock Batch Inference 지원** - 대량 데이터 처리 시 비동기 배치 가능
- ✅ **더 긴 컨텍스트** - 8K 토큰 (Cohere의 16배)
- ✅ **더 많은 언어** - 200개 언어 지원 (한국어 포함)
- ✅ **유연한 차원 선택** - 768/1024/3072 중 선택
- ✅ **OpenSearch 호환** - 벡터 임베딩을 OpenSearch에 저장 가능

### Nova 추가 기능 (선택사항)

- 📹 **비디오/오디오 임베딩** - 멀티미디어 콘텐츠 지원
- ✂️ **자동 세그먼테이션** - 긴 비디오/오디오 자동 분할 (15초 단위 등)

### 모델 변경 방법

Nova는 Cohere와 API 형식이 다르므로 몇 가지 수정이 필요합니다:

#### 1. 배치 작업 생성 시 (섹션 7-3)

```python
# 모델 ID 변경 (이미 적용됨)
modelId = 'amazon.nova-2-multimodal-embeddings-v1:0'
```

#### 2. 임베딩 차원 선택 및 OpenSearch 인덱스 생성

⚠️ **중요**: Nova는 Cohere와 다른 차원을 사용하므로 **새 인덱스**를 만들어야 합니다!

Nova는 여러 차원을 지원합니다:
- 768: 빠른 검색, 적은 메모리
- 1024: 균형잡힌 성능, Cohere와 유사
- 3072: 최고 정확도 (기본값)

**옵션 A: 1024 차원 사용 (권장 - Cohere와 유사)**
```python
# Nova 1024 차원 사용
INDEX_NAME_NOVA = 'fashion-items-nova-1024'

mapping = {
    "settings": {"index": {"knn": True}},
    "mappings": {
        "properties": {
            # ... 기존 필드들 ...
            "multimodal_embedding": {
                "type": "knn_vector",
                "dimension": 1024,  # Nova 1024 차원
                "method": {"name": "hnsw", "space_type": "cosinesimil", "engine": "nmslib"}
            }
        }
    }
}

client.indices.create(index=INDEX_NAME_NOVA, body=mapping)
```

**옵션 B: 3072 차원 사용 (최고 정확도)**
```python
# Nova 3072 차원 사용
INDEX_NAME_NOVA = 'fashion-items-nova-3072'

mapping = {
    "settings": {"index": {"knn": True}},
    "mappings": {
        "properties": {
            # ... 기존 필드들 ...
            "multimodal_embedding": {
                "type": "knn_vector",
                "dimension": 3072,  # Nova 기본 차원
                "method": {"name": "hnsw", "space_type": "cosinesimil", "engine": "nmslib"}
            }
        }
    }
}

client.indices.create(index=INDEX_NAME_NOVA, body=mapping)
```

**기존 Cohere 인덱스 유지:**
```python
# Cohere 1536 차원 (기존)
INDEX_NAME_COHERE = 'fashion-items-1'  # 기존 인덱스 유지
```

이렇게 하면 두 모델을 동시에 사용하고 비교할 수 있습니다!

#### 3. 입력 데이터 형식 변경

```python
# Cohere 형식 (기존)
{
    "recordId": "item_001",
    "modelInput": {
        "input_type": "search_document",
        "embedding_types": ["float"],
        "inputs": [{"content": [...]}]
    }
}

# Nova 형식 (변경 필요)
{
    "recordId": "item_001",
    "modelInput": {
        "taskType": "SINGLE_EMBEDDING",
        "singleEmbeddingParams": {
            "embeddingPurpose": "GENERIC_INDEX",
            "embeddingDimension": 1536,
            "text": {"truncationMode": "END", "value": "..."},
            "image": {"format": "jpeg", "source": {"bytes": "base64..."}}
        }
    }
}
```

#### 4. prepare_batch_input_data 함수 수정

노트북의 `prepare_batch_input_data` 함수를 Nova 형식으로 수정:

```python
def prepare_batch_input_data_nova(items, embedding_dimension=3072):
    """
    Amazon Nova용 배치 입력 데이터 준비
    텍스트+이미지 임베딩을 위한 형식
    """
    jsonl_lines = []
    for item in items:
        text = f"Title: {item['title']}\nDescription: {item['description']}\nCategory: {item['category']}\nBrand: {item['brand']}\nColor: {item['color']}\nPrice: ${item['price']}"
        image_base64 = encode_image(f"data/images/{item['item_id']}.jpg")
        
        record = {
            "recordId": item['item_id'],
            "modelInput": {
                "taskType": "SINGLE_EMBEDDING",
                "singleEmbeddingParams": {
                    "embeddingPurpose": "GENERIC_INDEX",
                    "embeddingDimension": embedding_dimension,  # 768, 1024, 3072 중 선택
                    "text": {"truncationMode": "END", "value": text},
                    "image": {"format": "jpeg", "source": {"bytes": image_base64}}
                }
            }
        }
        jsonl_lines.append(json.dumps(record))
    return '\n'.join(jsonl_lines)
```

#### 5. 배치 결과를 OpenSearch에 인덱싱

배치 작업 완료 후 S3에서 결과를 가져와 OpenSearch에 저장:

```python
# S3에서 배치 결과 다운로드
response = s3_client.get_object(Bucket=bucket, Key=output_key)
results = [json.loads(line) for line in response['Body'].read().decode('utf-8').strip().split('\n')]

# OpenSearch에 벌크 인덱싱
bulk_body = []
for result in results:
    record_id = result['recordId']
    embedding = result['embeddings'][0]['embedding']  # Nova 출력 형식
    
    # 원본 아이템 찾기
    item = next((i for i in sample_items if i['item_id'] == record_id), None)
    
    if item:
        bulk_body.append({"index": {"_index": INDEX_NAME}})
        bulk_body.append({
            **item,
            'multimodal_embedding': embedding,
            'created_at': datetime.now().isoformat()
        })

# OpenSearch에 업로드
client.bulk(body=bulk_body)
```

### 참고 자료

- [Amazon Nova 공식 블로그](https://aws.amazon.com/ko/blogs/aws/amazon-nova-multimodal-embeddings-now-available-in-amazon-bedrock/)
- [Bedrock Batch Inference 문서](https://docs.aws.amazon.com/bedrock/latest/userguide/batch-inference.html)

### 배치 처리 비교

| 방식 | 데이터 규모 | 처리 시간 | 비용 | 즉시 응답 | Cohere Embed v4 |
|------|------------|----------|------|----------|-----------------|
| 순차 처리 | < 10개 | 느림 | 표준 | ✅ | ✅ |
| 다중 입력 처리 | 10-10000개 | 빠름 | 표준 | ✅ | ✅ 권장 |
| Bedrock Batch API | > 1000개 | 매우 빠름 | 50% 할인 | ❌ | ❌ 미지원 |

자세한 내용은 [BATCH_PROCESSING_GUIDE.md](BATCH_PROCESSING_GUIDE.md)를 참고하세요.

## 🔧 Environment Variables

`.env` 파일에서 다음 환경 변수를 설정할 수 있습니다:

```env
# OpenSearch 엔드포인트 (선택사항 - 자동 감지 가능)
OPENSEARCH_ENDPOINT=https://your-endpoint.us-east-1.aoss.amazonaws.com

# 비동기 배치 처리용 (섹션 7-3 사용 시)
S3_BUCKET_NAME=my-bedrock-batch-bucket
BEDROCK_BATCH_ROLE_ARN=arn:aws:iam::123456789012:role/BedrockBatchRole
```

## 🚀 Scaling

실제 운영 환경에서는:
- 실제 패션 데이터셋 연동
- 데이터 규모에 맞는 배치 처리 방식 선택
- 필터링 기능 추가 (가격, 브랜드, 카테고리)
- 웹 인터페이스 구축
- 캐싱 및 성능 최적화

## 📄 License

MIT License

## 🤝 Contributing

Issues와 Pull Requests를 환영합니다.
