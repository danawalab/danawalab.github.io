---
layout: post
title:  "AWS EC2 인스턴스 자동 시작/중지"
description: "이번에 소개 드릴 내용은 AWS의 EC2 인스턴스를 시작과 중지를 자동으로 적용하는방법에 대해 알아보도록 하겠습니다. EC2 인스턴스 활용하기에는 편하고 좋은 도구 이지만, 정해진 시간에 인스턴스를 시작 또는 중비하는 기능을 제공하지 않아, 실수로 인스턴스를 중지 하지 않으면 가동한 시간만큼 과금이 발생하게 됩니다. Lambda와 CloudWatch를 조합하여 인스턴스에 설정된 태그를 기반으로 인스턴스를 시작/중지하는 방법에 대해 알아보도록 하겠습니다. AWS의 CloudWatch에서 규칙으로 1시간 주기로 Lambda를 호출 되도록 하고, Lambda에서는 인스턴스에 등록된 태그를 해석하여 인스턴스 시작, 종료 호출하게 됩니다."
date:   2021.05.03 
writer: "김준우" 
categories: aws
---
## 소개

안녕하세요. 다나와 김준우입니다. 이번에 소개 드릴 내용은 AWS의 EC2 인스턴스를 시작과 중지를 자동으로 적용하는방법에 대해 알아보도록 하겠습니다. EC2 인스턴스 활용하기에는 편하고 좋은 도구 이지만, 정해진 시간에 인스턴스를 시작 또는 중비하는 기능을 제공하지 않아, 실수로 인스턴스를 중지 하지 않으면 가동한 시간만큼 과금이 발생하게 됩니다. Lambda와 CloudWatch를 조합하여 인스턴스에 설정된 태그를 기반으로 인스턴스를 시작/중지하는 방법에 대해 알아보도록 하겠습니다. AWS의 CloudWatch에서 규칙으로 1시간 주기로 Lambda를 호출 되도록 하고, Lambda에서는 인스턴스에 등록된 태그를 해석하여 인스턴스 시작, 종료 호출하게 됩니다.

### 사용방법

서울리전(ap-northeast-2)의 ec2 인스턴스에 아래 태그에 입력하게 되면 자동으로 정해진 시간에 인스턴스를 시작 / 중지 하게 됩니다.


|태그명|값|비고|
|--|--|--|
|AUTOSTOP_ENABLE|true 또는 false|인스턴스 시작종료하는 기능을 사용할지 여부입니다. true일때 인스턴스 자동 시작/중지 됩니다.|
|DAY|월,화,수,목,금|자동 시작/중지 기능을 사용할 요일을 입력합니다. (콤마구분)|
|TIME|09~18|~(물결표) 기준으로 앞의 값은 시작, 뒤에는 값은 중지 시간입니다.<br/> - 분단위 지원하지 않습니다. <br/> - 시작 과 중지 사이의 시간에 대해서는 아무런 작업하지 않습니다.|



## Lambda

람다에서 인스턴스를 시작, 종료하도록 구현하였습니다. 우선 람다가 호출되는 시점의 요일과 시간을 메모리에 적재합니다. 그리고 ec2 인스턴스에 등록된 모든 태그를 조회하여 설정된 요일과 시간이 람다 호출한 시점의 요일과 시간이 일치하게 되면 start_instances 또는 stop_instances 함수를 호출합니다.

```python
# -*- coding: utf-8 -*-
import boto3
from datetime import datetime
# 서울 리전
region = 'ap-northeast-2'
# 월 ~ 일요일 
t = ["월", "화", "수", "목", "금", "토", "일"]

def lambda_handler(event, context):
    # 람다 호출된시점의 시간을 구합니다.
    print "start"
    now_date = t[datetime.today().weekday()]
    now_hour = int(datetime.now().strftime('%H')) + 9
    print "now >> date: " + now_date + ", hour: " + str(now_hour)

    # ec2 인스턴스의 모든 태그를 조회합니다.
    ec2 = boto3.client('ec2', region_name=region)
    response = ec2.describe_tags(
        Filters=[
            {
                'Name': 'resource-type',
                'Values': ['instance']
            }
        ]
    )
    
    # 값 임시 저장
    enable_instances = []
    day_instances = {}
    time_instances = {}
    
    # AUTO_STOP_ENABLE 태그가 true인 값만 추출합니다/
    for tag in response['Tags']:
        if tag['Key'] == "AUTOSTOP_ENABLE" and tag['Value'].lower() == "true":
            enable_instances.append(tag['ResourceId'])
        if tag['Key'] == "DAY":
            day_instances[tag['ResourceId']] = tag['Value']
        if tag['Key'] == "TIME":
            time_instances[tag['ResourceId']] = tag['Value']

    for instance in enable_instances:
        try:
            # 요일이 일치하는지 확인합니다.
            days = day_instances[instance].split(",")            
            is_day = False
            for d in days:
                if now_date == d:
                    is_day = True
            
            # 시간이 일치하는지 확인합니다.
            times = time_instances[instance].split("~")
            is_start_time = False
            is_end_time = False
            if int(times[1].strip()) == now_hour:
                is_end_time = True
            elif int(times[0].strip()) == now_hour:
                is_start_time = True
            

            if is_day == True and is_end_time == True:
                # 중지 인스턴스 호출
                ec2.stop_instances(InstanceIds=[instance])
            elif is_day == True and is_start_time == True:
                # 시작 인스턴스 호출
                ec2.start_instances(InstanceIds=[instance])
        except Exception as ex:
            print ex
    
    print "end"
```



## CloudWatch

cloudwatch에서 1시간 주기로 람다를 호출하는 규칙을 생성합니다. cron 설정은 0 */1 * * ? * 으로 매시 정각에 람다를 호출하게 됩니다.

![/images/2021-05-03-aws-autostop/Untitled.png](/images/2021-05-03-aws-autostop/Untitled.png)



## ec2 인스턴스 태그

인스턴스에 아래 이미지 처럼 태그를 입력해두면 작업외 시간에는 중지상태가 되고, 사용예정일땐 인스턴스를 시작하게 됩니다.

![/images/2021-05-03-aws-autostop/Untitled%201.png](/images/2021-05-03-aws-autostop/Untitled%201.png)



## 정리

ec2 인스턴스를 자동으로 시작, 중지하는 방법에 대해 알아보았습니다. 간단한 설정으로 폭탄과금을 예방할 수 있고, 업무 시간에 마춰 인스턴스를 시작할 수 있어 준비하는 시간이 줄어들게 됩니다. 불필요한 과금을 절약하는데 도움이 되었으면 좋겠습니다. 감사합니다.