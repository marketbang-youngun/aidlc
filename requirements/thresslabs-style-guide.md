# Thresslabs Java/Spring Boot 코딩 스타일 가이드

> thresslabs-backend-master 프로젝트 실제 소스 기반 분석. Spring Boot 3.5.10 / Java 17 기준.

---

## 1. 기술 스택

| 항목 | 버전 |
|------|------|
| Java | 17 |
| Spring Boot | 3.5.10 |
| 빌드 도구 | Gradle |
| ORM | Spring Data JPA + QueryDSL 5.1.0 (jakarta) |
| MyBatis | 3.0.3 |
| DB | MySQL 8 (Master/Replica 분리, HikariCP) |
| 캐시 | Redis (Redisson 3.29.0 + commons-pool2) |
| 인증 | JWT Cookie 방식 (Auth0 java-jwt 4.4.0) |
| 검색 | Elasticsearch |
| 파일 스토리지 | AWS S3 + CloudFront |
| API 문서 | Springdoc OpenAPI 2.8.5 (Boot3 대응) |
| 코드 생성 | Lombok |
| 이메일 | Spring Mail (Gmail SMTP) |
| 알림 | Slack API (bolt 1.44.2) |
| 세금계산서 | Popbill (linkhub 1.15.1) |
| XSS 방지 | OWASP AntiSamy 1.7.5 |
| OTP | Google Authenticator (warrenstrange 1.5.0) |
| Excel | Apache POI 5.3.0 |
| 결제 | NicePay, PayU |
| 알림톡 | 카카오 |

---

## 2. build.gradle

```gradle
plugins {
    id 'org.springframework.boot' version '3.5.10'
    id 'io.spring.dependency-management' version '1.1.5'
    id 'java'
}

group = 'com'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

dependencies {
    // Spring Boot Starters (버전 명시 없음 - BOM 관리)
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-mail'
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'org.springframework.boot:spring-boot-starter-websocket'
    implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.session:spring-session-core'
    runtimeOnly 'com.mysql:mysql-connector-j'

    // Thymeleaf Extras (버전 명시 필요)
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6:3.1.2.RELEASE'
    implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect:3.2.1'

    // Swagger (Boot3 대응 - springdoc v2)
    implementation "org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.5"

    // QueryDSL (Boot3: jakarta 필수)
    implementation 'com.querydsl:querydsl-jpa:5.1.0:jakarta'
    implementation 'com.querydsl:querydsl-core:5.1.0'
    annotationProcessor 'com.querydsl:querydsl-apt:5.1.0:jakarta'
    annotationProcessor 'jakarta.persistence:jakarta.persistence-api:3.1.0'
    annotationProcessor 'jakarta.annotation:jakarta.annotation-api:2.1.1'

    // MyBatis (Boot3 대응)
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'

    // AWS (Boot3 대응)
    implementation 'io.awspring.cloud:spring-cloud-aws-starter:3.1.1'
    implementation 'software.amazon.awssdk:s3:2.28.21'
    implementation 'software.amazon.awssdk:cloudfront:2.28.21'
    implementation 'software.amazon.awssdk:apache-client:2.28.21'

    // JWT
    implementation 'com.auth0:java-jwt:4.4.0'

    // ModelMapper
    implementation 'org.modelmapper:modelmapper:3.2.0'

    // XSS 방지
    implementation 'org.owasp.antisamy:antisamy:1.7.5'

    // Excel
    implementation 'org.apache.poi:poi-ooxml:5.3.0'

    // Google OTP
    implementation 'com.warrenstrange:googleauth:1.5.0'

    // Slack
    implementation 'com.slack.api:bolt:1.44.2'

    // Redis 확장
    implementation 'org.redisson:redisson-spring-boot-starter:3.29.0'
    implementation 'org.apache.commons:commons-pool2:2.11.1'

    // 링크허브 (팝빌 - 알림톡, 세금계산서)
    implementation 'kr.co.linkhub:popbill-spring-boot-starter:1.15.1'

    // Lombok
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // Devtools
    developmentOnly 'org.springframework.boot:spring-boot-devtools'

    // Test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
    enabled = false  // 기본 비활성화. 실수로 실행되면 안 되는 코드들이 있음
}
```

> **Boot3 핵심 변경점**
> - QueryDSL: `:jakarta` classifier 필수
> - Springdoc: `springdoc-openapi-starter-webmvc-ui` (v2.x)
> - `javax.*` → `jakarta.*` 전환 완료

---

## 3. 패키지 구조

