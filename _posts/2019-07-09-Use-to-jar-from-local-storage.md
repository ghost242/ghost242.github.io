---
layout: post
title: Maven에서 Local storage에 있는 jar를 의존성패키지로 사용하는 방법
categories: ["Programming", "Java", "Maven"]
tags: ["java", "maven", "depandencies"]
---

## 문제가 발생했다

회사내에서 개발중인 java 프로젝트 작업을 하던 도중에 발생한 문제였다.

일종의 Library형태로 개발중이던 패키지를 테스트하는 목적으로,

1. 기능을 windows에서 수정하고,
2. 이 패키지를 dependency로 사용하는 별도의 프로젝트를 만들어서
3. runnable jar로 다시 한번 패키지로 만든 뒤에 linux에서 테스트를 하겠다
는 목적으로 뚝딱거리고 있었다.

개발중인 프로젝트는 Maven프로젝트였는데, 그대로 maven repository서버로 올리기에는 우선은 아직 개발중이고, 개발자수준에서 테스트중인 셈이라 아직은 maven서버로 올리지 않을 생각이었다.
이때 그냥 maven package로 jar파일을 만들었을 때 하위 의존성이 걸린 패키지들을 불러오지 못하는 문제가 있었다. 그래서 그 부분에 대한 해결방법을 대충 찾아서 기록해두려고 한다.

## 간단한 참고

* 사용중인 IDE툴: Intellij(Jetbrains)
* 개발 환경: Windows 10
* jdk 버전: 1.8.0_161

## 설정 방법

### jar 패키징

기존 프로젝트를 maven package 명령으로 패키지 한 뒤에 테스트용 프로젝트에서 dependency로 추가하는 경우에 이 jar에 있는 dependency를 제대로 못가져오는 문제가 있다.
따라서 아래와 같이 pom.xml을 일부 수정해서 의존성 패키지들을 불러올 수 있도록 해줘야 한다.

```xml
<build>
    <plugins>
    ...
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <configuration>
                <outputDirectory>
                    ${project.build.directory}/lib
                </outputDirectory>
                <excludeTransitive>false</excludeTransitive>
                <stripVersion>false</stripVersion>
            </configuration>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>2.4</version>
            <configuration>
                <archive>
                    <manifest>
                        <addClasspath>true</addClasspath>
                        <classpathPrefix>lib/</classpathPrefix>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

* 출처: [[Stackoverflow] Including dependencies in a jar with Maven](https://stackoverflow.com/a/49398943) 의 답변 중에서 Leslie Li의 답변

이렇게 하면 jar가 패키지된 폴더에 lib라는 폴더를 새로 만들고, 여기에 이 프로젝트가 의존하는 jar를 몽땅 복사해넣는다.

테스트 프로젝트 생성

Maven 프로젝트를 생성하면 pom.xml이 프로젝트폴더 기준 root에 있을것이다.
이 pom.xml을 로컬 스토리지에 있는 jar를 의존성으로 쓰기위해 아래 내용을 추가한다.

```xml
<dependency>
    <groupId>[[그룹 이름]]</groupId>
    <artifactId>[[프로젝트 이름]]</artifactId>
    <version>[[버전]]</version>
    <scope>system</scope>
    <systemPath>[[jar 파일 위치]]</systemPath>
</dependency>
```

위 작업들을 마무리한 뒤 maven package를 하면 의존성이 걸린 jar파일들은 전부 타겟 jar파일이 있는 폴더의 하위에 lib파일안으로 복사된다. 그리고 테스트용 프로젝트에서 해당 jar를 포함하면 알아서 의존성 패키지를 긁어다 쓴다. 마찬가지로 더 좋은 방법이 있으면, 내용을 좀 더 수정/보완 하기로 한다.
