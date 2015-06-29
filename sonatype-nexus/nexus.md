Introduction
============

Nexus는 3가지 에디션이 있음 

- OSS
- Pro
- Pro+

### OSS

- 주요 기능

  - Hosting Repositories 
  - Proxy Remote Repositories
  - Repository Groups
  - Numerous Repository Formats
  - Hosting Project Web Sites
  - Fine-grained Security Model
  - Flexible LDAP Integration 
  - Comonent Search
  - Scheduled Tasks
  - REST Services
  - Integration with m2eclipse 

- 우리가 알아야하는 기능은 Fine-grained Security Model, Flexible LDAP Integration 인 거 같음
- Pro 버전을 사용했을 때 Multiple LDAP Servers 기능이 추가되지만 아직은 고려하지 않음 
- 문서상으로 2.8 버전에서는 6-1을 1.7 버전에서는 6.9를 확인해보면 될 거 같지만 먼저 기본 조작법을 먼저 익혀보는 게 role을 이해하는 데 더 도움이 될 거같다.

### Managing Privileges 

- repository target privileges 기능 활용 
- 사내에 설치된 버전에는 없는 기능
- privileges -> role -> user -> target repository 설정으로 유저에 대해 업로드 제약이 가능함 
    - privilege를 생성해 특정 레파지토리에 CRUD 권한을 부여 
    - repository targets을 생성해 특정 레파지토리 타입(e.g. maven2)에 업로드 가능한 정규식 규칙을 생성(e.g. groupid 기준)
    - role에 기존 role과(상속) 새로운 privilege 를 가진 role 생성
    - 추가한 role을 가진 user 계정 생성  

### 결론 

이 기능을 사용불가한 것처럼 보이는데 이유는 업로드 권한을 user 계정에 주는 것이 아닌 repository에 주기 때문입니다. 

- privilege는 특정 레파지토리에 CRUD 기능임
- repository targets는 특정 레파지토리에 대한 업로드 규칙임


### Nexus LDAP Integration

넥서스 pro 버전에서 추가 제공되는 기능 (아직 볼 필요는 없다.)

In addition to the basic LDAP support from Nexus OSS, Nexus Pro offers LDAP support features for enterprise LDAP deployments. These include the ability to cache authentication information, support for multiple LDAP servers and backup mirrors, the ability to test user logins, support for common user/group mapping templates, and the ability to support more than one schema across multiple servers.


상용버전과 무료보전의 기능상 비교(별거없음)

넥서스 LDAP은 순수 LDAP의 기능을 지원하므로 LDAP테스트를 직접해봐야겠다. 
