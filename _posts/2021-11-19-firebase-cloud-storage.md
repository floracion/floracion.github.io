---
layout: post
title: Firebase Cloud Storage
subtitle: Sangmin Lee
categories: Cloud Firebase
tags: [Cloud , Firebase]
---

## Firebase Cloud Storage

### 개요

Firebase용 Cloud Storage는 사진이나 동영상을 저장하기 위한 저장소이다.

해당 서비스에서 제공하는 SDK를 활용하여 iOS/Android/JS로 이미지, 오디오 동영상 등의 오브젝트를 저장할 수 있다.



### 주요 기능

- 업로드 및 다운로드가 중지된 위치부터 다시 시작되므로 사용자의 시간과 대역폭이 절약된다.
- 선언전 보안 모델을 사용하여 파일 이름, 크기, 콘텐츠 유형 및 기타 메타데이터를 기준으로 액세스를 허용할 수 있다.
- 앱 사용자가 급증할 때 엑사바이트급 규모로 확장이 가능하다.



### 기본 원리

개발자는 Cloud Storage용 Firebase SDK를 사용하여 클라이언트에서 직접 파일을 업로드하고 다운로드한다.

Firebase용 Cloud Storage는 Google Cloud Storage 버킷에 파일을 저장하므로 Firebase와 Google Cloud를 통해 파일에 액세스할 수 있다.



### 구현 경로

1. Cloud Storage용 Firebase SDK 적용
   
    : Gradle이나 NPM 등을 활용해 프로젝트에 SDK를 적용한다.
    
2. 참조 만들기
   
    : 업로드, 다운로드 또는 삭제할 파일의 경로를 참조한다. ex) `images/mount.png`
    
3. 업로드 또는 다운로드
   
    : 메모리 또는 디스크에 기본 형식으로 업로드 또는 다운로드한다.
    
4. 파일 보안 설정
   
    : Cloud Storage용 Firebase 보안 규칙을 사용하여 파일의 보안을 설정한다.



## 보안 규칙

Cloud Storage에 대한 Firebase 보안 규칙으로 손쉽게 사용자를 승인하고 요청을 검증할 수 있다.

스토리지 보안 규칙을 사용하면 경로 기반 권한을 지정하여 복잡성을 해소할 수 있다.

또한, 특정 사용자에게만 허용하거나 업로드 크기를 제한하는 승인 규칙을 작성할 수 있다.



### 인증

Firebase 인증은 클라이언트에서만 구현하면 되는 인증 솔루션이다.

사용자가 Firebase 인증으로 인증되면 스토리지 보안 규칙의 `request.auth` 변수가 사용자의 고유 ID( `request.auth.uid` ) 및 
기타 모든 사용자 정보를 토큰( `request.auth.token` ) 에 포함하는 객체가 된다.

사용자가 인증되지 않은 경우 `request.auth` 는 `null` 이다.



### 승인

인증 이외에도 파일별 및 경로별 승인 규칙을 지정하고 앱의 파일에 대한 액세스 권한을 결정할 수 있다.



예를 들어 모든 파일에서 `read` 또는 `write` 작업을 수행하려면 기본 스토리지 보안 규칙에 Firebase 인증이 필요하도록 구성할 수 있다.

```go
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```



### 데이터 검증

Firebase 보안 규칙은 `contentType` 및 `size` 와 같은 파일 메타데이터 속성뿐만 아니라 파일 이름 및 경로 유효성 검사에도 사용할 수 있다.

```go
service firebase.storage {
  match /b/{bucket}/o {
    match /images/{imageId} {
      // Only allow uploads of any image file that's less than 5MB
      allow write: if request.resource.size < 5 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
    }
  }
}
```



### Firebase 보안 규칙 언어

#### 서비스 및 데이터베이스 선언

```
service firebase.storage {
	// ...
}
```



#### 기본 읽기/쓰기 규칙

`match` 는 Cloud Storage의 bucket이나 파일 이름을 지정하는 기본 규칙이다.

`allow` 는 특정 데이터의 접근 방법(읽기, 쓰기)에 대해 허용되거나 거부되는 조건을 지정한다.

```go
service firebase.storage {
  // {bucket} 와일드 카드는 모든 클라우드 스토리지 버킷과 일치함을 나타낸다.
  match /b/{bucket}/o {
    // 파일이름 일치 여부 확인
    match /filename {
      allow read: if <condition>;
      allow write: if <condition>;
    }
  }
}
```



***와일드카드 일치***

