---
layout: post
title: Ubuntu 22.04 에서 절전모드와 최대절전모드를 활성화 하는 방법
category: ["Linux", "Utility"]
tag: ["Linux", "Ubuntu", "Power management", "Hibernate"]
---

## 절전 모드, 최대 절전 모드, 시스템 종료에 대해서

`절전 모드`와 `최대 절전 모드`는 한국어로 보면 의미를 알기 힘들지만, 이 둘은 완전히 다르다고 볼 수 있다. 현대의 OS 환경에는 ACPI(Advanced Configuration and Power Interface) 라고 하는 장치를 통해 전원을 제어하고있다. 하드웨어 밴더사에서 device 형태로 제공하는 기능을 OS에서 driver를 통해 다루고있는데, 마이크로소프트에서 문서로 정리해둔 전원 관리 상태를 적당히 요약하면 이런 내용을 볼 수 있다.

### 절전 모드(Suspend - S3)

사용중인 상태를 기준으로, 현재 프로세스 상태를 메모리에 기록하고 메모리를 유지하기위한 전력을 제외한 대부분의 장치에 공급되는 전원을 차단한다. 사용자가 시스템을 다시 사용하기위해 입력장치를 통해 기기를 깨우면(Wake up), 시스템은 다시 장치에 전원을 공급하고 이전처럼 사용할 수 있게된다.

### 최대 절전 모드(Hibernate - S4)

마찬가지로 사용중인 상태를 기준으로 할때, 현재 프로세스 상태를 디스크에 기록하고 CPU, 메모리를 포함한 모든 기기에 전원 공급을 차단한다. 이 상태를 기록하기위해 OS에서는 메모리 스왑 영역을 따로 확보해야 하고 전원 공급이 차단되기까지 시간이 절전 모드(Suspend) 보다 오래 걸린다. 마찬가지로 사용자가 시스템을 다시 사용하기 위해 최종 메모리 상태를 다시 복원(Resume)하면, 시스템은 디스크에 기록되어있던 스냅샷을 다시 메모리에 로드하고 작업 환경을 복원한다.
절전 모드(Suspend)와 다르게 메모리를 유지하기위한 전원공급이 없기 때문에 전력 소모량이 훨씬 적다.

### 시스템 종료(Power off/Shutdown - S5)

현재 메모리에 있는 프로세스를 전부 종료하고 OS까지 완전히 종료된다. 위의 두 모드와 다르게 현재 작업 상태가 저장되지 않기 때문에 사용자가 시스템을 다시 사용할 때는 부트업이 필요하고 초기 상태에서 시작하게된다.

## 설정하는 방법

여기에서 하는 설정은 지금 사용중인 환경을 기준으로 한다.

* Dell inspiron 15 3525 (AMD ryzen 5850U + 32GB memory)
* Ubuntu 22.04(Kernel 5.15.0-76-generic)

그리고 `Hibernate` 설정은 다음 단계를 거쳐 진행하게 된다.

1. `Hibernate` 기능 지원 여부 확인
2. `Hibernate image`를 만들기 위한 디스크 공간 확보
3. 시스템 부트업시 `grub` 실행 명령에서 불러올 `Hibernate image` 디스크 위치 지정
4. 시스템에 `Hibernate` 기능 활성화

### 시스템에서 지원하는 전원 관리 기능 확인

설정하기 전에 어떤 기능을 지원하는지 확인해야 한다.

```shell
$ cat /sys/power/state
freeze mem disk
```

현재 시스템에서 터미널을 통해 위 명령을 입력하면 이런 결과가 출력된다. 이것은 이 시스템이 `freeze`, `mem`, `disk` 상태로 변경이 가능하다는 의미인데, 이 세가지 상태는 대충 이런 상태를 의미한다.

* freeze: Suspend To Idle(S0); 시스템에서 전원을 많이 소모하는 장치에 전원공급을 중단하는 저전력 대기상태라고 볼 수 있는데, 그냥 PC가 켜져있는 것으로 생각해도 될것 같다.
* mem: 이 상태는 같은 디렉토리 위치에 있는 `/sys/power/mem_sleep` 파일에 있는 상태로 전환한다는 의미인데, 이 파일의 내용을 출력하면 이렇게 나온다.

    ```shell
    $ cat /sys/power/mem_sleep
    [s2idle]
    ```

    만약 이 시스템이 S3를 지원한다면 여기에 `[s2idle] deep` 이라고 내용이 출력될 것이다. 현재 사용중인 시스템은 S3를 지원하지 않는다는 것을 알 수 있다.
