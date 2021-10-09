---
title: 前端 | Vue2 初始化流程 - compile
date: 2021-8-2
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

## compile

在 init 的最后，会进入挂载阶段，倘若有定义 el ，那么就会执行挂载操作，对于组件则会直接跳过

```js
if (vm.$options.el) {
  vm.$mount(vm.$options.el) // 若当前组件参数中有节点，则执行挂载操作
}
```

### mount -- with compiler

由于这里跑的的 web-full-dev , 所以会有编译过程。这里的编译过程是拦截 mount 方法，在得到编译方法后挂载在 options 上再调用原来的 mount 方法执行真正的挂载过程

首先是查询传入的挂载点 el ，允许用户直接传入 dom 节点，如果传入的是字符串，那么就是获取 dom 节点再返回；接着排除掉 body 和 html 节点

接着检测用户是否有写入 render ，倘若有 render 则直接执行挂载操作；倘若无，
再判断用户是否有直接写入 template ，再判断 template 是否是字符串，或者 template 是有 nodeType 属性的，那就说明 template 直接是个 dom，这时获取 innerHTML ，或者传入无效的 template 就直接抛出警告；若用户没有写入 template ，那么就直接令 template 等于 el的 outerHtml，这个也没有就只能创建个 div 了

那么经上，template 便准备好了，接着提取 render, staticRenderFns 两个方法，绑在 Vue 的 $options 上

至此编译初始化就走完了，最后再用开头保存的原本 mount 进行挂载

```js
// platforms/web/entry-runtime-with-compiler.js

const mount = Vue.prototype.$mount

Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options // 为合并后的配置项
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      // compileToFunctions 函数由 ./compiler/index 文件 createCompiler(baseOptions) 执行返回的对象中获取
      // baseOptions 作为基本编译配置被传入到 compiler/index 文件中 createCompiler = createCompilerCreator(baseCompile) 函数执行后返回的 createCompiler 函数中
      // 也就是 createCompilerCreator 以 baseCompile 函数为参数执行 , 返回的函数被传入之前的配置 baseOptions , 后赋值给 createCompiler 函数

      // createCompiler 函数内定义的 compile 函数使用 baseCompile 函数来进行编译 , 并将结果返回
      // createCompiler 函数再将 compile 函数与 createCompileToFunctionFn(compile) 的执行结果 compileToFunctions 打包成对象一并返回
      // 而此处调用的 compileToFunctions 实际上就是 createCompileToFunctionFn(compile) 执行返回的同名函数 compileToFunctions , 接收以下三个参数
      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters, // Vue 实例的配置
        comments: options.comments
      }, this) // 传入编译的配置
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}
```

### 编译方法加载过程

#### compileToFunctions <- createCompiler

获取编译方法的过程比较复杂，被封装了很多层，主要针对不同的运行环境，浏览器、服务端，会有不同的编译参数配置，但也提取出了公共的部分，以此做到各自分离

首先来看，调用的 compileToFunctions 方法是从 `./compiler/index` 当前目录下编译目录导入的，如下：

而此处的 baseOptions 也是 web 端对应的编译配置，被传入 createCompiler 

```js
// platforms/web/compiler/index.js

import { baseOptions } from './options'
import { createCompiler } from 'compiler/index'

const { compile, compileToFunctions } = createCompiler(baseOptions)

export { compile, compileToFunctions }
```

#### createCompiler <- createCompilerCreator

接着来看这个方法, 来自通用的编译方法 `compiler/index`, 这里查看导入的路径很重要，因为断点会直接执行到 return 返回的函数中

可以看到，这里的 createCompiler 函数是 createCompilerCreator 函数传入 baseCompile 函数后执行返回的函数

而此处需要注意的是，baseCompile 函数是编译的关键步骤，涉及模板的解析，优化，最终生成代码

```js
// compiler/index.js

import { parse } from './parser/index'
import { optimize } from './optimizer'
import { generate } from './codegen/index'
import { createCompilerCreator } from './create-compiler'

export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options) // 将属性，标签解析出，生成 AST 树
  if (options.optimize !== false) {
    optimize(ast, options) // 优化语法树
  }
  const code = generate(ast, options) // 生成代码
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```

#### createCompilerCreator

首先可以注意到 createCompilerCreator 函数接受 baseCompile 函数执行后返回的就是上一级 createCompiler 函数；

而 createCompiler 函数其内则将 基本的编译配置 baseOptions 合并后联通 template 传入 baseCompile 函数进行实质上的编译

最终 createCompiler 函数返回的对象就有我们一开始看到的 compileToFunctions ，而这个函数实际上又是 createCompileToFunctionFn 函数传入 createCompiler 内定义的 compile 函数执行后返回的函数

