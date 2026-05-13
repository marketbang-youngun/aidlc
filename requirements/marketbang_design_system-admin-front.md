# AI Guide
## 기술 스택
| 항목 | 버전 |
|------|------|
| Vue | 3.2.x |
| Vue Router | 4.x |
| Pinia | 2.x (페이지 단위 상태) |
| CoreUI Vue | 4.1.x (UI 컴포넌트) |
| Axios | 1.2.x |
| CKEditor 5 | 35.4.x |
| SASS | scoped lang="sass" |
| Vue CLI | 5.x (빌드 도구) |
| 배포 | AWS S3 + CloudFront |
---
## 디렉토리 구조
```
src/
├── api/                  # API 호출 모듈 (도메인별 분리)
│   ├── common/           # 공통 API (upload, category, channel...)
│   └── ...
├── pinia/                # Pinia 스토어 (페이지 단위)
│   └── ...
├── store/                # Vuex 스토어 (전역 세션/유저)
│   └── index.js
├── views/                # 페이지 컴포넌트
│   ├── product/
│   └── ...
├── components/           # 공용 컴포넌트
│   └── modal/            # 모달 컴포넌트
├── layouts/              # 레이아웃 (DefaultLayout.vue)
├── router/               # 라우터
├── environments.js       # 환경별 API URL
├── Send.js               # Axios 인터셉터 래퍼
└── main.js
```
---
## 코딩 컨벤션
### Vue SFC 작성 순서
```
<template> → <script setup> → <style scoped lang="sass">
```
### Script Setup
- `import{ ref }from"vue"` — 공백 없이 붙여 쓰는 스타일 (프로젝트 전반 규칙)
- `defineProps`, `defineEmits` 는 반드시 `"vue"`에서 명시적 import
- `const` 변수명은 `snake_case`
- 컴포넌트명(import) 은 `PascalCase`
```js
// 올바른 예
import{defineProps, ref, computed}from"vue";
const product_store = ProductStore();
const image_show = ref(false);
```
### Template
- CoreUI 컴포넌트 사용: `CCard`, `CRow`, `CCol`, `CForm`, `CFormInput`, `CFormSelect` 등
- 폼 요소에는 `name` 속성 필수 (form 직접 접근 시 사용)
- `size="sm"` 기본 사용
- https://coreui.io/demos/bootstrap/latest/free/?theme=light(템플릿으로)
### SASS (scoped)
- `::v-deep()` 로 자식 컴포넌트 스타일 침투
- `%placeholder` 활용하여 중복 제거
---
### Pinia — 페이지 단위 상태
등록/수정 페이지처럼 복잡한 폼 상태 관리.
```js
// 사용 방법
import{ ProductStore }from"@store/product";
const product_store = ProductStore();
// provide/inject 패턴 (ProductCreate.vue 방식)
provide("wine_create_store", wine_create_store);
// 자식 컴포넌트에서
const wine_create_store = inject("wine_create_store");
```
> **규칙**: `RelatedProduct.vue` 계열은 Pinia store를 자식 컴포넌트에서 직접 import.
> `ProductCreate.vue` 계열은 `provide/inject` 패턴 사용.

### Provide/Inject 자동 unwrap 주의
```js
const store = inject("store");
store.value          // :x: ref가 아니므로 .value 불필요
store.someProp       // :white_check_mark: 직접 접근
```
---
## API 패턴
모든 API 호출은 `Send.js` 래퍼를 통해 Axios 호출.
```js
// src/api/product/product.js
import Send from"@/Send.js";
export default {
    getData(params, data){
        return Send({"method":"post", "url":"/api/seller/v1/goods/list", params, data});
    },
};
```
```js
// 컴포넌트에서 사용
import ApiProduct from"@/api/product/product";
ApiProduct.getData(params, data).then(response=>{
    const data = response.data;
    if(data.code !== 0){alert(data.message); return;}
    // 성공 처리
});
```
---
## 모달 컴포넌트 패턴
CModal visibility는 `isOpen` prop + `@close` fallthrough 방식 사용.
`defineEmits(["close"])` 사용 금지 — Vue 3 fallthrough attrs 로 처리.
```vue
<!-- ModalComponent.vue -->
<CModal :visible="isOpen">...</CModal>
<script setup>
import{ defineProps }from"vue";
const props = defineProps({
    "isOpen": {"type": Boolean, "default": false},
    "onSuccess": {"type": Function, "default": null},    // emit 대신 Function prop
    "onCompleteClose": {"type": Function, "default": null},
});
</script>
<!-- 부모에서 사용 -->
<MyModal
    :isOpen="show_modal"
    :onSuccess="data=>{ /* 처리 */ }"
    :onCompleteClose="()=>{show_modal=false;}"
    @close="show_modal=false"
/>
```
---
## CKEditor 5 패턴
```js
// 에디터 인스턴스를 store에 저장 → 제출 시 getData() 로 직접 읽음
// (source editing 모드에서 v-model sync 누락 방지)
@ready="editor=>{
    product_store.ck_editor = editor;
    editor.plugins.get("FileRepository").createUploadAdapter = loader=>new MyUploadAdapter(loader);
}"
```
```js
// store dataSetting 에서
"wineGoodAbstractText": ck_editor.value ? ck_editor.value.getData() : wineGoodAbstractText.value,
```
## 주요 규칙 요약
1. **`defineProps` / `defineEmits`는 반드시 `"vue"`에서 import** (eslint `no-undef`)
2. **모달 close는 fallthrough attrs 방식** — `defineEmits(["close"])` 금지
3. **CKEditor 인스턴스는 store에 저장** — 제출 시 `getData()` 사용
4. **form 직접 접근**: `form.$el["input_name"]` 방식
5. **스크롤**: `window.scrollTo` + `.header` 높이 보정 + `nextTick`
6. **textarea Enter 허용**: `@keydown.enter` 핸들러에서 `tagName !== "TEXTAREA"` 예외 처리
7. **Pinia store 반환 시 `ref` 그대로 노출** — storeToRefs 사용 안 함
8. 부연설명과 추가 설명 없이 되도록 짧고 간결하게 답변
9. My code style preference is: import test from"test"; import{ test }from"test"; import{test1, test2}from"test"; const test = ref("test"); const{ test }=test(); const{test1, test2}=test(); const test = {"key1": value1, "key2": value2, "key3": value3}; const test = ["test1", "test2"]; const test = ()=>{ return; } const test = test=>{ } const test = (test1, test2)=>{ } const test = function( test ){ } const test = function(test1, test2){ } if(test === "test"){ }else if( test ){ }else{ } test("test"); test(test1, test2); test( test );
10. 문자열에는 ' 대신 "를 사용하고, 들여쓰기는 탭 4 크기로 하되 처음부터 들여쓰지 말고, 코드 마지막에는 항상 세미콜론을 붙이고, 객체 스타일은 {"key": value}로, import 스타일은 import test from"test"; (단일일 경우 import{ test }from"test";, 복수일 경우 import{test1, test2}from"test";)로 하며, function 스타일은 ()=>{} , test=>{} param이 한개일 경우 () 생략 , (test1, test2)=>{}로 =>의 공백제거 하고, if 스타일은 if(test === "test"){ }else if( test ){ }else{ }로 한다. 대체적으로 불필요한 공백은 제거된 스타일이다.
11. vue3 composition api script setup을 기준으로 코드를 보여주세요.