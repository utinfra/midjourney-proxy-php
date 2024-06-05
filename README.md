# midjourney-proxy
모든 기능의 Midjourney 전이, Midjourney의 모든 이미지 생성 기능을 지원하며, 고성능, 안정적이고 무료입니다


## 지원 기능

- [x] Imagine 지원 (그리기)
- [x] Imagine 이미지첨부 지원
- [x] UPSCALE  Pan ⬅️ ➡️ ⬆️ ⬇️ 지원
- [x] ZoomOut 🔍 지원
- [x] Custom Zoom 🔍 지원
- [x] 부분 다시그리기 Vary (Region) 🖌 지원
- [x] Make Square
- [x] 실시간 진행상황 업무 지원
- [x] Blend 지원
- [x] Describe 지원
- [x] 계정 풀 지원
- [x] 금지어 설정 지원
- [x] CDN 이미지 대체 지원

** 설치
웹맨 프레임워크를 설치해야 합니다(이미 설치한 경우 이 단계를 무시하십시오).
composer create-project workerman/webman

webman 디렉터리로 이동하여 webman/midjourney를 설치하세요.
cd webman
composer require webman/midjourney

**CONFIGURATION
config/plugin/webman/midjourney/process.php 파일을 열고 다음과 같이 설정하세요
<?php

use Webman\Midjourney\TaskStore\File;

return [
    'server' => [
        'handler' => Webman\Midjourney\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'reloadable' => false,
        'constructor' => [
            'config' => [
                'accounts' => [
                    [
                        'enable' => true,
                        'token' => '설정 방식은 아래를 참조하세요',
                        'guild_id' => '설정 방식은 아래를 참조하세요',
                        'channel_id' => '설정 방식은 아래를 참조하세요',
                        'useragent' => '설정 방식은 아래를 참조하세요',
                        'concurrency' => 3, // 동시 실행 수, 10달러/30달러 사용자 3 동시 실행, 60달러/120달러 사용자 12 동시 실행
                        'timeoutMinutes' => 10, // Discord에 작업을 제출한 후 10분 이내에 응답이 없으면 시간 초과로 간주됩니다.
                    ]
                ],
                'proxy' => [
                    'server' => 'https://discord.com',      // 국내에서는 프록시가 필요하며, 프록시 설정은 아래를 참조하십시오.
                    'cdn' => 'https://cdn.discordapp.com',  // 국내에서는 프록시가 필요하며, 프록시 설정은 아래를 참조하십시오.
                    'gateway' => 'wss://gateway.discord.gg', // 국내에서는 프록시가 필요하며, 프록시 설정은 아래를 참조하십시오.
                    'upload' => 'https://discord-attachments-uploads-prd.storage.googleapis.com', // 국내에서는 프록시가 필요하며, 프록시 설정은 아래를 참조하십시오.
                ],
                'store' => [
                    'handler' => File::class, // 작업 저장 방식
                    'expiredDates' => 30, // 만료일자 30일
                    File::class => [
                        'dataPath' => runtime_path() . '/data/midjourney', // 작업 저장 디렉토리
                    ]
                ],
                'settings' => [
                    'debug' => false,  // 디버그 모드는 터미널에 더 많은 정보를 표시할 것입니다.
                    'secret' => '',    // 인터페이스 키는 비어 있지 않은 경우 http 헤더 mj-api-secret에 전달해야 합니다
                    'notifyUrl' => '', // 웹맨 AI 프로젝트는 비워 두세요
                    'apiPrefix' => '', // API Prefix
                    'tmpPath' => runtime_path() . '/tmp/midjourney' // 파일 업로드용 임시 디렉토리
                ]
            ]
        ]
    ]
];

** token、guild_id、channel_id useragent 얻기 (알아서 하삼)
** 프록시 예제
* https 서비스 nginx 프록시
discord.com cdn.discordapp.com discord-attachments-uploads-prd.storage.googleapis.com 도메인마다 프록시를 설정해야 합니다. 예를 들어, discord.com의 경우 프록시 설정은 다음과 유사합니다.
server {
  listen 80;
  server_name your_domain.com;
  proxy_buffer_size  64k;
  proxy_buffers   32 64k;
  proxy_busy_buffers_size 128k;
  location ^~ / {
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_ssl_server_name on;
    proxy_pass https://discord.com;
    proxy_set_header Host discord.com;
    proxy_set_header Referer "";
  }
}

