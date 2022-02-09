# InversifyJS

InversifyJS 功能点整理（xmind 脑图文件）

## reflect-medata

### ECMAScript 元数据反射 API 的原型

### Decorator
装饰器

### 像 c# (. net) 和 Java 这样的语言支持向类型添加元数据的属性或注释，以及用于读取元数据的反射 API

### reflect-metadata polyfill 应该在整个应用程序中只导入一次，因为 Reflect 对象意味着是一个全局单例

## inversify

### Service Identifier
服务标识符

- 作为依赖注入和解析依赖的唯一标识符
- 推荐使用 Symbol 类型作为服务标识符（因为 Symbol 是一种唯一且不可变的数据类型，用来作为 id 的数据类型非常合适）

	- Symbol('Weapon')

### @injectable()
实现类(实现接口的具体类 class)必须使用该装饰器来注解

- @injectable
class a {}

### injection
注入

- @inject([ServiceIdentifier])
当一个类依赖于一个接口时，需要使用该装饰器来为这个接口定义一个在运行时可用的标识符

	- @inject('') weapon: Weapon

- @multiInject([ServiceIdentifier])
获取一个抽象的多重实现（所有）

	- @multiInject('') weapons: Weapon[]

### bindings
绑定

- @named([Tag])
命名绑定(Named Bindings)

	- 在 bind 的时候使用 withTargetTagged([Tag]) 注册

- @tagged([key], [value])
带标签的绑定(Tagged Bindings)

	- 在 bind 的时候使用 withTargetTagged([key], [value]) 注册

- Default Targets
指定某个实现为抽象的默认实现

	- container.bind...whenTargetIsDefault()
	- 这样就可以不使用 @named/@tagged，也不会抛出 AMBIGUOUS_MATCH 异常

- Contextual bindings & @targetName
上下文绑定

	- @targetName([name of the constructor arguments])
	- container.bind...when((request: interfaces.Request) => {
  return request.target.name.equals([targetName])
})
	- @targetName 装饰器用于从上下文约束中访问构造函数参数的名称，即使在代码被压缩时也是如此
	- 多亏了@targetName，我们仍然可以在运行时引用设计时的名字
	- 内置自定义约束

		- traverseAncerstors
		- taggedConstraint
		- namedConstraint
		- typeConstraint

	- whenXXX

		- container.bind...whenXXX 中提供了一些通用的上下文约束

### Container
IOC 容器

- 推荐创建名为 inversify.config.ts 来配置容器。
这是唯一有耦合的地方，其他类中不应该存在对任何类的引用（因为引用的都是抽象，而不是具体实现）

	- import { Container} from 'inversify'
	- const container = new Container()
	- container.bind<[Interface]>([ServiceIdentifier]).to(Implementation)[.whenTargetNamed/whenParentNamed([Tag])]
	- export default container

- Container API

	- Options

		- options 通过 Container 的构造函数传递

	- default scope

		- transient: to/toSelf/toDynamicValue/toService
		- singleton: other types
		- scope declaring

			- container.bind<[Interface]>([ServiceIdentifier]).to([Implementation]).inSingletonScope()
			- container.bind<[Interface]>([ServiceIdentifier]).to([Implementation]).inTransientScope()
			- container.bind<[Interface]>([ServiceIdentifier]).to([Implementation]).inRequestScope()

	- autoBindInjectable

		- 用它来激活 @injectable() 装饰类的自动绑定
		- 手动定义的绑定优先于自动绑定
		- new Container({ autoBindInjectable: true })

	- skipBaseClassChecks

		- 跳过检查基类是否有 @injectable 属性，如果你的 @injectable 类扩展了你不控制的类(第三方类)，那么它就特别有用。默认值为 false

	- Container.merge

		- 创建一个包含两个或多个容器绑定(克隆绑定)的新容器
		- Container.merge(containerA, containerB, ....)

	- advanced feature

		- container.applyCustomMetadataReader
不建议创建自己的自定义元数据读取器

		- container.applyMiddleware
可以控制解析前后自定义行为，比如解析日志等

	- container.createChild

		- 分层 DI 系统（父、子 容器）

	- 解析依赖

		- 获取单个绑定关联

			- container.get/getAsync

				- 根据运行时标识符解析依赖项
				- 绑定时使用 container.bind...to(...)

			- container.getNamed/getNamedAsync

				- 根据匹配给定命名约束的运行时标识符解析依赖项
				- 绑定时使用 container.bind...whenTargetNamed(..)

			- container.getTagged/getTaggedAsync

				- 根据匹配给定标记约束的运行时标识符解析依赖项
				- 绑定时使用 container.bind...whenTargetTagged(...)

		- 获取多个绑定关联

			- container.getAll/getAllAsync

				- 获取给定标识符的所有可用绑定
				- 绑定时使用 container.bind...to(...)

			- container.getAllNamed/getAllNamedAsync

				- 根据匹配给定命名约束的运行时标识符解析所有依赖项
				- 绑定时使用 container.bind...whenTargetNamed(..)

			- container.getAllTagged/getAllTaggedAsync

				- 根据匹配给定标记约束的运行时标识符解析所有依赖项
				- 绑定时使用 container.bind...whenTargetTagged(...)

		- container.resolve

			- resolve 类似 container.get，但它允许用户创建一个实例，即使没有声明绑定（即不会报错）

	- 检查绑定

		- container.isBound

			- 检查给定服务标识符是否注册了绑定
			- 子容器继承父容器中的绑定，在父容器中绑定，子容器中检查绑定结果为 true

		- container.isCurrentBound

			- 检查当前容器中是否只有给定服务标识符注册了绑定
			- 用于父、子容器中隔离检查，在父容器中绑定，子容器中检查绑定结果为 false

		- container.isBoundNamed

			- 检查给定服务标识符和给定命名约束是否注册了绑定

		- container.isBoundTagged

			- 检查给定的服务标识符和给定的标记约束是否注册了绑定

	- 绑定依赖关系

		- container.bind

		- container.rebind/rebindAsync

			- 替换给定 serviceIdentifier 的所有现有绑定
			- 相当于清空给定 serviceIdentifier 的所有现有绑定，然后重新 bind

		- container.unbind/unbindAsync

			- 移除此容器中绑定到服务标识符的所有绑定。这将导致失活过程（onDeactivation）

		- container.unbindAll/unbindAllAsync

			- 移除此容器中绑定的所有绑定。这将导致失活过程（onDeactivation）

	- 实例声明周期事件

		- container.onActivation

			- 为使用指定标识符注册的所有依赖项添加激活处理程序（监听指定服务标识符的实例创建）

		- container.onDeactivation

			- 为依赖项的标识符添加去激活处理程序（监听指定服务标识符的实例销毁）

	- snapshot