```
src/main/java/com/threelabs/
├── config/
│   ├── db/                  # DataSourceConfig, MasterJpaConfig, ReplicaJpaConfig
│   ├── security/
│   │   ├── filter/          # JwtAuthenticationFilter, JwtAuthorizationFilter, CorsLogFilter
│   │   ├── handler/         # CustomLoginSuccessHandler, CustomLoginFailureHandler
│   │   │                      CustomAuthenticationEntryPoint, CustomAccessDeniedHandler
│   │   ├── provider/        # JwtTokenProvider, JwtTokenConstants
│   │   ├── request/         # JwtLoginRequest
│   │   ├── user_details/    # CustomUserDetails, CustomUserDetailsService
│   │   ├── SecurityConfig.java
│   │   └── JwtConfig.java
│   ├── querydsl/            # JPAQueryFactory 빈 설정
│   ├── redis/               # Redis/Redisson 설정
│   ├── log/                 # 로깅 설정
│   ├── socket/              # WebSocket 설정
│   ├── swagger/             # Springdoc 설정
│   ├── cookie/              # 쿠키 설정
│   └── valid/               # 커스텀 Validation
├── controller/
│   ├── admin/               # 관리자 API (/api/admin/v1/**)
│   │   ├── banner/
│   │   ├── stat/
│   │   └── system/
│   ├── buyer/               # 점주 API (/api/buyer/v1/**)
│   ├── seller/              # 수입사 API (/api/seller/v1/**)
│   ├── common/              # 공통 API (/api/common/v1/**)
│   │   ├── chat/
│   │   ├── code/
│   │   └── info/
│   └── nicepay/             # 결제 콜백
├── service/
│   ├── admin/
│   ├── buyer/
│   ├── seller/
│   ├── common/
│   ├── banner/
│   ├── chat/
│   ├── goods/
│   ├── order/
│   ├── payment/
│   ├── coupon/
│   ├── point/
│   ├── notification/        # 카카오/슬랙/SMS 발송
│   ├── correspondent/
│   ├── bill/
│   └── etc/
├── repository/
│   ├── admin/
│   ├── buyer/
│   ├── seller/
│   ├── banner/
│   ├── chat/
│   ├── common/
│   ├── coupon/
│   ├── order/
│   ├── payment/
│   └── wine/
├── replica/
│   └── repository/          # Replica DB 전용 Repository (읽기 전용)
│       ├── buyer/
│       ├── seller/
│       ├── order/
│       └── ...
├── entity/
│   ├── admin/
│   ├── buyer/
│   ├── seller/
│   ├── banner/
│   ├── chat/
│   ├── common/
│   ├── coupon/
│   ├── order/
│   ├── payment/
│   └── wine/
├── dto/
│   ├── ResponseDto.java     # 공통 응답 래퍼
│   ├── admin/
│   ├── buyer/
│   ├── seller/
│   │   ├── request/
│   │   └── response/
│   ├── chat/
│   │   ├── request/
│   │   └── response/
│   ├── common/
│   ├── payment/
│   └── wine/
├── enumration/              # (오탈자 주의: enumeration → enumration으로 패키지 고정)
│   ├── Role.java
│   ├── code/                # YnCode 등
│   ├── adimn/               # 관리자 권한 Enum
│   ├── banner/
│   ├── goods/
│   ├── seller/
│   ├── wine/
│   └── etc/
├── error/
│   ├── Handler/             # ErrorExceptionHandler (@RestControllerAdvice)
│   ├── exception/           # CustomErrorCodeException, ErrorCodeException
│   ├── enumration/          # ErrorCode, CustomErrorCode
│   └── response/            # ErrorCodeResponse, ErrorCodeResponseEntityFactory
├── component/               # @Component (banner, cache, payment, notification 등)
├── event/                   # 이벤트 기반 비동기 처리 (slack, sms, kakao, elasticsearch 등)
├── converter/               # AttributeConverter (DB 컬럼 암호화 등)
├── elasticsearch/
│   ├── entity/
│   └── repository/
├── extract/
│   └── file/                # Excel 파일 생성
├── log/                     # 로그 관련
├── schedule/                # @Scheduled 스케줄러
├── util/                    # LoginUserProviderUtil, RoleRouteUtil 등
├── vo/                      # UserVO 등 Value Object
└── constant/                # 상수 정의
```

---

## 4. application.yml 구조

```yaml
server:
  port: 8888
  shutdown: graceful
  tomcat:
    uri-encoding: UTF-8
    basedir: .
    accesslog:
      enabled: true
      directory: logs
      suffix: .log
      prefix: access
      file-date-format: .yyyy-MM-dd
      max-days: 7
      pattern: "%{yyyy-MM-dd HH:mm:ss.SSS}t %a %{X-Forwarded-For}i %s %D %r %b %{Referer}i %{User-Agent}i"
    relaxed-query-chars: ['[', ']', '{', '}', '|', '^', '`', '"', '<', '>']

spring:
  lifecycle:
    timeout-per-shutdown-phase: 10s
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 20MB
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher

  datasource:
    master:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://host:3306/db?characterEncoding=UTF-8&useSSL=false&allowPublicKeyRetrieval=true&autoReconnection=true
      username: user
      password: pass
      hikari:
        pool-name: MASTER
        maximumPoolSize: 5
        minimumIdle: 1
        maxLifetime: 1800000      # 30분
        idleTimeout: 600000       # 10분
        connectionTimeout: 3000   # 3초
        keepaliveTime: 30000      # 30초 ping
        leakDetectionThreshold: 500
    replica:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://host:3306/db?characterEncoding=UTF-8&useSSL=false&allowPublicKeyRetrieval=true&autoReconnection=true
      username: user_readonly
      password: pass
      hikari:
        pool-name: REPLICA
        maximumPoolSize: 5
        minimumIdle: 1
        maxLifetime: 1800000
        idleTimeout: 600000
        connectionTimeout: 3000
        keepaliveTime: 30000
        leakDetectionThreshold: 500
        read-only: true

  jpa:
    open-in-view: true  # LAZY 로딩 지원 목적

  mail:
    host: smtp.gmail.com
    port: 587
    username: example@gmail.com
    password: 'app-password'
    properties.mail.smtp:
      auth: true
      starttls.enable: true
      connectiontimeout: 5000
      timeout: 5000
      writetimeout: 5000

  data:
    redis:
      repositories.enabled: false
      connect-timeout: 1000ms
      timeout: 500ms
    elasticsearch:
      repositories.enabled: false

  redis:
    host: 127.0.0.1
    port: 36379
    password:
    database: 0

  elasticsearch:
    uris: host:9200

  cache:
    type: redis

logging:
  file:
    path: logs
  charset:
    console: utf-8
  level:
    com.threelabs: debug
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
    com.zaxxer.hikari: DEBUG
  logback:
    rollingpolicy:
      max-history: 7

# 커스텀 설정 (회사명 접두사)
threelabs:
  token:
    secret-key: ${JWT_SECRET}
    access-token:
      admin: marketbang_admin_access_token    # 쿠키명
      buyer: marketbang_buyer_access_token
      seller: marketbang_seller_access_token
      valid-time: '#{3 * 60 * 60 * 1000L}'           # 3시간
      valid-time-buyer: '#{3 * 60 * 60 * 1000L}'
      valid-time-seller: '#{3 * 60 * 60 * 1000L}'
    refresh-token:
      admin: marketbang_admin_refresh_token
      buyer: marketbang_buyer_refresh_token
      seller: marketbang_seller_refresh_token
      valid-time: '#{24 * 60 * 60 * 1000L}'          # 24시간
      valid-time-buyer: '#{24 * 60 * 60 * 1000L}'
      valid-time-seller: '#{24 * 60 * 60 * 1000L}'
    domain: localhost:8080
    cookie-secure: false   # 운영: true
  origin: http://127.0.0.1:8889
  origin_www: http://127.0.0.1:8889
  url:
    seller: "http://127.0.0.1:8080"
    admin: "http://127.0.0.1:8081"
    api: "http://127.0.0.1:8888"
  storage: /path/to/storage/
  cloud-front:
    protocol: https
    domain: cdn.example.kr
    key-pair-id: KEY_PAIR_ID
    private-key: private.pem
    s3-upload-prefix: upload/
  is-run-schedule: true
  database:
    column-sec-key: "32byte-encryption-key"
    column-sec-iv: "16byte-iv-string"
  scheduler:
    pool-size: 2

