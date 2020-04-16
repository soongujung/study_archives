# 작업과정 정리 (회원가입, 로그인(JWT))
# 샘플 REST API 서버 설치
여기서는 Vue.js Front End 앱에서 
- 회원가입
- 로그인,인증 (JWT)
만을 정리하려고 한다.  
  
작업 내용을 따라하기에 앞서 예제 REST API 서버를 세팅하자. 서버 로직도 여기서 정리하기에는 장황하다. 미리 준비된 API 서버를 활용하자.
  
> 주의할 점은 nodejs 8.x 버전이 설치되어 있어야 한다. 현재 시점으로 nodejs 의 최신 버전은 12.x이다.  
> .pkg 나 .exe .msi를 활용해서 설치하는 방식 말고 .tar.gz, zip 파일로 특정 디렉터리에 해당 패키지를 위치시키고 [노드디렉터리]/bin을 운영체제의 환경변수로 등록하는 방식으로 설치하자.  
> 이렇게 해야 원할때마다 개발 PC OS의 node 버전을 바꿔가며 테스트를 할 수 있기 때문이다.
> 이렇게 커스텀환경설정 과정은 여기서는 정리하지 않는다.  
  
참고)  
- nodejs previos version download
    - [Node.js Previous Releases](https://nodejs.org/en/download/releases/)

```bash
$ git clone https://github.com/CanDoVueJS/RESTful-api-server.git

$ cd RESTful-api-server

$ npm install

# Sequelize CLI를 전역으로 설치
$ npm install -g sequelize-cli

$ npm run dev
```

# 회원가입 페이지 구현
## Signup 컴포넌트
### Signup 컴포넌트 작성
src/pages 밑에 Signup.vue 컴포넌트를 작성한다.  
**src/pages/Signup.vue**  
```html
<template>
    <div class="sign-up-page">
        <h3>회원가입</h3>
    </div>
</template>
<script>
export default {
    name: 'Signup',
}
</script>
```
### 라우터 연동
src/router/index.js 파일 내에 "/signup" 라우트, Path에 Signup 컴포넌트를 등록한다.  
**src/router/index.js**  
```javascript
import Vue from 'vue'
import Router from 'vue-router'

// ...

// Signup 컴포넌트 import
import Signup from '@/pages/Signup';

export default new Router({
    mode: 'history',
    routes: [
        // ...
        {
            path: '/signup',
            name: 'Signup',
            component: Signup,
        }
    ]
})
```
http://localhost:8080/signup 으로 접속해보자
![이미자](./img/ROUTER_SIGNUP_SCREENSHOT_1.png)

## SignupForm 컴포넌트
SignupForm 컴포넌트는 Signup 컴포넌트의 자식 컴포넌트이다.  
### SignupForm 컴포넌트 작성
```html
<template>
    <form novalidate>
        <fieldset>
            <input type="text" v-model="name" placeholder="이름"/>
            <input type="email" v-model="email" placeholder="이메일"/>
            <input type="password" v-model="password" placeholder="비밀번호"/>
            <input type="password" v-model="passwordConfirm" placeholder="비밀번호 확인"/>
            <button type="submit"> 회원가입 </button>
        </fieldset>
    </form>
</template>

<script>
export default {
    name: 'SignupForm',
    data(){
        return {
            name: '',
            email: '',
            password: '',
            passwordConfirm: ''
        }
    }
}
</script>
```
### Signup 컴포넌트 내에 연동
이제 SignupForm 컴포넌트를 Signup 컴포넌트 내에 연동해보자.  
```html
<template>
    <div class="sign-up-page">
        <h3>회원가입</h3>
        <signup-form/>
    </div>
</template>
<script>
import SignupForm from '@/components/SignupForm'

export default {
    name: 'Signup',
    components:{
        SignupForm
    }
}
</script>
```
**결과화면**  
![이미자](./img/ROUTER_SIGNUP_FORM_SCREENSHOT_1.png)  

### submit 이벤트 연결
#### 참고) axios 커스텀 설정
> **참고)**  
> axios 관련 설정은 setting-axios-simple.md 를 확인하고 설정하자.  