*wss 서비스 프록시
gateway.discord.gg는 websocket 프로토콜을 사용합니다. 위의 https 프록시와는 다른 방식으로 프록시 설정을 해야 합니다. gateway.discord.gg 프록시는 다음과 유사합니다
server {
  listen 80;
  server_name your_wss_domain.com;
  proxy_buffer_size  64k;
  proxy_buffers   32 64k;
  proxy_busy_buffers_size 128k;

  location ^~ / {
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_http_version 1.1;
    proxy_ssl_server_name on;
    proxy_pass https://gateway.discord.gg;
    proxy_set_header Host gateway.discord.gg;
    proxy_set_header Referer "";
  }
}

**Interface
* /image/imagine 이미지그리기
 SEND POST
{
  "prompt": "a cat",
  "images": [url1, url2, ...], // 옵션
  "notifyUrl": "https://your-server.com/notify", // 옵션
}

RECEIVE
{
  "code": 0,
  "msg": "ok",
  "taskId": "1710816049856103374",
  "data": []
}

* /image/action  이미지편집
 SEND POST
{
    "taskId": "1710816049856103374",
    "customId": "MJ::JOB::upsample::1::749b4d14-75ec-4f16-8765-b2b9a78125fb",
    "notifyUrl": "https://your-server.com/notify", // 옵션
}

RECEIVE
{
  "code": 0,
  "msg": "ok",
  "taskId": "1710816302060986090",
  "data": []
}

* /image/describe  이미지를 설명
 SEND POST
{
    "images": [url],
    "notifyUrl": "https://your-server.com/notify", // 옵션
}

RECEIVE
{
  "code": 0,
  "msg": "ok",
  "taskId": "1710816302060386071",
  "data": []
}

* /image/blend 이미지 합치기
 SEND POST
{
    "images": [url1, url2],
    "notifyUrl": "https://your-server.com/notify", // 옵션
}

RECEIVE
{
  "code": 0,
  "msg": "ok",
  "taskId": "1710816302060354172",
  "data": []
}

* /task/fetch?taskId=1710816049856103374 작업 상태
SEND GET
RECEIVE
{
  "code": 0,
  "msg": "success",
  "data": {
    "id": "1710816049856103374",
    "action": "IMAGINE",
    "status": "FINISHED",
    "submitTime": 1710903739,
    "startTime": 1710903739,
    "finishTime": 1710903844,
    "progress": "100%",
    "imageUrl": "https:\/\/your_cdn.com\/attachments\/1148151204884726471\/121984387748450658284\/a_cat._65e72369d-1db1-5be4-9566-71056a5b0caf.png?ex=660cc723&is=65fa5223&hm=0d9b721610b62101c7cb4c0f3bf4e364cdd69be3441b9c3b1c200d20b309d97e&",
    "imageRawUrl": "https:\/\/cdn.discordapp.com\/attachments\/1148151204884726471\/121984387748450658284\/a_cat._65e72369d-1db1-5be4-9566-71056a5b0caf.png?ex=660cc723&is=65fa5223&hm=0d9b721610b62101c7cb4c0f3bf4e364cdd69be3441b9c3b1c200d20b309d97e&",
    "prompt": "A cat. --v 6.0 --relax",
    "finalPrompt": "A cat. --v 6.0 --relax",
    "params": [],
    "images": [],
    "description": null,
    "failReason": null,
    "discordId": "1148151204875075657",
    "data": [],
    "buttons": [
      [
        {
          "type": 2,
          "style": 2,
          "label": "U1",
          "custom_id": "MJ::JOB::upsample::1::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "label": "U2",
          "custom_id": "MJ::JOB::upsample::2::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "label": "U3",
          "custom_id": "MJ::JOB::upsample::3::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "label": "U4",
          "custom_id": "MJ::JOB::upsample::4::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "emoji": {
            "name": "🔄"
          },
          "custom_id": "MJ::JOB::reroll::0::65e72369d-1db1-5be4-9566-71056a5b0caf::SOLO"
        }
      ],
      [
        {
          "type": 2,
          "style": 2,
          "label": "V1",
          "custom_id": "MJ::JOB::variation::1::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "label": "V2",
          "custom_id": "MJ::JOB::variation::2::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "label": "V3",
          "custom_id": "MJ::JOB::variation::3::65e72369d-1db1-5be4-9566-71056a5b0caf"
        },
        {
          "type": 2,
          "style": 2,
          "label": "V4",
          "custom_id": "MJ::JOB::variation::4::65e72369d-1db1-5be4-9566-71056a5b0caf"
        }
      ]
    ]
  }
}