springdoc:
  api-docs:
    path: /api-docs
  show-login-endpoint: true
  swagger-ui:
    tags-sorter: alpha
    defaultModelsExpandDepth: -1
    path: /swagger-ui.html

cloud:
  aws:
    region.static: ap-northeast-2
    stack.auto: false
    s3.bucket: bucket-name
    credentials:
      access-key: ACCESS_KEY
      secret-key: SECRET_KEY
```

---

## 5. Entity 패턴

```java
package com.threelabs.entity.admin;

import jakarta.persistence.*;           // Boot3: javax → jakarta
import java.io.Serializable;
import java.time.LocalDateTime;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import com.threelabs.enumration.Role;
import com.threelabs.enumration.code.YnCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Setter
@Getter
@Entity(name = "admin_user")        // 테이블명 소문자 스네이크케이스
@NoArgsConstructor
public class AdminUser implements Serializable {
    static final long serialVersionUID = 2022L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "USER_NO")        // 컬럼명: 대문자 스네이크케이스
    private Long userNo;

    @Column(name = "USER_ID")
    private String userId;

    @Column(name = "USER_PWD")
    private String userPwd;

    @Column(name = "USER_AUTH")
    private Role userAuth;           // Enum: @Enumerated 없이 사용(ordinal 저장)

    @Column(name = "USE_YN")
    @Enumerated(EnumType.STRING)     // YnCode는 STRING으로 저장
    private YnCode useYn;

    @JsonSerialize(using = LocalDateTimeSerializer.class)
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    @Column(name = "APPEND_DATE", updatable = false)
    private LocalDateTime appendDate;

    @JsonSerialize(using = LocalDateTimeSerializer.class)
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    @Column(name = "UPDATE_DATE")
    private LocalDateTime updateDate;

    // 패키지 private 생성자 (createEntity를 통해서만 생성)
    AdminUser(String userId, String userPwd, Role userAuth,
              String userName, LocalDateTime appendDate) {
        this.userId = userId;
        this.userPwd = userPwd;
        this.userAuth = userAuth;
        this.useYn = YnCode.Y;
        this.appendDate = appendDate;
    }

    // 정적 팩토리 메서드로 외부에서 생성
    public static AdminUser createEntity(String userId, String userPwd, Role userAuth,
                                         String userName, LocalDateTime appendDate) {
        return new AdminUser(userId, userPwd, userAuth, userName, appendDate);
    }
}
```

**규칙:**
- `implements Serializable` 선언 (세션/Redis 저장 대비)
- `@Entity(name = "테이블명")` — 소문자 스네이크케이스
- `@Column(name = "COLUMN_NAME")` — 대문자 스네이크케이스
- `LocalDateTime` 필드: `@JsonSerialize` / `@JsonDeserialize` 명시
- `YnCode` Enum: `@Enumerated(EnumType.STRING)` 필수
- `Role` Enum: `@Enumerated` 없이 사용 (DB에 ordinal 저장)
- 생성: `createEntity()` 정적 팩토리 메서드 사용
- `@Getter` + `@Setter` + `@NoArgsConstructor` 기본 조합

---

## 6. Enum 패턴

### Role (권한 Enum)
```java
package com.threelabs.enumration;

import lombok.Getter;
import org.springframework.security.core.GrantedAuthority;

public enum Role implements GrantedAuthority {
    // 절대 순서 변경 금지 — DB에 ordinal로 저장됨
    ROLE_ADMIN("관리자 계정", "관리자"),
    ROLE_BUYER("점주 계정(정상사용자)", "점주"),
    ROLE_BUYER_STOP("점주 계정(정지사용자)", "점주"),
    ROLE_SELLER_OWNER("수입사 대표 계정(정상사용자)", "수입사"),
    ROLE_SELLER_OWNER_STOP("수입사 대표 계정(정지사용자)", "수입사"),
    ROLE_SELLER_MANAGER("수입사 매니저 계정(정상사용자)", "수입사"),
    ROLE_SELLER_MANAGER_STOP("수입사 매니저 계정(정지사용자)", "수입사"),
    ROLE_ANONYMOUS("비회원", "비회원"),
    ROLE_ADMIN_MANAGER("관리자 매니저 계정", "관리자");

    private String description;
    @Getter
    private String display;

    Role(String description, String display) {
        this.description = description;
        this.display = display;
    }

    @Override
    public String getAuthority() {
        return String.valueOf(this);
    }

    // 상수 문자열 참조용 내부 클래스
    public static class Secured {
        public static final String ADMIN = "ROLE_ADMIN";
        public static final String BUYER = "ROLE_BUYER";
    }
}
```

### YnCode
```java
package com.threelabs.enumration.code;

import lombok.Getter;

@Getter
public enum YnCode {
    Y(true),
    N(false);

    private boolean value;

    YnCode(boolean value) {
        this.value = value;
    }
}
```

### CustomErrorCode (에러 코드 Enum)
```java
@Getter
public enum CustomErrorCode {
    ERROR_PAYMENT_FAIL(7001, "결제 실패했습니다.", "결제오류"),
    ERROR_WORK_NOTICE(9001, "시스템 점검중입니다. 잠시 후에 이용해주세요.", "점주/수입사 작업 공지");

    private final Integer code;
    private final String message;
    private final String description;   // 개발자용 설명

    CustomErrorCode(Integer code, String message, String description) {
        this.code = code;
        this.message = message;
        this.description = description;
    }
}
```

---

## 7. Controller 패턴

```java
package com.threelabs.controller.admin;

import com.threelabs.dto.ResponseDto;
import com.threelabs.dto.admin.AdminUserChangePasswordRequestDto;
import com.threelabs.entity.admin.AdminUser;
import com.threelabs.service.admin.AdminUserService;
import com.threelabs.util.LoginUserProviderUtil;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@Tag(name = "관리자")                          // Swagger 그룹핑
@RequestMapping("/api/admin/v1/me")           // URL: /api/{role}/v1/{resource}
@RequiredArgsConstructor
@RestController
public class AdminUserController {

