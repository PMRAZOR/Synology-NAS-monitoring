# Synology NAS 모니터링 (Grafana, InfluxDB, Telegraf)

![Overall](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/overall.png)
![Disk](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/disk.png)
![Cpu and Ram](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/cpuram.png)
![Network and Log](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/networklog.png)

프로젝트의 기존 목표:
* 유지관리 및 지속성을 위해 도커 컨테이너에 볼륨 마운트
* Grafana가 MySQL 대신 자체 SQLite로 데이터를 저장
* Snmp 패키지랑 시놀로지 NAS MIBS를 추가
* 정상 작동

추가된 목표:
* 정상 작동 (Telegraf 1.16.3 이 버스터 기반이라 소스 업데이트가 끝남)
* 올인원 컨테이너가 아닌 서비스별로 컨테이너 구분
* InfluxDB 버전을 1에서 2로 업데이트

## SNMP 활성화
1. 시놀로지 메뉴에서 제어판에 들어간 다음, "터미널 및 SNMP"에 들어갑니다.
2. "SNMP 서비스 활성화"를 체크하고, "SNMPv1, SNMPv2 서비스" 도 체크하세요.
3. 커뮤니티에는 ***public*** 이라고 입력하세요.
4. 적용합니다.
![snmp](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/snmp.png)

## 시놀로지에서 도커 이미지 실행
1. 이 깃을 다운받거나 클론합니다. (마스터 브랜치)
2. 이 mib들을 다운받아서 클론받은 git 내부의 mibs폴더에 넣습니다. (mibs 폴더안에 파일들이 27개가 되어야 합니다(13 + 14))
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/IF-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/SNMPv2-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/SNMPv2-SMI.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/SNMPv2-TC.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/SNMPv2-CONF.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/UCD-SNMP-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/HOST-RESOURCES-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/HOST-RESOURCES-TYPES.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/SNMP-FRAMEWORK-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/DISMAN-EVENT-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/IP-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/TCP-MIB.txt
     * https://raw.githubusercontent.com/net-snmp/net-snmp/master/mibs/UDP-MIB.txt
3. 시놀로지에서 설치할 공간에 폴더를 생성하고 깃에서 클론받은 ***mibs*** 폴더와 ***telegraf*** 폴더를 복사합니다. (각각 로그를 따기 위한 설정 파일들입니다)
![movefolder](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/movefolder.png)
4. ***telegraf*** 폴더안에 telegraf.conf를 텍스트 편집기로 엽니다
![editconf1](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/editconf1.png)
5. ***[[inputs.snmp]]*** 를 검색한 다음 밑에 있는 agent를 자신의 시놀로지 내부망 IP를 넣어주고 저장합니다.
![editconf2](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/editconf2.png)
5. 컨테이너 매니저에서 프로젝트를 선택하고 생성을 누른다음 프로젝트 이름과, 폴더 2개를 넣었던 폴더, 그리고 docker-compose.yml을 업로드 하거나 복사한 후 생성합니다.
![project1](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/project1.png)
![project2](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/project2.png)
![project3](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/project3.png)
6. 기다리면 컨테이너들이 생성됩니다.
![containerdone1](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/containerdone1.png)
7. 컨테이너 생성이 완료되었습니다.
![containerdone2](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/containerdone2.png)

## 그라파나 실행
1. [http://시놀로지_IP:3000](http://시놀로지_IP:3000) 에 접속하여 로그인합니다. (기본 설정은 admin, admin 입니다)
2. 비밀번호 변경창이 나오면 변경합니다.
3. 좌측 메뉴창에서 Connections - Data sources에 들어간 다음 "Add data source" 버튼을 클릭합니다.
![1](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/1.png)
4. InfluxDB를 검색하고 클릭합니다.
![2](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/2.png)
5. Query language를 Flux로 변경하고 URL에 "http://synology-influxdb:8086" 를 입력합니다.
![3](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/3.png)
6. InfluxDB Details를 동일하게 설정하고 "Save & Test" 버튼을 눌러 작동하는지 테스트 합니다. (home, synologymonitor, monitoring)
    * (해당 옵션들은 docker-compose.yml 에서 원하는대로 설정 가능하나, 설정 변경 시 telegraf.conf에도 동일하게 수정하셔야 합니다)
![4](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/4.png)
7. 좌측 메뉴창에서 Dashboards를 선택하고 오른쪽 New - Import 버튼을 눌러줍니다. (또는 http://시놀로지_IP:3000/dashboard/import 에 접속하셔도 됩니다)
![5](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/5.png)
8. 깃에서 클론 받은 Synology-DashBoard.json 을 드래그 해서 옮기고, 임포트를 눌러줍니다.
![6](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/6.png)
9. 빨간색 느낌표가 뜨면서 No data 또는 N/A 라고 나올텐데, 각각 카드 오른쪽 위에 점3개 버튼을 클릭하고 Edit을 클릭합니다.
![7](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/7.png)
10. 밑의 Queries 탭에서 Data source를 클릭하고, ***${InfluxDB}*** 를 선택합니다
![8](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/8.png)
![9](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/9.png)
11. 오른쪽 상단의 Save dashboard를 클릭하고 Save를 클릭합니다.
![10](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/10.png)
![11](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/11.png)
12. 오른쪽 상단의 Back to dashboard 버튼을 클릭하여 뒤로 나와 정상적으로 데이터가 나오는지 확인합니다.
![12](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/grafana/12.png)
13. 모든 카드들에 동일하게 설정합니다.
![Overall](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/overall.png)

## 로깅 활성화
1. 시놀로지 패키지 센터에서 로그 센터를 설치합니다
2. 로그 센터를 실행합니다
3. 로그 전송 탭에 들어가서 > "syslog 서버로 로그 보내기" 를 체크합니다
4. 서버 = ***localhost***, 포트 = ***8094***, 전송 프로토콜 = ***UDP***, 로그 형식 = ***BSD (RFC 3164)*** 으로 지정합니다
5. "테스트 로그 보내기"를 클릭하여 테스트 할 수 있습니다
6. 적용합니다
![snmp](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/syslog.png)
![test](https://github.com/PMRAZOR/Synology-NAS-monitoring/blob/master/image/settings/test.png)

# Synology NAS monitoring (ENG)

# Synology NAS monitoring (Readme before forked)

The main points of this project are:

* Persistence is supported via mounting volumes to a Docker container.
* Grafana will store its data in SQLite files instead of a MySQL.
* Added snmp packages and Synology NAS MIBS.
* Enable 

## Enable SNMP
1. From Control panel in your Synology NAS go to Terminal & SNMP
2. Click on SNMP tab, and enable SNMPv1, SNMPv2 service
3. in Community input put ***public***
4. Save

## Run Docker image in your Synology

1. Install Docker from Synology package center
2. Create two empty folders in your Synology ***influxdb*** and ***grafana***, we need to use it later to mount it to our container.
3. Open Docker client from Synology > Image > Add > Add from url and paste Hub page url "https://hub.docker.com/r/alhazmy13/telegraf-influxdb-grafana"
4. Wait until it finishes downloading the image
5. Click on the image "alhazmy13/telegraf-influxdb-grafana" and then click on Launch
6. Network Tab keep it in bridge mode 
7. Check "Enable auto-restart."
8. Port settings, just change Local port for 3003 from Auto to 3003, and port 514 from Auto to 5144
9. In Volume settings, click Add folder and select the first folder that we created, "grafana" and on mount Path, paste ***/var/lib/grafana***
10. In Volume settings again, click Add Folder and select the second folder that we created "influxdb" and on mount Path paste ***/var/lib/influxdb***
12. [OPTIONAL] Environment Tab > Add new variable "TZ" with your local time zone **ignore this if you want to use the default UTC**
14. Apply, Next, Done and your container should be ready.

## Start Grafana

1. Open [http://YOUR_LOCAL_NAS_IP:3003](http://YOUR_LOCAL_NAS_IP:3003) and login with the default username ***root*** and password ***root***
2. You need to import the dashboard. To do this, go to [http://YOUR_LOCAL_NAS_IP:3003/dashboard/import](http://YOUR_LOCAL_NAS_IP:3003/dashboard/import) and put ***14590*** in "Import via grafana.com" input
3. Click on load and complete the process

## Enable Logging
1. Install Log center From Synology package center
2. Open Log center app
3. Click on Log Sending > check "Send log to syslog server"
3. Set Server = ***localhost***,  port = ***5144***, Protocol = ***UDP***, Format = ***BSD (RFC 3164)***
4. For testing, click on "Send test log" 
4. Apply

## Configure Firewall
If the firewall is enabled, then you need to add a new rule for port UDP/161, This is mandatory otherwise, some data will be missing from the dashboard https://github.com/alhazmy13/Synology-NAS-monitoring/issues/7 .

1.  Open Control panel
2.  Security -> Firewall
3.  Edit Rules -> Create New Rule
4.  In the ports section, select from a built-in applications and chose SNMP service
5.  In the IP section select Spesifc IP -> subnet -> Source: 172.12.0.0 / subnet: 255.255.0.0
6.  Action = Allow
7.  Disable and re-enable the firewall for it to take effect
