> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/2105c48b45c7)

async-validator
===============

async-validator 是一个表单的异步验证的第三方库，它是 [](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ftmpfs%2Fasync-validate)[https://github.com/tmpfs/async-validate](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ftmpfs%2Fasync-validate) 的演变。也是 [](https://links.jianshu.com/go?to=https%3A%2F%2Felement.eleme.cn%2F%23%2Fzh-CN%2Fcomponent%2Finstallation)[element-ui](https://links.jianshu.com/go?to=https%3A%2F%2Felement.eleme.cn%2F%23%2Fzh-CN%2Fcomponent%2Finstallation) 中的 form 组件所使用的验证方式。

API
---

> 注意：以下内容是从早期版本的异步验证修改而来的。

### install

```
npm install --save async-validator
```

### 使用

基本用法包括定义一个 descriptor，将其分配给 schema，并将要验证的对象和回调函数传递给 schema 创建出来的 validator 的 validate 方法：

```
// 基本用法
var schema = require('async-validator'); // 引用组件
var descriptor = {
 name: {
  type: "string",
  required: true,
  validator: (rule, value) => value === 'muji',
 }
}; // 定义一个descriptor
var validator = new schema(descriptor); // descriptor分配给schema，创建一个validator
validator.validate({name: "muji"}, (errors, fields) => {
 if(errors) {
  // validation failed, errors is an array of all errors
  // fields is an object keyed by field name with an array of
  // errors per field
 return handleErrors(errors, fields);
 }
  // validation passed
}); // 参数一：要验证的对象、参数二：回调函数

// 使用 promise
validator.validate({
  name: "muji",
 asyncValidator: (rule, value) => axios.post('/nameValidator', { name: value }),
}, (errors, fields) => {
 if(errors) {
  // validation failed, errors is an array of all errors
  // fields is an object keyed by field name with an array of
  // errors per field
 return handleErrors(errors, fields);
 }
  // validation passed
})
.then(() => {
  // validation passed
})
.catch(({ errors, fields }) => {
 return handleErrors(errors, fields);
})
```

### Validate

```
function(source, [options], callback): Promise
```

*   `source`: 要验证的对象（必需)
*   `options`: 描述验证处理选项的对象（可选）
*   `callback`: 验证完成时调用的回调函数（必需）

该方法将返回 Promise 对象，如：

*   `then()`: 验证通过
*   `catch({ errors, fields })`: 验证失败，errors 是一个包含所有错误的数组，fields 是由字段名称做键值对的对象所组成的数组。

#### Options

*   `suppressWarning`: Boolean，是否禁止有关无效值的内部警告。
*   `first`: Boolean，当第一个验证规则生成错误时调用回调，不再处理其他验证规则。如果验证涉及多个异步调用（例如数据库查询），并且只需要第一个错误，请使用此选项。
*   `firstFields`: Boolean|String[], 当指定字段的第一个验证规则生成错误时调用回调，不处理同一字段的其他验证规则。“真” 表示所有字段。

### Rules

> Rules 可以是执行验证的函数，例如上面：var descriptor = {name: {} } 中的`name`，不仅仅是一个对象，还可以是一个函数。相关如下：

```
function(rule, value, callback, source, options)
```

*   `rule`: 源描述符中与要验证的字段名相对应的验证规则。它始终被分配一个字段属性，该属性具有要验证的字段的名称。
*   `value`: 正在验证的源对象属性的值。
*   `callback`: 验证完成后调用的回调函数。它需要传递一个错误实例数组来指示验证失败。如果检查是同步的，则可以直接返回一个错误、错误或错误数组。
*   `source`: 传递给 validate 方法的源对象。
*   `options`: 附加选项。
*   `options.messages`: 包含验证错误消息的对象将与 defaultmessages 深度合并。

传递给 validate 或 asyncvalidate 的选项将传递给验证函数，以便您可以在验证函数中临时引用数据（例如模型引用）。但是，某些选项名是保留的；如果使用选项对象的这些属性，它们将被覆盖。保留的属性是消息、异常和错误。

