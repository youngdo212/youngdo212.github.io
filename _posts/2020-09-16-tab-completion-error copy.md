---
layout: post
title: "터미널 탭 자동 완성 오류 해결"
description: ""
date: 2020-09-16
tags: [linux, aws]
comments: true
---

aws의 ec2 인스턴스(우분투)를 원격 접속 중, 터미널의 탭 자동 완성을 하기 위해 탭 키를 누르면 다음과 같은 문구가 출력되는 문제가 발생하였다.

`bash: cannot create temp file for here-document: 장치에 남은 공간이 없음`

찾아보니 /tmp에 오버플로우가 생겼을 수도 있다는 말이 있었다. /tmp가 어떤 역할을 하는지 몰라서 다시 또 검색해봤다.

/tmp 폴더는 unix와 linux에서 임시 파일을 보관하는 장소(temporary folder라고 불림). 부팅시 또는 규칙적인 간격으로 이 디렉토리가 비워진다고 한다.

- 참고로 /tmp와 /var/tmp 두 종류의 temporary folder가 존재하는데 후자의 경우 더 오래 유지가 필요한 파일을 보관하며 부팅 후에도 유지될 수 있다.

결론적으로 root 파일 시스템 공간이 꽉 찼기 때문에 temp folder 또한 꽉 차있어서 발생하는 문제라고 한다. 디스크 여유 공간을 확인하기 위해 `df -h` 커맨드를 실행하면 다음과 같은 결과를 얻을 수 있다.

```
Filesystem      Size  Used Avail Use% Mounted on
udev            476M     0  476M   0% /dev
tmpfs            98M   11M   88M  11% /run
/dev/xvda1      7.7G  7.7G     0 100% /
tmpfs           490M     0  490M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           490M     0  490M   0% /sys/fs/cgroup
/dev/loop0       29M   29M     0 100% /snap/amazon-ssm-agent/2012
/dev/loop1       97M   97M     0 100% /snap/core/9804
/dev/loop2       97M   97M     0 100% /snap/core/9665
/dev/loop3       18M   18M     0 100% /snap/amazon-ssm-agent/1566
tmpfs            98M     0   98M   0% /run/user/1000
```

여기서 루트 파일 시스템인 /dev/xvda1을 보면 7.7G 용량을 전부 사용한 것을 알 수 있다. 참고로 /dev/xvda1은 우분투의 가상 저장 장치 Xen Virtual block Device로서 xvd**a1**의 a1은 가장 첫 번째 파티션을 나타낸다. 

이 문제를 해결하기 위해서 ec2 인스턴스의 디스크 용량을 늘리기로 했다. 다음과 같은 순서로 진행된다.

1. [EBS(Elastic Block Store) 볼륨 확장하기](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html)
2. [볼륨의 파일 시스템 확장하기](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html#extend-file-system)
    - 파일 시스템 볼륨 확장시에도 tmp dir의 공간 부족 문제가 발생하였다. [해결법](https://aws.amazon.com/ko/premiumsupport/knowledge-center/ebs-volume-size-increase/)

위의 방법대로 인스턴스의 볼륨을 늘려 문제를 해결하였다.

## 참고자료

[https://unix.stackexchange.com/questions/277387/tab-completion-errors-bash-cannot-create-temp-file-for-here-document-no-space](https://unix.stackexchange.com/questions/277387/tab-completion-errors-bash-cannot-create-temp-file-for-here-document-no-space)

[https://en.wikipedia.org/wiki/Temporary_folder#:~:text=In Unix and Linux%2C the,tmp and %2Fvar%2Ftmp.&text=Typically%2C %2Fvar%2Ftmp is,is for more temporary files](https://en.wikipedia.org/wiki/Temporary_folder#:~:text=In%20Unix%20and%20Linux%2C%20the,tmp%20and%20%2Fvar%2Ftmp.&text=Typically%2C%20%2Fvar%2Ftmp%20is,is%20for%20more%20temporary%20files).

[https://askubuntu.com/questions/166083/what-is-the-dev-xvda1-device#:~:text=Virtual storage devices%2C representing cloud,SCSI-like storage device](https://askubuntu.com/questions/166083/what-is-the-dev-xvda1-device#:~:text=Virtual%20storage%20devices%2C%20representing%20cloud,SCSI%2Dlike%20storage%20device)).

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modify-volume.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modify-volume.html)

[https://aws.amazon.com/ko/premiumsupport/knowledge-center/ebs-volume-size-increase/](https://aws.amazon.com/ko/premiumsupport/knowledge-center/ebs-volume-size-increase/)