# nova-bedrock-outpainting-demo
bedrock-nova-outpainting-demo

AWS Bedrock Nova 2 Omni를 사용하여 3:1 비율의 배너 이미지를 생성하는 두 가지 방법을 제공합니다.

## 📋 개요

Nova 2 Omni는 기본적으로 2880x1440 (2:1 비율) 이미지를 생성합니다. 이 프로젝트는 2:1 이미지를 3072x1024 (3:1 비율)로 변환하는 두 가지 방법을 제공합니다:

1. **Simple Stretch** (`generate_image.py`) - 좌우를 늘려서 3:1로 변환
2. **AI Outpainting** (`generate_image_novacanvas.py`) - Nova Canvas로 양옆에 새로운 내용 생성

## 🚀 빠른 시작

### 1. 환경 설정

```bash
# 가상환경 생성 및 활성화
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 패키지 설치
pip install -r requirements.txt
```

### 2. AWS 자격증명 설정

`.env` 파일 생성:
```bash
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_DEFAULT_REGION=us-east-1
```

또는 AWS CLI 설정:
```bash
aws configure
```

### 3. 이미지 생성

#### 방법 1: Simple Stretch (빠르고 간단)
```bash
python generate_image.py
```

**생성 파일:**
- `cityscape_banner_2x1_nova_omni_original.png` (2880x1440, 2:1 원본)
- `cityscape_banner_3x1_nova_omni.png` (3072x1024, 3:1 변환)

#### 방법 2: AI Outpainting (자연스러운 확장)
```bash
python generate_image_novacanvas.py
```

**생성 파일:**
- `cityscape_banner_2x1_nova_omni_original.png` (2880x1440, 2:1 원본)
- `cityscape_banner_3x1_nova_canvas_outpaint.png` (3072x1024, 3:1 확장)

## 📊 두 방법 비교

| 특징 | Simple Stretch | AI Outpainting |
|------|----------------|----------------|
| **속도** | 빠름 (1회 API 호출) | 느림 (2회 API 호출) |
| **비용** | 저렴 | 비쌈 |
| **품질** | 좌우가 약간 늘어남 | 자연스러운 확장 |
| **일관성** | 항상 동일한 결과 | 매번 다른 결과 |
| **위아래 보존** | 100% 보존 | 100% 보존 |
| **추천 용도** | 빠른 프로토타입, 테스트 | 최종 프로덕션, 고품질 |

## 🔧 작동 원리

### Method 1: Simple Stretch (`generate_image.py`)

```
1. Nova 2 Omni로 2880x1440 (2:1) 이미지 생성
2. 원본 저장
3. PIL로 3072x1024 (3:1)로 리샘플링
   - 가로: 2880 → 3072 (1.07배 늘림)
   - 세로: 1440 → 1024 (0.71배 축소)
4. 변환된 이미지 저장
```

**장점:**
- 빠른 처리 속도
- 추가 API 비용 없음
- 예측 가능한 결과

**단점:**
- 가로 방향이 약간 늘어나 보일 수 있음

### Method 2: AI Outpainting (`generate_image_novacanvas.py`)

```
1. Nova 2 Omni로 2880x1440 (2:1) 이미지 생성
2. 원본 저장
3. 이미지를 2048x1024로 스케일 (높이 맞춤)
4. 3072x1024 캔버스 생성
5. 중앙에 스케일된 이미지 배치
6. 좌우 빈 공간 (각 512px) 마스킹
7. Nova Canvas Outpainting으로 좌우 확장
   - AI가 원본 스타일 분석
   - 자연스럽게 좌우 내용 생성
8. 확장된 이미지 저장
```

**장점:**
- 자연스러운 확장
- 새로운 콘텐츠 생성
- 고품질 결과

**단점:**
- 추가 API 호출 필요
- 처리 시간 증가
- 매번 다른 결과

## 📝 프롬프트 커스터마이징

두 파일 모두 `main()` 함수의 `full_prompt` 변수를 수정하여 원하는 이미지를 생성할 수 있습니다:

```python
full_prompt = """Your custom prompt here..."""
```

**현재 프롬프트:** 미래형 도시 풍경 (석양, 사이버펑크 스타일, 캐릭터 실루엣)

## 🔐 Private VPC 환경에서 사용

Nova 2 Omni와 Nova Canvas 모두 Private VPC 환경에서 사용 가능합니다.

### VPC Endpoint 설정

```bash
# Bedrock Runtime VPC Endpoint 생성
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --service-name com.amazonaws.us-east-1.bedrock-runtime \
  --route-table-ids rtb-xxxxx
```

### 필요한 IAM 권한

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:us-east-1::foundation-model/us.amazon.nova-2-omni-v1:0",
        "arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-canvas-v1:0"
      ]
    }
  ]
}
```

**중요:** Web Grounding과 달리 이미지 생성은 외부 인터넷 접속이 필요 없으며, 모든 처리가 AWS 내부에서 이루어집니다.

## 📦 Requirements

```
boto3>=1.35.0
Pillow>=10.0.0
python-dotenv>=1.0.0
```

## 🎯 사용 사례

### Simple Stretch 추천
- 빠른 프로토타이핑
- A/B 테스트용 이미지
- 내부 검토용 자료
- 비용 절감이 중요한 경우

### AI Outpainting 추천
- 최종 프로덕션 배너
- 마케팅 캠페인 소재
- 고품질이 중요한 경우
- 창의적인 확장이 필요한 경우

## 🐛 트러블슈팅

### Nova 2 Omni Preview 권한 오류
```
Error: Access denied to model
```
**해결:** AWS 계정에 Nova 2 Omni Preview 권한이 필요합니다. AWS Support에 요청하세요.

### Outpainting 실패
```
Error: Invalid mask image
```
**해결:** 이미지 크기가 Nova Canvas 제한(최대 4096x4096)을 초과하지 않는지 확인하세요.

### 메모리 부족
```
MemoryError: Unable to allocate array
```
**해결:** 더 작은 해상도로 생성하거나 시스템 메모리를 늘리세요.

## 📄 라이선스

MIT License

## 🤝 기여

이슈와 PR을 환영합니다!

## 📧 문의

질문이나 제안사항이 있으시면 이슈를 등록해주세요.