```
var schema = require('async-validator');
var descriptor = {
 name(rule, value, callback, source, options) {
 var errors = [];
 if(!/^[a-z0-9]+$/.test(value)) {
 errors.push(
 new Error(
 util.format("%s must be lowercase alphanumeric characters",
 rule.field)));
 }
 return errors;
 }
}
var validator = new schema(descriptor);
validator.validate({name: "Firstname"}, (errors, fields) => {
 if(errors) {
 return handleErrors(errors, fields);
 }
  // validation passed
});
```

针对单个字段测试多个验证规则通常很有用，这样可以使规则成为对象数组，例如：

```
var descriptor = {
 email: [
 { type: "string", required: true, pattern: schema.pattern.email },
 {
 validator (rule, value, callback, source, options) {
 var errors = [];
  // test if email address already exists in a database
  // and add a validation error to the errors array if it does
 return errors;
 }
 }
 ]
}
```

### 规则的参数

##### type

要使用的验证程序的类型，识别的类型值如下：

*   `string`: 必须是 String 类型。这是默认类型。
*   `number`: 必须是 Number 类型。
*   `boolean`: 必须是 Boolean 类型。
*   `method`: 必须是 Function 类型。
*   `regexp`: 必须是 RegExp 的实例或在创建新 RegExp 时不生成异常的字符串。
*   `integer`: 必须是 Number 和整数类型。
*   `float`: 必须是 Number 和浮点数类型。
*   `array`: 必须是由 array.isarray 确定的数组。
*   `object`: 必须是 Object 类型，而不是 Array.IsArray 类型。
*   `enum`: 值必须存在于枚举中。
*   `date`: 必须是 Date 类型。
*   `url`: 必须是 url 类型。
*   `hex`: 必须是十六进制类型。
*   `email`: 必须是电子邮件类型。

##### Required

boolean, 是否必填

##### Pattern:

模式规则属性表示正则表达式, 该值必须匹配才能通过验证。

##### Range:

使用 min 和 max 属性定义范围。对于字符串和数组类型，将根据长度进行比较，对于数字类型，数字不得小于 min，也不得大于 max。

##### Length:

验证字段的确切长度。对于字符串和数组类型，对 length 属性执行比较，对于数字类型，此属性指示数字的完全匹配，即，它可能仅严格等于 len。如果 len 属性与最小和最大范围属性组合，则 len 优先。

##### Enumerable:

枚举，要从可能值列表中验证值，请使用带有枚举属性的枚举类型，列出该字段的有效值。

```
var descriptor = {
 role: { type: "enum", enum: ['admin', 'user', 'guest'] }
}
```

##### Whitespace:

空白，通常将仅包含空格的必填字段视为错误。要为仅包含空格的字符串添加其他测试，请将空白属性添加到值为 true 的规则。规则必须是字符串类型。

##### Deep Rules:

如果需要验证深层对象属性，则可以通过将嵌套规则分配给规则的 fields 属性来为对象或数组类型的验证规则执行此操作。

```
// 深度规则
var descriptor = {
 address: {
 type: "object", required: true,
 fields: {
 street: {type: "string", required: true},
 city: {type: "string", required: true},
 zip: {type: "string", required: true, len: 8, message: "invalid zip"}
 }
 },
 name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({ address: {} }, (errors, fields) => {
  // errors for address.street, address.city, address.zip
});
```

> 注意：如果您没有在父规则上指定必需的属性，那么对于不在源对象上声明的字段是完全有效的，并且不会执行深度验证规则，因为没有要验证的内容。

深度规则验证为嵌套规则创建架构，因此还可以指定传递给 schema.validate（）方法的选项。

```
var descriptor = {
 address: {
 type: "object", required: true, options: {single: true, first: true},
 fields: {
 street: {type: "string", required: true},
 city: {type: "string", required: true},
 zip: {type: "string", required: true, len: 8, message: "invalid zip"}
 }
 },
 name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({ address: {} }).catch(({ errors, fields }) => {
// now only errors for street and name    
});
```

