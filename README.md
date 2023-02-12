# 🎯 챌린지 코스 사이드 프로젝트

* 아래 요구사항은 내용을 직접 기입하셔도 좋고, 링크로 공유해주셔도 좋습니다.

### 4단계 - 테스트 코드 작성

- JaCoCo 결과를 공유해주세요

- SonarQube 혹은 SonarCloud 결과를 공유해주세요


### 5단계 - 배포

- 배포한 서비스 URL을 공유해주세요

비용 절감을 위해 사용하지 않을 때는 인스턴스를 실행하지 않고 있습니다. 실행 및 테스트를 위해 AWS IAM 사용자 계정, 배포 URL을 슬랙 메시지로 공유하겠습니다.

- 배포 스크립트를 공유해주세요

```shell
#!/bin/bash

## 변수 설정
txtrst='\033[1;37m' # White
txtred='\033[1;31m' # Red
txtylw='\033[1;33m' # Yellow
txtpur='\033[1;35m' # Purple
txtgrn='\033[1;32m' # Green
txtgra='\033[1;30m' # Gray

EXECUTION_PATH=$(pwd)
SHELL_SCRIPT_PATH=$(dirname $0)
BRANCH=$1
PROFILE=$2

## 변경사항 확인
function check_df() {
  echo -e "${txtylw}=======================================${txtrst}"
  echo -e "${txtgrn} << check_df >>${txtrst}"
  echo -e "${txtylw}=======================================${txtrst}"

  git fetch
  master=$(git rev-parse $BRANCH)
  remote=$(git rev-parse origin $BRANCH)
  remote=($(echo $remote | tr " ", "\n"))

  if [[ $master == ${remote[1]} ]]; then
    echo -e "[$(date)] Nothing to do!!! 😫"
    exit 0
  fi
}

## 저장소 pull
function pull() {
  echo -e "${txtylw}=======================================${txtrst}"
  echo -e "${txtgrn} << Pull Request 🏃♂️ >>${txtrst}"
  echo -e "${txtylw}=======================================${txtrst}"

  git pull origin $BRANCH
  git submodule foreach git pull https://$BRANCH:$SUBMODULE_TOKEN@github.com/$BRANCH/admin-site-server-config.git main
}

## gradle build
function build () {
  echo -e "${txtylw}=======================================${txtrst}"
  echo -e "${txtgrn} << gradle build >>${txtrst}"
  echo -e "${txtylw}=======================================${txtrst}"

  ./gradlew clean build
  JAR_FILE=$(basename — build/libs/*.jar)
}

## 프로세스 pid를 찾고 종료
function kill_process () {
  echo -e "${txtylw}=======================================${txtrst}"
  echo -e "${txtgrn} << find and kill process >>${txtrst}"
  echo -e "${txtylw}=======================================${txtrst}"

  PID=$(pgrep -f $JAR_FILE)
  if [ -z "$PID" ]
  then
    echo -e ">> 실행중인 프로세스가 없습니다."
  else
    sudo kill -2 $PID
    echo -e ">> 실행중인 프로세스를 종료했습니다."
  fi
}

## 어플리케이션 실행
function run () {
  echo -e "${txtylw}=======================================${txtrst}"
  echo -e "${txtgrn} << 어플리케이션을 실행합니다. >>${txtrst}"
  echo -e "${txtylw}=======================================${txtrst}"

  nohup java -jar -Dspring.profiles.active=$PROFILE build/libs/$JAR_FILE 1> application.log 2>&1  &
}

## 배포 스크립트 실행
if [[ $# -ne 2 ]]
then
    echo -e "${txtylw}=======================================${txtrst}"
    echo -e "${txtgrn}  << 스크립트 🧐 >>${txtrst}"
    echo -e ""
    echo -e "${txtgrn} $0 브랜치이름 ${txtred}{ prod | dev }"
    echo -e "${txtylw}=======================================${txtrst}"
    exit
else
  check_df
  pull
  build
  kill_process
  run
fi
```

### 6단계 - 성능테스트

- 부하 테스트 결과를 공유해주세요

- 모니터링 대시보드를 공유해주세요