필드의미

id     작업ID
action 작업유형 (IMAGINE, UPSCALE, VARIATION 아래를참조 action 값설명)
status 작업상태 (PENDING, STARTED, SUBMITTED, RUNNING, FINISHED, FAILED)
submitTime 작업생성시간
startTime 시작시간
finishTime 종료시간
progress 작업진행률 0% - 100%，성공하든 실패하든 최종상태는 100%임.
imageUrl 이미지주소 cdn교체주소
imageRawUrl 이미지원본주소 (국내접근불가)
prompt 프롬프트
finalPrompt 최종프롬프트
params 작업관련매개변수
images 작업관련이미지，(URL배열)
description 그림해석결과，작업설명
failReason 작업실패원인，이 갑이 비어있지않으면, 작업이 실패한 것임.
discordId discord에 할당된 작업 id
data 작업맞춤데이터
buttons 작업실행버튼，CustomerID가 /image/action인터페이스의 CustomerID매개변수임
action 값 설명

IMAGINE 그림그리기
UPSCALE 이미지선택
VARIATION 부분그리기
REROLL 다시생성
DESCRIBE 이미지에서 텍스트로
BLEND 이미지혼합
ZOOMOUT 이미지 확장
ZOOMOUT_CUSTOM 사용자정의 이미지 확대
PANLEFT  그림을 왼쪽으로 확장
PANRIGHT 그림을 오른쪽으로 확장
PANUP 그림을 윗쪽으로 확장
PANDOWN 그림을 아랫쪽으로 확장
MAKE_SQUARE 그림을 정사각형으로 확장
PIC_READER 이미지에서 텍스트를 추출한 후 새 이미지를 생성.
CANCEL_JOB 작업 취소
UPSCALE_V5_2X v5 2배고화질
UPSCALE_V5_4X v5 4배고화질
UPSCALE_V6_2X_CREATIVE v6 2배 창의 고화질
UPSCALE_V6_2X_SUBTLE v6 2배 미세조정 고해상도 이미지
VARIATION_STRONG 강렬한 변환
VARIATION_SUBTLE 미세조정 변환
VARIATION_REGION 부분 재 그리기
notifyUrl 알림형식
만약 notifyUrl 매개변수가 설정되어 있다면, 작업 상태가 변경될 때 이 주소로 POST 요청을 보냅니다. 
요청 내용은 작업 상태의 JSON 형식이며, 이는 /task/status 인터페이스가 반환하는 data 내용과 동일합니다

{
    "id": "1710816049856103374",
    "action": "IMAGINE",
    "status": "FINISHED",
    "submitTime": 1710903739,
    "startTime": 1710903739,
    "finishTime": 1710903844,
    "progress": "100%",
    "imageUrl": "https:\/\/your_cdn.com\/attachments\/1148151204884726471\/121984387748450658284\/a_cat._65e72369d-1db1-5be4-9566-71056a5b0caf.png?ex=660cc723&is=65fa5223&hm=0d9b721610b62101c7cb4c0f3bf4e364cdd69be3441b9c3b1c200d20b309d97e&",
    "imageRawUrl": "https:\/\/cdn.discordapp.com\/attachments\/1148151204884726471\/121984387748450658284\/a_cat._65e72369d-1db1-5be4-9566-71056a5b0caf.png?ex=660cc723&is=65fa5223&hm=0d9b721610b62101c7cb4c0f3bf4e364cdd69be3441b9c3b1c200d20b309d97e&",
    "prompt": "A cat. --v 6.0 --relax",
    "finalPrompt": "A cat. --v 6.0 --relax",
    "params": [],
    "images": [],
    "description": null,
    "failReason": null,
    "discordId": "1148151204875075657",
    "data": [],
    "buttons": [
       ...
    ]
  }

## 관련 프로젝트
![image](https://github.com/webman-php/midjourney-proxy/assets/6073368/2d249e52-5e2a-4ca3-b356-99ea95c238e1)



  [https://bla.cn](https://bla.cn/#module=painting)  
  [https://jey.cn](https://jey.cn)  
  [webman AI](https://www.workerman.net/app/view/ai)  

## webman AI QQ2000人群
![image](https://github.com/webman-php/midjourney-proxy/assets/6073368/7b7aa50c-9f4b-4825-95a5-d034ce8f54fa)

**QQ群 789898358**

## 文档
[webman/midjourney](https://www.workerman.net/plugin/159)
