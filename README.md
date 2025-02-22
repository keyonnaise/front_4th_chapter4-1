## 프론트엔드 배포 파이프라인

### 개요

프론트엔드 배포 파이프라인은 소스 코드가 사용자에게 도달하기까지의 자동화된 프로세스를 관리하는 핵심 인프라입니다. 작성된 글에서는 GitHub Actions, AWS S3, CloudFront를 활용하여 신뢰성 있고 효율적인 배포 환경을 구축하는 방법을 설명합니다.

### 1. GitHub Actions 워크플로우 구성

GitHub Actions는 GitHub에서 제공하는 CI/CD 도구로, 코드 변경 사항이 발생할 때마다 자동으로 빌드, 테스트, 배포 등의 작업을 수행할 수 있습니다. 이를 통해 개발자는 코드 품질을 지속적으로 유지하며, 신뢰성 있는 배포를 수행할 수 있습니다.

![Image](https://github.com/user-attachments/assets/dae0208a-484b-42e8-b432-3c94abb91279)

#### 주요 단계 설명

1. GitHub Checkout 액션을 사용하여 소스 코드를 저장소에서 가져옵니다.
2. npm ci 명령어를 실행하여 프로젝트 의존성을 설치합니다. npm ci는 npm install과 달리 package-lock.json 파일을 기반으로 정확히 동일한 의존성을 설치하므로, 개발 환경 간 일관성을 유지할 수 있습니다.
3. npm run build 명령어를 통해 Next.js 프로젝트를 빌드합니다.
4. AWS 접근에 필요한 자격 증명을 설정합니다. 다음 값들은 GitHub Secrets에서 관리됩니다.
    1. AWS Access Key ID
    2. AWS Secret Access Key
    3. AWS Region
5. S3 배포
    1. `aws s3 sync` 명령어로 빌드된 파일들을 S3 버킷에 동기화합니다.
    2. `--delete` 옵션으로 버킷의 기존 파일들을 정리합니다.
6. CloudFront 캐시 무효화
    1. 전체 경로("/*") 대신 특정 경로만 무효화하여 비용 절감
    2. 배포 전략에 따른 단계적 무효화 고려

전체적인 과정을 도식화하면 다음과 같습니다.

![Image](https://github.com/user-attachments/assets/c6385168-140e-47cb-be1d-3e398251c970)

이러한 체계적인 빌드 과정은 개발팀이 높은 품질의 일관된 소프트웨어를 신속하게 배포할 수 있도록 지원합니다.

### 2. AWS 자격 증명 구성에 대해

클라우드 인프라스트럭처를 활용하는 개발 환경에서 AWS 자격 증명 구성은 중요한 보안 요소입니다. AWS 자격 증명은 AWS 리소스에 안전하게 접근하고 관리할 수 있는 핵심 요소로, 잘못 관리될 경우 보안 위험을 초래할 수 있습니다.

GitHub Actions와 같은 CI/CD 파이프라인에서 AWS 자격 증명을 안전하게 관리하기 위해서는 다음과 같은 접근 방법이 권장됩니다.

- GitHub Secrets 활용: GitHub 저장소의 Secrets 기능을 사용하여 AWS 액세스 키와 같은 중요한 자격 증명 정보를 안전하게 저장할 수 있습니다. 이 방식은 소스 코드에 직접 자격 증명을 포함시키지 않고 암호화된 환경 변수로 관리할 수 있게 해줍니다.
- IAM 역할 기반 접근: 최소 권한 원칙에 따라 특정 작업에 필요한 최소한의 권한만을 가진 IAM 역할을 생성하는 것이 중요합니다. 이를 통해 잠재적인 보안 위험을 크게 줄일 수 있습니다.

이러한 접근 방식을 통해 AWS 자격 증명을 안전하고 효율적으로 관리할 수 있으며, 보안성을 크게 향상시킬 수 있습니다.

### 3. S3 버킷 동기화

Amazon S3 버킷은 클라우드 환경에서 확장 가능하고 안전한 객체 스토리지 서비스입니다. 프론트엔드 배포 파이프라인에서 S3 버킷은 빌드된 정적 파일(HTML, CSS, JavaScript 등)을 저장하고 호스팅하는 컴포넌트로 작동합니다.

S3 버킷에 빌드된 파일을 업로드하는 주요 이유는 다음과 같습니다.

- 고가용성: 데이터가 여러 장소에 분산 저장되어 항상 접근 가능하도록 보장합니다.
- 글로벌 접근성: 전 세계 어디서나 빠르고 안정적으로 정적 웹사이트 자원에 접근할 수 있습니다.
- 정적 웹사이트 호스팅: S3는 정적 파일을 직접 호스팅할 수 있는 기능을 제공하여, 별도의 서버 없이도 웹사이트를 운영할 수 있습니다.

S3 버킷 동기화는 단순한 파일 업로드가 아닌닌 지속적이고 신뢰할 수 있는 웹 애플리케이션 배포 전략의 핵심 메커니즘입니다.

### 4. CloudFront와 CDN

Amazon CloudFront는 AWS의 콘텐츠 전송 네트워크(CDN) 서비스로, 전 세계의 엣지 로케이션을 통해 사용자에게 빠르고 안전하게 콘텐츠를 전송합니다. CloudFront의 주요 기능은 다음과 같습니다.

- 빠른 콘텐츠 전송: 사용자와 가장 가까운 엣지 로케이션에서 콘텐츠를 제공함으로써 로딩 시간을 최소화합니다.
- 보안: SSL/TLS 암호화를 지원하여 데이터 전송의 안전성을 높입니다.
- 스케일링: 트래픽이 급증할 경우 자동으로 스케일링하여 안정적인 서비스를 제공합니다.

### 5. 캐시 무효화(Cache Invalidation)

캐시 무효화는 CDN에서 오래된 콘텐츠를 제거하고 최신 콘텐츠를 가져오는 과정입니다. CloudFront에서 캐시 무효화는 다음과 같은 이유로 중요합니다:

- 최신 콘텐츠 제공: 웹사이트가 업데이트되었을 때, 사용자가 항상 최신 버전을 경험할 수 있도록 합니다.
- 장애 대응: 잘못된 콘텐츠가 배포되었을 경우, 빠르게 수정된 버전으로 대체할 수 있습니다.
- 비용 효율적 관리: 무효화 요청은 필요할 때만 수행하여 비용을 절감할 수 있습니다.

### 6. Repository Secret과 환경변수

GitHub Actions에서 Repository Secrets는 중요한 자격 증명(예: AWS 액세스 키)을 안전하게 저장하는 방법입니다. 이를 통해 소스 코드에 민감한 정보를 하드코딩하지 않고도 CI/CD 파이프라인에서 안전하게 사용할 수 있습니다.

- 환경변수: CI/CD 파이프라인에서 사용되는 다양한 설정을 저장하는 데 사용됩니다. 환경변수를 통해 배포 환경에 따라 다른 설정을 쉽게 적용할 수 있습니다.
- 보안: Repository Secrets는 암호화되어 저장되며, 워크플로우에서만 접근 가능하여 보안성을 높입니다.

### 7. 결론

효율적인 프론트엔드 배포 파이프라인은 최적화된 빌드 프로세스 그리고 안전한 배포 전략의 조화로 구성됩니다. S3와 CloudFront를 통한 효율적인 스토리지 및 콘텐츠 전송, 캐시 무효화를 통한 신속한 업데이트, Repository Secrets와 환경변수를 통한 보안 관리가 결합되어 안정적이고 효율적인 배포 환경을 구축할 수 있습니다.

---

## CDN을 통한 성능 개선

### 개요

콘텐츠 전송 네트워크(CDN)는 웹 애플리케이션의 성능을 최적화하기 위한 필수적인 인프라 요소입니다. 본 문서에서는 CloudFront를 활용한 구체적인 성능 개선 사례와 최적화 전략을 다룹니다.

### CDN 도입이 웹 성능 개선에 미치는 영향

CDN 도입은 웹 성능 개선에 획기적인 영향을 미칩니다. 가장 큰 이점은 사용자와 가까운 서버에서 콘텐츠를 제공함으로써 네트워크 지연 시간을 최소화할 수 있다는 점입니다. 이는 웹사이트의 로딩 속도를 크게 향상시켜 사용자 만족도를 높이고 이탈률을 감소시키는 효과가 있습니다.

또한 CDN은 원본 서버의 트래픽을 여러 서버로 분산시키는 역할을 합니다. 이러한 부하 분산을 통해 서버의 안정성이 높아지고 전체적인 웹사이트의 가용성이 향상됩니다. 특히 트래픽이 갑자기 증가하는 상황에서도 안정적인 서비스 제공이 가능합니다.

CDN의 효율적인 캐싱 시스템도 주목할 만한 장점입니다. 정적 콘텐츠뿐만 아니라 동적 콘텐츠까지 캐싱하여 반복되는 요청에 대한 응답 속도를 크게 개선합니다. 이는 서버의 응답 시간을 단축시키고 전체적인 시스템 성능을 향상시키는 데 핵심적인 역할을 합니다.

### CDN 도입 전후 성능 비교

![Image](https://github.com/user-attachments/assets/1b8f7c1c-ba68-457f-a2ac-5136d589e3ba)

CDN을 사용하지 않았을 때

![Image](https://github.com/user-attachments/assets/8caa7516-a22f-4494-8716-5b820760356d)

CDN을 사용했을 때

본 사례에서 CDN 도입 효과는 드라마틱 했습니다. TTFB(Time To First Byte)는 206.93ms에서 6.72ms로 97% 감소했으며, 전체 페이지 로딩 시간은 212.71ms에서 10.30ms로 95% 개선되었습니다. 이는 CDN의 효과적인 콘텐츠 전송 및 캐싱 메커니즘을 잘 보여줍니다.

### CDN을 도입 했을 때 주의할 점

CDN의 도입은 웹 성능을 극대화하는 중요한 전략입니다. 하지만 효과적으로 운영하기 위해서는 몇 가지 주의해야 할 사항이 있습니다.

1. 캐시 관리:
먼저, 캐시 관리는 CloudFront의 핵심 기능 중 하나입니다. CloudFront에서는 다양한 캐시 정책을 설정하여 콘텐츠의 종류에 따라 적절한 캐시 TTL(유효 기간)을 설정할 수 있습니다. 예를 들어, 정적 이미지나 CSS 파일과 같은 정적 콘텐츠는 긴 TTL을 설정하여 성능을 극대화하고, 동적 웹 페이지나 사용자 맞춤형 콘텐츠는 짧은 TTL을 설정해 최신 정보를 제공할 수 있도록 합니다. 또한, CloudFront의 Invalidation 기능을 활용하여 콘텐츠가 업데이트될 때 기존 캐시를 신속하게 무효화할 수 있습니다. 이를 통해 사용자에게 항상 최신의 콘텐츠를 제공할 수 있습니다.

2. 보안 설정:
보안은 CloudFront의 또 다른 중요한 측면입니다. AWS는 CloudFront에 다양한 보안 기능을 제공하여 데이터 보호를 강화할 수 있습니다. 예를 들어, AWS WAF(Web Application Firewall)를 사용하여 DDoS 공격과 같은 위협으로부터 웹 애플리케이션을 보호할 수 있습니다. 또한, CloudFront의 Signed URLs 및 Signed Cookies 기능을 통해 특정 콘텐츠에 대한 접근을 제어하여 민감한 데이터에 대한 보안을 강화할 수 있습니다.

그 외에도 다양한 지역 고려, 비용 관리, 모니터링 및 성능 분석 등이 중요한 주의 사항입니다. 이러한 요소들을 고려하여 CDN을 도입하면 성능뿐만 아니라 안정성과 보안성도 높일 수 있습니다.

### 결론

CDN 도입을 통해 웹 페이지의 성능이 크게 향상되었으며, TTFB 속도가 206.93ms에서 6.72ms로 개선되었습니다. 이는 사용자 경험을 극대화하고 비즈니스 성과에 긍정적인 영향을 미칩니다. 앞으로는 지속적인 CDN 최적화와 AI 기반 모니터링을 통해 성능을 더욱 향상시킬 필요가 있습니다.
