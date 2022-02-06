# css 밖으로 빼서 scss로 관리하기

참고자료

[https://uxgjs.tistory.com/256](https://uxgjs.tistory.com/256)

[https://velog.io/@jipro/Vue.js-컴포넌트에서-scss-파일-import-할-때-주의사항](https://velog.io/@jipro/Vue.js-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%97%90%EC%84%9C-scss-%ED%8C%8C%EC%9D%BC-import-%ED%95%A0-%EB%95%8C-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD)

[https://qiita.com/oka91/items/71cc1ab7aadbc41aa0c1](https://qiita.com/oka91/items/71cc1ab7aadbc41aa0c1)

---

> nuxt 이든 그냥 vue든 사용할 수 있다. 여기서는 nuxt로 예를 들겠다
> 

### assets/scss로 variables.scss를 옮긴다

- 큰 이유는 없고 assets 폴더 안에서 관리를 쉽게 하기 위해 (이미지 파일도 assets으로 들어오게 되므로)

### variables.scss를 보고있던 nuxt.config.js에서 경로를 재설정 해준다

- nuxt.config.js에서 보고있던 경로가 아닌 scss폴더로 varialbes.scss를 옮겼으므로

```swift
vuetify: {
    customVariables: ['~/assets/scss/variables.scss'], // 이부분 scss 폴더를 추가했다
    treeShake: true,
    theme: {
      dark: true,
      themes: {
        dark: {
          primary: colors.blue.darken2,
          accent: colors.grey.darken3,
          secondary: colors.amber.darken3,
          info: colors.teal.lighten1,
          warning: colors.amber.base,
          error: colors.deepOrange.accent4,
          success: colors.green.accent3,
        },
      },
    },
  },
```

### 아래와 같이 폴더를 생성 해 줄 것 (정해진건 아니지만 관리를 쉽게 하기 위해 나눔)

<img src="https://user-images.githubusercontent.com/46176241/152669869-ef82a2fd-24b2-49a5-a563-65bafef7d67c.png" width="300"> 


### 각 파일의 정의

- _index.scss : variables.scss를 제외한 다른 scss를 모으기 위해 사용, 즉 import만 함
- variables.scss : 변수 정의만 함(변수 등록하면 어떤  scss에서도 사용가능) 
따로 클래스를 생성하진 않는다. 이유로서는 컴포넌트 개수만큼 실행되기 때문
- common.scss : 웹사이트에 공통으로 정의 될  css
- 나머지 폴더에 들어있는 파일들 일반 scss 정의 파일 ( vue 파일과 같은 이름으로 정의해놓고 사용하는게 좋을 듯 싶다)

### _index.scss파일에 각 폴더에서 정의한  scss 파일들을 import 해 줄것

_index.scss

```jsx
//전체 페이지에서 공통으로 사용할 스타일
@import "./common.scss";

//layout에 적용 될 스타일
@import "./1_layouts/admin.scss";
@import "./1_layouts/user.scss";

//about.vue 스타일
@import "./2_pages/about.scss";
```

### 정의한 _index.scss를 프로젝트 전체에서 사용할 수 있게 해 줘야 하므로 vuetify 정의파일에서 import해 줘야 한다

- nuxt의 경우 
.nuxt/vuetify/plugin.js

```jsx
import Vue from 'vue'
import Vuetify from 'vuetify/lib/framework'

import options from './options'
import '../../assets/scss/_index.scss'  // <- 이 부분 추가

Vue.use(Vuetify, {
})

export default (ctx) => {
  const vuetifyOptions = typeof options === 'function' ? options(ctx) : options

  vuetifyOptions.icons = vuetifyOptions.icons || {}
  vuetifyOptions.icons.iconfont = 'mdi'

  const vuetify = new Vuetify(vuetifyOptions)

  ctx.app.vuetify = vuetify
  ctx.$vuetify = vuetify.framework
}
```

- vue의 경우
frontend(프로젝트 폴더 이름)/src/plugins/vuetify.js 에 위처럼 import 해 주자

### variables.scss 는 변수만 정의 한다. 이곳에 정의를 하면 어떤  scss 에서도 사용할 수 있다

variables.scss 아래와 같이 변수만 정의해 두자

```jsx
$font-size-root: 20px;
$title-color:white;
```

# 하지만 단점 ! (내가 모를 수도 있다)

만약 about.scss 와 index.scss에서 같은.title이란 클래스를 정의 했을 때

vue 파일에서 .title을 쓰면 어느쪽 title을 불러와야할까?

그렇기 때문에(해결 방법을 못 찾았기 때문에) _index.scss없이 그냥 각 vue 컴포넌트에서 불러오는게 편할 것 같았다

about.vue 의 style 부분

```jsx
<style lang="scss" scoped>
  @import '~scss/2_pages/about.scss';
</style>
```

위의 ~ 부분은 nuxt.config.js 혹은 vue.config.js에서 alias를 설정해 줘야 사용할 수 있다

nuxt.config.js

```jsx
alias: {
  'scss': resolve(__dirname, './assets/scss'),
},
```

이렇게 되면 import한 정의만 사용할 수 있기 때문에 다른  scss에 영향없이 독립적으로 사용할 수 있다
