# Ubuntu 16.04에 EtherCAT master 설치하기

[EtherLAB홈페이지의 PDF](http://www.etherlab.org/download/ethercat/ethercat-1.5.2.pdf)와 [Etherlab Users Q&A](http://lists.etherlab.org/pipermail/etherlab-users/2015/002820.html)를 참고했습니다.

## 1.	Ubuntu 16.04.1 LTS 설치하기
EtherCAT-1.5.2 설치는 리눅스 커널 버전에 큰 영향을 받는 것 같습니다. 4.4.x-generic 버전의 리눅스 커널에서 정상적으로 동작하는 것을 확인했습니다.   
[Ubuntu 16.04.1 LTS](http://old-releases.ubuntu.com/releases/xenial/)에서 자신의 컴퓨터에 맞는 이미지를 다운받아 설치합니다.

## 2.	m4, autoconf, automake 설치하기
EtherCAT-1.5.2를 빌드할 때 쓰이는 툴을 먼저 설치해 주어야 합니다.
터미널에 다음 명령어를 입력해서 설치했습니다.

```
$ sudo apt-get install m4
$ sudo apt-get install autoconf
$ sudo apt-get install automake
```

## 3.	ethercat-1.5.2.tar 다운로드 하기
Etherlab 홈페이지에 올라와 있는 압축파일은 리눅스 커널 4.4.x 버전에는 맞지 않는 코드들이 있습니다. 맞지 않는 부분들을 수정하여 github에 올려놓았고 다음의 명령어로 다운로드 받으면 됩니다.   
```
$ wget ~~~~수정하기
```

## 4.	압축을 풀고 /usr/local/src/로 디렉터리 이동시키기
```
$ tar -xvf ethercat-1.5.2.tar
$ sudo mv ethercat-1.5.2 /usr/local/src/
$ cd /usr/local/src/
```
ethercat이라는 링크를 생성합니다.
```
$ sudo ln -s ethercat-1.5.2 ethercat
$ cd ethercat
```

## 5.	./configure & make 하기
Default로 8139too 드라이버를 이용하는 것으로 되어있는데, 4.4.x-generic커널에서는 generic드라이버를 이용해야 정상적으로 스크립트가 실행됩니다.   
```
$ ./configure --enable-generic and --disable-8139too
$ sudo make
$ sudo make modules
$ sudo make install
$ sudo make modules_install
$ sudo depmod
```
make modules 를 실행했을 때 warning이 뜨지만 큰 문제 없었습니다.

## 6.	/etc/sysconfig/ethercat 에 MAC Address 기록하기
```
$ sudo mkdir /etc/sysconfig/
$ sudo cp /opt/etherlab/etc/sysconfig/ethercat /etc/sysconfig/
```
텍스트 에디터를 이용하여 /etc/sysconfig/ethercat을 수정해야 합니다.
```
$ sudo vi /etc/sysconfig/ethercat
```

다른 터미널에서 ifconfig 명령어를 이용해서 유선 랜포트의 MAC address를 알아냅니다.   
**MASTER0_DEVICE=“??:??:??:??:??:??"**   
이 항목을 찾아 “??:??:??:??:??:??”에 찾은 MAC address를 기입합니다.   
**DEVICE_MODULES=“generic"**   
아래쪽에 떨어져 있어서 놓치기 쉽습니다. generic 드라이버를 썼으므로 “generic”을 기입합니다.

## 7.	initialization script 복사하기
```
$ sudo cp /opt/etherlab/etc/init.d/ethercat /etc/init.d/
$ sudo chmod a+x /etc/init.d/ethercat
```
보드 실행시 EtherCAT을 켜고 싶다면 다음 명령어를 사용하면 됩니다.
```
$ sudo update-rc.d ethercat defaults
```

## 8.	ethercat tool 이라는 CLI툴 사용 가능하게 하기
```
$ sudo ln -s /opt/etherlab/bin/ethercat /usr/local/bin/ethercat
```
텍스트 에디터로 /etc/udev/rules.d/99-EtherCAT.rules를 만들어 편집합니다.   
```
$ sudo vi /etc/udev/rules.d/99-EtherCAT.rules
```
다음 줄을 입력하고 저장합니다.   
**KERNEL=="EtherCAT[0-9]*", MODE="0664"**

## 9.	sudo /etc/init.d/ethercat start 명령어를 이용하여 EtherCAT 실행
```
$ sudo /etc/init.d/ethercat start
$ sudo /etc/init.d/ethercat restart
$ sudo /etc/init.d/ethercat stop
```
등의 명령어가 사용가능합니다.

```
$ ethercat master
```
을 입력하여 정상 실행을 확인합니다.

## 참고 링크

전반적인 가이드   
http://lists.etherlab.org/pipermail/etherlab-users/2015/002820.html 

EtherLAB 홈페이지   
https://etherlab.org/


./configure 옵션 관련   
https://etherlab-users.etherlab.narkive.com/H9VjgQsW/linux-distribution-for-igh-ethercat-master

make 또는 make modules 할때 생기는 에러 해결법  
https://etherlab-users.etherlab.narkive.com/TrOErKh4/make-module-errors
https://etherlab-users.etherlab.narkive.com/nGJzHwXR/ethercat-master-error-make-modules 



make modules 도중 sock_create_kern 에러 해결법   
https://www.mail-archive.com/etherlab-users@etherlab.org/msg02694.html