    private final LoginUserProviderUtil<AdminUser> loginAdminUserProviderUtil;
    private final AdminUserService adminUserService;

    @Operation(summary = "관리자 비밀번호 변경")
    @PostMapping("/password")
    public ResponseDto<Object> changePassword(
            @RequestBody AdminUserChangePasswordRequestDto dto) {
        AdminUser adminUser = loginAdminUserProviderUtil.getUser();
        adminUserService.changePwdAdminMe(adminUser.getUserNo(), dto.getPwd(), dto.getNewPwd());
        return new ResponseDto<>(0, "", null);
    }
}
```

**URL 패턴:**
```
/api/{role}/v1/{resource}

역할: admin, buyer, seller, common
예시:
  GET  /api/buyer/v1/goods/{id}        상품 상세
  POST /api/seller/v1/goods            상품 등록
  GET  /api/admin/v1/member/list       회원 목록
  POST /api/common/v1/code/list        공통 코드 조회
```

**응답 규칙:**
```java
// 성공 (데이터 있음)
return new ResponseDto<>(0, "", resultData);

// 성공 (데이터 없음)
return new ResponseDto<>(0, "", null);

// 성공 (메시지 있음)
return new ResponseDto<>(0, "저장되었습니다.", null);
```

---

## 8. ResponseDto (공통 응답 포맷)

```java
package com.threelabs.dto;

import com.threelabs.config.security.user_details.CustomUserDetails;
import lombok.Getter;
import lombok.Setter;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;

@Getter
@Setter
public class ResponseDto<T> {

    private final String message;
    private final int code;       // 0: 성공, 1 이상: 오류
    private final T data;
    private final boolean isLogin;  // 현재 로그인 여부 자동 감지

    public ResponseDto(int code, String message, T data) {
        boolean tempIsLogin = false;
        try {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            Object principal = authentication.getPrincipal();
            if (principal instanceof CustomUserDetails) {
                tempIsLogin = true;
            }
        } catch (Exception e) {
            // 비인증 상태는 false 유지
        }
        this.isLogin = tempIsLogin;
        this.code = code;
        this.message = message;
        this.data = data;
    }
}
```

---

## 9. Service 패턴

```java
package com.threelabs.service.admin;

import com.threelabs.error.exception.CustomErrorCodeException;
import com.threelabs.enumration.code.YnCode;
import lombok.RequiredArgsConstructor;
import lombok.extern.apachecommons.CommonsLog;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@CommonsLog                  // Apache Commons Logging
@RequiredArgsConstructor
public class AdminUserService {

    private final AdminUserRepository adminUserRepository;
    private final PasswordEncoder passwordEncoder;

    // 조회: readOnly 트랜잭션
    @Transactional(readOnly = true)
    public Page<MemberAdminSearchListResponseDto> findAdminUser(MemberAdminSearchListRequestDto dto) {
        Pageable pageRequest = dto.getPageable();
        String keyword = dto.getKeyword() != null && !dto.getKeyword().trim().isEmpty()
                ? dto.getKeyword().trim() : null;
        return adminUserRepository.listAdminSearch(keyword, dto.getPermissionType(), pageRequest, dto.getOrderType());
    }

    // 생성: @Transactional 필수
    @Transactional
    public boolean addAdminUser(MemberAdminAddRequestDto dto) {
        // 중복 체크 후 예외
        AdminUser existing = adminUserRepository
                .findAdminUserByUserIdAndUseYn(dto.getUserId(), YnCode.Y)
                .orElse(null);
        if (existing != null) {
            throw new CustomErrorCodeException("이미 등록된 이메일입니다.", 1);
        }
        adminUserRepository.save(AdminUser.createEntity(
                dto.getUserId(),
                passwordEncoder.encode(dto.getUserPwd()),
                Role.ROLE_ADMIN_MANAGER,
                dto.getUserName(),
                LocalDateTime.now()
        ));
        return true;
    }

    // 수정: orElseThrow로 존재 확인
    @Transactional
    public void changePwdAdminMe(Long userNo, String pwd, String newPwd) {
        AdminUser adminUser = adminUserRepository.findById(userNo)
                .orElseThrow(() -> new CustomErrorCodeException("잘못된 요청입니다.", 1));
        if (passwordEncoder.matches(pwd, adminUser.getUserPwd())) {
            adminUser.setUserPwd(passwordEncoder.encode(newPwd));
            adminUserRepository.save(adminUser);
        } else {
            throw new CustomErrorCodeException("비밀번호가 일치하지 않습니다.", 1);
        }
    }

    // 소프트 삭제
    @Transactional
    public boolean deleteAdminUser(Long userNo) throws Exception {
        AdminUser adminUser = adminUserRepository.findById(userNo).orElse(null);
        if (adminUser != null && adminUser.getUseYn().equals(YnCode.Y)) {
            adminUser.setUseYn(YnCode.N);   // 소프트 삭제
            adminUserRepository.save(adminUser);
            return true;
        } else {
            throw new Exception("삭제되었거나 존재하지 않는 사용자입니다.");
        }
    }
}
```

**규칙:**
- `@CommonsLog` 로깅
- 조회: `@Transactional(readOnly = true)`
- 생성/수정/삭제: `@Transactional`
- 존재 확인 후 예외: `orElseThrow(() -> new CustomErrorCodeException("메시지", 1))`
- 비즈니스 규칙 위반: `throw new CustomErrorCodeException("메시지", 코드번호)`
- 물리 삭제 대신 `useYn = YnCode.N` 소프트 삭제

---

## 10. Repository 패턴

### JpaRepository + 커스텀 인터페이스

```java
package com.threelabs.repository.admin;

import com.threelabs.entity.admin.AdminUser;
import com.threelabs.enumration.Role;
import com.threelabs.enumration.code.YnCode;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

