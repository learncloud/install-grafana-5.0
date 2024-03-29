
# Grafana 설정 가이드

## 구성 요소
* grafana ([grafana/grafana:6.4.3](https://grafana.com/grafana/download))

## Step 0. Keycloak 연동
- `참고` : DOMAIN = grafana.192.168.178.82.nip.io (192~.nip.io는 api-gateway주소임)
- `참고` : DOMAIN_NAME = 192.168.178.82.nip.io (192~.nip.io는 api-gateway주소임)


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
	
## log 설정 가이드
* 목적: 'log level 설정'
* 순서: ([grafana-config.yaml](https://github.com/tmax-cloud/install-grafana/blob/5.0/yaml/grafana-config.yaml)) 에
```
data:
  grafana.ini: |
     [log]
     level = 해당 부분에 설정(debug,info, warn, error,critical) (default 는 info)
```


## Hyperauth selfsigned CA 설정
* 목적: selfsigned_CA를 마운트하여 grafana와 hyperauth에 연동(공인인증서 사용하지 않을 경우)

* 순서:
  1. selfsigned-ca.yaml에 api-gateway-system namespace에 tmaxcloud-gateway-selfsigned secret의 ca.crt를 넣어준다.
  
  2. selfsigned-ca.yaml 생성
  
  ```
  kubectl apply -f selfsigned-ca.yaml
  ```
  
  3. grafana.yaml 에 selfsigned-ca volume mount
  ```
  volumeMounts:
  - name: selfsigned-ca
    mountPath: /etc/grafana
    readOnly: true
  volumes:
  - name: selfsigned-ca
    secret:
      secretName: selfsigned-ca
  ```
  4. grafana-config.yaml 수정
  ```
  [auth.generic_oauth]
  tls_skip_verify_insecure = false
  tls_client_cert = /etc/grafana/ca.crt
  ```
