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

## 🔧 Configuration

환경 변수를 통해 설정을 관리합니다:

```bash
# .env.example을 .env로 복사
cp .env.example .env
```

`.env` 파일 설정:
```env
# 기존 OpenSearch 엔드포인트 (선택사항)
OPENSEARCH_ENDPOINT=https://your-endpoint.us-east-1.aoss.amazonaws.com
```

**자동 감지**: 엔드포인트를 설정하지 않으면 자동으로 활성 OpenSearch 컬렉션을 검색합니다.

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

## 🚀 Scaling

실제 운영 환경에서는:
- 실제 패션 데이터셋 연동
- 배치 처리를 통한 대량 데이터 임베딩
- 필터링 기능 추가 (가격, 브랜드, 카테고리)
- 웹 인터페이스 구축

## 📄 License

MIT License

## 🤝 Contributing

Issues와 Pull Requests를 환영합니다.
