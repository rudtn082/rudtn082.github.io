---
layout: post
title: "[인사이드 안드로이드] 챕터 4 - JNI와 NDK(2)"
description:
headline:
modified: 2019-04-04
category: Android
tags: [Android]
imagefeature: cover_1.jpg
mathjax:
chart:
comments: true
share: ture
featured: true
---

# [인사이드 안드로이드]


## Chapter4 - JNI와 NDK(2)


---------------------------------------


### JNI 함수 이용하기  

#### JNI 함수를 활용하는 예제 프로그램의 구조  

예제 프로그램의 전체적인 구조는 다음과 같다.  
① 네이티브 메서드가 선언된 JniFuncMain 클래스  
② JniTest 객체  
③ 네이티브 메서드가 실제 구현이 포함된 jnitest.so  

#### 자바측 코드 살펴보기(JniFuncMain.java)  

JniFuncMain 클래스는 다음과 같다.  

![jni5](/images/post/jni5.png "jni5")  

JniTest 클래스는 다음과 같다.

![jni6](/images/post/jni6.png "jni6")  

#### JNI 네이티브 함수의 코드 살펴보기  

JniFuncMain.h 헤더 파일은 javah명령어를 통해 생성한다.

![jni7](/images/post/jni7.png "jni7")  

실제 함수 코드인 jnifunc.cpp는 다음과 같다.  

![jni8](/images/post/jni8.png "jni8")  

cpp파일 까지 작성을 완료한 후 이전 포스트 처럼 .so파일을 생성한다.  
여기서는 c가 아닌 cpp이기때문에 gcc가 아닌 g++로 컴파일을 한다.  

```
g++ -I/(jdk경로)/include/ -I/(jdk경로)/include/linux -shared -fPIC jnifunc.cpp -o libjnifunc.so

sudo mv libjnifunc.so /usr/lib/

java JniFuncMain
```

JniFuncMain의 결과는 다음과 같다.  

![jni9](/images/post/jni9.png "jni9")  

#### 안드로이드에서의 활용 예  
* frameworks/base/core/jni
* frameworks/base/services/jni
* frameworks/base/media/jni

  
  
  
### C프로그램에서 자바 클래스 실행하기  

지금까지는 자바 코드가 메인 프로그램이고 자바 쪽 코드에서 네이티브 메서드를 통해 C 함수를 호출해서 JNI를 이용하는 방식이었다.  
지금부터는 C/C++로 구현된 메인 애플리케이션에서 자바 클래스를 실행하는 JNI이용 방식이다.  

#### C 코드(invocationApi.c) 살펴보기  

자바 가상 머신을 생성하고 실행할 클래스 및 메서드를 받아와 호출하는 코드이다.  

![jni10](/images/post/jni10.png "jni10")  

#### 자바 코드(InvocationApiTest.java) 살펴보기

main() 메서드 하나만 포함한 간단한 클래스이다.  

![jni11](/images/post/jni11.png "jni11")  

#### 컴파일 및 실행

자바 코드 컴파일은 다음과같이 javac를 이용하여 진행한다.  
```
javac InvocationApiTest.java
```

책에서와 달리 우분투에서 진행했기 때문에 C 코드는 .exe 파일이 아닌 .out 파일을 생성한다.  
컴파일시 libjvm.so를 포함해야 하므로 /etc/profile의 LD_LIBRARY_PATH를 수정해준다.  
```
sudo nano /etc/profile
```

export LD_LIBRARY_PATH 부분이 존재한다면 libjvm.so 파일이 존재하는 위치를 추가해준다.  
나는 /usr/lib/jvm/java5/jdk1.5.0_22/jre/lib/amd64/server 위치에 있었기 때문에 다음과 같이 입력했다.  

export LD_LIBRARY_PATH=/usr/lib:/usr/local/lib:/usr/lib/jvm/java5/jdk1.5.0_22/jre/lib/amd64/server  

```
source /etc/profile
```

이제 C파일을 다음과 같은 명령어를 통해 컴파일한다.  
```
gcc -I/(JAVA_HOME)/include/ -I/(JAVA_HOME)/include/linux -L/(libjvm.so 위치) -fPIC invocationApi.c -o invocationApi.out -ljvm
```

나와 같은 경우에는 다음과 같이 입력했다.

