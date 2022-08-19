# 安装eslint
## 安装依赖
```csharp
	"@typescript-eslint/eslint-plugin": "^5.30.5",
    "@typescript-eslint/parser": "^5.30.5",
	"eslint": "^8.19.0",
    "eslint-config-prettier": "^8.5.0",
    "eslint-plugin-prettier": "^4.2.1",
    "eslint-plugin-react": "^7.30.1",
    "eslint-plugin-react-hooks": "^4.6.0",
```
## 配置.eslintrc.js

```javascript
module.exports = {
  extends: ["eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/jsx-runtime",
    "plugin:react-hooks/recommended",
    "prettier",
    "plugin:prettier/recommended"],
  env: {
    browser: true,
    commonjs: true,
    es6: true,
  },
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
      modules: true,
    },
    sourceType: 'module',
    ecmaVersion: 6
  },
  plugins: ["react", "@typescript-eslint", "react-hooks", "prettier"],
  settings: {
    'import/resolver': {
      node: {
        extensions: ['.tsx', '.ts', '.js', '.json'],
      },
      alias: [['@', './src']],
    },
    'react': {
      version: "18.2.0"
    }
  },
  /*
	 * "off" 或 0    ==>  关闭规则
	 * "warn" 或 1   ==>  打开的规则作为警告（不影响代码执行）
	 * "error" 或 2  ==>  规则作为一个错误（代码不能执行，界面报错）
	 */
  rules: {
    // eslint (http://eslint.cn/docs/rules)
    "quotes": [2, "single"],
    "semi": [1, "never"],
    "class-methods-use-this": "off",
    "import/no-named-as-default": "off",
    "import/no-extraneous-dependencies": "off",
    "import/no-unresolved": "off",
    "import/extensions": "off",
    "react/jsx-filename-extension": "off",
    "no-param-reassign": "off",
    "no-unused-expressions": "off",
    "jsx-a11y/alt-text": "off",
    "no-shadow": "off",
    "react/jsx-one-expression-per-line": "off",
    "no-use-before-define": "off",
    "react/button-has-type": "off",
    "no-undef": "off",
    "no-unused-vars": "off",
    "import/prefer-default-export": "off",
    "jsx-a11y/click-events-have-key-events": "off",
    "jsx-a11y/no-noninteractive-element-interactions": "off",
    "import/order": "off",
    "jsx-a11y/anchor-is-valid": "off",
    "jsx-a11y/no-static-element-interactions": "off",
    "react/require-default-props": "off",
    "react/destructuring-assignment": "off",
    "no-nested-ternary": "off",
    "no-console": "off",
    "no-sequences": "off",
    "no-multi-assign": "off",
    "func-names": "off",
    "prefer-destructuring": "off",
    "consistent-return": "off",
    "no-useless-escape": "off",
    "no-new-func": "off",
    "global-require": "off",
    "import/no-dynamic-require": "off",
    "react/display-name": "off",
    "react/prop-types": "off",

    // typeScript (https://typescript-eslint.io/rules)
    "@typescript-eslint/no-unused-vars": "off", // 禁止定义未使用的变量
    "@typescript-eslint/no-inferrable-types": "off", // 可以轻松推断的显式类型可能会增加不必要的冗长
    "@typescript-eslint/no-namespace": "off", // 禁止使用自定义 TypeScript 模块和命名空间。
    "@typescript-eslint/no-explicit-any": "off", // 禁止使用 any 类型
    "@typescript-eslint/ban-ts-ignore": "off", // 禁止使用 @ts-ignore
    "@typescript-eslint/ban-types": "off", // 禁止使用特定类型
    "@typescript-eslint/explicit-function-return-type": "off", // 不允许对初始化为数字、字符串或布尔值的变量或参数进行显式类型声明
    "@typescript-eslint/no-var-requires": "off", // 不允许在 import 语句中使用 require 语句
    "@typescript-eslint/no-empty-function": "off", // 禁止空函数
    "@typescript-eslint/no-use-before-define": "off", // 禁止在变量定义之前使用它们
    "@typescript-eslint/ban-ts-comment": "off", // 禁止 @ts-<directive> 使用注释或要求在指令后进行描述
    "@typescript-eslint/no-non-null-assertion": "off", // 不允许使用后缀运算符的非空断言(!)
    "@typescript-eslint/explicit-module-boundary-types": "off", // 要求导出函数和类的公共类方法的显式返回和参数类型

    // react (https://github.com/jsx-eslint/eslint-plugin-react)
    "react-hooks/rules-of-hooks": "off",
    "react-hooks/exhaustive-deps": "off"
  }
}
```
## 配置.eslintignore

```bash
*.sh
node_modules
*.md
*.woff
*.ttf
.vscode
.idea
dist
/public
/docs
.husky
.local
/bin
.eslintrc.js
.prettierrc.js
/mock/*
/dist/*


```

# 安装eslint
## 依赖
```bash
	"stylelint": "^14.9.1",
    "stylelint-config-prettier": "^9.0.3",
    "stylelint-config-recess-order": "^3.0.0",
    "stylelint-config-standard": "^26.0.0",
    "stylelint-less": "^1.0.6"
```
## 配置.stylelintrc.js