규칙에는 특정 문자열 접두어와 슬래시를 포함한 와일드카드를 사용할 수도 있다. ex) `match /images/{imageId}`



#### 계층적 데이터

파일 이름에 슬래시를 포함하여 계층적 구조를 나타낼 수 있다.

```go
service firebase.storage {
  match /b/{bucket}/o {
    match /images/{imageId} {
      allow read, write: if <condition>;
    }

    // Explicitly define rules for the 'mp3s' pattern
    match /mp3s/{mp3Id} {
      allow read, write: if <condition>;
    }
  }
}
```



다음과 같이 중첩 구조로도 나타낼 수 있다.

```go
service firebase.storage {
  match /b/{bucket}/o {
    match /images {
      // Exact match for "images/profilePhoto.png"
      match /profilePhoto.png {
        allow write: if <condition>;
      }
    }
  }
}
```



***재귀 일치 와일드카드***

다중 세그먼트 와일드카드를 활용해 다양한 경로 일치 패턴을 만들 수 있다.

```go
match /images {

  // 패턴 : images/**
  // 일치 예시
  //   - images/users/user:12345/profilePhoto.png
  //   - images/profilePhoto.png
  match /{allImages=**} {
    allow read: if <other_condition>;
  }
}
```



#### 세분화된 작업

`read` 와 `write` 에 세부적인 연산을 적용할 수 있다.

예를 들어 파일 생성과 삭제에 다른 조건을 적용할 수 있다.



`read` 세분화 ⇒ `get` , `list`

`write` 세분화 ⇒ `create` , `update` , `delete`

```go
service firebase.storage {
  match /b/{bucket}/o {
    match /images/{imageId} {
      // 파일 하나를 다운로드하기 위한 조건
      allow get: if <condition>;
      // 파일 리스트를 다운로드하기 위한 조건
      allow list: if <condition>;

    match /images/{imageId} {
      // 파일을 생성하기 위한 조건 
      allow create: if <condition>;

      // 파일을 수정하기 위한 조건
      allow update: if <condition>;

      // 파일을 제거하기 위한 조건
      allow delete: if <condition>;
    }
  }
 }
}
```



### 보안 규칙의 조건

Cloud Storage에 대한 Firebase 보안 규칙은 추가적으로 사용자 인증 확인이나 수신 데이터 검증과 같은 조건을 적용할 수 있다.



#### 인증

사용자 지정 인증을 사용하는 경우 `request.auth.token` 값을 활용한다.

인증되지 않은 사용자의 요청을 수행할 때는 `request.auth` 변수가 `null` 로 활용한다.



사용자 인증을 활용하여 파일을 보호하는 방법은 다음과 같다.

- Public : `request.auth` 를 무시한다.
- Authenticated : `request.auth` 가 `null` 이 아닌지 확인한다.
- User private : `request.auth.uid` 가 경로의 `uid` 와 일치하는지 확인한다.



***Public***

공개는 사용자의 인증을 고려하지 않는다.

```go
// 누구든지 100KB이하의 파일을 다운로드 할 수 있으며, txt 파일을 업로드할 수 있다.
match /public/{imageId} {
  allow read: if resource.size < 100 * 1024;
  allow write: if imageId.matches(".*\\.txt");
}
```



***Authenticated***

파일을 인증되지 않은 사용자는 볼 수 없도록 할 수 있다.

```go
match /internal/{imageId} {
  allow read: if request.auth != null;
}
```



***User private***

자신의 파일에 세분화 된 권한을 가진 개별 사용자 권한을 제공할 수 있다.

```go
match /users/{userId}/profilePicture.png {
  allow read;
  allow write: if request.auth.uid == userId;
}
```



***Group private***

여러 팀 구성원이 공유 문서에서 공동 작업을 할 수 있도록 그룹 권한을 제공할 수 있다.

```go
match /files/{groupId}/{fileName} {
  allow read: if resource.metadata.owner == request.auth.token.groupId;
  allow write: if request.auth.token.groupId == groupId;
}
```



#### 데이터 검증

파일 이름과 경로뿐만 아니라 파일의 메타 데이터 속성 검증을 할 수 있다.

```go
service firebase.storage {
  match /b/{bucket}/o {
    match /images/{imageId} {
      // Only allow uploads of any image file that's less than 5MB
      allow write: if request.resource.size < 5 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
    }
  }
}
```



## Reference

[https://firebase.google.com/docs/storage](https://firebase.google.com/docs/storage)

