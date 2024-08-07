사이트 개편 이후 리소스 및 비용에 대한 팔로우 중 발생한 S3 DataTransfer 비용 이슈 회고입니다.


### 문제 발생

사이트 개편 이후 일 단위로 AWS 비용을 팔로우 하던 중 DataTransfer 비용이 급증한 것을 확인하였습니다.

사이트 개편 이전 월간 10달러 정도 발생하던 것에 비해, 개편 이후 일일 100달러 이상 발생하고 있었습니다.


### 원인 분석

비용 상세 분석 결과, 개편 이후 S3 APN2-DataTransfer-Out-Bytes 비용이 급증한 것을 확인하였습니다.

대부분의 리소스를 S3에 저장하고있었으나 CloudFront를 통해 서비스하고 있었기 때문에, S3 DataTransfer 비용이 최소화 되어야 했습니다.

우선 S3 DataTransfer 비용이 급증한 경우를 다음과 같이 정리했습니다.

1. 서비스 개편으로 인한 단순 트래픽 증가
2. 광고 캠페인의 리소스를 S3에 저장하고 있어 DataTransfer 비용이 발생한 경우
3. S3에 저장된 리소스를 다른 리전의 서비스에서 사용하는 경우
4. CloudFront를 사용하지 않고 S3에 직접 접근하는 경우


### 원인 확인

서비스 개편 이후 광고 및 이벤트 캠페인이 활발하게 진행되었으나, 트래픽 증가로 인한 비용 발생이라고 보기에는 DataTransfer 비용이 너무 많이 발생하고 있었으며

CloudFront 지표를 확인한 결과 사용량이 증가하기는 했지만, S3 DataTransfer 비용이 급증한 것과는 비교할 수 없었습니다.

또한 마케팅팀에 문의한 결과, 광고 캠페인의 리소스 중 S3에 저장된 리소스가 존재하지 않았습니다.

S3 버킷의 리전을 확인한 결과, 모든 리전이 `ap-northeast-2`로 설정되어 있었으며, 다른 리전의 서비스에서 S3 리소스를 사용하는 경우도 없었습니다.

결과적으로 S3 직접 접근으로 인한 비용 증가가 예상되었으나 S3에서 트래픽에 대한 지표를 제공하지 않아 확인이 어려웠습니다.


### 원인 발견 및 해결

S3 DataTransfer 비용이 급증한 이유를 찾기 위해 AWS S3의 [사용 보고서](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/aws-usage-report-understand.html)를 확인한 결과, `region-DataTransfer-Out-Bytes` 유형의 경우 Amazon S3에서 인터넷으로 전송된 데이터의 양에 대한 비용을 청구한다는 것을 확인하였습니다.

종합적으로 S3 직접 접근으로 인한 비용 증가로 판단하고 S3에 직접 접근하는 서비스를 확인한 결과, 사이트 개편 이후 상품의 상세정보 내 이미지를 S3에 저장하고 있었으며, 일부 상품의 상세정보 내 이미지의 URL이 S3 주소로 설정되어 있었음을 확인하였습니다.

이로 인해 DataTransfer 비용 증가는 물론 캐싱이 되지 않아 성능 저하도 발생하고 있었으며, 이를 해결하기 위해 상품의 상세정보 내 이미지 URL을 CloudFront 주소로 변경하였습니다.


### 마치며

S3 DataTransfer 비용이 급증한 이유를 찾기 위해 AWS S3의 사용 보고서를 확인하였으며, S3 직접 접근으로 인한 비용 증가로 판단하였습니다.

이로 인해 DataTransfer 비용 증가는 물론 캐싱이 되지 않아 성능 저하도 발생하고 있었으며, 이를 해결하기 위해 상품의 상세정보 내 이미지 URL을 CloudFront 주소로 변경하였습니다.

월간 2000달러 이상 발생할 것으로 예상되었던 DataTransfer 비용을 최소화하고 성능을 개선할 수 있었으며, 이를 통해 서비스의 안정성을 높일 수 있었습니다.


### 참조
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/aws-usage-report-understand.html