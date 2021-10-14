## 리눅스 명령어 모음

|Command|Description|
|-------|-----------|
|`cat [filename]`|기본 출력 디바이스에 파일 내용을 출력합니다.|
|`cd /dir`|디렉토리를 변경합니다.|
|`chmod [options] mode filename`|파일의 권한을 변경합니다.|
|`chown [options] filename`|파일의 소유권을 변경합니다.|
|`clear`|터미널을 한번 정리합니다. (출력 내용들을 지웁니다.)|
|`cp [options] source destination`|파일과 디렉토리를 복사합니다.|
|`date [options]`|시스템 날짜와 시간을 출력 또는 설정합니다.|
|`df [options]`|사용된/사용가능한 디스크 공간을 출력합니다.|
|`du [options]`|각 파일이 어느정도의 공간을 차지하는지 보여줍니다.|
|`file [options] filename`|파일의 종류를 확인하거나, 변경합니다.|
|`find [pathname] [expression]`|주어진 패턴에 매칭되는 파일을 찾습니다.|
|`grep [options] pattern [filesname]`|주어진 패턴에 일치하는 파일들을 찾거나 출력합니다.|
|`kill [options] pid`|프로세스를 멈춥니다. 강제종료를 위해서는 `-9` 옵션을 사용하세요.|
|`less [options] [filename]`|한 파일의 내용들은 한 페이지에 한번에 보여줍니다.|
|`ln [options] source [destination]`|숏컷을 생성합니다.|
|`locate filename`|특정 파일명에 대해 파일시스템의 복사본을 탐색합니다.|
|`lpr [options]`|인쇄 작업을 전송합니다.|
|`ls [options]`|디렉토리 요소들을 나열합니다.|
|`man [command]`|특정 커맨드에 대한 도움말 정보를 출력합니다.|
|`mkdir [options] directory`|새로운 디렉토리를 생성합니다.|
|`mv [options] source destination`|파일 또는 디렉토리의 이름을 변경하거나 이동합니다.|
|`passwd [name [password]]`|패스워드를 변경하거나 시스템 관리자가 패스워드를 변경하도록 허용합니다.|
|`ps [options]`|현재 실행중인 프로세스들의 스냅샷을 출력합니다.|
|`pwd`|현재 디렉토리의 경로명을 출력합니다.|
|`rm [options] directory`|파일 또는 디렉토리를 삭제합니다.|
|`rmdir [options] directory`|빈 디렉토리를 삭제합니다.|
|`ssh [options] user@machine`|다른 리눅스 머신으로 원격 로그인합니다. `exit`를 통해 ssh 세션을 떠날 수 있습니다.|
|`su [options] [user [arguments]]`|다른 이용자 계정으로 변경합니다.|
|`tail [options] filename`|한 파일의 마지막 `n` 줄만큼을 출력합니다.|
|`tar [options] filename`|`.tar` 또는 `.tar.gz` 또는 `.tgz` 파일들을 압축해제하거나, 압축합니다.|
|`top`|시스템에서 사용 중인 리소스들을 출력합니다. `q`를 눌러 벗어납니다.|
|`touch filename`|특정한 이름을 가진 빈 파일을 만듭니다.|
|`who [options]`|누가 로그인 했는지에 대해 출력합니다.|
