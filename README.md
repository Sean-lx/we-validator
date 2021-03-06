# we-validator
简单灵活的表单验证插件

[![NPM][img-npm]][url-npm]

[![Build Status][img-travis]][url-travis]
[![Language][img-javascript]][url-github]
[![License][img-mit]][url-mit]

支持微信小程序、支付宝小程序、浏览器以及Nodejs端使用。

## 特点

- 使用简单灵活
- 不依赖任何框架
- 既支持原生微信小程序方式，也支持 mpvue、wepy等小程序框架使用
- 支持web浏览器以及Nodejs端使用
- [支持自定义规则](#自定义规则)
- 支持实例化和[直接调用验证函数](#直接调用验证函数)两种使用方式
- [支持自定义错误消息提示](#自定义错误消息提示)



## 安装
非npm安装方式，直接引入lib目录下的we-validator.js到项目即可
```bash
npm install we-validator --save
```


## 使用

下面是微信小程序的用法，支付宝小程序类似
```html
<form bindsubmit="onSubmitForm">
    <input type="text" name="username" placeholder="用户名" />
    <input type="number" name="phoneno" placeholder="手机号" />
    <input type="text" name="str" placeholder="长度为3的字符串" />

    <button type="default" formType="submit">提交</button>
</form>
```

```javascript
const WeValidator = require('we-validator')

Page({
    onReady(){
        this.initValidator()
    },
    onSubmitForm(e){
        let { value } = e.detail

        if(!this.oValidator.checkData(value)) return

        // 开始提交表单
        // wx.request
    },
    initValidator(){
        // 实例化
        this.oValidator = new WeValidator({
            rules: {
                username: {
                    required: true
                },
                phoneno: {
                    required: true,
                    mobile: true
                },
                str: {
                    required: true,
                    stringLength: 3
                },
            },
            messages: {
                username: {
                    required: '请输入用户名'
                },
                phoneno: {
                    required: '请输入手机号',
                    mobile: '手机号格式不正确'
                },
                str: {
                    required: '请输入字符串',
                    stringLength: '字符串长度不对'
                },
            },
        })
    },
})
```

您可参考当前项目下对应示例
- [原生微信小程序使用方式](./example/wechat/pages/index/index.js)
- [mpvue 使用方式](./example/mpvue/src/pages/index/index.vue)
- [web 浏览器使用方式](./example/web/index.html)

## 参数

- `rules` - 验证字段的规则
- `messages` - 验证字段错误的提示信息
- `onMessage` - 错误提示显示方式，默认会自动检测环境。小程序默认使用`showToast`，普通web浏览器默认使用`alert`，Nodejs端不做处理建议自己配置，[配置方式](#自定义错误消息提示)

## 支持的正则类型

具体正则内容可查看[源码](./src/rules.js)
- `bankCard` - 银行卡
- `mobile` -  手机号
- `mobileWithSpace` -  手机号（带空格`131 2233 4455`）
- `idCard` -  身份证
- `chinese` -  中文
- `chinese2to8` -  中文（2-8位）
- `intOrFloat` -  整数或小数
- `int` -  整数
- `noZeroStart` -  非零开头的数字
- `specialStr` -  含有^%&',;=?$\"等特殊字符
- `email` -  邮箱
- `httpUrl` -  InternetURL地址
- `tel` -  电话号码,正确格式为："XXX-XXXXXXX"、"XXXX-XXXXXXXX"、"XXX-XXXXXXX"、"XXX-XXXXXXXX"、"XXXXXXX"和"XXXXXXXX"
- `money` -  货币
- `month` -  一年的12月，正确格式为："01"～"09"和"1"～"12"
- `day` -  一个月的31天,正确格式为；"01"～"09"和"1"～"31"
- `html` -  匹配html标签
- `spaceEnter` -  匹配空行
- `qq` -   qq号码
- `zip` -  邮编
- `doubleByte` -  匹配双字节字符(包括汉字在内)

## 规则

除了支持以上所有正则规则，以下特殊规则也支持，可动态传参（上面正则和下面特殊规则的使用方式一样）

- `required`: true，必填
- `regex`: RegExp，正则通用校验
- `equal`: WeValidator.$value(compareName)，字段值是否相同，例如二次密码校验，[参考下面栗子](#值相同校验)
- `intGreater`: n，大于n的数字
- `intLength`: n，只能输入n位的数字
- `intLessLength`: n，至少n位数字
- `intLengthRange`: [n, m]，n到m位数字
- `decimalLength`: n，只能输入有n位小数的正实数
- `decimalLengthRange`: [n, m]，只能输入有n~m位小数的正实数
- `stringLength`: n，长度为n的字符串
- `stringLetter`: aorA，由26个英文字母组成的字符串，大写或小写类型，A表示大写，a表示小写，其它表示不限制大小写
- `stringLetterDefault`: true，由数字、26个英文字母或者下划线组成的字符串

#### 动态参数的使用

```javascript
const WeValidator = require('we-validator')

// 使用方式一，实例化（推荐）
new WeValidator({
    rules: {
        field1: {
            intGreater: 6 // 大于6的数字
        },
        field2: {
            intLengthRange: [2, 5] // 2-5位数字
        }
    },
    messages: {
        field1: {
            intGreater: '请输入大于6的数字'
        },
        field2: {
            intLengthRange: '请输入2-5位数字'
        }
    }
})

// 使用方式二，调用函数
let b1 = WeValidator.intGreater('str', 6) // 大于6的数字
let b2 = WeValidator.intLengthRange('str', 2, 5) // 2-5位数字
```

#### 值相同校验

例如：二次密码校验
```javascript
const WeValidator = require('we-validator')

// 实例化
new WeValidator({
    rules: {
        pwd1: {
            required: true
        },
        pwd2: {
            required: true,
            equal: WeValidator.$value('pwd1')
        }
    },
    messages: {
        pwd1: {
            required: '请输入密码'
        },
        pwd2: {
            required: '请输入确认密码',
            equal: '两次密码不一致'
        }
    }
})
```

## 方法

#### 自定义规则

```javascript
const WeValidator = require('we-validator')

// 添加自定义规则
WeValidator.addRule('theRuleName', function(value, param){
    return /\d/.test(value)
})

// 使用方式一，实例化
new WeValidator({
    rules: {
        field1: {
            theRuleName: true
        }
    },
    message: {
        field1: {
            theRuleName: '提示信息'
        }
    }
})

// 使用方式二，调用函数
WeValidator.theRuleName('str')
```

#### 直接调用验证函数

更灵活，支持以上所有规则类型的函数调用
```javascript
const WeValidator = require('we-validator')

let b1 = WeValidator.required('str')
let b2 = WeValidator.mobile('str')
let b3 = WeValidator.idCard('str')
let b4 = WeValidator.intLengthRange('str', 2, 5) // 2-5位数字
```

#### 自定义错误消息提示

可以全局配置一个，也可以单独配置。优先级是：验证的时候onMessage > 实例化参数onMessage > 全局onMessage > 默认检测
```javascript
const WeValidator = require('we-validator')

// 全局配置
WeValidator.onMessage = function(data){
    /*
    data 参数
    {
        msg, // 提示文字
        name, // 表单控件的 name
        value, // 表单控件的值
        param // rules 验证字段传递的参数 []
    }
    */
}

// 实例化配置
new WeValidator({
    rules: {},
    message: {},
    onMessage: function(data){
        alert(data.msg)
    }
})

// 验证的时候配置 onMessage
if(!obj.checkData(formData, (data) => {
    alert(data.msg)
})){
     return
}
```

## 协议

[![license][img-mit]][url-mit]


[url-github]: https://github.com/ChanceYu/we-validator
[url-npm]: https://www.npmjs.com/package/we-validator
[url-travis]: https://travis-ci.org/ChanceYu/we-validator
[url-mit]: https://opensource.org/licenses/mit-license.php

[img-npm]: https://nodei.co/npm/we-validator.png?compact=true
[img-travis]: https://travis-ci.org/ChanceYu/we-validator.svg?branch=master
[img-javascript]: https://img.shields.io/badge/language-JavaScript-brightgreen.svg
[img-mit]: https://img.shields.io/badge/license-MIT-blue.svg

