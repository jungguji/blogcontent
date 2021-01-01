---
layout: posts
title: Codedeploy 연동 중 발생한 에러
date: 2020-06-08 15:08:41
tags:
    - AWS
    - CodeDeploy
---

## Codedeploy에서 배포그룹 생성 시 인스턴스를 잘못 등록

```text
The deployment failed because no instances were found for your deployment group.
Check your deployment group settings to make sure the tags for your Amazon EC2 instances or Auto Scaling groups correctly identify the instances you want to deploy to, and then try again.
```

 

![CodeDeploy](/images/20200608/codedeploy_인스턴스_설정.png)

배포 그룹에서 태그 그룹으로 인스턴스를 지정할 때 위 사진처럼 일치하는 인스턴스가 존재한다는 문구가 나타나는 걸 확인해야한다.

* * *

## 생성한 버킷 리전과 인스턴스의 리전이 틀려서 에러 발생

```text
payload:"{\"error_code\":5,\"script_name\":\"\",\"message\":\"
The bucket you are attempting to access must be addressed using the specified endpoint. Please send all future requests to this endpoint.\",\"log\":\"\"}"}
```

위 에러 로그는 아래 디렉토리에 위치한 파일을 열어서 확인 할 수 있다.
shell script에서 작성한 echo도 아래 파일에서 함께 확인 할 수 있다.

```shell
vim /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log
```

 
 
![CodeDeploy Log File](/images/20200608/codedeploy_logfile.PNG)