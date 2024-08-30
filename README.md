# Frida를 활용한 앱 함수 후킹

## Frida란?
앱의 동적 분석과 조작을 위한 강력한 디버깅 및 프로그래밍 도구이다.


## 사용한 도구
- Jadx
- Nox
- Adb
- Frida

## 수행 및 결과

### Frida-Sever 준비

- 녹스 에뮬레이터 접속

```
nox_adb shell 
//nox플레이 에뮬레이터 쉘에 접속

pm list packages -f 
// pm -> Package Manager 
// -f는 각 애플리케이션의 APK 파일 경로를 함께 출력
```

- frida-server를 Nox에뮬레이터로 옮기기 (Desktop 쉘)

```
//로컬 cmd에서 nox_adb push 를 이용해여 frida-server넣기
//frida-server가 있는 위치로 이동후 실습 진행
// cmder를 사용하여 윈도우에서도 리눅스 명령어 사용가능 한 것임..


ls -al |grep frida
// frida-server 있는지 확인

nox_adb push frida-server-16.2.1-android-x86_64  /sdcard
// nox 에뮬레잍터 안으로 local에 있는 frida-server파일 옮기기

```
![Untitled](https://github.com/user-attachments/assets/85f5ef0b-68b3-4432-92fc-91983564f5a7)


![Untitled (1)](https://github.com/user-attachments/assets/23073e1e-b786-47e8-8c87-07a6d9b9834e)




- 권한 설정 (Nox 에뮬레이터)

```
// nox 에뮬레이터 쉘

cd sdcard
// frida-server 파일 넣은 위치로 이동

ls
// 잘 넣어졌는지 확인

mv frida-server-16.2.1-android-x86_64 /data/local/tmp/
// frida-server파일을 /data/local/tmp 파일로 이동

cd /data/local/tmp
// frida-server파일 옮긴 곳으로 이동

chmod 777  frida-server-16.2.1-android-x86_64
// frida-server파일 권한 777로 설정

ls -al
// 권한 변경 되었는지 확인

 ./frida-server-16.2.1-android-x86_64 &
 // 백그라운드로 frida-server 실행

```

![Untitled (2)](https://github.com/user-attachments/assets/fd7eb298-473f-4c40-9996-87e6c538f2d1)


![Untitled (3)](https://github.com/user-attachments/assets/5625e0a3-396c-48a7-97a3-ee696f37f0b0)


![Untitled (4)](https://github.com/user-attachments/assets/9e16f214-fdb5-4f20-9d13-b0442c6f84f0)


![Untitled (5)](https://github.com/user-attachments/assets/03de843a-7af6-42f7-b874-f87771ff9b9a)


### 함수 후킹

```
// 에뮬레이터 쉘

pm list package
// 설치된 패키지 들을 모두 반환한다.

pm list package -f
// 설치된 패키지의 위치까지 반환한다.
// 미리 옮겨논 Rootbeer라는 패키지 위치를 확인한다.

pm list package -f | grep mino
// mino hunter라는 앱을 후킹 하기위해 위치를 기억한다.
```

![Untitled (6)](https://github.com/user-attachments/assets/8d8d9483-120c-42e1-a442-12ebfb20782f)


![Untitled (7)](https://github.com/user-attachments/assets/fe7518d8-c940-4855-9fa1-1d47bec93cc5)



![Untitled (8)](https://github.com/user-attachments/assets/dfac42c2-1455-4304-9770-655e4a2fcb19)



```
// window Local PC 쉘

nox_adb pull /data/app/ksy.useworlds.minohunter-yP76JWNpF_eWZCClrvrFpA==/base.apk
// 에뮬레이터 쉘에서 확인한 mino 패키지위치를 기입하는데 ~~.apk까지만 기입해준다.

```

![Untitled (9)](https://github.com/user-attachments/assets/8ab2729d-05f2-43cf-94bd-f6353ec8ff32)



-  jadx를 키고 apk파일을 열어 코드를 분석해본다. (Desktop)

![Untitled (10)](https://github.com/user-attachments/assets/37895f19-b4c9-4818-8f55-4acdb10d38b7)


![Untitled (11)](https://github.com/user-attachments/assets/5a926bc6-217e-4f52-95f8-ef0db1bfa788)


> 코드 분석을 하다보면 Mino클래스의 nextStage라는 메소드를 볼 수 있다.

![Untitled (12)](https://github.com/user-attachments/assets/2f207bdf-942d-4127-a31e-3a300ce51c17)


> vscode에 복사해준후 java로 만들어진 앱이란것을 위해 아래와 같이 작성해준다.

```
// test3.js
Java.perform(() => { //자바로 개발된 앱이기 때문에 넣어줘야 하는 코드이다.
    let Mino = Java.use("ksy.useworlds.minohunter.Mino");//객체를 넣어준다.
    Mino["nextStage"].implementation = function () {
        console.log(`Mino.nextStage is called`);
        console.log(this.stage.value);
        this.stage.value=999; //이렇게 후킹해준다.
        this["nextStage"]();
        console.log(this.stage.value);
    };
});
```

```
// 윈도우 쉘
frida -U -f ksy.useworlds.minohunter -l test3.js
// frida를 이용해서 mino hunter를 실행시키고 후킹할 코드도 함께 넣어서 실행해준다.
```

![Untitled (13)](https://github.com/user-attachments/assets/057b464d-3d7b-4a6c-a3ef-9e7b70425c67)

함수 후킹을 통해 스테이지를 건너뛰었다.

![Untitled (14)](https://github.com/user-attachments/assets/34e46d24-0227-4f2a-8b67-27ce3cd1c4e7)

