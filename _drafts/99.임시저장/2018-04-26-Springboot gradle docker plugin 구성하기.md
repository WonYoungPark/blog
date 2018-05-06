

2018-04-26 기준 최신버전인 3.2.7을 적용해 보도록 하곘습니다.

```groovy
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.bmuschko:gradle-docker-plugin:3.2.7'
    }
}
```



## 인증 속성

공용 Docker Hub Registry 또는 개인 레지스트리에 대한 이미지를 pull 하거나 push 할때 인증이 필요할 수 있습니다.

registryCredentials 클로저에 자격 파라미터 값을 입력하여 인증정보를 제공할 수 있습니다.

| Property name | Type   | Default value                 | Description               |
| ------------- | ------ | ----------------------------- | ------------------------- |
| `url`         | String | <https://index.docker.io/v1/> | 레지스트리 URL.           |
| `username`    | String | null                          | 레지스트리 username.      |
| `password`    | String | null                          | 레지스트리 password.      |
| `email`       | String | null                          | 레지스트리 email address. |

```bash
registryCredentials {
    url = 'http://cicd.mrblue.com:5000'
    username = 'wyparks2'
    password = 'mrblue-developer'
}
```

