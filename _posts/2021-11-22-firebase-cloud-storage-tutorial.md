---
layout: post
title: Firebase Cloud Storage & Nuxt.js Tutorial
subtitle: Sangmin Lee
categories: Firebase Nuxt
tags: [Firebase , Nuxt]
---

## Firebase Cloud Storage & Nuxt 튜토리얼

[Project Repository](https://github.com/LeeSM0518/nuxt-firebase-storage)

해당 튜토리얼은 Nuxt.js를 활용해 Firebase용 Cloud Storage에 이미지를 업로드 및 다운로드를 하는 튜토리얼이다.

튜토리얼 진행에 관련된 소프트웨어 버전은 다음과 같다.

```
node : v16.13.0
npm  : 8.1.0
nuxt : 2.15.7
firebase : 8.6.7
```

### Firebase 프로젝트 만들기 및 프로젝트 설정

먼저 [Firebase Console](https://console.firebase.google.com/) 에서 프로젝트 추가를 진행한다.

<img src="https://user-images.githubusercontent.com/43431081/142874959-9ca89681-f39a-4b3f-8dec-5631a8f3e380.png" alt="Add project" width="400">


프로젝트를 추가한 후, 콘솔에서 `앱 추가` 버튼을 누르고 `웹` 으로 플랫폼을 선택한다.

![Untitled 1](https://user-images.githubusercontent.com/43431081/142874809-de7010b2-d1ff-495e-8474-81ed4b3a55ac.png)

![Untitled 2](https://user-images.githubusercontent.com/43431081/142874923-043b860a-a933-4ecb-bc83-91f3047dd6b8.png)


웹으로 애플리케이션을 등록하고 자신의 프로젝트에 Firebase SDK 를 추가한다.

```bash
$ npm install firebase
```

그 후 Nuxt.js 에서 제공하는 Firebase SDK도 추가한다.

```bash
$ npm install @nuxtjs/firebase
```

`nuxt.config.js` 에서 firebase를 설정한다.

```jsx
export default {
  ...

  modules: [
    ...
    '@nuxtjs/firebase'  // firebase 모듈 추가
  ],

  // firebase 설정값
  firebase: {
    config: {
      apiKey: '<API_KEY>',
      authDomain: '<AUTH_DOMAIN>',
      projectId: '<PROJECT_ID>',
      storageBucket: '<STORAGE_BUCKET>',
      messagingSenderId: '<MESSAGING_SENDER_ID>',
      appId: '<APP_ID>'
    },
    services: {
      storage: true
    }
  },

  ...
}
```

해당 튜토리얼에서는 css 프레임워크인 `element-ui` 를 활용할 것이다.

먼저 `element-ui` 를 설치한다.

```jsx
$ npm install element-ui
```


`nuxt.config.js` 파일에서 `element-ui` 를 plugins과 css에 적용한다.

```jsx
export default {
  ...

  // Global CSS: https://go.nuxtjs.dev/config-css
  css: [
    'element-ui/lib/theme-chalk/index.css'
  ],

  // Plugins to run before rendering page: https://go.nuxtjs.dev/config-plugins
  plugins: [
    '@/plugins/element-ui'
  ],

	...
}
```


`plugins` 폴더를 만들고 `element-ui.js` 파일을 만든 뒤, 다음과 같이 코드를 작성한다.

```jsx
import Vue from 'vue'
import Element from 'element-ui'
import locale from 'element-ui/lib/locale/lang/en'

Vue.use(Element, { locale })
```


### 이미지 업로드 및 다운로드 코드 작성

`pages` 폴더에서 `index.vue` 를 다음과 같이 작성한다.

```jsx
<template>
  <el-container>
    <el-main>
      <el-upload
        class='avatar-uploader'
        action='#'
        list-type='picture-card'
        :on-success='handleSuccess'>
        <i slot='default' class='el-icon-plus'></i>
        <div slot='file' slot-scope='{file}'>
          <img class='el-upload-list__item-thumbnail' :src='file.url' alt=''>
        </div>
      </el-upload>
      <el-button @click='uploadFiles'>upload to server</el-button>
      <el-button @click='downloadFile'>download from server</el-button>
      <div v-if='downloadFiles.length !== 0'>
        <img v-for='file in downloadFiles' :key='file' :src='file' class='avatar'>
      </div>
    </el-main>
  </el-container>
</template>

<script>

export default {
  data() {
    return {
      uploadedFiles: [],
      downloadFiles: [],
    }
  },
  methods: {
    handleSuccess(_, file) {
      this.uploadedFiles.push(file.raw)
    },
    async uploadFiles() {
      for (const file of this.uploadedFiles) {
        try {
          const storageRef = await this.$fire.storage.ref(`image/${file.name}`)
          storageRef.put(file)
          console.log(`uploaded file : ${file.name}`)
        } catch (e) {
          console.log(e.message)
        }
      }
    },
    async downloadFile() {
      for (const file of this.uploadedFiles) {
        try {
          const storageRef = await this.$fire.storage.ref(`image/${file.name}`)
          const url = await storageRef.getDownloadURL()
          this.downloadFiles.push(url)
          console.log(`downloaded file : ${file.name}`)
        } catch (e) {
          console.log(e.message)
        }
      }
    }
  }
}
</script>

<style>
</style>
```

- 파일 업로드 함수

  ```jsx
  async uploadFiles() {
    for (const file of this.uploadedFiles) {
      try {
        const storageRef = this.$fire.storage.ref(`image/${file.name}`)
        await storageRef.put(file)
        console.log(`uploaded file : ${file.name}`)
      } catch (e) {
        console.log(e.message)
      }
    }
  },
  ```

  - 파일을 가리키는 참조를 만들어 애플리케이션에 액세스 권한을 부여한다.

    ```jsx
    const storageRef = this.$fire.storage.ref(`image/${file.name}`)
    ```

    - 참조 객체를 생성해 변수에 저장해놓는다.


  - 참조 객체를 통해 파일을 업로드한다.

    ```jsx
    await storageRef.put(file)
    ```

    - 파일을 참조 객체의 함수에 전달해서 Firebase에 저장한다.

- 파일 다운로드 함수

  ```jsx
  async downloadFile() {
    for (const file of this.uploadedFiles) {
      try {
        const storageRef = await this.$fire.storage.ref(`image/${file.name}`)
        const url = await storageRef.getDownloadURL()
        this.downloadFiles.push(url)
        console.log(`downloaded file : ${file.name}`)
      } catch (e) {
        console.log(e.message)
      }
    }
  }
  ```

  - data에 저장해놓은 참조 객체를 통해 `getDownloadURL` 함수를 호출하여 파일을 다운로드 받는다.


### 실행결과

실행했을 때의 화면이 아래와 같이 나오며, 사진을 추가해보자.

<img width="500" alt="Screen_Shot_2021-11-19_at_9 58 21_AM" src="https://user-images.githubusercontent.com/43431081/142875942-806b733c-457d-45ff-b226-92498419726b.png">

사진을 업로드 했을 때, 다음과 같이 파일이 추가되는 것을 확인할 수 있다.

<img width="500" alt="Screen_Shot_2021-11-19_at_9 58 57_AM" src="https://user-images.githubusercontent.com/43431081/142876021-fc3924c0-4d6a-4485-820e-e5bdc945601e.png">

`upload to server` 를 클릭했을 때, 다음과 같이 업로드된 파일의 이름이 콘솔에 출력되는 것을 확인할 수 있다.

<img width="500" alt="Screen_Shot_2021-11-19_at_9 59 23_AM" src="https://user-images.githubusercontent.com/43431081/142876089-6ec03dce-64d9-49c5-87e3-755b4663203e.png">

Firebase Storage로 접속해보면 전송한 파일들이 저장되어 있는 것을 확인할 수 있다.

![Untitled 3](https://user-images.githubusercontent.com/43431081/142876175-d2387443-6abd-4fc6-94ae-4fe7bbb6ba4b.png)

그 다음으로 `download from server` 를 클릭했을 때, 다음과 같이 버튼 아래에 사진이 출력되며 콘솔에 다운로드된 파일 이름들이 출력되는 것을 확인할 수 있다.

<img width="500" alt="Screen_Shot_2021-11-19_at_10 03 33_AM" src="https://user-images.githubusercontent.com/43431081/142876226-348af7ff-cb1e-47bc-a1b9-94a7fc6b3276.png">

<img width="500" alt="Screen_Shot_2021-11-19_at_10 03 57_AM" src="https://user-images.githubusercontent.com/43431081/142876296-b7f589a8-ecd5-4ecb-8888-7302165e0ab7.png">


### 파일 업로드 및 다운로드 플러그인화

파일을 firebase storage에 업로드하거나 다운로드 하는 기존 코드를 Vue 인스턴스 메서드로 추출하여 사용하기 편리하도록 리팩토링한다.

먼저 파일 이름을 uuid 포맷으로 랜덤 생성하기 위해 `uuid` 라이브러리를 추가한다.

```bash
$ npm install uuid
```

`plugins` 에 `object-storage.js` 파일을 만든 후, 다음과 같이 코드를 작성한다.

```jsx
import Vue from 'vue'
import { v4 as uuidv4 } from 'uuid'

// 연도-월 포맷을 만들기 위한 함수
const getYearAndMonth = () => {
  const now = new Date()
  return `${now.getFullYear()}-${now.getMonth() + 1}`
}

// 파일 경로를 만들기 위한 함수
const generateFilePath = (fileType) => {
  const fileName = uuidv4()
  return `files/${getYearAndMonth()}/${fileName}.${fileType}`
}

const objectStorage = {
  install(Vue) {

		// 파일 업로드 함수
    Vue.prototype.$uploadFiles = async function(fileList) {
      try {
        const pathList = []
        for (const file of fileList) {
          const fileType = file.type.split('/')[1]
          const filePath = generateFilePath(fileType)

					// firebase storage에 랜덤으로 만든 경로로 커넥션 객체 생성
          const storageRef = this.$fire.storage.ref(filePath)

					// 커넥션 객체를 통해 파일 업로드
          await storageRef.put(file)

					// 랜덤 경로 저장
          pathList.push(filePath)
        }
        return pathList
      } catch (e) {
        console.log(e.message)
        return []
      }
    }

		// 파일 다운로드 함수
    Vue.prototype.$downloadFiles = async function(pathList) {
      try {
        const urlList = []
        for (const path of pathList) {
					// 파일 경로로 커넥션 객체 생성
          const storageRef = this.$fire.storage.ref(path)

					// 커넥션 객체로부터 파일 URL 조회
          urlList.push(await storageRef.getDownloadURL())
        }
        return urlList
      } catch (e) {
        console.log(e.message)
        return []
      }
    }
  }
}

Vue.use(objectStorage)
```

 

위에서 구현한 플러그인을 `nuxt.config.js` 에 명시한다.

```jsx
export default {
  ...

  // Plugins to run before rendering page: https://go.nuxtjs.dev/config-plugins
  plugins: [
    '@/plugins/element-ui',
    '@/plugins/object-storage'
  ],

  ...
}
```

기존의 `index.vue` 코드에서 플러그인을 사용하도록 리팩토링한다.

```jsx
<template>
  <el-container>
    <el-main>
      <el-upload
        class='avatar-uploader'
        action='#'
        list-type='picture-card'
        :on-success='handleSuccess'>
        <i slot='default' class='el-icon-plus'></i>
        <div slot='file' slot-scope='{file}'>
          <img class='el-upload-list__item-thumbnail' :src='file.url' alt=''>
        </div>
      </el-upload>
      <el-button @click='uploadFiles'>upload to server</el-button>
      <el-button @click='downloadFile'>download from server</el-button>
      <div v-if='downloadFiles.length !== 0'>
        <img v-for='file in downloadFiles' :key='file' :src='file' class='avatar'>
      </div>
    </el-main>
  </el-container>
</template>

<script>

export default {
  data() {
    return {
      uploadedFiles: [],
      uploadedPaths: [],
      downloadFiles: []
    }
  },
  methods: {
    handleSuccess(_, file) {
      this.uploadedFiles.push(file.raw)
    },
    async uploadFiles() {
      console.log(this.uploadedFiles)
      this.uploadedPaths = await this.$uploadFiles(this.uploadedFiles) // 업로드 함수 호출
      console.log(this.uploadedPaths)
      // for (const file of this.uploadedFiles) {
      //   try {
      //     const storageRef = this.$fire.storage.ref(`image/${file.name}`)
      //     await storageRef.put(file)
      //     console.log(`uploaded file : ${file.name}`)
      //   } catch (e) {
      //     console.log(e.message)
      //   }
      // }
    },
    async downloadFile() {
      this.downloadFiles = await this.$downloadFiles(this.uploadedPaths) // 다운로드 함수 호출
      console.log(this.downloadFiles)
      // for (const file of this.uploadedFiles) {
      //   try {
      //     const storageRef = await this.$fire.storage.ref(`image/${file.name}`)
      //     const url = await storageRef.getDownloadURL()
      //     this.downloadFiles.push(url)
      //     console.log(`downloaded file : ${file.name}`)
      //   } catch (e) {
      //     console.log(e.message)
      //   }
      // }
    }
  }
}
</script>

<style>
</style>
```

# Reference

[Cloud Storage for Firebase | Firebase Documentation](https://firebase.google.com/docs/storage)