```js
// compiler/create-compiler.js
import { extend } from 'shared/util'
import { detectErrors } from './error-detector'
import { createCompileToFunctionFn } from './to-function'

export function createCompilerCreator (baseCompile: Function): Function { // 创建一个编译器构造器，用于传递
  return function createCompiler (baseOptions: CompilerOptions) {
    function compile (
      template: string,
      options?: CompilerOptions
    ): CompiledResult {
      const finalOptions = Object.create(baseOptions) // 创建 finalOptions 用来合并 base 的和挂载时传入的 options
      const errors = []
      const tips = []

      let warn = (msg, range, tip) => {
        (tip ? tips : errors).push(msg)
      }

      if (options) {
        if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
          // $flow-disable-line
          const leadingSpaceLength = template.match(/^\s*/)[0].length

          warn = (msg, range, tip) => {
            const data: WarningMessage = { msg }
            if (range) {
              if (range.start != null) {
                data.start = range.start + leadingSpaceLength
              }
              if (range.end != null) {
                data.end = range.end + leadingSpaceLength
              }
            }
            (tip ? tips : errors).push(data)
          }
        }
        // merge custom modules
        if (options.modules) {
          finalOptions.modules =
            (baseOptions.modules || []).concat(options.modules) // 合并 baseOptions x options 里模块部分
        }
        // merge custom directives
        if (options.directives) {
          finalOptions.directives = extend( //  // 合并 baseOptions x options 里指令部分
            Object.create(baseOptions.directives || null),
            options.directives
          )
        }
        // copy other options
        for (const key in options) { //  // 合并 options 里其他配置
          if (key !== 'modules' && key !== 'directives') {
            finalOptions[key] = options[key]
          }
        }
      }

      finalOptions.warn = warn

      // 实际进行编译的步骤 ，此时 finalOptions 为最终的编译配置
      const compiled = baseCompile(template.trim(), finalOptions)
      if (process.env.NODE_ENV !== 'production') {
        detectErrors(compiled.ast, warn)
      }
      compiled.errors = errors
      compiled.tips = tips
      return compiled
    }

    return {
      compile,
      compileToFunctions: createCompileToFunctionFn(compile)
    }
  }
}
```

#### createCompileToFunctionFn

该函数返回的 compileToFunctions 便是我们编译的开始，核心也就是 compile 执行的步骤，给 compile 传入的 options 会与之前 createCompiler 函数传入 baseOptions 合并形成最终的编译配置项，再由 baseCompile 来完成编译

```js
// compiler/to-function.js

type CompiledFunctionResult = {
  render: Function;
  staticRenderFns: Array<Function>;
};

function createFunction (code, errors) {
  try {
    return new Function(code)
  } catch (err) {
    errors.push({ err, code })
    return noop
  }
}

// 定义了 createCompileToFunctionFn(compile) 函数，返回 compileToFunctions 函数，在 $mount 时被调用
export function createCompileToFunctionFn (compile: Function): Function {
  const cache = Object.create(null)

  return function compileToFunctions (
    template: string,
    options?: CompilerOptions,
    vm?: Component
  ): CompiledFunctionResult {
    options = extend({}, options) // 将 options 的属性混入一个空对象
    const warn = options.warn || baseWarn
    delete options.warn

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      // detect possible CSP restriction
      try {
        new Function('return 1')
      } catch (e) {
        if (e.toString().match(/unsafe-eval|CSP/)) {
          warn(
            'It seems you are using the standalone build of Vue.js in an ' +
            'environment with Content Security Policy that prohibits unsafe-eval. ' +
            'The template compiler cannot work in this environment. Consider ' +
            'relaxing the policy to allow unsafe-eval or pre-compiling your ' +
            'templates into render functions.'
          )
        }
      }
    }

    // check cache
    const key = options.delimiters
      ? String(options.delimiters) + template
      : template
    if (cache[key]) {
      return cache[key]
    }

    // compile
    const compiled = compile(template, options) // 将模板与编译配置传入 compile 函数

    // check compilation errors/tips
    if (process.env.NODE_ENV !== 'production') {
      if (compiled.errors && compiled.errors.length) {
        if (options.outputSourceRange) {
          compiled.errors.forEach(e => {
            warn(
              `Error compiling template:\n\n${e.msg}\n\n` +
              generateCodeFrame(template, e.start, e.end),
              vm
            )
          })
        } else {
          warn(
            `Error compiling template:\n\n${template}\n\n` +
            compiled.errors.map(e => `- ${e}`).join('\n') + '\n',
            vm
          )
        }
      }
      if (compiled.tips && compiled.tips.length) {
        if (options.outputSourceRange) {
          compiled.tips.forEach(e => tip(e.msg, vm))
        } else {
          compiled.tips.forEach(msg => tip(msg, vm))
        }
      }
    }

    // turn code into functions
    const res = {}
    const fnGenErrors = []
    res.render = createFunction(compiled.render, fnGenErrors)
    res.staticRenderFns = compiled.staticRenderFns.map(code => {
      return createFunction(code, fnGenErrors)
    })

    // check function generation errors.
    // this should only happen if there is a bug in the compiler itself.
    // mostly for codegen development use
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      if ((!compiled.errors || !compiled.errors.length) && fnGenErrors.length) {
        warn(
          `Failed to generate render function:\n\n` +
          fnGenErrors.map(({ err, code }) => `${err.toString()} in\n\n${code}\n`).join('\n'),
          vm
        )
      }
    }

    return (cache[key] = res)
  }
}
```

