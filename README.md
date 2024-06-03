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

字段含义

id 任务ID
action 任务类型 (IMAGINE, UPSCALE, VARIATION 等参考下方 action 值说明)
status 任务状态 (PENDING, STARTED, SUBMITTED, RUNNING, FINISHED, FAILED)
submitTime 任务创建时间
startTime 开始时间
finishTime 完成时间
progress 任务进度 0% - 100%，不管成功还是失败，最终状态为100%
imageUrl 图片地址 cdn替换后的地址
imageRawUrl 图片原始地址 国内无法访问
prompt 提示词
finalPrompt MJ最终使用的提示词
params 任务相关参数
images 任务相关图片，格式为url数组
description 图生文的结果，只有describe任务有
failReason 任务失败原因，只要此处值不为空代表任务失败
discordId 任务所属的discord id
data 任务自定义数据
buttons 任务操作按钮，其中 custom_id 为 /image/action 接口的 customId 参数
action 值说明

IMAGINE 画图
UPSCALE 选图
VARIATION 局部重绘
REROLL 重新生成
DESCRIBE 图生文
BLEND 图片混合
ZOOMOUT 扩图
ZOOMOUT_CUSTOM 自定义扩图
PANLEFT 扩图左移
PANRIGHT 扩图右移
PANUP 扩图上移
PANDOWN 扩图下移
MAKE_SQUARE 扩图成正方形
PIC_READER 从图片中提取文字后生成新图
CANCEL_JOB 取消任务
UPSCALE_V5_2X v5 2倍高清图
UPSCALE_V5_4X v5 4倍高清图
UPSCALE_V6_2X_CREATIVE v6 2倍创意高清图
UPSCALE_V6_2X_SUBTLE v6 2倍微调高清图
VARIATION_STRONG 强烈变换
VARIATION_SUBTLE 微调变换
VARIATION_REGION 局部重绘
notifyUrl 通知格式
如果有设置 notifyUrl 参数，当任务状态变化时会向此地址发送 POST 请求，请求内容为任务状态的 json 格式，格式与 /task/status 接口返回的data内容一致。

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