#### SignupForm
##### 템플릿
@submit.prevent 이벤트에 대해 submit 함수를 연결시켜주자.  
```html
<template>
    <form @submit.prevent="submit" novalidate>
        ...
    </form>
</template>
```
##### 스크립트
submit 함수를 호출하도록 해주었으니 submit() 메서드를 컴포넌트내에서 찾을 수 있도록 methods 내에 선언해주자.  

그리고 부모 컴포넌트인 Signup 컴포넌트로 데이터를 전달할 수 있도록 data() 영역 내에 데이터의 객체 리터럴을 지정해주자.  

```javascript
export default {
    name: 'SignupForm',
    data(){
        return {
            name: '',
            email: '',
            password: '',
            passwordConfirm: ''
        }
    },
    methods:{
        submit(){
            const {name, email, password, passwordConfirm} = this;
            if(!name || !email || !password || !passwordConfirm){
                alert('항목이 누락되었습니다.');
            }
            if(password !== passwordConfirm){
                alert('비밀번호를 확인해주세요.');
                return;
            }
            this.$emit('submit', {name, email, password});
        }
    }
}
```
#### Signup
##### 템플릿
submit 이벤트를 onSubmit 함수에 연결시켜주자.
```html
<template>
    <div class="sign-up-page">
        ...
        <signup-form @submit="onSubmit"/>
    </div>
</template>
```
##### 스크립트
- axios 를 커스터마이징한 api 를 import 한다.
- onSubmit() 함수를 methods 항목에 추가해주자.
- onSubmit() 함수 내에는 api 모듈을 이용해 /auth/signup 에 회원가입 요청을 보내는 로직을 작성한다.
```javascript
// ...
import api from '@/api';

export default {
    name: 'Signup',
    // ... 
    methods: {
        onSubmit(payload){
            console.log(payload);
            const{email,password,name} = payload;

            api.post('/auth/signup', { name, email, password })
            .then(res => {
                alert('회원가입이 완료되었습니다.')
                // this.$router.push({ name: 'Signin' })
            })
            .catch(err => {
                alert(err.response.data.msg)
            })
        }
    }
}
```
![이미자](./img/SIGNUP_SCREENSHOT_1.png)  
  
# 로그인(JWT) 기능 구현
## 컴포넌트 구성
### Signup.vue - 로그인 페이지 이동 링크 추가
회원가입 페이지에서 이미 가입한 회원일 경우에 로그인하도록 유도하는 링크를 추가해주자.
#### 템플릿
```html
<template>
    <div class="sign-up-page">
        ...
        <p>
            이미 가입하셨나요?
            <router-link :to="{name: 'Signin'}">
                로그인하러 가기
            </router-link>
        </p>
    </div>
</template>
```

### Signin.vue 
#### 컴포넌트 작성
src/pages 밑에 Signin.vue 를 만들어주자.
```html
<template>
    <div class="sign-in-page">
        <h3>로그인</h3>
        <p>회원이 아니신가요? 
            <router-link :to="{name: 'Signup'}">
                회원가입하러 가기
            </router-link>
        </p>
    </div>
</template>
<script>
export default {
    name: 'Signin'
}
</script>
```
#### 라우터 등록
**src/router/index.js**  
```javascript
import Vue from 'vue'
import Router from 'vue-router'

// ...

// Signin 컴포넌트 import
import Signin from '@/pages/Signin';

Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    // ...
    {
      path: '/signin',
      name: 'Signin',
      component: Signin,
    }
  ]
})
```

### SigninForm.vue
로그인 폼을 표현해주는 컴포넌트이다. SigninForm.vue 라는 파일명으로 src/components/ 밑에 생성해주자.  
  
부모 컴포넌트인 Signin 컴포넌트에게 'submit' 이벤트를 전파하고 있다.
**src/components/SigninForm.vue**  
```html
<template>
    <form @submit.prevent="submit">
        <fieldset>
            <input type="text" v-model="email" placeholder="이메일">
            <input type="password" v-model="password" placeholder="비밀번호">
            <button type="submit"> 로그인 </button>
        </fieldset>
    </form>
</template>
<script>
export default {
    name: 'SigninForm',
    data(){
        return {
            email: '',
            password: '',
        }
    },
    methods: {
        submit(){
            const {email, password} = this
            this.$emit('submit', {email, password})
        }
    }
}
</script>
```