@Repository
public interface AdminUserRepository extends JpaRepository<AdminUser, Long>,
                                             CustomAdminUserRepository {
    // 메서드명 기반 쿼리 생성
    Optional<AdminUser> findAdminUserByUserId(String userId);
    Optional<AdminUser> findAdminUserByUserIdAndUseYn(String userId, YnCode useYn);
    List<AdminUser> findAllByUserAuthAndUseYn(Role userAuth, YnCode useYn);

    // 직접 UPDATE 쿼리
    @Modifying
    @Transactional
    @Query("UPDATE admin_user au SET au.recentLoginDate = :recentLoginDate WHERE au.userNo = :userNo")
    int updateRecentLoginDateByUserNo(Long userNo, LocalDateTime recentLoginDate);
}
```

### QueryDSL 커스텀 인터페이스

```java
public interface CustomAdminUserRepository {
    Page<MemberAdminSearchListResponseDto> listAdminSearch(
            String keyword, AdminPermissionType permissionType,
            Pageable pageable, String orderType);
    List<AdminUser> listAllAdminByPermission(AdminPermissionType adminPermissionType);
}
```

### QueryDSL 구현체

```java
@RequiredArgsConstructor
public class CustomAdminUserRepositoryImpl implements CustomAdminUserRepository {

    private final JPAQueryFactory queryFactory;
    final private EntityManager entityManager;

    @Override
    public Page<MemberAdminSearchListResponseDto> listAdminSearch(
            String keyword, AdminPermissionType permissionType,
            Pageable pageable, String orderType) {

        QAdminUser qAdminUser = QAdminUser.adminUser;
        QAdminUserPermission qAdminUserPermission = QAdminUserPermission.adminUserPermission;

        // 동적 WHERE 절
        BooleanExpression keywordExp = null;
        if (keyword != null && keyword.length() > 0) {
            keywordExp = qAdminUser.userName.containsIgnoreCase(keyword);
        }

        // 서브쿼리 활용
        BooleanExpression permissionTypeExp = null;
        if (permissionType != null) {
            JPQLQuery<Long> subQuery = JPAExpressions
                    .select(qAdminUserPermission.userNo)
                    .from(qAdminUserPermission)
                    .where(qAdminUserPermission.permissionType.eq(permissionType))
                    .where(qAdminUserPermission.useYn.eq(YnCode.Y));
            permissionTypeExp = qAdminUser.userNo.in(subQuery);
        }

        // 동적 ORDER BY
        List<OrderSpecifier> orderSpecifierList = new LinkedList<>();
        switch (orderType) {
            case "user_id_desc" -> orderSpecifierList.add(new OrderSpecifier<>(Order.DESC, qAdminUser.userId));
            case "user_id_asc"  -> orderSpecifierList.add(new OrderSpecifier<>(Order.ASC,  qAdminUser.userId));
            case "user_name_desc" -> orderSpecifierList.add(new OrderSpecifier<>(Order.DESC, qAdminUser.userName));
            case "user_name_asc"  -> orderSpecifierList.add(new OrderSpecifier<>(Order.ASC,  qAdminUser.userName));
        }
        orderSpecifierList.add(new OrderSpecifier<>(Order.DESC, qAdminUser.userNo)); // 기본 정렬

        // 데이터 조회
        List<MemberAdminSearchListResponseDto> fetch = queryFactory
                .select(Projections.constructor(MemberAdminSearchListResponseDto.class, qAdminUser))
                .from(qAdminUser)
                .where(qAdminUser.userAuth.eq(Role.ROLE_ADMIN_MANAGER))
                .where(qAdminUser.useYn.eq(YnCode.Y))
                .where(permissionTypeExp)
                .where(keywordExp)
                .orderBy(orderSpecifierList.toArray(new OrderSpecifier[0]))
                .limit(pageable.getPageSize())
                .offset(pageable.getOffset())
                .fetch();

        // 전체 카운트 (페이징용)
        Long total = queryFactory
                .select(qAdminUser.countDistinct())
                .from(qAdminUser)
                .where(qAdminUser.userAuth.eq(Role.ROLE_ADMIN_MANAGER))
                .where(qAdminUser.useYn.eq(YnCode.Y))
                .where(permissionTypeExp)
                .where(keywordExp)
                .fetchOne();

        return new PageImpl<>(fetch, pageable, total);
    }

    // JOIN 활용
    public List<AdminUser> listAllAdminByPermission(AdminPermissionType adminPermissionType) {
        QAdminUser qAdminUser = QAdminUser.adminUser;
        QAdminUserPermission qAdminUserPermission = QAdminUserPermission.adminUserPermission;
        return queryFactory
                .select(qAdminUser)
                .from(qAdminUser)
                .join(qAdminUserPermission)
                    .on(qAdminUserPermission.userNo.eq(qAdminUser.userNo)
                        .and(qAdminUserPermission.useYn.eq(YnCode.Y))
                        .and(qAdminUserPermission.permissionType.eq(adminPermissionType)))
                .where(qAdminUser.useYn.eq(YnCode.Y))
                .fetch();
    }
}
```

**규칙:**
- 클래스명: `Custom{Entity}RepositoryImpl`
- 단순 조회: JpaRepository 메서드명 자동 생성
- 복잡한 쿼리: QueryDSL 커스텀 구현
- DTO 직접 생성: `Projections.constructor()`
- 페이징: `Pageable` + `PageImpl<>` 반환
- 동적 조건: `BooleanExpression` + null 허용 (where 절에 null 전달 시 무시됨)
- 동적 정렬: `LinkedList<OrderSpecifier>` 후 마지막에 기본 정렬 추가

---

## 11. DTO 패턴

### Response DTO

```java
package com.threelabs.dto.admin;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;
import lombok.Setter;

@Getter
public class MemberAdminSearchListResponseDto {

    @Schema(description = "사용자 번호")
    private Long userNo;

    @Schema(description = "사용자 ID(이메일)")
    private String userId;

    @Schema(description = "이름")
    private String userName;

    @Setter  // QueryDSL에서 fetch 후 추가 데이터 설정 시 필요
    private List<AdminUserPermission> menuPermissionList;

    // Entity → DTO 변환은 생성자에서 처리
    public MemberAdminSearchListResponseDto(AdminUser adminUser) {
        this.userNo = adminUser.getUserNo();
        this.userId = adminUser.getUserId();
        this.userName = adminUser.getUserName();
    }

