References
- openssl: https://gist.github.com/cecilemuller/9492b848eb8fe46d462abeb26656c4f8
- subnet tag: https://repost.aws/ko/knowledge-center/eks-vpc-subnet-discovery

```
kubernetes.io/cluster/cluster-name
shared
```
```
kubernetes.io/role/elb
1
```
```
kubernetes.io/role/internal-elb
1
```

# localhost 도메인에 대한 HTTPS 인증서를 만드는 방법 

이는 개발 전용으로 컴퓨터에서 호스팅되는 로컬 가상 호스트를 로드하기 위한 인증서 생성에 중점을 둡니다.

프로덕션 환경에서는 자체 서명된 인증서를 사용하지 마십시오!


**Do not use self-signed certificates in production !**
For online certificates, use Let's Encrypt instead ([tutorial](https://gist.github.com/cecilemuller/a26737699a7e70a7093d4dc115915de8)).



## 인증 기관(CA) 

생성하다 RootCA.pem, RootCA.key& RootCA.crt: 

	openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout RootCA.key -out RootCA.pem -subj "/C=US/CN=Example-Root-CA"
	openssl x509 -outform pem -in RootCA.pem -out RootCA.crt

참고하세요 Example-Root-CA예시이므로 이름을 맞춤설정할 수 있습니다. 

## 도메인 이름 인증서 

두 개의 도메인이 있다고 가정해 보겠습니다. fake1.local그리고 fake2.local로컬 컴퓨터에서 호스팅되는 개발을 위해 (사용 hosts그들을 가리키는 파일 127.0.0.1).

먼저 파일을 생성합니다 domains.ext모든 로컬 도메인을 나열합니다. 

	authorityKeyIdentifier=keyid,issuer
	basicConstraints=CA:FALSE
	keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
	subjectAltName = @alt_names
	[alt_names]
	DNS.1 = localhost
	DNS.2 = fake1.local
	DNS.3 = fake2.local

생성하다 localhost.key, localhost.csr, 그리고 localhost.crt: 

	openssl req -new -nodes -newkey rsa:2048 -keyout localhost.key -out localhost.csr -subj "/C=US/ST=YourState/L=YourCity/O=Example-Certificates/CN=localhost.local"
	openssl x509 -req -sha256 -days 1024 -in localhost.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -extfile domains.ext -out localhost.crt

첫 번째 명령의 국가/주/도시/이름은 사용자 정의할 수 있습니다.

이제 Apache 등을 사용하여 웹 서버를 구성할 수 있습니다. 

	SSLEngine on
	SSLCertificateFile "C:/example/localhost.crt"
	SSLCertificateKeyFile "C:/example/localhost.key"