### Signin 컴포넌트에 SigninForm 컴포넌트 연결
#### 템플릿
SigninForm 컴포넌트를 템플릿에 추가해주고, SigninForm 으로부터 올라오는 @submit 이벤트에 대해 onSubmit() 메서드를 핸들러로 사용하도록 템플릿을 구성하자.  

```html
<template>
    <div class="sign-in-page">
        ...
        <signin-form @submit="onSubmit"></signin-form>
        <p>
            ...
        </p>
    </div>
</template>
```
이제 Sigin.vue의 스크립트 단에서
- onSubmit() 함수 구현
- SigninForm 컴포넌트를 script에서 import
- components에 SigninForm을 등록
해주는 작업이 남았다.  

#### 스크립트
```javascript
// SigninForm 컴포넌트 import
import SigninForm from '@/components/SigninForm';

// 커스텀 axios 모듈 import
import api from '@/api/'

export default {
    // ...
    // component 'SigninForm' 등록
    components:{
        SigninForm
    }
    // 
    methods: {
        onSubmit(payload){
            console.log(payload);
        },
    },
}
```
![이미자](./img/SIGNIN_SCREENSHOT_1.png)

#### API 요청 로직 추가
Signin.vue 내의 onSubmit() 함수 내에 아래와 같이 로그인을 요청하는 로직을 작성하자.
```javascript
// SigninForm 컴포넌트 import 
import SigninForm from '@/components/SigninForm';

// 커스텀 axios 모듈 import 
import api from '@/api/'

export default {
    // ...
    methods: {
        onSubmit(payload){
            const { email, password } = payload;
            api.post('/auth/signin', { email, password })
            .then(res => {
                console.log('success');
                const { accessToken } = res.data;
                console.log('accessToken :: ', accessToken)
            })
        },
    }
    // ...
}
```

### Signup.vue :: 회원가입 성공시 로그인페이지로 리다이렉트
회원가입 컴포넌트인 Signup.vue에서 우리가 하나 빼먹고 하지 않은 것이 있다. 회원가입이 완료되면 로그인 페이지로 이동하게 하는 것이다. 이제 소스를 보는게 더 익숙하니... 소스부터 보자!!...  
```html
<template>
    ...
    템플릿에서는 손댈게 없다.
<template>
<script>
// ...
import Signin from '@/pages/Signin' 
// ...

export default {
    // ... 

    methods: {
        onSubmit(payload){
            const{email,password,name} = payload;

            api.post('/auth/signup', { name, email, password })
            .then(res => {
                alert('회원가입이 완료되었습니다.')
                // 이 부분이 추가되었다. 
                this.$router.push({ name: 'Signin' })
            })
            .catch(err => {
                alert(err.response.data.msg)
            })
        }
    }
}
</script>
```
onSubmit 함수에 회원가입 API 요청이 성공하면 
> this.$router.push({name: 'Signin'});
과 같은 소스를 입력해주어야 Signin으로 이동이된다. 라우터에 대한 자세한 내용은 추후 정리할..... 지 말지 고민중 ㅋㅋㅋ  

## 애플리케이션(front-end)에서 JWT 인증
우리가 이미 github에서 clone 받은 샘플 서버는 DB에 데이터(email, password)가 존재하면 accessToken을 만들어 브라우저에게 전송해준다.

### accessToken 의 모습
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiZW1haWwiOiJzZ2p1bmdAZ21haWwuY29tIiwibmFtZSI6InNnanVuZyIsImlhdCI6MTU4NzAyODg5NiwiZXhwIjoxNTg3NjMzNjk2fQ.QToQO8AG8HMjAHYcKLcd1IMCRXEaNNQGr7TEHCgswt8
```

### accessToken 의 형식
... ㅋㅋㅋ 아.... 이건 진짜 내일 정리해야겠다... 너무 힘들쟈나~...  

### 심플 로그인 로직 구현
### vuex) 스토어 기반 로그인 로직으로의 전환
### vuex) accessToken 으로 서버에 사용자 정보 요청