* disk:  이 상태는 같은 디렉토리 위치에 있는 `/sys/power/disk` 파일에 있는 상태로 전환한다는 의미이고, 위와 마찬가지로 내용을 출력하면 이렇게 나온다.

    ```shell
    $ cat /sys/power/disk
    [platform] shutdown reboot suspend test_resume 
    ```

    현재 disk의 상태는 `platform` 이라는 상태를 가리키고있다는 의미인데, 이 상태가 S4인 `Hibernate`를 의미한다. 그 외에 다른 상태는 이름 그대로인데, 여기에서 `suspend` 상태가 있는 것을 볼 수 있다. linux kernel 문서에서 해당 내용을 찾아보면 "Hybrid system suspend. Put the system into the suspend sleep state selected through the `mem_sleep` file described above." 라는 내용으로 설명하고 있는데, 내용 아래에는 `suspend` 상태를 지원하는 시스템에서만 유효하다 라는 설명이 같이 있다. 이 시스템은 당연히 S3를 지원하지 않기 때문에 사용할 수 없다.

종합해보면 현재 사용중인 시스템은 총 3가지 상태(S0, S4, S5)를 기능으로 제공하고있음을 알 수 있다.

### 메모리 스왑 설정

디스크에 스왑 공간을 만들어줘야 한다. 리눅스는 설치 과정에서 디스크 공간에 적당히 스왑 공간을 잡는데, `/swapfile` 이라는 이름으로 관리된다. 스왑 설정을 확인하고 싶다면 이런 명령을 사용할 수 있다.

```shell
$ swapon --show
NAME       TYPE  SIZE USED PRIO
/swapfile  file  2.0G   0B   -2
```

파일 타입으로 스왑 공간이 확보되어있고 현재는 사용되고있지 않음을 알 수 있다. 크기는 2GB 공간만큼 확보되어있다. 이제 `Hibernate`를 위한 디스크 공간을 확보해야 한다.

#### Hibernate image

시스템이 `Hibernate` 상태를 기록하기 위한 이미지 크기를 확인해야 한다.

```shell
$ cat /sys/power/image_size
13132705792
```

#### Disk block size

사용중인 파일시스템이 설정하고있는 디스크 블록의 크기를 확인한다. 이것은 스왑 이미지가 파일 형태로 디스크에서 관리되기 때문에 확인이 필요하다.

```shell
$ df
Filesystem     1K-blocks      Used Available Use% Mounted on
tmpfs            3218944      2396   3216548   1% /run
/dev/nvme0n1p3 481918488 130174404 327190460  29% /
...

$ blkid
/dev/nvme0n1p3: LABEL="UBUNTU" UUID="12345678-abcd-1a2b-asdf-qwerasdfzxcv
" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="DISK PARTITION UUID" 
...
```

이 시스템은 ext4 파일시스템에 디스크 블록 크기를 4096B 로 관리하고 있음을 알 수 있다.

#### 디스크 공간 생성

Linux에서 일정 크기의 디스크 공간을 파일 형태로 확보할 필요가 있다. 그렇게 해야 시스템이 메모리 상태를 디스크에 옮길 수 있다. 이 공간은 스왑 파일과 동일한 형태로 만들어진다.

```shell
$ sudo dd if=/dev/zero of=/hibernate bs=4096 count=3206227
...
```

시간이 좀 걸리긴 하지만 파일이 하나 만들어진다. 파일 전체 크기는 `4096B * 3206227B = 13132705792B` 만큼 확보된다. 이 파일을 스왑 공간으로 정의해준다.

#### Swap enable

위에서 만든 파일을 스왑 공간으로 지정하기 위해 블록 헤더를 생성하고 시스템에 메모리 스왑 공간임을 선언해줄 필요가 있다. 우선 생성한 파일을 스왑 공간으로 만든다.

```shell
$ mkswap -U /hibernate
mkswap: /hibernate: insecure permissions 0644, fix with: chmod 0600 /hibernate    
Setting up swapspace version 1, size = 12.2 GiB (13132705792 bytes)                 
no label, UUID="swapfile mount UUID"
$ swapon /hibernate
```

이 공간은 `mount` 명령으로 지정하지 않았지만, 스왑 공간으로 마운트가 되어있다. 그래서 시스템이 시작될 때 항상 마운트 설정을 할 수 있게 `fstab` 파일에 내용을 추가해줘야 한다. 시스템에서 이미 swapfile을 하나 만들어서 스왑 공간으로 지정하고있기 때문에 해당 라인을 복사해서 여기서 만든 스왑 공간의 파일이름을 넣어주면 된다.

