
# Grafana 설정 가이드

## 구성 요소
* grafana ([grafana/grafana:6.4.3](https://grafana.com/grafana/download))

## Step 0. Keycloak 연동
- `추측` : DOMAIN = console.192.168.178.82.nip.io로 추측하고 진행해볼거
* 목적 : 'Keycloak 연동'
* 순서: 
	* keycloak에서 client 생성 후
	* Client protocol = openid-connect , Access type = confidential Standard Flow Enabled = On, Direct Access Grants Enabled = On
	* Client > grafana > Credentials > Secret 복사 후 version.conf CLIENT_SECRET,CLIENT_ID 를 채운다
	* version.conf KETCLOAK_ADDR에  keycloak 주소를 채운다.
	* Root URL = https://${DOMAIN}/api/grafana/, Valid Redirect URIs = https://${DOMAIN}/api/grafana/login/generic_oauth/* , Admin URL = https://${DOMAIN}/api/grafana/, Web Origins = https://${DOMAIN}/api/grafana/ 
	* DOMAIN = grafana dns 주소(ex grafana.tmaxcloud.org)
![image](https://user-images.githubusercontent.com/66110096/118447268-8a7f3000-b72b-11eb-9bdd-01d4252427c6.png)

## Step 1. Grafana Config 설정

* 목적 : `version.conf 파일에 설치를 위한 정보 기입`
* 순서: 
	* 환경에 맞는 config 내용 작성
		* version.conf 에 알맞는 버전(6.4.3)과 registry, pvc  정보를 입력한다.
		* GRAFANA_HOME=~/install-grafana-5.0
		* GRAFANA_VERSION=6.4.3
		* REGISTRY = private registry가 존재하지 않는다면 그대로 둔다. `ex. 192.168.178.17:5000`
		* DOMAIN = grafana dns 주소 `ex. grafana.tmaxcloud.org`
		* KEYCLOAK_ADDR = 192.168.178.81
		* CLIENT_ID = hyperauth에 등록한 grafana client id( `grafana2`로 설정해주세요)
		* CLIENT_SECRET = hyperauth에서 생성한 `Client > grafana > Credentials > Secret의 문자열`
		* GRAFANA_PVC = grafana pvc 용량 `ex. 10Gi`
		* DOMAIN_NAME = apigateway dns address `ex. 192.168.178.82`
	

## Step 2. installer 실행
* 목적 : `설치를 위한 shell script 실행`
* 순서: 
	* 권한 부여 및 실행
	``` bash
	$ sudo chmod +x install.sh
	$ ./install.sh
	```