    // 필드 직접 주입용 생성자
    public MemberAdminSearchListResponseDto(Long userNo, String userId, String userName, String userPhone) {
        this.userNo = userNo;
        this.userId = userId;
        this.userName = userName;
    }
}
```

### Request DTO

```java
package com.threelabs.dto.admin;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter  // Request는 바인딩 필요
public class AdminUserChangePasswordRequestDto {
    private String pwd;
    private String newPwd;
}
```

**규칙:**
- Response DTO: `@Getter`만 (기본 불변). fetch 후 세팅 필요한 필드만 `@Setter`
- Request DTO: `@Getter` + `@Setter`
- 변환 로직: 생성자에서 직접 처리 (MapStruct 미사용)
- Swagger 문서화: 필드에 `@Schema(description = "설명")` 추가
- 네이밍: `{Domain}{Action}RequestDto` / `{Domain}{Action}ResponseDto`
- seller DTO는 `dto/seller/request/`, `dto/seller/response/`로 분리

---

## 12. 예외 처리

### 커스텀 예외

```java
@Getter
public class CustomErrorCodeException extends RuntimeException {
    private final String message;
    private final Integer code;

    // 직접 메시지/코드 지정
    public CustomErrorCodeException(String message, Integer code) {
        super();
        this.message = message;
        this.code = code;
    }

    public CustomErrorCodeException(String message, Integer code, Throwable throwable) {
        super(throwable);
        this.message = message;
        this.code = code;
    }

    // Enum으로 지정
    public CustomErrorCodeException(CustomErrorCode customErrorCode) {
        this(customErrorCode.getMessage(), customErrorCode.getCode());
    }

    // Enum 코드 + 커스텀 메시지
    public CustomErrorCodeException(CustomErrorCode customErrorCode, String customMsg) {
        this(customMsg, customErrorCode.getCode());
    }
}
```

### Global Exception Handler

```java
@CommonsLog
@RequiredArgsConstructor
@RestControllerAdvice
public class ErrorExceptionHandler {

    @ExceptionHandler(ErrorCodeException.class)
    protected ResponseEntity<ErrorCodeResponse> customErrorException(ErrorCodeException exception) {
        log.debug("CUSTOM ERROR", exception);
        return ErrorCodeResponseEntityFactory.make(exception.getResponseCode());
    }

    @ExceptionHandler(CustomErrorCodeException.class)
    protected ResponseEntity<ErrorCodeResponse> customErrorException(CustomErrorCodeException exception) {
        log.debug("CUSTOM ERROR", exception);
        return ErrorCodeResponseEntityFactory.make(HttpStatus.OK, exception.getCode(), exception.getMessage());
    }

    @ExceptionHandler(NoSuchElementException.class)
    protected ResponseEntity<ErrorCodeResponse> noSuchElementException(NoSuchElementException exception) {
        return ErrorCodeResponseEntityFactory.make(ErrorCode.NOT_FOUND);
    }

    @ExceptionHandler(HttpMessageNotReadableException.class)
    protected ResponseEntity<ErrorCodeResponse> httpMessageNotReadableException(HttpMessageNotReadableException exception) {
        log.error("HttpMessageNotReadableException", exception);
        return ErrorCodeResponseEntityFactory.make(HttpStatus.SERVICE_UNAVAILABLE,
                CustomErrorCode.ERROR_WORK_NOTICE.getCode(), "시스템 오류입니다.(관리자에게 문의해주세요.)");
    }
}
```

### ErrorCodeResponse (에러 응답 형식)

```java
@Getter
public class ErrorCodeResponse {
    private final LocalDateTime timestamp = LocalDateTime.now();
    @JsonIgnore
    private final HttpStatus status;
    private final Integer code;
    private final String message;

    public ErrorCodeResponse(HttpStatus status, Integer code, String message) {
        this.status = status;
        this.code = code;
        this.message = message;
    }
}
```

**사용 예:**
```java
// Service에서
throw new CustomErrorCodeException("이미 등록된 이메일입니다.", 1);
throw new CustomErrorCodeException(CustomErrorCode.ERROR_PAYMENT_FAIL);
throw new CustomErrorCodeException(CustomErrorCode.ERROR_PAYMENT_FAIL, "결제 금액이 맞지 않습니다.");
```

---

## 13. Security 설정 (Boot3 방식)

```java
@EnableWebSecurity
@RequiredArgsConstructor
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig {

    @Value("${threelabs.origin}")
    private String origin;

    private final AuthenticationFailureHandler authenticationFailureHandler;
    private final AuthenticationSuccessHandler authenticationSuccessHandler;
    private final AuthenticationEntryPoint authenticationEntryPoint;
    private final AccessDeniedHandler accessDeniedHandler;
    private final CustomUserDetailsService userDetailsService;
    private final JwtTokenProvider jwtTokenProvider;
    private final JwtAuthorizationFilter jwtAuthorizationFilter;
    private final RoleRouteUtil roleRouteUtil;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public JwtAuthenticationFilter jwtAuthenticationFilter(AuthenticationManager authenticationManager) {
        JwtAuthenticationFilter filter = new JwtAuthenticationFilter(jwtTokenProvider, roleRouteUtil);
        filter.setFilterProcessesUrl("/api/**/v1/user/login");  // 로그인 URL 패턴
        filter.setUsernameParameter("userId");
        filter.setPasswordParameter("userPwd");
        filter.setAuthenticationManager(authenticationManager);
        filter.setAuthenticationSuccessHandler(authenticationSuccessHandler);
        filter.setAuthenticationFailureHandler(authenticationFailureHandler);
        return filter;
    }

    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.addAllowedHeader("*");
        configuration.setAllowedMethods(List.of("GET", "HEAD", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        configuration.addExposedHeader("*");
        configuration.setAllowedOriginPatterns(List.of(
                "https://yourdomain.kr", "https://*.yourdomain.kr",
                "null", "NULL"  // 모바일 결제 완료 시 origin이 null로 들어옴
        ));
        if ("LOCAL".equals(App._ENV)) {
            configuration.addAllowedOriginPattern("http://127.0.0.1:*");
            configuration.addAllowedOriginPattern("http://localhost:*");
        }
        configuration.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
                                           AuthenticationManager authenticationManager) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.OPTIONS, "/**").permitAll()  // OPTIONS 전체 허용
                .requestMatchers("/healthcheck", "/").permitAll()
                // 공개 API
                .requestMatchers("/api/buyer/v1/public/**").permitAll()
                .requestMatchers("/v3/api-docs/**", "/swagger-ui/**", "/api-docs/**").permitAll()
                // 역할별 접근 제어
                .requestMatchers("/api/admin/v1/**").hasAnyRole("ADMIN", "ADMIN_MANAGER")
                .requestMatchers("/api/seller/**").hasAnyRole("SELLER_OWNER", "SELLER_MANAGER")
                .requestMatchers("/api/buyer/**", "/page/buyer/**").hasRole("BUYER")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form.disable())
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .exceptionHandling(exception ->
                exception.authenticationEntryPoint(authenticationEntryPoint)
                         .accessDeniedHandler(accessDeniedHandler));

        http.addFilter(jwtAuthenticationFilter(authenticationManager));
        http.addFilterBefore(jwtAuthorizationFilter, UsernamePasswordAuthenticationFilter.class);
        http.addFilterBefore(corsOptionsHeaderLogFilter(), CorsFilter.class);

        return http.build();
    }
}
```

**Boot3 변경점:**
- `WebSecurityConfigurerAdapter` → 사용 안 함 (삭제됨)
- `SecurityFilterChain` Bean 방식 사용
- `http.authorizeRequests()` → `http.authorizeHttpRequests()`
- `antMatchers()` → `requestMatchers()`

---

## 14. JWT 토큰 (Cookie 기반)

```java
@RequiredArgsConstructor
@Component
public class JwtTokenProvider {