快照

		- container.snapshot

			- 保存容器的状态，稍后使用restore方法进行恢复

		- container.restore

			- 将容器状态恢复到上次快照

	- module
模块

		- container.load/loadAsync

			- 调用每个模块的注册方法
			- 容器模块可以帮助您在非常大的应用程序中管理绑定的复杂性

		- container.unload/unloadAsync

			- 移除模块添加的绑定和处理程序。这将导致失活过程（onDeactivation）

	- 属性

		- container.parent

			- 访问容器层次结构，获取/设置当前容器的父级容器

		- container.id

			- 自动生成的唯一标识符

### 可选依赖

- @optional

	- 被 @optional 修饰的依赖可以声明默认值

		- @inject("Shuriken") @optional() shuriken: Shuriken = { name: "DefaultShuriken" }

### 注入常量或动态值

- constant value

	- container.bind...toConstantValue(1)

- dynamic value

	- container.bind...toDynamicValue((context: interfaces.Context) => new Katana())
	- container.bind...toDynamicValue((context: interfaces.Context) => Promise.resolve(new Katana()))

解析依赖时，返回的是 Promise<Katana>，需要 await

### Post Construct Decorator
post construct 装饰器

- 该装饰器将在对象实例化之后和任何激活处理程序之前运行
- 一个 class 中只能有一个函数使用该装饰器，出现多个会直接报错
- @postConstruct

### Middleware

- 执行阶段 before or after the

	- planning
	- resolution
	- activation

- 应用一个中间件

	- container.applyMiddleware(...)

### 自定义 Tag 装饰器

- 为多个标签创建一个可重用的装饰器

### hierarchical DI systems
分层依赖系统

- 在分层DI系统中，一个容器可以有一个父容器，多个容器可以在一个应用程序中使用。容器形成一个层次结构
- 当位于层次结构底部的容器请求依赖时，容器会尝试使用自己的绑定来满足该依赖。如果容器没有绑定，它就把请求传递给它的父容器。如果该容器不能满足请求，则将其传递给其父容器。请求一直冒泡，直到我们找到一个可以处理该请求的容器或容器的祖先

### circular dependencies
循环依赖

- InversifyJS能够识别循环依赖，如果检测到循环依赖，它会抛出一个异常来帮助你识别问题的位置
- 解决方案

	- LazyServiceIdentifer

		- @inject(new LazyServiceIdentifer(() => TYPES.Katana)) katana: Katana
		- lazy 标识符不会延迟依赖的注入，所有的依赖都是在创建类实例时注入的。但是，它确实延迟了对属性标识符的访问

	- @lazyInject

		- 会将依赖项的注入延迟到实际使用时，这发生在创建类实例之后

### inheritance
继承

- rules
限制继承的规则

	- 派生类必须显式声明其构造函数
	- 派生类中的构造函数参数的个数必须为 >= 其基类的构造函数参数的个数

- 克服规则的方案

	- 使用 @unmanaged 装饰器

		- 允许用户标记参数将被手动注入到基类中
		- // parent
public constructor(@unmanaged() arg: string) {}

// child
public constructor(){
  super('unmanaged-injected-value')
}

	- 属性设置

		- 使用 public、protected、private 访问修饰符和属性setter来避免注入基类

	- 属性注入

		- 可以使用属性注入来避免注入基类

	- 注入到派生类

		- 先注入派生类，然后再使用基类的构造函数 (super)注入基类

	- 跳过基类的 @injectable 检查

		- 将容器的 skipBaseClassChecks 选项设置为 true 将禁用所有基类检查。这意味着完全由开发人员来确保 super() 构造函数在正确的时间以正确的参数被调用

- 基类由第三方模块提供时

	- 自定义子类继承第三方模块基类，会抛出缺少 @injectable 注释的错误
	- 使用装饰函数来解决这个问题

		- import { decorate, injectable } from 'inversify'
		- import SomeClass from 'some-module'
		- decorate(injectable(), SomeClass)

### 常用原型链

- container.bind<Samurai>(Samurai).toSelf()
注册具体类型（class自注册）

### 资料参考

- InversifyJS 特性和 API wiki

	- 中文版本