### compileToFunctions

接上编译过程，执行到 compileToFunctions ，传入 template、编译配置、Vue

compileToFunctions 在得到生成的代码 render 后，又将代码转换为 函数与 staticRenderFns 合并为对象保存在缓存 cache 里并返回

```js
const { render, staticRenderFns } = compileToFunctions(template, {
  outputSourceRange: process.env.NODE_ENV !== 'production',
  shouldDecodeNewlines,
  shouldDecodeNewlinesForHref,
  delimiters: options.delimiters, // Vue 实例的配置
  comments: options.comments
}, this) // 传入编译的配置
options.render = render
options.staticRenderFns = staticRenderFns
```

再来看 compileToFunctions 方法

首先将传入的 options 保存, 再将模板与编译配置传入 compile 函数，结果保存为 compiled

```js
// compiler/to-function.js

export function createCompileToFunctionFn (compile: Function): Function {
  const cache = Object.create(null)

  return function compileToFunctions (
    template: string,
    options?: CompilerOptions,
    vm?: Component
  ): CompiledFunctionResult {
    options = extend({}, options) // 将 options 的属性混入一个空对象
    const warn = options.warn || baseWarn
    delete options.warn

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      // detect possible CSP restriction
      try {
        new Function('return 1')
      } catch (e) {
        if (e.toString().match(/unsafe-eval|CSP/)) {
          warn(
            'It seems you are using the standalone build of Vue.js in an ' +
            'environment with Content Security Policy that prohibits unsafe-eval. ' +
            'The template compiler cannot work in this environment. Consider ' +
            'relaxing the policy to allow unsafe-eval or pre-compiling your ' +
            'templates into render functions.'
          )
        }
      }
    }

    // check cache
    const key = options.delimiters
      ? String(options.delimiters) + template
      : template
    if (cache[key]) {
      return cache[key]
    }

    // compile
    const compiled = compile(template, options) // 将模板与编译配置传入 compile 函数

    // check compilation errors/tips
    if (process.env.NODE_ENV !== 'production') {
      if (compiled.errors && compiled.errors.length) {
        if (options.outputSourceRange) {
          compiled.errors.forEach(e => {
            warn(
              `Error compiling template:\n\n${e.msg}\n\n` +
              generateCodeFrame(template, e.start, e.end),
              vm
            )
          })
        } else {
          warn(
            `Error compiling template:\n\n${template}\n\n` +
            compiled.errors.map(e => `- ${e}`).join('\n') + '\n',
            vm
          )
        }
      }
      if (compiled.tips && compiled.tips.length) {
        if (options.outputSourceRange) {
          compiled.tips.forEach(e => tip(e.msg, vm))
        } else {
          compiled.tips.forEach(msg => tip(msg, vm))
        }
      }
    }

    // turn code into functions
    const res = {}
    const fnGenErrors = []
    res.render = createFunction(compiled.render, fnGenErrors)
    res.staticRenderFns = compiled.staticRenderFns.map(code => {
      return createFunction(code, fnGenErrors)
    })

    // check function generation errors.
    // this should only happen if there is a bug in the compiler itself.
    // mostly for codegen development use
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      if ((!compiled.errors || !compiled.errors.length) && fnGenErrors.length) {
        warn(
          `Failed to generate render function:\n\n` +
          fnGenErrors.map(({ err, code }) => `${err.toString()} in\n\n${code}\n`).join('\n'),
          vm
        )
      }
    }

    return (cache[key] = res)
  }
}
```

### compile

将 baseOptions 和 options 合并为 finalOptions ，将 template 和 finalOptions 传入 baseCompile 进行编译