    private final Algorithm algorithm;       // JwtConfig에서 @Bean으로 주입
    private final JwtTokenConstants jwtTokenConstants;
    private final RoleRouteUtil roleRouteUtil;

    // Access Token 생성
    public String createAccessToken(HttpServletRequest request, UserDetails user, boolean passOTP) {
        String whoAmI = roleRouteUtil.whoAmI();  // URL 기반으로 admin/buyer/seller 판별
        long validTime = switch (whoAmI) {
            case "buyer"  -> jwtTokenConstants.getAccessTokenValidTimeBuyer();
            case "seller" -> jwtTokenConstants.getAccessTokenValidTimeSeller();
            default       -> jwtTokenConstants.getAccessTokenValidTime();
        };
        return JWT.create()
                .withSubject(user.getUsername())
                .withExpiresAt(new Date(System.currentTimeMillis() + validTime))
                .withIssuer(request.getRequestURI())
                .withClaim("roles", user.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority).toList())
                .withClaim("passOTP", passOTP)
                .sign(algorithm);
    }

    // 토큰을 HttpOnly Cookie에 저장
    public void setToken(HttpServletResponse response, String accessToken, String refreshToken) throws IOException {
        String whoAmI = roleRouteUtil.whoAmI();
        String accessTokenName = switch (whoAmI) {
            case "admin"  -> jwtTokenConstants.getAdminAccessTokenName();
            case "buyer"  -> jwtTokenConstants.getBuyerAccessTokenName();
            case "seller" -> jwtTokenConstants.getSellerAccessTokenName();
            default       -> null;
        };
        Cookie accessCookie = new Cookie(accessTokenName, accessToken);
        accessCookie.setPath("/");
        accessCookie.setHttpOnly(true);
        accessCookie.setMaxAge((int) (validTime / 1000) - 10);
        accessCookie.setSecure(jwtTokenConstants.isCookieSecure());  // 운영: true
        response.addCookie(accessCookie);
        // ... refresh cookie도 동일하게 처리
    }

    // Cookie에서 토큰 추출
    public Optional<String> resolveAccessToken(HttpServletRequest request) {
        String tokenName = switch (roleRouteUtil.whoAmI()) {
            case "admin"  -> jwtTokenConstants.getAdminAccessTokenName();
            case "buyer"  -> jwtTokenConstants.getBuyerAccessTokenName();
            case "seller" -> jwtTokenConstants.getSellerAccessTokenName();
            default       -> null;
        };
        if (tokenName == null) return Optional.empty();
        Cookie cookie = WebUtils.getCookie(request, tokenName);
        return cookie != null ? Optional.of(cookie.getValue()) : Optional.empty();
    }
}
```

**특징:**
- Authorization Header 방식 미사용 → **HttpOnly Cookie** 방식
- 역할(admin/buyer/seller)별 쿠키명 분리
- `whoAmI()`: 요청 URL의 `/api/{role}/` 부분으로 역할 판별
- 운영 환경: `cookie-secure: true`

---

## 15. CustomUserDetails

```java
@Setter
public class CustomUserDetails implements UserDetails {

    @Getter
    private String email;

    @JsonIgnore
    private String password;

    @Getter
    private Role role;

    @Getter
    private final Long userNo;

    private final boolean accountNonExpired;
    private final boolean isEnabled;

    public CustomUserDetails(UserVO user) {
        this.email = user.getUserId();
        this.userNo = user.getUserNo();
        this.password = user.getUserPwd();
        this.role = user.getUserAuth();
        this.accountNonExpired = true;
        this.isEnabled = true;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority(role.toString()));
    }

    @Override public String getPassword() { return this.password; }
    @Override public String getUsername() { return this.email; }
    @Override public boolean isAccountNonExpired() { return this.accountNonExpired; }
    @Override public boolean isAccountNonLocked() { return true; }
    @Override public boolean isCredentialsNonExpired() { return true; }
    @Override public boolean isEnabled() { return this.isEnabled; }
}
```

---

## 16. LoginUserProviderUtil (현재 로그인 사용자 조회)

```java
@Service
@RequiredArgsConstructor
public class LoginUserProviderUtil<T> {

    private final AdminUserRepository adminUserRepository;
    private final BuyerUserRepository buyerUserRepository;
    private final SellerUserRepository sellerUserRepository;

    @SuppressWarnings("unchecked")
    public T getUser() {
        CustomUserDetails user = getUserDetails().orElseThrow();
        Role role = user.getRole();
        if (role == Role.ROLE_ADMIN || role == Role.ROLE_ADMIN_MANAGER) {
            return (T) adminUserRepository.findById(user.getUserNo())
                    .orElseThrow(() -> new UsernameNotFoundException("USER NOT FOUND"));
        } else if (role == Role.ROLE_BUYER) {
            return (T) buyerUserRepository.findById(user.getUserNo())
                    .orElseThrow(() -> new UsernameNotFoundException("USER NOT FOUND"));
        } else if (role == Role.ROLE_SELLER_MANAGER || role == Role.ROLE_SELLER_OWNER) {
            return (T) sellerUserRepository.findById(user.getUserNo())
                    .orElseThrow(() -> new UsernameNotFoundException("USER NOT FOUND"));
        }
        throw new ErrorCodeException(ErrorCode.LOGIN_FAILURE);
    }