```shell
$ sudo echo "/hibernate                                 none            swap    sw              0       0" >> /etc/fstab
```

이제 시스템이 재시작 되더라도 항상 `/hibernate` 파일을 스왑 공간으로 마운트 하게된다.

### GRUB 설정 변경

Ubuntu 22.04는 부트로더로 `GRUB`를 사용하고 있기 때문에, 기본 부트로더인 `GRUB`가 실행될 때 명령어로 `Hibernate image`가 있을 때 이 이미지를 불러오도록 설정해줄 수 있다. 이 작업은 최초의 부트 과정에서 커널 이미지를 파일시스템에 로드할 때 실행되어야 하기 때문에 우리가 알고있는 `/hibernate`라는 파일 이름을 사용할 수 없다. 따라서 디스크에 기록되어있는 물리위치를 알아야 한다.

#### 디스크의 UUID 확인

처음 할 일은 `/hibernate` 파일이 저장되어있는 볼륨 마운트의 UUID 를 알아내는 것이다. 

```shell
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0   9.6M  1 loop /snap/canonical-livepatch/229
...
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   950M  0 part /boot/efi
├─nvme0n1p2 259:2    0     8G  0 part /var/snap/firefox/common/host-hunspell
└─nvme0n1p3 259:3    0   468G  0 part /
```

우선은 루트 디렉토리인 `/` 가 어떤 파티션과 마운트되어있는지 알아야 한다. 이 시스템에서는 위 명령을 이용해서 `nvme0n1p3`이 목표로 하는 디스크 파티션임을 알 수 있다. 다음은 디스크 파티션의 UUID이다.

```shell
$ blkid
/dev/nvme0n1p3: LABEL="UBUNTU" UUID="12345678-abcd-1a2b-asdf-qwerasdfzxcv
" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="DISK PARTITION UUID" 
...
```

위에서 같은 명령으로 디스크의 블록 사이즈를 확인했는데, 이번에는 디스크의 UUID인 `12345678-abcd-1a2b-asdf-qwerasdfzxcv` 값이 필요하다.

#### 스왑 파일의 섹터 시작위치 확인

이제 `/hibernate` 파일의 물리위치를 확인해야 한다. 이 지점은 이런 명령어를 이용해서 알 수 있다.

```shell
$ filefrag -v /hibernate | head
Filesystem type is: ef53
File size of /hibernate is 12824908000 (3131082 blocks of 4096 bytes)
 ext:     logical_offset:        physical_offset: length:   expected: flags:
   0:        0..   20479:   27930624..  27951103:  20480:            
   1:    20480..   53247:   27885568..  27918335:  32768:   27951104:
   2:    53248..   86015:   27852800..  27885567:  32768:   27918336:
   3:    86016..  112639:   27826176..  27852799:  26624:   27885568:
   4:   112640..  145407:   27754496..  27787263:  32768:   27852800:
   5:   145408..  178175:   27721728..  27754495:  32768:   27787264:
   6:   178176..  210943:   27688960..  27721727:  32768:   27754496:
```

`filefrag`는 디스크 상에서의 파일 위치를 추적해서 보여주는 프로그램이다. 여기서 `logical_offset`이 OS에서 제공하는 File description으로 다룰 수 있는 파일의 위치이고, 이 위치와 대응하는 물리위치가 `physical_offset` 컬럼에 있는 값이다. 여기서는 이 파일의 시작지점을 알아야 하기 때문에 `logical_offset`의 0에 해당하는 `27930624` 값을 가져온다.

#### GRUB에 설정값 기입

`Hibernate` 단계에서 시스템을 복원하는 `resume` 작업을 하기 위해 부트로더에게 어떤 위치를 참조해야 하는지 알려줘야 한다. 그 설정은 `/etc/default/grub` 파일에 `GRUB_CMDLINE_LINUX_DEFAULT` 값을 수정하는 것이다.

```text
...
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=12345678-abcd-1a2b-asdf-
qwerasdfzxcv resume_offset=27930624"                                            
...
```

처음 설치하고 나면 "quiet splash"만 적혀있는데, 두개는 각각의 의미를 갖는 명령어이다. 여기에 디스크의 UUID를 `resume=UUID=`로 적고, 파일의 물리 위치를 `resume_offset=`으로 추가해주는 것이다. = 뒤에 각각에 해당하는 값이 들어가게 된다.

그리고 GRUB 설정을 업데이트해준다.