```js
// compiler/create-compiler.js

function compile (
  template: string,
  options?: CompilerOptions
): CompiledResult {
  const finalOptions = Object.create(baseOptions) // 以 baseOptions 为原型创建 finalOptions
  const errors = []
  const tips = []

  let warn = (msg, range, tip) => {
    (tip ? tips : errors).push(msg)
  }

  if (options) {
    if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
      // $flow-disable-line
      const leadingSpaceLength = template.match(/^\s*/)[0].length

      warn = (msg, range, tip) => {
        const data: WarningMessage = { msg }
        if (range) {
          if (range.start != null) {
            data.start = range.start + leadingSpaceLength
          }
          if (range.end != null) {
            data.end = range.end + leadingSpaceLength
          }
        }
        (tip ? tips : errors).push(data)
      }
    }
    // merge custom modules
    if (options.modules) {
      finalOptions.modules =
        (baseOptions.modules || []).concat(options.modules) // 合并 baseOptions x options 里模块部分
    }
    // merge custom directives
    if (options.directives) {
      finalOptions.directives = extend( //  // 合并 baseOptions x options 里指令部分
        Object.create(baseOptions.directives || null),
        options.directives
      )
    }
    // copy other options
    for (const key in options) { //  // 合并 options 里其他配置
      if (key !== 'modules' && key !== 'directives') {
        finalOptions[key] = options[key]
      }
    }
  }

  finalOptions.warn = warn

  // 实际进行编译的步骤 ，此时 finalOptions 为最终的编译配置
  const compiled = baseCompile(template.trim(), finalOptions)
  if (process.env.NODE_ENV !== 'production') {
    detectErrors(compiled.ast, warn)
  }
  compiled.errors = errors
  compiled.tips = tips
  return compiled
}
```

注意是以 baseOptions 为原型创建 finalOptions, finalOptions 再与 options 进行合并，所以合并后 baseOptions 的配置方法参数等会在 finalOptions 的原型中, 最终 finalOptions 如下

```js
finalOptions 
  comments:undefined
  delimiters:(2) ['$[', ']']
  outputSourceRange:true
  shouldDecodeNewlines:false
  shouldDecodeNewlinesForHref:false
  warn:ƒ (msg, range, tip) 
  [[prototype]]
    canBeLeftOpenTag:ƒ (val) 
    directives:{model: ƒ, text: ƒ, html: ƒ}
    expectHTML:true
    getTagNamespace:ƒ getTagNamespace (tag) 
    isPreTag:ƒ (tag) 
    isReservedTag:ƒ (tag) 
    isUnaryTag:ƒ (val) 
    modules:(3) [{…}, {…}, {…}]
      0:{staticKeys: Array(1), transformNode: ƒ, genData: ƒ}
      1:{staticKeys: Array(1), transformNode: ƒ, genData: ƒ}
      2:{preTransformNode: ƒ}
    mustUseProp:ƒ (tag, type, attr)
    staticKeys:'staticClass,staticStyle'
```

### baseCompile

baseCompile 是编译的核心，此时传入了模板和编译配置，开始编译

baseCompile 做了三件事：模板的解析，优化，最终生成代码

```js
// compiler/index.js

export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options) // 将属性，标签解析出，生成 AST 树
  if (options.optimize !== false) {
    optimize(ast, options) // 优化语法树
  }
  const code = generate(ast, options) // 生成代码
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```

#### parse

获取一些前置的解析方法，接着进入到 parseHTML 解析 html，传入参数模板 template，以及各项配置，其中配置中带有 start、end、chars、comment 方法；

```js
// compiler/parser/index.js

function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
  warn = options.warn || baseWarn

  platformIsPreTag = options.isPreTag || no
  platformMustUseProp = options.mustUseProp || no
  platformGetTagNamespace = options.getTagNamespace || no
  const isReservedTag = options.isReservedTag || no
  maybeComponent = (el: ASTElement) => !!el.component || !isReservedTag(el.tag)

  transforms = pluckModuleFunction(options.modules, 'transformNode')
  preTransforms = pluckModuleFunction(options.modules, 'preTransformNode')
  postTransforms = pluckModuleFunction(options.modules, 'postTransformNode')

  delimiters = options.delimiters

  const stack = []
  const preserveWhitespace = options.preserveWhitespace !== false
  const whitespaceOption = options.whitespace
  let root
  let currentParent
  let inVPre = false
  let inPre = false
  let warned = false

    // 传入模板，以及一个配置对象
  parseHTML(template, {
    warn,
    expectHTML: options.expectHTML,
    isUnaryTag: options.isUnaryTag,
    canBeLeftOpenTag: options.canBeLeftOpenTag,
    shouldDecodeNewlines: options.shouldDecodeNewlines,
    shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
    shouldKeepComment: options.comments,
    outputSourceRange: options.outputSourceRange,
    start (tag, attrs, unary, start, end) {
      // check namespace.
      // inherit parent ns if there is one
      const ns = (currentParent && currentParent.ns) || platformGetTagNamespace(tag)

      // handle IE svg bug
      /* istanbul ignore if */
      if (isIE && ns === 'svg') {
        attrs = guardIESVGBug(attrs)
      }

      let element: ASTElement = createASTElement(tag, attrs, currentParent) // 创建 AST 元素
      if (ns) {
        element.ns = ns
      }

      if (process.env.NODE_ENV !== 'production') {
        if (options.outputSourceRange) {
          element.start = start
          element.end = end
          element.rawAttrsMap = element.attrsList.reduce((cumulated, attr) => {
            cumulated[attr.name] = attr
            return cumulated
          }, {})
        }
        attrs.forEach(attr => {
          if (invalidAttributeRE.test(attr.name)) {
            warn(
              `Invalid dynamic argument expression: attribute names cannot contain ` +
              `spaces, quotes, <, >, / or =.`,
              {
                start: attr.start + attr.name.indexOf(`[`),
                end: attr.start + attr.name.length
              }
            )
          }
        })
      }

      // 处理 AST 元素
      // 判断是否为禁止的 tag 和服务端渲染
      if (isForbiddenTag(element) && !isServerRendering()) {
        element.forbidden = true
        process.env.NODE_ENV !== 'production' && warn(
          'Templates should only be responsible for mapping the state to the ' +
          'UI. Avoid placing tags with side-effects in your templates, such as ' +
          `<${tag}>` + ', as they will not be parsed.',
          { start: element.start }
        )
      }

      // apply pre-transforms
      // 对创建的 ASTElement 做预转换
      for (let i = 0; i < preTransforms.length; i++) {
        element = preTransforms[i](element, options) || element
      }

      if (!inVPre) {
        processPre(element)
        if (element.pre) {
          inVPre = true
        }
      }
      if (platformIsPreTag(element.tag)) {
        inPre = true
      }
      if (inVPre) {
        processRawAttrs(element)
      } else if (!element.processed) {
        // structural directives
        // 构建以前命令
        processFor(element) // 从元素中拿到 v-for 指令的内容，然后分别解析出 for、alias、iterator1、iterator2 等属性的值添加到 AST 的元素
        processIf(element) // 从元素中拿 v-if 指令的内容
        processOnce(element)
      }

      // 如果不存在 root 则将新建的 element 作为 root ,在检查 root 的约束条件 , 有问题的 tag 及 属性 v-for 均不能在 root 存在
      if (!root) {
        root = element
        if (process.env.NODE_ENV !== 'production') {
          checkRootConstraints(root)
        }
      }

      if (!unary) {
        currentParent = element
        stack.push(element)
      } else {
        closeElement(element)
      }
    },

    end (tag, start, end) {
      const element = stack[stack.length - 1]
      // pop stack
      stack.length -= 1
      currentParent = stack[stack.length - 1]
      if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
        element.end = end
      }
      closeElement(element)
    },

    chars (text: string, start: number, end: number) {
      if (!currentParent) {
        if (process.env.NODE_ENV !== 'production') {
          if (text === template) {
            warnOnce(
              'Component template requires a root element, rather than just text.',
              { start }
            )
          } else if ((text = text.trim())) {
            warnOnce(
              `text "${text}" outside root element will be ignored.`,
              { start }
            )
          }
        }
        return
      }
      // IE textarea placeholder bug
      /* istanbul ignore if */
      if (isIE &&
        currentParent.tag === 'textarea' &&
        currentParent.attrsMap.placeholder === text
      ) {
        return
      }
      const children = currentParent.children
      if (inPre || text.trim()) {
        text = isTextTag(currentParent) ? text : decodeHTMLCached(text)
      } else if (!children.length) {
        // remove the whitespace-only node right after an opening tag
        text = ''
      } else if (whitespaceOption) {
        if (whitespaceOption === 'condense') { // 如果此处文本样式 whitespace 配置为 condense，有换行符的取消，没有的单空格
          // in condense mode, remove the whitespace node if it contains
          // line break, otherwise condense to a single space
          text = lineBreakRE.test(text) ? '' : ' '
        } else {
          text = ' '
        }
      } else {
        text = preserveWhitespace ? ' ' : ''
      }
      if (text) {
        if (!inPre && whitespaceOption === 'condense') {
          // condense consecutive whitespaces into single space
          text = text.replace(whitespaceRE, ' ')
        }
        let res
        let child: ?ASTNode
        if (!inVPre && text !== ' ' && (res = parseText(text, delimiters))) {
          child = {
            type: 2, // 有表达式
            expression: res.expression,
            tokens: res.tokens,
            text
          }
        } else if (text !== ' ' || !children.length || children[children.length - 1].text !== ' ') {
          child = {
            type: 3, // 纯文本
            text
          }
        }
        if (child) {
          if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
            child.start = start
            child.end = end
          }
          children.push(child)
        }
      }
    },
    comment (text: string, start, end) {
      // adding anything as a sibling to the root node is forbidden
      // comments should still be allowed, but ignored
      if (currentParent) {
        const child: ASTText = {
          type: 3,
          text,
          isComment: true
        }
        if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
          child.start = start
          child.end = end
        }
        currentParent.children.push(child)
      }
    }
  })
```

##### parseHTML

parseHTML 将传入的 template 虚幻遍历，解析出 AST 树

过程中会匹配注释的tag，doctype 的地方，结束tag，开始tag

首先肯定是开始 tag 的匹配，以栈的形式先压入开始的 tag，记录 tag 的名字，接着截取一下，循环匹配它对应的属性。接着进入 handleStartTag 处理开始标签，将之前分割开的属性按 name, value 的形式放入对象，按顺序存入 attrs 中。将非一元的标签压入栈中，再由 parseRndTag 弹出匹配结束标签

接着判断 parseHtml 中传入的配置是否有 start ，调用 start 处理刚才得到的 tag 名字，属性名字，开始结束的索引。

调用 createASTElement 函数创建 AST 树，接着处理生成的 AST 树，看属性中是否有禁止的属性，是否存在 v-pre ；然后继续构建 vue 的指令，从元素中拿到 v-for 指令的内容，然后分别解析出 for、alias、iterator1、iterator2 等属性的值添加到 AST 的元素；从元素中拿 v-if 指令的内容；从元素中拿 v-once 指令的内容。

然后判断是否存在 root ， 如果不存在 root 则将新建的 element 树作为 root ， 再检查 root 的约束条件 , 有问题的 tag 及 属性 v-for 均不能在 root 存在。

再判断是否是一元标签，如不是令 element 树作为当前的父级，并将 element 树入栈

之后重复上边的操作，遍历模板，匹配下一个开始标签 < , 用 advance 将模板不断裁剪，生成 AST 树，不断完善栈

如果遇到文本，则会在 chars 方法里边解析文本，获取用户定义的变量边框 delimiters 或者就是 vue 默认的双花括号 ，匹配其中的变量，再用 _s() 来预置以便之后渲染变量

```js
// compiler/parser/html-parser.js

export function parseHTML (html, options) {
  const stack = []
  const expectHTML = options.expectHTML
  const isUnaryTag = options.isUnaryTag || no
  const canBeLeftOpenTag = options.canBeLeftOpenTag || no
  let index = 0
  let last, lastTag
  while (html) {
    last = html
    // Make sure we're not in a plaintext content element like script/style
    if (!lastTag || !isPlainTextElement(lastTag)) {
      let textEnd = html.indexOf('<')
      if (textEnd === 0) {
        // Comment:
        if (comment.test(html)) {
          const commentEnd = html.indexOf('-->')

          if (commentEnd >= 0) {
            if (options.shouldKeepComment) {
              options.comment(html.substring(4, commentEnd), index, index + commentEnd + 3)
            }
            advance(commentEnd + 3)
            continue
          }
        }

        // http://en.wikipedia.org/wiki/Conditional_comment#Downlevel-revealed_conditional_comment
        if (conditionalComment.test(html)) {
          const conditionalEnd = html.indexOf(']>')

          if (conditionalEnd >= 0) {
            advance(conditionalEnd + 2)
            continue
          }
        }

        // Doctype:
        const doctypeMatch = html.match(doctype)
        if (doctypeMatch) {
          advance(doctypeMatch[0].length)
          continue
        }

        // End tag:
        const endTagMatch = html.match(endTag)
        if (endTagMatch) {
          const curIndex = index
          advance(endTagMatch[0].length)
          parseEndTag(endTagMatch[1], curIndex, index)
          continue
        }

        // Start tag:
        const startTagMatch = parseStartTag() // 解析开始的标签，把 template 转化成节点名和属性
        if (startTagMatch) {
          handleStartTag(startTagMatch)
          if (shouldIgnoreFirstNewline(startTagMatch.tagName, html)) {
            advance(1)
          }
          continue
        }
      }

      let text, rest, next
      if (textEnd >= 0) {
        rest = html.slice(textEnd) // 将结尾标签截出
        while (
          !endTag.test(rest) &&
          !startTagOpen.test(rest) &&
          !comment.test(rest) &&
          !conditionalComment.test(rest)
        ) {
          // < in plain text, be forgiving and treat it as text
          next = rest.indexOf('<', 1)
          if (next < 0) break
          textEnd += next
          rest = html.slice(textEnd)
        }
        text = html.substring(0, textEnd)
      }

      if (textEnd < 0) {
        text = html
      }

      if (text) {
        advance(text.length) // 更新 index html
      }

      if (options.chars && text) {
        options.chars(text, index - text.length, index)
      }
    } else {
      let endTagLength = 0
      const stackedTag = lastTag.toLowerCase()
      const reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'))
      const rest = html.replace(reStackedTag, function (all, text, endTag) {
        endTagLength = endTag.length
        if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') {
          text = text
            .replace(/<!\--([\s\S]*?)-->/g, '$1') // #7298
            .replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1')
        }
        if (shouldIgnoreFirstNewline(stackedTag, text)) {
          text = text.slice(1)
        }
        if (options.chars) {
          options.chars(text)
        }
        return ''
      })
      index += html.length - rest.length
      html = rest
      parseEndTag(stackedTag, index - endTagLength, index)
    }

    if (html === last) {
      options.chars && options.chars(html)
      if (process.env.NODE_ENV !== 'production' && !stack.length && options.warn) {
        options.warn(`Mal-formatted tag at end of template: "${html}"`, { start: index + html.length })
      }
      break
    }
  }

  // Clean up any remaining tags
  parseEndTag()
  // ...
}
```

##### AST

例子中最后生成 AST 树如下:

```js
ast
  attrs:(1) [{…}]
    0:{name: 'id', value: '"app"', dynamic: undefined, start: 5, end: 13}
  attrsList:(1) [{…}]
    0:{name: 'id', value: 'app', start: 5, end: 13}
  attrsMap:{id: 'app'}
  start:0
  end:110
  parent:undefined
  plain:false
  rawAttrsMap:{id: {…}}
  children:(3) [{…}, {…}, {…}]
    0:{type: 1, tag: 'div', attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
    1:{type: 3, text: ' ', start: 71, end: 80}
    2:{type: 1, tag: 'balance', attrsList: Array(0), attrsMap: {…} }
  tag:'div'
  type:1
```

#### optimize

Vue 是数据驱动，是响应式的，但是我们的模板并不是所有数据都是响应式的，也有很多数据是首次渲染后就永远不会变化的
遍历已生成的 AST 树，并侦测完全稳定的副树，而 dom 部分不需要改变
侦测副树我们可以提升它们到常量，以便于我们不需要在每个 re-render 时给它们创建新的节点；在打包过程中完全跳过它们

因此 optimize 环节也是优化 ast ， 标记 静态节点 和 静态根 的时候

会将生成好的 ast 树递归进行标记，判断当前节点 isStatic(node)

```js
// compiler/optimizer.js

export function optimize (root: ?ASTElement, options: CompilerOptions) {
  if (!root) return
  isStaticKey = genStaticKeysCached(options.staticKeys || '')
  isPlatformReservedTag = options.isReservedTag || no
  // first pass: mark all non-static nodes.
  markStatic(root) // 标记静态节点
  // second pass: mark static roots.
  markStaticRoots(root, false) // 标记静态根
}

// 用来判断是否静态节点
function isStatic (node: ASTNode): boolean {
  if (node.type === 2) { // expression
    return false
  }
  if (node.type === 3) { // text
    return true
  }
  return !!(node.pre || ( // 使用 v-for 指令是静态元素
    !node.hasBindings && // no dynamic bindings
    !node.if && !node.for && // not v-if or v-for or v-else
    !isBuiltInTag(node.tag) && // not a built-in
    isPlatformReservedTag(node.tag) && // not a component
    !isDirectChildOfTemplateFor(node) &&
    Object.keys(node).every(isStaticKey)
  ))
}
```

#### generate

传入优化好后的 ast 树与 options 传入 generate 用来生成 render 代码

先生成 el 的代码，根据各项有无生成 data，再生成子树的代码，拼接起来

```js
// compiler/codegen/index.js

export function generate (
  ast: ASTElement | void,
  options: CompilerOptions
): CodegenResult {
  const state = new CodegenState(options)
  const code = ast ? genElement(ast, state) : '_c("div")'
  return {
    render: `with(this){return ${code}}`,
    staticRenderFns: state.staticRenderFns
  }
}

// genElement
export function genElement (el: ASTElement, state: CodegenState): string {
  if (el.parent) {
    el.pre = el.pre || el.parent.pre
  }

  if (el.staticRoot && !el.staticProcessed) { // 将 optimize 里标记的静态节点先生成
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    return genSlot(el, state)
  } else {
    // component or element
    let code
    if (el.component) {
      code = genComponent(el.component, el, state)
    } else {
      let data
      if (!el.plain || (el.pre && state.maybeComponent(el))) {
        data = genData(el, state)
      }

      const children = el.inlineTemplate ? null : genChildren(el, state, true)
      code = `_c('${el.tag}'${
        data ? `,${data}` : '' // data
      }${
        children ? `,${children}` : '' // children
      })`
    }
    // module transforms
    for (let i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code)
    }
    return code
  }
}

// genData
export function genData (el: ASTElement, state: CodegenState): string {
  let data = '{'

  // directives first.
  // directives may mutate the el's other properties before they are generated.
  const dirs = genDirectives(el, state)
  if (dirs) data += dirs + ','

  // key
  if (el.key) {
    data += `key:${el.key},`
  }
  // ref
  if (el.ref) {
    data += `ref:${el.ref},`
  }
  if (el.refInFor) {
    data += `refInFor:true,`
  }
  // pre
  if (el.pre) {
    data += `pre:true,`
  }
  // record original tag name for components using "is" attribute
  if (el.component) {
    data += `tag:"${el.tag}",`
  }
  // module data generation functions
  for (let i = 0; i < state.dataGenFns.length; i++) {
    data += state.dataGenFns[i](el)
  }
  // attributes
  if (el.attrs) {
    data += `attrs:${genProps(el.attrs)},`
  }
  // DOM props
  if (el.props) {
    data += `domProps:${genProps(el.props)},`
  }
  // event handlers
  if (el.events) {
    data += `${genHandlers(el.events, false)},`
  }
  if (el.nativeEvents) {
    data += `${genHandlers(el.nativeEvents, true)},`
  }
  // slot target
  // only for non-scoped slots
  if (el.slotTarget && !el.slotScope) {
    data += `slot:${el.slotTarget},`
  }
  // scoped slots
  if (el.scopedSlots) {
    data += `${genScopedSlots(el, el.scopedSlots, state)},`
  }
  // component v-model
  if (el.model) {
    data += `model:{value:${
      el.model.value
    },callback:${
      el.model.callback
    },expression:${
      el.model.expression
    }},`
  }
  // inline-template
  if (el.inlineTemplate) {
    const inlineTemplate = genInlineTemplate(el, state)
    if (inlineTemplate) {
      data += `${inlineTemplate},`
    }
  }
  data = data.replace(/,$/, '') + '}'
  // v-bind dynamic argument wrap
  // v-bind with dynamic arguments must be applied using the same v-bind object
  // merge helper so that class/style/mustUseProp attrs are handled correctly.
  if (el.dynamicAttrs) {
    data = `_b(${data},"${el.tag}",${genProps(el.dynamicAttrs)})`
  }
  // v-bind data wrap
  if (el.wrapData) {
    data = el.wrapData(data)
  }
  // v-on data wrap
  if (el.wrapListeners) {
    data = el.wrapListeners(data)
  }
  return data
}
```

经上此步骤，依次生成各部分 code，再进行拼接得到最后的 render

```js
// 根生成的 code ：
'{attrs:{"id":"app"}}'

// 子树生成的 code ：
'[_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))]),_v(" "),_c('balance')],1'

// 接着将两者拼接成 ：
'_c('div',{attrs:{"id":"app"}},[_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))]),_v(" "),_c('balance')],1)'

// 而最后输出的 render
'with(this){return _c('div',{attrs:{"id":"app"}},[_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))]),_v(" "),_c('balance')],1)}'
```

##### createFunction

根据上边生成的 render code，生成函数

```js
// compiler/to-function.js

function createFunction (code, errors) {
  try {
    return new Function(code)
  } catch (err) {
    errors.push({ err, code })
    return noop
  }
}
```

将 code 生成函数后，就成了如下形式：

_c 为 f createElement(vm, a, b, c, d, false)
_v 为 ƒ createTextVNode (val)
_s 为 ƒ toString (val)

```js
ƒ anonymous() {
  with(this){return _c('div',{attrs:{"id":"app"}},[_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))]),_v(" "),_c('balance')],1)}
}
```

后续见 mount 篇