    public Optional<CustomUserDetails> getUserDetails() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication == null) return Optional.empty();
        Object principal = authentication.getPrincipal();
        if (principal instanceof CustomUserDetails customUserDetails) {
            return Optional.of(customUserDetails);
        }
        return Optional.empty();
    }
}
```

**Controller에서 사용:**
```java
private final LoginUserProviderUtil<AdminUser> loginAdminUserProviderUtil;

AdminUser adminUser = loginAdminUserProviderUtil.getUser();
Long userNo = adminUser.getUserNo();
```

---

## 17. Master/Replica DB 분리 설정

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.master")
    public DataSourceProperties masterProps() {
        return new DataSourceProperties();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.master.hikari")
    public HikariConfig masterHikariConfig() {
        return new HikariConfig();
    }

    @Bean
    @Primary
    public DataSource masterDataSource(
            @Qualifier("masterProps") DataSourceProperties props,
            @Qualifier("masterHikariConfig") HikariConfig cfg) {
        cfg.setJdbcUrl(props.getUrl());
        cfg.setUsername(props.getUsername());
        cfg.setPassword(props.getPassword());
        cfg.setDriverClassName(props.getDriverClassName());
        return new HikariDataSource(cfg);
    }

    // Replica도 동일 구조로 구성 (read-only: true)
}
```

- `replica/repository/` 패키지: Replica DataSource를 사용하는 Repository 별도 분리
- `@Primary` Bean이 Master DataSource
- 서비스 레이어에서 `@Transactional(readOnly = true)` 시 Replica 라우팅

---

## 18. 이벤트 기반 비동기 처리

```java
// 이벤트 발행 (Service에서)
// Slack, SMS, 카카오 알림은 별도 이벤트로 발행 → 트랜잭션 완료 후 실행
@Transactional
public boolean setAdminUserPermission(Long userNo, AdminSetPermissionRequestDto dto) {
    // ... 비즈니스 로직
    slackMessageService.sendAdminSetAdminUserPermission(
            adminUser.getUserNo(), adminUser.getUserId(), adminUser.getUserName());
    return true;
}
```

**이벤트 패키지 구조:**
```
event/
├── slack/     SlackMessageDto, SlackMessageEventAfterCommit
├── sms/
├── kakao/
├── coupon/
├── bizcall/
└── elasticsearch/
```

---

## 19. 네이밍 컨벤션 요약

| 구분 | 패턴 | 예시 |
|------|------|------|
| Entity | `{Domain}` | `AdminUser`, `BuyerUser` |
| JpaRepository | `{Domain}Repository` | `AdminUserRepository` |
| QueryDSL 인터페이스 | `Custom{Domain}Repository` | `CustomAdminUserRepository` |
| QueryDSL 구현체 | `Custom{Domain}RepositoryImpl` | `CustomAdminUserRepositoryImpl` |
| Replica Repository | `{Domain}ReplicaRepository` | `BuyerUserReplicaRepository` |
| Service | `{Domain}Service` | `AdminUserService` |
| Controller | `{Role}{Domain}Controller` | `AdminUserController` |
| Request DTO | `{Domain}{Action}RequestDto` | `AdminUserChangePasswordRequestDto` |
| Response DTO | `{Domain}{Action}ResponseDto` | `MemberAdminSearchListResponseDto` |
| Enum 패키지 | `enumration` | (오탈자지만 프로젝트 규칙) |
| URL | `/api/{role}/v1/{resource}` | `/api/admin/v1/me/password` |
| 테이블명 | `@Entity(name = "소문자_스네이크")` | `admin_user` |
| 컬럼명 | `@Column(name = "대문자_스네이크")` | `USER_NO`, `USE_YN` |
| 소프트 삭제 | `useYn` 필드 + `YnCode.N` | — |

---

## 20. 신규 프로젝트 체크리스트

### 초기 설정
- [ ] `build.gradle` — QueryDSL `:jakarta`, Springdoc v2, Lombok, 의존성 구성
- [ ] `application.yml` — Master/Replica, Redis, JWT 쿠키명, 커스텀 prefix 설정
- [ ] `logback-spring.xml` 로그 설정

### 공통 클래스 (순서대로 작성)
1. [ ] `enumration/Role.java` — `GrantedAuthority` 구현, 순서 고정 주석 필수
2. [ ] `enumration/code/YnCode.java`
3. [ ] `error/enumration/ErrorCode.java`, `CustomErrorCode.java`
4. [ ] `error/exception/CustomErrorCodeException.java`, `ErrorCodeException.java`
5. [ ] `error/response/ErrorCodeResponse.java`, `ErrorCodeResponseEntityFactory.java`
6. [ ] `error/Handler/ErrorExceptionHandler.java` — `@RestControllerAdvice`
7. [ ] `dto/ResponseDto.java` — 공통 응답 래퍼
8. [ ] `vo/UserVO.java`
9. [ ] `config/security/user_details/CustomUserDetails.java`
10. [ ] `config/security/user_details/CustomUserDetailsService.java`
11. [ ] `config/security/provider/JwtTokenConstants.java`, `JwtTokenProvider.java`
12. [ ] `config/security/filter/JwtAuthenticationFilter.java`, `JwtAuthorizationFilter.java`
13. [ ] `config/security/handler/` — 4개 핸들러 (Success, Failure, EntryPoint, AccessDenied)
14. [ ] `config/security/SecurityConfig.java`
15. [ ] `config/db/DataSourceConfig.java`, `MasterJpaConfig.java`, `ReplicaJpaConfig.java`
16. [ ] `config/querydsl/QueryDslConfig.java` — `JPAQueryFactory` Bean
17. [ ] `util/LoginUserProviderUtil.java`
18. [ ] `util/RoleRouteUtil.java` — URL 기반 역할 판별

### 도메인별 작성 순서
```
Entity → Repository(JpaRepository) → Repository(QueryDSL Custom) → DTO → Service → Controller
```
