---
title: typescript入门实战
date: 2019-04-23 09:48:37
tags: typescript
---

### 基础教程  

```
function Hello({ name, enthusiasmLevel = 1 }: Props) {
  if (enthusiasmLevel <= 0) {
    throw new Error('You could be a little more enthusiastic. :D');
  }

  return (
    <div className="hello">
      <div className="greeting">
        Hello {name + getExclamationMarks(enthusiasmLevel)}
      </div>
    </div>
  );
}
```
这种是通过无状态组件（stateless function component （SFC）））的形式引入，但是我们更常用的是通过类形式引入一个组件.
```
class Hello extends React.Component<Props, object> {
  render() {
    const { name, enthusiasmLevel = 1 } = this.props;

    if (enthusiasmLevel <= 0) {
      throw new Error('You could be a little more enthusiastic. :D');
    }

    return (
      <div className="hello">
        <div className="greeting">
          Hello {name + getExclamationMarks(enthusiasmLevel)}
        </div>
      </div>
    );
  }
}
我们只需要考虑到props的类型，所以通过泛型接口的形式定义为React.Component<Props, object>,其中Props是我们类中的this.props,object是类中的this.state,此处我们只对props进行了校验。
```

createStore<StoreState>函数的泛型就收四个参数，按照教程提示就会报相应的错误，应该修正为
```
const store = createStore<StoreState, any, {}, {}>(enthusiasm, {
  enthusiasmLevel: 1,
  languageName: 'TypeScript',
});
```

- withRouter的作用  

把不是通过路由切换过来的组件中，将react-router的history、location、match三个对象传入props对象上。  

```
// 使用方式
***
class Test extends Component {
  render() {
  }
}
export default withRouter(Test)
```




问题： /Users/kirin/node_modules/@types/react/index.d.ts
(2312,14): Duplicate identifier 'LibraryManagedAttributes'.

解决：*"skipLibCheck"*: true此方法不能从根本上解决问题。

问题：Parsing error: The keyword 'import' is reserved eslint
解决：项目根目录加入.eslintrc.json文件，并加入以下配置。
```
"ecmaFeatures": {
    "jsx": true,
    "modules": true
}
替换为
"parserOptions": {
    "ecmaFeatures": {
        "jsx": true,
        "modules": true
    }
}

或者加入以下配置，并安装相关依赖
"parser": "babel-eslint",

- npm install -g eslint@1.x.x babel-eslint@4.x.x
- npm install -g eslint-plugin-react@3.x.x
```

问题： The class method 'render' must be marked either 'private', 'public', or 'protected'
解决： 此问题主要是由于tslint的规范，可以关闭此校验，通过在tslint.json中配置`member-access: false`,[详情可参考](https://palantir.github.io/tslint/rules/member-access/)

问题： Class name must be in pascal case tslint
解决： 没找到合适的解决方案，最后只能取消eslint中extends中的配置，将tslint.json文件中`extends: []`

问题： Module '"/Users/kirin/study/front/ts-study/node_modules/@types/react/index"' has no default export.
解决： 添加`"allowSyntheticDefaultImports": true`配置到tsconfig.json,或者改成`import * as React from 'react'`