```javascript
// @see: https://stylelint.io

module.exports = {
  extends: [
    "stylelint-config-standard", // 配置stylelint拓展插件
    "stylelint-config-prettier", // 配置stylelint和prettier兼容
    "stylelint-config-recess-order" // 配置stylelint css属性书写顺序插件,
  ],
  plugins: ["stylelint-less"], // 配置stylelint less拓展插件
  rules: {
    indentation: null, // 指定缩进空格
    "no-descending-specificity": null, // 禁止在具有较高优先级的选择器后出现被其覆盖的较低优先级的选择器
    "function-url-quotes": "always", // 要求或禁止 URL 的引号 "always(必须加上引号)"|"never(没有引号)"
    "string-quotes": "double", // 指定字符串使用单引号或双引号
    "unit-case": null, // 指定单位的大小写 "lower(全小写)"|"upper(全大写)"
    "color-hex-case": "lower", // 指定 16 进制颜色的大小写 "lower(全小写)"|"upper(全大写)"
    "color-hex-length": "long", // 指定 16 进制颜色的简写或扩写 "short(16进制简写)"|"long(16进制扩写)"
    "rule-empty-line-before": "never", // 要求或禁止在规则之前的空行 "always(规则之前必须始终有一个空行)"|"never(规则前绝不能有空行)"|"always-multi-line(多行规则之前必须始终有一个空行)"|"never-multi-line(多行规则之前绝不能有空行。)"
    "font-family-no-missing-generic-family-keyword": null, // 禁止在字体族名称列表中缺少通用字体族关键字
    "block-opening-brace-space-before": "always", // 要求在块的开大括号之前必须有一个空格或不能有空白符 "always(大括号前必须始终有一个空格)"|"never(左大括号之前绝不能有空格)"|"always-single-line(在单行块中的左大括号之前必须始终有一个空格)"|"never-single-line(在单行块中的左大括号之前绝不能有空格)"|"always-multi-line(在多行块中，左大括号之前必须始终有一个空格)"|"never-multi-line(多行块中的左大括号之前绝不能有空格)"
    "property-no-unknown": null, // 禁止未知的属性(true 为不允许)
    "no-empty-source": null, // 禁止空源码
    "declaration-block-trailing-semicolon": null, // 要求或不允许在声明块中使用尾随分号 string："always(必须始终有一个尾随分号)"|"never(不得有尾随分号)"
    "selector-class-pattern": null, // 强制选择器类名的格式
    "value-no-vendor-prefix": null, // 关闭 vendor-prefix(为了解决多行省略 -webkit-box)
    "at-rule-no-unknown": null,
    "selector-pseudo-class-no-unknown": [
      true,
      {
        ignorePseudoClasses: ["global", "v-deep", "deep"]
      }
    ]
  }
}

```
## 配置.stylelintignore

```bash
/dist/*
/public/*
public/*

```

# 安装prettier + husky + lint-staged
## 依赖

```bash
"husky": "^8.0.1",
"lint-staged": "^13.0.3",
"prettier": "^2.7.1"
```
## 配置
```bash
1、npx husky install

2、npx husky add .husky/pre-commit "yarn lint-staged --allow-empty"
```
### 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ad9c5a6c6a64c7f8a7105c0602f0fc8.png)
### package.json

```bash
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write --loglevel warn"
    ],
    "src/**/*.{less,postcss,css,scss}": [
      "stylelint --fix --custom-syntax postcss-less --cache --cache-location node_modules/.cache/stylelint/"
    ]
  }
```

# 安装 commitlint

```bash
"@commitlint/cli": "^17.0.3",
"@commitlint/config-conventional": "^17.0.3",
```
## 配置
commitlint.config.js

```javascript
const checkType = (header) => {
  header = `${header}`;
  const enumType = ['feat', 'fix', 'style', 'chore', 'test', 'ci', 'refactor', 'revert', 'reformat', 'docs'];
  const realType = header.split(':')[0];
  return enumType.includes(realType);
};

const checkSubject = (header) => {
  header = `${header}`;
  const realSubject = header.split(':')[1];
  if (!realSubject) {
    return false;
  }
  return realSubject.length > 0;
};


/*
 * @Description: commit-msg提交信息格式规范
 *
 * commit-msg格式: <type>: <subject>
 *     - chore: 其他修改, 比如改变构建流程、或者增加依赖库、工具等
 *     - ci: 持续集成修改
 *     - docs: 文档修改
 *     - feat: 新特性、新功能
 *     - fix: 修改bug
 *     - perf: 优化相关，比如提升性能、体验
 *     - refactor: 代码重构
 *     - revert: 回滚到上一个版本
 *     - style: 代码格式修改, 注意不是 css 修改
 *     - test: 测试用例修改
 */
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum-rule': [2, 'never'],
    'subject-enum-rule': [2, 'never'],
    'type-enum': [0, 'never'],
    'type-empty': [0, 'always'],
    'subject-empty': [0, 'always'],
  },
  plugins: [
    {
      rules: {
        'type-enum-rule': ({ header }) => {
          return [
            checkType(header),
            '需要包含提交类型，格式如: "feat: 开发新功能" 中的feat, ' +
            '可选值有: feat/fix/style/test/chore/ci/..., 类型后面紧跟英文冒号分隔主题信息',
          ];
        },
        'subject-enum-rule': ({ header }) => {
          return [checkSubject(header), '需要包含提交主题, 格式如: "feat: 开发新功能" 中的 开发新功能'];
        },
      },
    },
  ],
};

```

```bash
yarn husky add .husky/commit-msg 'yarn commitlint --edit "$1"'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/08ed4d8ff85741bf9ff96b62a0b0245b.png)

在commit代码时
1、执行 pre-commit 格式化 代码格式
2、执行 commit-msg 进行 commit 校验

效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7641fa4701c4fa38f9cd1ef1ff54a6d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f8ef9c6510b46b2887aeacd111ff962.png)

# 参考这里
https://github.com/GuoguoDad/react-template-vite

## 结束语
- 👀 目前专注于前端
- ⚙️ 在react、react-native开发方面有丰富的经验
- 🔭 最近在学习安卓，有自己的开源安卓项目，集成react-native热更新功能
- 我❤️ 思考、学习、编码和健身
- 如果文章对您有帮助，三连支持一下～O(∩_∩)O谢谢！