```shell
$ sudo update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Sourcing file `/etc/default/grub.d/oem-flavour.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-76-generic
Found initrd image: /boot/initrd.img-5.15.0-76-generic
Found linux image: /boot/vmlinuz-5.15.0-75-generic
Found initrd image: /boot/initrd.img-5.15.0-75-generic
Found linux image: /boot/vmlinuz-5.14.0-1027-oem
Found initrd image: /boot/initrd.img-5.14.0-1027-oem
Memtest86+ needs a 16-bit boot, that is not available on EFI, exiting
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

이런 결과값이 출력되고 나면 위에서 수정한 값이 정상적으로 grub 설정에 반영된다. 그리고 이제 부트업시 커널 이미지를 초기화할 때 사용하는 `ramfs` 도 수정해줄 필요가 있다.

```shell
$ cat /etc/initramfs-tools/conf.d/resume

```

아무 것도 추가하지 않았기 때문에 아직 내용이 없다. 여기에 `/etc/default/grub` 파일에 추가했던 내용을 똑같이 옮겨 적는다.

```shell
$ echo "resume=UUID=12345678-abcd-1a2b-asdf-qwerasdfzxcv resume_offset=27930624" >> /etc/initramfs-tools/conf.d/resume
```

이제 `GRUB` 설정과 마찬가지로 업데이트를 통해 이미지를 수정해준다.

```shell
$ update-initramfs -k all -u
update-initramfs: Generating /boot/initrd.img-5.15.0-76-generic
...
```

이제 `Hibernate` 실행시 정상적으로 이미지를 디스크에 기록하고, 다시 불러올 수 있게 되었다.

### Hibernate 기능 활성화

여기에서는 노트북 덮개 스위치 장치에 대한 설정을 지정해서 사용자가 노트북 덮개를 덮었을 때 `Hibernate` 모드로 변경하도록 설정한다. 노트북에 따라서는 이 기능을 OS 설정에서 활성화 하더라도 실제로 적용되지 않을 수 있다. 어디까지나 하드웨어 밴더를 다룰 수 있는 드라이버가 있는 경우 사용이 가능하다.

이 설정은 `/etc/systemd/logind.conf` 파일에서 수정할 수 있다.

```text
...
[Login]
...
#HandleLidSwitch=hibernate
#HandleLidSwitchExternalPower=ignore
#HandleLidSwitchDocked=ignore
...
```

이 파일 내용은 전체적으로 주석처리되어있는데, 이 중에서 `HandleLidSwitch` 라인의 "\#"을 제거해서 활성화해준다. 여기에 넣을 수 있는 값이 몇가지 있는데, 각각 `ignore`, `suspend`, `hibernate`, `poweroff` 이렇게 세가지 값이다. 지금까지 `hibernate` 모드를 설정했는데 당연히 여기서도 덮개를 닫았을 때 `hibernate` 모드로 진입하도록 수정해야 한다. 추가로 외부 전원이 연결되어있는 경우에도 `hibernate` 모드가 실행되도록 하고싶다면 그 아래 라인도 활성화해주고 `ignore`를 `hibernate`로 바꿔준다.

## 결론

Windows 환경에서는 `\[전력 관리\]` > `\[고급 설정\]`에서 `덮개를 닫을 때` 동작을 `최대절전모드`로 바꿔주면 위의 모든 과정을 알아서 해준다. 하지만 리눅스에서 이 기능을 이해하고 정리하는데 3개월이나 걸렸다. 시스템을 초기화할 때마다 헷깔리는 내용이라서 전체적으로 정리해봤는데, 리눅스란 워낙 날것같은 운영체제이고 배포판, 버전, 시스템 환경 등등에 따라 추가되거나 빠져야 할 단계들이 있다. 지금은 이 노트북을 기준으로만 작성했지만, 필요한 정보는 계속해서 찾아서 내용을 정리하고 빠진 내용이 있다면 추가해 나갈 예정이다.

## 참고

* [How to Enable Hibernate Function in Ubuntu 22.04 LTS](https://ubuntuhandbook.org/index.php/2021/08/enable-hibernate-ubuntu-21-10/)
    | 가장 처음 찾아서 참고했던 자료이다. 이 문서에서 작성한 단계도 이 링크의 내용을 바탕으로 필요한 내용들을 추가했다.
* [The linux kernel - System sleep states](https://www.infradead.org/~mchehab/kernel_docs/admin-guide/pm/sleep-states.html)
* [How to Hibernate Ubuntu: A Step-by-Step Guide](https://www.linuxandubuntu.com/home/how-to-enable-hibernate-in-ubuntu-linux)
* [Power management/Suspend and hibernate](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate)