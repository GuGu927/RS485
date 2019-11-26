RS485 Compilation
=================
Version : 1.0.2b
----------------

---  
## 패치노트
### Ver 1.0.2
> 코콤) 패킷 분류작업 추가하여 패킷오류 시 로그출력하도록 함  
> 코콤) 환기팬 오류 수정  
> 코콤) 난방 MQTT 송수신 관련 개선  
> 코콤) 난방관련 HA에서 작동 시 월패드 회신 기다리지 않고 HA상에 반영하도록 수정  
### Ver 1.0.1
> 코콤) 패킷에서 write 시 interval을 주도록 변경  
> 코콤) 조명, 콘센트의 방 별 일괄제어 추가(light0, plug0)  
> 코콤) 월패드로 패킷을 5회 송신 후 응답없을 경우 해당 명령을 무시하도록 변경  
> 코콤) 환기팬 추가  
### Ver 1.0.0
> 코콤(조명, 플러그, 난방, 환기, 가스, 엘레베이터) 지원  
> 그렉스 환기장치[바닥열환기장치][grex_link] 지원  
---  
## MQTT 명령어
> ### topic : rs485/bridge/config/restart  
> payload : none  
> 설명 : HA 재시작 시 mqtt discovery를 재설정 한다.  

> ### topic : rs485/bridge/config/remove  
> payload : none  
> 설명 : HA에 mqtt discovery로 등록된 기기entity를 삭제한다.  

> ### topic : rs485/bridge/config/log_level  
> payload : debug 혹은 info  
> 설명 : 로그레벨을 바꾼다. 기본 : info  

> ### topic : rs485/bridge/config/scan  
> payload : none  
> 설명 : 월패드의 장치정보를 읽어온다.  

> ### topic : rs485/bridge/config/packet  
> payload : hexdata  
> 설명 : 월패드에 패킷을 보낸다  
> 예: AA5530BC000E0001003A0000000000000000350D0D  
> INFO:RS485:Message: rs485/bridge/config/packet = AA5530BC000E0001003A0000000000000000350D0D  
> INFO:RS485:[From kocom]light/livingroom/state = {'light2': 'off', 'light1': 'off', 'light3': 'on'}  
> INFO:RS485:[To HA]homeassistant/light/livingroom/state = {"light2": "off", "light1": "off", "light3": "on"}  
---  
## rs485.conf 파일 설명
```conf
[RS485]
type = Serial                    // serial 혹은 socket
[Socket]
server = 192.168.x.x           // socket 쓸 경우 socket IP주소
port = xx                        // socket 쓸 경우 socket PORT
[SocketDevice]
device = kocom               // socket 쓸 경우 월패드 이름{{장치이름}}
[Serial]
port1 = /dev/ttyUSB0        // serial 쓸 경우 (월패드 혹은 그렉스)의 장치경로 작성
port2 = /dev/ttyUSB1        // serial 쓸 경우 (월패드 혹은 그렉스)의 장치경로 작성
port3 = /dev/ttyUSB2        // serial 쓸 경우 (월패드 혹은 그렉스)의 장치경로 작성
[SerialDevice]
port1 = kocom               // serial 쓸 경우 (월패드 이름 혹은 그렉스 이름) 작성{{장치이름}}
port2 = grex_ventilator     // serial 쓸 경우 (월패드 이름 혹은 그렉스 이름) 작성{{장치이름}}
port3 = grex_controller     // serial 쓸 경우 (월패드 이름 혹은 그렉스 이름) 작성{{장치이름}}

# {{장치이름}}
# kocom = 코콤월패드
# grex_ventilator = 그렉스 환기장치
# grex_controller = 그렉스 환기장치의 리모콘(환기모드, 정지 등)

[MQTT]
anonymous = False           // MQTT 설정
server = 192.168.x.xx         // MQTT 서버
username = id                 // MQTT ID
password = pw                // MQTT PW
[Wallpad]
light = True                    // 조명 : True 혹은 False 대소문자 주의
plug = True                    // 플러그 : True 혹은 False 대소문자 주의
thermostat = True            // 난방 : True 혹은 False 대소문자 주의
fan = False                     // 환기팬 : True 혹은 False 대소문자 주의
gas = True                     // 가스 : True 혹은 False 대소문자 주의
elevator = True               // 엘레베이터 : True 혹은 False 대소문자 주의
```
---  
## rs485.py 스크립트 파일에서 수정할 항목

```py
# 보일러 초기값
INIT_TEMP = 22
# 조명 / 플러그 갯수
KOCOM_LIGHT_SIZE            = {'livingroom': 3, 'bedroom': 2, 'room1': 2, 'room2': 2, 'kitchen': 3}
KOCOM_PLUG_SIZE             = {'livingroom': 2, 'bedroom': 2, 'room1': 2, 'room2': 2, 'kitchen': 2}

# 방 패킷에 따른 방이름 (패킷1: 방이름1, 패킷2: 방이름2 . . .)
# 월패드에서 장치를 작동하며 방이름(livingroom, bedroom, room1, room2, kitchen 등)을 확인하여 본인의 상황에 맞게 바꾸세요
# 조명/콘센트와 난방의 방패킷이 달라서 두개로 나뉘어있습니다.
KOCOM_ROOM                  = {'00': 'livingroom', '01': 'bedroom', '02': 'room1', '03': 'room2', '04': 'kitchen'}
KOCOM_ROOM_THERMOSTAT       = {'00': 'livingroom', '01': 'bedroom', '02': 'room1', '03': 'room2'}

# TIME 변수(초)
SCAN_INTERVAL = 300         # 월패드의 상태값 조회 간격
SCANNING_INTERVAL = 0.5     # 상태값 조회 시 패킷전송 간격
```


[grex_link]: http://www.grex.co.kr/product-index/prd-a/