```
gcc -I/usr/lib/jvm/java5/jdk1.5.0_22/include/ -I/usr/lib/jvm/java5/jdk1.5.0_22/include/linux -L/usr/lib/jvm/java5/jdk1.5.0_22/jre/lib/amd64/server -fPIC invocationApi.c -o invocationApi.out -ljvm
```

이제 실행 파일인 invocationApit.out을 실행한다.  

```
./invocationApi.out
```

![jni12](/images/post/jni12.png "jni12")  

  
    
#### 안드로이드에서 활용 예: Zygote 프로세스  

5장에서 알아볼 자바 기반의 **Zygote 프로세스**도 **app_process**라는 C++ 기반의 네이티브 애플리케이션에서 JNI 호출 API를 통해 실행된다.  

app_process는 안드로이드 프레임워크가 부팅될 때 안드로이드 런타임을 초기화하고 Zygote 프로세스를 실행하는 역할을 하는 프로세스다. 또한 Zygote는 안드로이드 프레임워크의 성능을 향상 시키기 위한 특별한 프로세스로서 모든 안드로이드 애플리케이션의 프로세스는 Zygote에서 fork된다. 이러한 **Zygote 프로세스는 안드로이드 zygoteInit 클래스라는 자바 프로그램으로 구성돼있다.** 따라서 app_process는 안드로이드가 부팅할 때 JNI 호출 API를 이용해서 자신의 프로그램 영역에 달빅 가상 머신을 로드하고, ZygoteInit 클래스의 main() 메서드를 호출해서 Zygote를 실행한다.  
  
  
  
  
### JNI 네이티브 함수 직접 등록하기  

이전 포스트의 'JNI 기본 원리 이해하기'에서 설명했듯이 자바 가상 머신은 네이티브 메서드를 포함하는 자바 애플리케이션을 실행할 때 아래의 두 단계를 거친다.  
1. System.loadLibrary() 메서드를 이용해서 네이티브 메서드의 실제 구현이 포함된 C/C++ 라이브러리를 메모리 상에 로드한다.  
2. 자바 가상 머신은 위에서 로드된 라이브러리의 함수 심볼을 검색해서 자바에서 선언된 네이티브 메서드의 시그너처와 일치하는 JNI 네이티브 함수 심볼을 찾은 다음 네이티브 메서드와 실제 구현인 JNI 네이티브 함수를 매핑한다.  

JNI는 C/C++ 개발자가 JNI 네이티브 함수를 직접 자바 클래스의 네이티브 메서드에 매핑할 수 있게 해주는 RegisterNatives()라는 JNI 함수를 제공한다. RegisterNatives() 함수를 이용하면 자바 가상 머신이 자동으로 심볼을 검색해서 적장할 JNI 네이티브 함수를 연결하는 작업을 대신해 별도의 매핑 과정을 생략할 수 있어 로딩 속도를 향상시킬 수 있다.  

#### 네이티브 라이브러리 로드 시에 JNI네이티브 함수 등록하기  

이전 포스트에서의 hellojni.c 코드를 자바 가상 머신 대신에 직접 네이티브 메서드와 C 함수를 매핑한 hellojnimap.cpp 코드로 수정해본다.  

우선 System.loadLibrary() 메서드의 동작 방식을 알아보자.  
1. 자바 코드에서 System.loadLibrary() 메서드를 호출하면 자바 가상 머신은 주어진 이름을 가진 공유 라이브러리를 로드한다.  
2. 자바 가상 머신은 로드한 라이브러리 내의 함수 심볼을 검색해서 JNI_OnLoad()라는 함수가 구현돼 있는지 확인하고 라이브러리에 해당 함수가 포함돼 있으면 자동으로 JNI_OnLoad() 함수를 호출한다.  
3. 만약 JNI_OnLoad() 함수가 구현돼 있지 않다면 자바 가상 머신은 자동으로 네이티브 메서드와 라이브러리 내의 JNI 네이티브 함수의 심볼을 비교해서 매핑 작업을 수행한다.  

다음은 hellojnimap.cpp 소스 코드이다.  

![jni13](/images/post/jni13.png "jni13")  
  
  
컴파일 화면은 다음과 같다.  
  
![jni14](/images/post/jni14.png "jni14")  