父规则也会被验证，因此如果您有一组规则，例如：

```
var descriptor = {
 roles: {
 type: "array", required: true, len: 3,
 fields: {
 0: {type: "string", required: true},
 1: {type: "string", required: true},
 2: {type: "string", required: true}
 }
 }
}
```

并提供 `{roles: ["admin", "user"]}` 这样的源对象，将会创建两个错误。一个用于数组长度不匹配，另一个用于索引 2 处缺少的必需数组项。

##### defaultField:

defaultField 属性可与数组或对象类型一起使用，以验证容器的所有值。它可以是包含验证规则的对象或数组。例如：

```
var descriptor = {
 urls: {
 type: "array", required: true,
 defaultField: {type: "url"}
 }
}
```

##### Transform:

有时需要在验证之前转换值，可能是为了强制价值或以某种方式对其进行消毒。为此，请将验证规则添加到转换有时需要在验证之前转换一个值，可能是强制值或以某种方式对其进行清理。为此，请向验证规则添加转换函数。该属性在验证之前被转换，并重新分配给源对象，以在适当的位置改变该属性的值。

```
var schema = require('async-validator');
var sanitize = require('validator').sanitize;
var descriptor = {
 name: {
  type: "string",
 required: true, pattern: /^[a-z]+$/,
 transform(value) {
 return sanitize(value).trim();
 }
 }
}
var validator = new schema(descriptor);
var  source = {name: " user "};
validator.validate(source).then(() => assert.equal(source.name, "user"));
```

如果没有转换函数，验证将失败，因为模式不匹配，因为输入包含前导空格和尾随空格，但通过添加转换函数验证传递，同时清理字段值。

##### Messages:

消息，根据您的应用程序要求，您可能需要 i18n 支持，或者您可能更喜欢不同的验证错误消息。实现这一点的最简单方法是将消息分配给规则：

```
{name:{type: "string", required: true, message: "Name is required"}}
```

消息可以是任何类型，例如 jsx 格式。

```
{name:{type: "string", required: true, message: <b>Name is required</b>}}
```

消息也可以是一个函数，例如，如果使用 vue-i18n：

```
{name:{type: "string", required: true, message: () => this.$t( 'name is required' )}}
```

对于不同的语言，可能需要相同的模式验证规则，在这种情况下，为每种语言复制模式规则是没有意义的。在这个场景中，您只需为该语言提供您自己的消息并将其分配给模式：

```
var schema = require('async-validator');
var cn = {
 required: '%s 必填',
};
var descriptor = {name:{type: "string", required: true}};
var validator = new schema(descriptor);
// deep merge with defaultMessages
validator.messages(cn);
// ...
```

如果要定义自己的验证函数，最好将消息字符串分配给消息对象，然后通过验证函数内的 options.messages 属性访问消息。

##### asyncValidator

异步验证器，您可以为指定字段自定义异步验证函数：function(rule, value, callback)

```
const fields = {
 asyncField:{
 asyncValidator(rule,value,callback){
 ajax({
 url:'xx',
  value:value
 }).then(function(data){
 callback();
 },function(error){
 callback(new Error(error))
 });
 }
 },
 promiseField:{
 asyncValidator(rule, value){
 return ajax({
 url:'xx',
  value:value
 });
 }
 }
};
```

##### validator:

验证器，您可以为指定字段自定义验证函数：function(rule, value, callback)

```
const fields = {
 field:{
 validator(rule,value,callback){
 return value === 'test';
 },
 message: 'Value is not equal to "test".',
 },
 field2:{
 validator(rule,value,callback){
  return  new Error(`'${value} is not equal to "test".'`);
 },
 },
 arrField:{
 validator(rule, value){
 return [
 new Error('Message 1'),
 new Error('Message 2'),
 ];
 }
 },
};
```

常见问题
----

### 如何避免警告

```
var Schema = require('async-validator');
Schema.warning = function(){};
```