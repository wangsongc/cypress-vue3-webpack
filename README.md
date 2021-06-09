## Cypress组件级测试初探（Vue3+Webpack）

Cypress7.0开始将CypressComponent Test Runner与Cypress Test Runner捆绑在一起，除了原有的E2E测试，目前可以支持组件级测试了；它同时支持Vue2和vue3，并支持Typescript。

### 创建vue3+Webpack工程

```
vue create vue3-webpack
```

### 安装cypress

安装 Cypress 和 Webpack Dev Server 以及 Vue 适配器

```
npm install cypress @cypress/vue@next @cypress/webpack-dev-server --dev
```

### 配置Cypress组件测试运行器

- 找到vue项目下的/cypress/plugins/index.js，配置webpack为server启动器

```
<project dir>/cypress/plugins/index.js
```

```
const { startDevServer } = require('@cypress/webpack-dev-server')
const webpackConfig = require('@vue/cli-service/webpack.config.js')

module.exports = (on, config) => {
  on('dev-server:start', options =>
    startDevServer({
      options,
      webpackConfig
    })
  )

  return config
}
```

### 配置cypress.json，指定测试路径

```
{
  "component": {
    "componentFolder": "src",
    "testFiles": "**/*.spec.ts"
  }
}
```

### 编写测试用例

这里是针对vue3+webpack的demo中HelloWorld组件进行测试。

```
import { mount } from '@cypress/vue'
import HelloWorld from '../components/HelloWorld.vue'

describe('HelloWorld', () => {
  it('renders a message', () => {
    const msg = 'Hello Cypress Component Testing!'
    mount(HelloWorld, {
      propsData: {
        msg
      }
    })

    cy.get('h1').should('have.text', msg)
    cy.get('span').eq(0)
    cy.get('button').click()
    cy.get('span').should(($span) => {
      expect($span.get(0).innerText).to.eq('1') })
  })
})
```

### 运行测试

- 打开cypress运行器界面，执行测试

```
yarn cypress open-ct # or npx cypress open-ct
```

- 无界面测试

```
yarn cypress run-ct # or npx cypress run-ct
```

### 初体验

作为jest+vue-test-utils方案的替代

- 可以让用例在真实浏览器中运行，让测试更贴近用户体验
- 测试运行全程可视化，可以准确的看到页面渲染的内容，不需要再使用`console.log(wrapper.html())`去定位问题了
- 由Cypress提供支持 - 最流行和最可靠的 E2E 测试工具
- 目前cypress组件测试库还在Alpha阶段，功能上可能还不完善，API可能存在突破性变化，使用时也会遇到很多问题

### 使用时遇到的一些问题及解决办法
- eslint报错：error 'cy' is not defined no-undef

解决办法：安装eslint-plugin-cypress，并在eslint配置中添加
```
"eslintConfig": {
    "extends": [
        "plugin:cypress/recommended"
    ]
}
```

- 执行`npx cypress open-ct`时报错：webpack Dev Server Invalid Options, options.proxy should be {Object|Array}

解决办法：找到webpack.config.js 或者 vue.config.js 或者 vue.config.base.js中的proxy配置注释掉

- 找不到@vue/compiler-dom

解决办法：安装依赖，`npm i @vue/compiler-dom`

- cypress运行报错：object(...)is not a function

解决办法: 原因可能是cypress/vue的版本不对，如果是vue3的项目对应安装cypress/vue@next，vue2项目对应安装cypress/vue

- cypress运行报错：Cannot read property 'tap' of undefined

解决办法：确保html-webpack-plugin的版本与webpack的版本保持一致，如webpack用的版本是5，那么html-webpack-plugin版本也要是5，否则会报错