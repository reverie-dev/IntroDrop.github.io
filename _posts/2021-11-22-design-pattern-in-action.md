---
layout:     post
title:      "golang 实现设计模式 "
subtitle:   "design pattern in action"
date:       2021-11-22 19:00:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- tutorial
- golang
- design pattern
---

## 1. 抽象工厂模式

```go
abstract_factory.go
// package abstract_factory 抽象工厂模式
package abstract_factory

import "fmt"

// FruitFactory 水果工厂接口
type FruitFactory interface {
	// CreateFruit 生产水果
	CreateFruit() Fruit
}

//AppleFactory 苹果工厂，实现FruitFactory接口
type AppleFactory struct{}

//BananaFactory 香蕉工厂，实现FruitFactory接口
type BananaFactory struct{}

//OrangeFactory 橘子工厂，实现FruitFactory接口
type OrangeFactory struct{}

// CreateFruit 苹果工厂生产苹果
func (appleFactory AppleFactory) CreateFruit() Fruit {
	return &Apple{}
}

// CreateFruit 香蕉工厂生产香蕉
func (bananaFactory BananaFactory) CreateFruit() Fruit {
	return &Banana{}
}

// CreateFruit 橘子工厂生产橘子
func (orangeFactory OrangeFactory) CreateFruit() Fruit {
	return &Orange{}
}

// Fruit 水果接口
type Fruit interface {
	// Eat 吃水果
	Eat()
}

// Apple 苹果，实现Fruit接口
type Apple struct{}

// Banana 香蕉，实现Fruit接口
type Banana struct{}

// Orange 橘子，实现Fruit接口
type Orange struct{}

// Eat 吃苹果
func (apple Apple) Eat() {
	fmt.Println("吃苹果")
}

// Eat 吃香蕉
func (banana Banana) Eat() {
	fmt.Println("吃香蕉")
}

// Eat 吃橘子
func (orange Orange) Eat() {
	fmt.Println("吃橘子")
}
```



```go
abstract_factory_test.go
package abstract_factory

import "testing"

func Test(t *testing.T) {
	t.Run("abstract_factory: ", ProduceFruitAndEat)
}

func ProduceFruitAndEat(t *testing.T) {
	var factory FruitFactory
	var apple, banana, orange Fruit

	//创建苹果工厂，生产苹果，吃苹果
	factory = &AppleFactory{}
	apple = factory.CreateFruit()
	apple.Eat()

	//创建香蕉工厂，生产香蕉，吃香蕉
	factory = &BananaFactory{}
	banana = factory.CreateFruit()
	banana.Eat()

	//创建橘子工厂，生产橘子，吃橘子
	factory = &OrangeFactory{}
	orange = factory.CreateFruit()
	orange.Eat()
}
```



## 2. 适配器模式

adapter.go

```go
// package adapter 适配器，以手机充电为例，适配器将220v电压转为5v电压
package adapter

import "fmt"

// Volts220 源结构体，电压220v
type Volts220 struct{}

// OutputPower 输出电压，实现了Adaptee接口
func (v Volts220) OutputPower() {
	fmt.Println("电源输出了220V电压")
}

// Adaptee 源接口
type Adaptee interface {
	// OutputPower 输出电压
	OutputPower()
}

// Target 目的接口
type Target interface {
	// CovertTo5V 转换到5V电压
	CovertTo5V()
}

// Adapter 适配器结构体，嵌套Adaptee接口，实现Target接口
type Adapter struct {
	Adaptee
}

// CovertTo5V 转换到5V电压
func (a Adapter) CovertTo5V() {
	//这里实现自己的转换逻辑，会调用源结构体的方法
	a.OutputPower()
	fmt.Println("通过手机电源适配器，转成了5V电压，可供手机充电")
}
```



adapter_test.go

```go
package adapter

import "testing"

func Test(t *testing.T) {
	t.Run("adaptor: ", ConvertVolts)
}

func ConvertVolts(t *testing.T) {
	var adaptee Adaptee
	adaptee = &Volts220{}

	var target Target
	target = &Adapter{adaptee}
	target.CovertTo5V()
}
```



## 3. 桥接模式

```go
bridge.go
// package bridge 桥接模式，以订购咖啡为例，两个纬度，分别为大中小杯，加糖加奶
package bridge

import "fmt"

// ICoffee 咖啡接口
type ICoffee interface {
	// OrderCoffee 订购咖啡
	OrderCoffee()
}

// LargeCoffee 大杯咖啡，实现ICoffee接口
type LargeCoffee struct {
	ICoffeeAddtion
}

// MediumCoffee 中杯咖啡，实现ICoffee接口
type MediumCoffee struct {
	ICoffeeAddtion
}

// SmallCoffee 小杯咖啡，实现ICoffee接口
type SmallCoffee struct {
	ICoffeeAddtion
}

// OrderCoffee 订购大杯咖啡
func (lc LargeCoffee) OrderCoffee() {
	fmt.Println("订购了大杯咖啡")
	lc.AddSomething()
}

// OrderCoffee 订购中杯咖啡
func (mc MediumCoffee) OrderCoffee() {
	fmt.Println("订购了中杯咖啡")
	mc.AddSomething()
}

// OrderCoffee 订购小杯咖啡
func (sc SmallCoffee) OrderCoffee() {
	fmt.Println("订购了小杯咖啡")
	sc.AddSomething()
}

// CoffeeCupType 咖啡容量类型
type CoffeeCupType uint8

const (
	// CoffeeCupTypeLarge 大杯咖啡
	CoffeeCupTypeLarge = iota
	// CoffeeCupTypeMedium 中杯咖啡
	CoffeeCupTypeMedium = iota
	// CoffeeCupTypeSmall 小杯咖啡
	CoffeeCupTypeSmall = iota
)

// CoffeeFuncMap 全局可导出变量，咖啡类型与创建咖啡对象的map，用于减小圈复杂度
var CoffeeFuncMap = map[CoffeeCupType]func(coffeeAddtion ICoffeeAddtion) ICoffee{
	CoffeeCupTypeLarge:  NewLargeCoffee,
	CoffeeCupTypeMedium: NewMediumCoffee,
	CoffeeCupTypeSmall:  NewSmallCoffee,
}

// NewCoffee 创建咖啡接口对象的简单工厂，根据咖啡容量类型，获取创建接口对象的func
func NewCoffee(cupType CoffeeCupType, coffeeAddtion ICoffeeAddtion) ICoffee {
	if handler, ok := CoffeeFuncMap[cupType]; ok {
		return handler(coffeeAddtion)
	}
	return nil
}

// NewLargeCoffee 创建大杯咖啡对象
func NewLargeCoffee(coffeeAddtion ICoffeeAddtion) ICoffee {
	return &LargeCoffee{coffeeAddtion}
}

// NewMediumCoffee 创建中杯咖啡对象
func NewMediumCoffee(coffeeAddtion ICoffeeAddtion) ICoffee {
	return &MediumCoffee{coffeeAddtion}
}

// NewSmallCoffee 创建小杯咖啡对象
func NewSmallCoffee(coffeeAddtion ICoffeeAddtion) ICoffee {
	return &SmallCoffee{coffeeAddtion}
}

// ICoffeeAddtion 咖啡额外添加接口
type ICoffeeAddtion interface {
	//AddSomething 添加某种食物
	AddSomething()
}

// Milk 加奶，实现ICoffeeAddtion接口
type Milk struct{}

// Sugar 加糖，实现ICoffeeAddtion接口
type Sugar struct{}

// AddSomething Milk实现加奶
func (milk Milk) AddSomething() {
	fmt.Println("加奶")
}

//AddSomething Sugar实现加糖
func (sugar Sugar) AddSomething() {
	fmt.Println("加糖")
}

// CoffeeAddtionType 咖啡额外添加类型
type CoffeeAddtionType uint8

const (
	// CoffeeAddtionTypeMilk 咖啡额外添加牛奶
	CoffeeAddtionTypeMilk = iota
	// CoffeeAddtionTypeSugar 咖啡额外添加糖
	CoffeeAddtionTypeSugar = iota
)

// CoffeeAddtionFuncMap 全局可导出变量，咖啡额外添加类型与创建咖啡额外添加对象的map，用于减小圈复杂度
var CoffeeAddtionFuncMap = map[CoffeeAddtionType]func() ICoffeeAddtion{
	CoffeeAddtionTypeMilk:  NewCoffeeAddtionMilk,
	CoffeeAddtionTypeSugar: NewCoffeeAddtionSugar,
}

// NewCoffeeAddtion 创建咖啡额外添加接口对象的简单工厂，根据咖啡额外添加类型，获取创建接口对象的func
func NewCoffeeAddtion(addtionType CoffeeAddtionType) ICoffeeAddtion {
	if handler, ok := CoffeeAddtionFuncMap[addtionType]; ok {
		return handler()
	}
	return nil
}

// NewCoffeeAddtionMilk 创建咖啡额外加奶
func NewCoffeeAddtionMilk() ICoffeeAddtion {
	return &Milk{}
}

// NewCoffeeAddtionMilk 创建咖啡额外加糖
func NewCoffeeAddtionSugar() ICoffeeAddtion {
	return &Sugar{}
}
```



```go
bridge_test.go
package bridge

import "testing"

func Test(t *testing.T) {
	t.Run("large-milk: ", OrderLargeMilkCoffee)
	t.Run("large-sugar: ", OrderLargeSugarCoffee)
	t.Run("medium-milk: ", OrderMediumMilkCoffee)
	t.Run("smarll-sugar: ", OrderSmallSugarCoffee)
}

func OrderLargeMilkCoffee(t *testing.T) {
	var coffeeAddtion ICoffeeAddtion
	coffeeAddtion = NewCoffeeAddtion(CoffeeAddtionTypeMilk)
	var coffee ICoffee
	coffee = NewCoffee(CoffeeCupTypeLarge, coffeeAddtion)
	coffee.OrderCoffee()
}

func OrderLargeSugarCoffee(t *testing.T) {
	var coffeeAddtion ICoffeeAddtion
	coffeeAddtion = NewCoffeeAddtion(CoffeeAddtionTypeSugar)
	var coffee ICoffee
	coffee = NewCoffee(CoffeeCupTypeLarge, coffeeAddtion)
	coffee.OrderCoffee()
}

func OrderMediumMilkCoffee(t *testing.T) {
	var coffeeAddtion ICoffeeAddtion
	coffeeAddtion = NewCoffeeAddtion(CoffeeAddtionTypeMilk)
	var coffee ICoffee
	coffee = NewCoffee(CoffeeCupTypeMedium, coffeeAddtion)
	coffee.OrderCoffee()
}

func OrderSmallSugarCoffee(t *testing.T) {
	var coffeeAddtion ICoffeeAddtion
	coffeeAddtion = NewCoffeeAddtion(CoffeeAddtionTypeSugar)
	var coffee ICoffee
	coffee = NewCoffee(CoffeeCupTypeSmall, coffeeAddtion)
	coffee.OrderCoffee()
}
```



## 4. 建造者模式

```go
builder.go
//package builder 建造者模式
package builder

const (
	// Success 成功
	Success = 0
	// ErrCodeNameRequired 名字必填
	ErrCodeNameRequired = -10001
	// ErrCodeMinIdleMoreThanMaxIdle 最小空闲数超过了最大空闲数
	ErrCodeMinIdleMoreThanMaxIdle = -10002
	// ErrCodeMaxTotalLessThanOrEqualToZero 最大连接数必须大于0
	ErrCodeMaxTotalMustMoreThanZero = -10003
	// ErrCodeMaxIdleMustMoreThanZero 最大空闲数必须大于0
	ErrCodeMaxIdleMustMoreThanZero = -10004
	// ErrCodeMinIdleMustMoreThanZero 最小空闲数必须大于0
	ErrCodeMinIdleMustMoreThanZero = -10005
	// ErrCodeMaxTotalNotSet 最大连接数未设置
	ErrCodeMaxTotalNotSet = -10006
	// ErrCodeMaxIdleNotSet 最大空闲连接数未设置
	ErrCodeMaxIdleNotSet = -10007
	// ErrCodeMinIdleNotSet 最小空闲连接数未设置
	ErrCodeMinIdleNotSet = -10008
)

// errMap 错误信息与错误码的配置
var errMap = map[int16]string{
	Success:                         "ok",
	ErrCodeNameRequired:             "名字必填",
	ErrCodeMinIdleMoreThanMaxIdle:   "最小空闲数超过了最大空闲数",
	ErrCodeMaxTotalMustMoreThanZero: "最大连接数必须大于0",
	ErrCodeMaxIdleMustMoreThanZero:  "最大空闲连接数必须大于0",
	ErrCodeMinIdleMustMoreThanZero:  "最小空闲连接数必须大于0",
	ErrCodeMaxTotalNotSet:           "最大连接数未设置",
	ErrCodeMaxIdleNotSet:            "最大空闲连接数未设置",
	ErrCodeMinIdleNotSet:            "最小空闲连接数未设置",
}

// resourcePoolConfig 资源池配置，不可导出
type resourcePoolConfig struct {
	// name 名字，不可导出，必填
	name string
	// maxTotal 最大连接数，不可导出，必填
	maxTotal uint16
	// maxIdle 最大空闲连接数，不可导出，必填
	maxIdle uint16
	// minIdle 最小空闲连接数，不可导出，不必填，但填里必须大于0
	minIdle uint16
}

// Builder 建造者接口
type Builder interface {
	// Build 创建对象
	Build() int16
}

// ResourcePoolBuild 资源池建造者，实现Builder接口
type ResourcePoolBuild struct {
	// resourcePoolConfig 嵌套资源池配置
	resourcePoolConfig
}

// Build 创建对象
func (build *ResourcePoolBuild) Build() int16 {
	// 这一串的判断可以保证builder里要求必须设置的属性，都能够设置
	if len(build.name) == 0 {
		return ErrCodeNameRequired
	}
	if build.maxTotal == 0 {
		return ErrCodeMaxTotalNotSet
	}
	if build.maxIdle == 0 {
		return ErrCodeMaxIdleNotSet
	}

	// 对设置后的对象，做整体的逻辑性验证
	if build.minIdle > build.maxIdle {
		return ErrCodeMinIdleMoreThanMaxIdle
	}
	return Success
}

// SetName 设置名字，并进行校验
func (build *ResourcePoolBuild) SetName(name string) int16 {
	if len(name) == 0 {
		return ErrCodeNameRequired
	}
	build.name = name
	return Success
}

// SetMaxTotal 设置最大连接数，并进行校验
func (build *ResourcePoolBuild) SetMaxTotal(maxTotal uint16) int16 {
	if maxTotal <= 0 {
		return ErrCodeMaxTotalMustMoreThanZero
	}
	build.maxTotal = maxTotal
	return Success
}

// SetMaxIdle 设置最大空闲连接数，并信息校验
func (build *ResourcePoolBuild) SetMaxIdle(maxIdle uint16) int16 {
	if maxIdle <= 0 {
		return ErrCodeMaxIdleMustMoreThanZero
	}
	build.maxIdle = maxIdle
	return Success
}

// SetMinIdle 设置最小空闲连接数，并进行校验
func (build *ResourcePoolBuild) SetMinIdle(minIdle uint16) int16 {
	if minIdle <= 0 {
		return ErrCodeMinIdleMustMoreThanZero
	}
	build.minIdle = minIdle
	return Success
}
```



```go
builder_test.go
package builder

import (
	"fmt"
	"testing"
)

func Test(t *testing.T) {
	t.Run("builder-maxTotalNotSet: ", maxTotalNotSet)
	t.Run("builder-minIdleMoreThanMaxIdle: ", minIdleMoreThanMaxIdle)
	t.Run("builder-buildSuccess: ", buildSuccess)
}

func maxTotalNotSet(t *testing.T) {
	// 此为示例代码，在实践中，不应将建造者与使用者放在一个包内，所以这里假设不能直接访问resourcePoolConfig
	build := &ResourcePoolBuild{}
	build.SetName("mysql连接池")
	build.SetMaxIdle(10)
	errCode := build.Build()
	fmt.Println(fmt.Sprintf("建造资源池配置：%s", errMap[errCode]))
}

func minIdleMoreThanMaxIdle(t *testing.T) {
	// 此为示例代码，在实践中，不应将建造者与使用者放在一个包内，所以这里假设不能直接访问resourcePoolConfig
	build := &ResourcePoolBuild{}
	build.SetName("mysql连接池")
	build.SetMaxIdle(10)
	build.SetMinIdle(20)
	build.SetMaxTotal(50)
	errCode := build.Build()
	fmt.Println(fmt.Sprintf("建造资源池配置：%s", errMap[errCode]))
}

func buildSuccess(t *testing.T) {
	// 此为示例代码，在实践中，不应将建造者与使用者放在一个包内，所以这里假设不能直接访问resourcePoolConfig
	build := &ResourcePoolBuild{}
	build.SetName("mysql连接池")
	build.SetMaxIdle(10)
	build.SetMinIdle(5)
	build.SetMaxTotal(50)
	errCode := build.Build()
	fmt.Println(fmt.Sprintf("建造资源池配置：%s", errMap[errCode]))
}
```



## 5. 组合模式

```go
component.go
//package combination 组合模式
package combination

// UIComponent UI组件接口，对于任何UI控件都适用。
type UIComponent interface {
	// PrintUIComponent 打印UI组件
	PrintUIComponent()
	// GetUIControlName 获取控件名字
	GetUIControlName() string
	// GetConcreteUIControlName 获取控件具体名字
	GetConcreteUIControlName() string
}

// UIComponentAddtion UI组件附加接口，使用接口隔离原则，保证不需要实现接口声明方法的结构体，没有额外负担。仅对容器类型对UI控件适用。
type UIComponentAddtion interface {
	// AddUIComponent 添加UI组件
	AddUIComponent(component UIComponent)
	// AddUIComponents 添加UI组件列表
	AddUIComponents(components []UIComponent)
	// GetUIComponentList 获取UI组件列表
	GetUIComponentList() []UIComponent
}

// UIAttr UI属性
type UIAttr struct {
	// UI 名字
	Name string
}

//client 打印client
var client = &PrintClient{}
```



```go
container.go
package combination

// WinForm 窗口，实现UIComponent、UIComponentAddtion接口
type WinForm struct {
	// UIAttr 嵌套UI属性
	UIAttr
	// Components 容器的组件列表
	Components []UIComponent
}

// GetUIControlName 获取控件名字
func (window *WinForm) GetUIControlName() string {
	return "WinForm"
}

// GetConcreteUIControlName 获取控件具体名字
func (window *WinForm) GetConcreteUIControlName() string {
	return window.Name
}

// PrintUIComponent 打印UI组件
func (window *WinForm) PrintUIComponent() {
	client.printContainer(window, window)
}

// AddUIComponent 添加UI组件
func (window *WinForm) AddUIComponent(component UIComponent) {
	window.Components = append(window.Components, component)
}

// AddUIComponents 添加UI组件列表
func (window *WinForm) AddUIComponents(components []UIComponent) {
	window.Components = append(window.Components, components...)
}

// GetUIComponentList 获取UI组件列表
func (window *WinForm) GetUIComponentList() []UIComponent {
	return window.Components
}

// Frame 框架，实现UIComponent、接口
type Frame struct {
	// UIAttr 嵌套UI属性
	UIAttr
	// Components 容器的组件列表
	Components []UIComponent
}

// GetUIControlName 获取控件名字
func (frame *Frame) GetUIControlName() string {
	return "Frame"
}

// GetConcreteUIControlName 获取控件具体名字
func (frame *Frame) GetConcreteUIControlName() string {
	return frame.Name
}

// PrintUIComponent 打印UI组件
func (frame *Frame) PrintUIComponent() {
	client.printContainer(frame, frame)
}

// AddUIComponent 添加UI组件
func (frame *Frame) AddUIComponent(component UIComponent) {
	frame.Components = append(frame.Components, component)
}

// AddUIComponents 添加UI组件列表
func (frame *Frame) AddUIComponents(components []UIComponent) {
	frame.Components = append(frame.Components, components...)
}

// GetUIComponentList 获取UI组件列表
func (frame *Frame) GetUIComponentList() []UIComponent {
	return frame.Components
}
```



```go
leaf.go
package combination

// Picture 图片，实现UIComponent接口
type Picture struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (picture Picture) GetUIControlName() string {
	return "Picture"
}

// GetConcreteUIControlName 获取控件具体名字
func (picture Picture) GetConcreteUIControlName() string {
	return picture.Name
}

// PrintUIComponent 打印UI组件
func (picture Picture) PrintUIComponent() {
	client.printCurrentControl(picture)
}

// Button 按钮，实现UIComponent接口
type Button struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (button Button) GetUIControlName() string {
	return "Button"
}

// GetConcreteUIControlName 获取控件具体名字
func (button Button) GetConcreteUIControlName() string {
	return button.Name
}

// PrintUIComponent 打印UI组件
func (button Button) PrintUIComponent() {
	client.printCurrentControl(button)
}

// Label 标签，实现UIComponent接口
type Label struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (label Label) GetUIControlName() string {
	return "Label"
}

// GetConcreteUIControlName 获取控件具体名字
func (label Label) GetConcreteUIControlName() string {
	return label.Name
}

// PrintUIComponent 打印UI组件
func (label Label) PrintUIComponent() {
	client.printCurrentControl(label)
}

// TextBox 文本框，实现UIComponent接口
type TextBox struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (textBox TextBox) GetUIControlName() string {
	return "TextBox"
}

// GetConcreteUIControlName 获取控件具体名字
func (textBox TextBox) GetConcreteUIControlName() string {
	return textBox.Name
}

// PrintUIComponent 打印UI组件
func (textBox TextBox) PrintUIComponent() {
	client.printCurrentControl(textBox)
}

// PassWordBox 密码框，实现UIComponent接口
type PassWordBox struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (passWordBox PassWordBox) GetUIControlName() string {
	return "PassWordBox"
}

// GetConcreteUIControlName 获取控件具体名字
func (passWordBox PassWordBox) GetConcreteUIControlName() string {
	return passWordBox.Name
}

// PrintUIComponent 打印UI组件
func (passWordBox PassWordBox) PrintUIComponent() {
	client.printCurrentControl(passWordBox)
}

// CheckBox 复选框，实现UIComponent接口
type CheckBox struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (checkBox CheckBox) GetUIControlName() string {
	return "CheckBox"
}

// GetConcreteUIControlName 获取控件具体名字
func (checkBox CheckBox) GetConcreteUIControlName() string {
	return checkBox.Name
}

// PrintUIComponent 打印UI组件
func (checkBox CheckBox) PrintUIComponent() {
	client.printCurrentControl(checkBox)
}

// LinkLabel 关联的标签，实现UIComponent接口
type LinkLabel struct {
	// UIAttr 嵌套UI属性
	UIAttr
}

// GetUIControlName 获取控件名字
func (linkLabel LinkLabel) GetUIControlName() string {
	return "LinkLabel"
}

// GetConcreteUIControlName 获取控件具体名字
func (linkLabel LinkLabel) GetConcreteUIControlName() string {
	return linkLabel.Name
}

// PrintUIComponent 打印UI组件
func (linkLabel LinkLabel) PrintUIComponent() {
	client.printCurrentControl(linkLabel)
}
```



client.go

```go
package combination

import "fmt"

// PrintClient 打印客户端
type PrintClient struct{}

// printContainer 打印容器控件
func (client PrintClient) printContainer(component UIComponent, componentAddtion UIComponentAddtion) {
	client.printCurrentControl(component)
	for _, v := range componentAddtion.GetUIComponentList() {
		v.PrintUIComponent()
	}
}

// printCurrentControl 打印当前控件
func (client PrintClient) printCurrentControl(component UIComponent) {
	fmt.Println(fmt.Sprintf("print %s(%s)", component.GetUIControlName(), component.GetConcreteUIControlName()))
}
```



```go
combination_test.go
package combination

import (
	"testing"
)

func Test(t *testing.T) {
	t.Run("combination: ", Combination)
}

// Combination 单元测试
//print WinForm(WINDOW窗口)
//print Picture(LOGO图片)
//print Button(登录)
//print Button(注册)
//print Frame(FRAME1)
//print Label(用户名)
//print TextBox(文本框)
//print Label(密码)
//print PassWordBox(密码框)
//print CheckBox(复选框)
//print TextBox(记住用户名)
//print LinkLabel(忘记密码)
func Combination(t *testing.T) {
	window := &WinForm{UIAttr: UIAttr{Name: "WINDOW窗口"}}
	picture := &Picture{UIAttr{Name: "LOGO图片"}}
	loginButton := &Button{UIAttr{Name: "登录"}}
	registerButton := &Button{UIAttr{Name: "注册"}}
	frame := &Frame{UIAttr: UIAttr{Name: "FRAME1"}}
	userLable := &Label{UIAttr{Name: "用户名"}}
	textBox := &TextBox{UIAttr{Name: "文本框"}}
	passwordLable := &Label{UIAttr{Name: "密码"}}
	passwordBox := &PassWordBox{UIAttr{Name: "密码框"}}
	checkBox := &CheckBox{UIAttr{Name: "复选框"}}
	rememberUserTextBox := &TextBox{UIAttr{Name: "记住用户名"}}
	linkLable := &LinkLabel{UIAttr{Name: "忘记密码"}}

	window.AddUIComponents([]UIComponent{picture, loginButton, registerButton, frame})
	frame.AddUIComponents([]UIComponent{userLable, textBox, passwordLable, passwordBox, checkBox, rememberUserTextBox, linkLable})

	window.PrintUIComponent()
}
```



## 6. 命令模式

```go
command.go
// package command 命令模式，引入调用者和接收者实现解耦
package command

import "fmt"

// Command 命令接口
type Command interface {
	// Execute 执行命令
	Execute()
}

// CreateCommand 创建命令，实现Command接口
type CreateCommand struct {
	// receiver 接收者接口对象
	receiver Receiver
}

// UpdateCommand 更新命令，实现Command接口
type UpdateCommand struct {
	// receiver 接收者接口对象
	receiver Receiver
}

// Execute 创建命令实现执行命令方法
func (command CreateCommand) Execute() {
	command.receiver.Action()
}

// Execute 修改命令实现执行命令方法
func (command UpdateCommand) Execute() {
	command.receiver.Action()
}

// Receiver 接收者接口
type Receiver interface {
	// Action 动作
	Action()
}

// CreateReceiver 创建接收者，实现Receiver接口
type CreateReceiver struct{}

// UpdateReceiver 更新接收者，实现Receiver接口
type UpdateReceiver struct{}

func (receiver CreateReceiver) Action() {
	fmt.Println("执行了创建")
}

func (receiver UpdateReceiver) Action() {
	fmt.Println("执行了修改")
}

// Invoker 调用者
type Invoker struct {
	// command 命令接口对象
	command Command
}

// Call 调用者调用执行命令方法
func (invoker Invoker) Call() {
	invoker.command.Execute()
}
```



```go
command_test.go
package command

import "testing"

func Test(t *testing.T) {
	t.Run("create_command: ", CreateCommandExecute)
	t.Run("update_command: ", UpdateCommandExecute)
}

func CreateCommandExecute(t *testing.T) {
	var receiver Receiver
	receiver = &CreateReceiver{}

	var command Command
	command = &CreateCommand{receiver: receiver}

	invoker := Invoker{command: command}
	invoker.Call()
}

func UpdateCommandExecute(t *testing.T) {
	var receiver Receiver
	receiver = &UpdateReceiver{}

	var command Command
	command = &UpdateCommand{receiver: receiver}

	invoker := Invoker{command: command}
	invoker.Call()
}
```



## 7. 装饰者模式

*decorator_add.go*

```go
// package decorator 装饰者模式，在运行期动态地给对象添加额外的指责，比子类更灵活。第二个功能：用于添加功能的装饰模式。
// 若不想修改原来的接口，则可以使用装饰者。go这种语言，若增加接口的方法，则原来实现接口的结构体都需要增加一个新方法，这样就不符合开闭原则了。
package decorator

import "fmt"

// Booker 书的接口
type Booker interface {
	// Reading 读书
	Reading()
}

// Book 书，实现Booker接口
type Book struct {
}

// Reading 读书，实现Booker接口
func (book Book) Reading() {
	fmt.Println("我正在读书")
}

// Underliner 划线接口，装饰者接口
type Underliner interface {
	// Booker 继承Booker接口
	Booker
	// Underline 划线
	Underline()
}

// NotesTaker 记笔记接口，装饰者接口
type NotesTaker interface {
	// Booker 继承Booker接口
	Booker
	// TakeNotes 记笔记
	TakeNotes()
}

// ConcreteUnderline 具体的划线类，实现Underliner接口
type ConcreteUnderline struct {
	// Booker 书的接口对象
	Booker Booker
}

// ReadingBooks ConcreteUnderline提供读书的方法，包装了Booker接口
func (underline ConcreteUnderline) Reading() {
	underline.Booker.Reading()
}

// Underline 划线，实现Underliner接口
func (underline ConcreteUnderline) Underline() {
	fmt.Println("我正在划线")
}

// ConcreteNotesTake 具体的记笔记类，实现NotesTaker接口
type ConcreteNotesTake struct {
	// Booker 书的接口对象
	Booker Booker
}

// ReadingBooks ConcreteNotesTake提供读书的方法，包装了Booker接口
func (notesTake ConcreteNotesTake) Reading() {
	notesTake.Booker.Reading()
}

// Underline 划线，实现NotesTaker接口
func (notesTake ConcreteNotesTake) TakeNotes() {
	fmt.Println("我正在记笔记")
}
```



```go
decorator_strengthen.go
// package decorator 装饰者模式，在运行期动态地给对象添加额外的指责，比子类更灵活。第一个功能：用于增强功能的装饰模式。
package decorator

// HappinessIndex 幸福指数接口
type HappinessIndex interface {
	// GetHappinessIndex 获取幸福指数
	GetHappinessIndex() int
}

// MySelf 我自己，实现HappinessIndex接口
type MySelf struct{}

// GetHappinessIndex 获取幸福指数，实现HappinessIndex接口
func (myself MySelf) GetHappinessIndex() int {
	return 100
}

// DrinkCoffee 喝咖啡
type DrinkCoffee struct {
	// HappinessIndex 幸福指数对象
	HappinessIndex HappinessIndex
}

// GetHappinessIndex 获取幸福指数，实现HappinessIndex接口
func (coffee DrinkCoffee) GetHappinessIndex() int {
	return coffee.HappinessIndex.GetHappinessIndex() + 20
}

// EatFriedChicken 吃炸鸡，实现HappinessIndex接口
type EatFriedChicken struct {
	// HappinessIndex 幸福指数对象
	HappinessIndex HappinessIndex
}

// GetHappinessIndex 获取幸福指数，实现HappinessIndex接口
func (friedChicken EatFriedChicken) GetHappinessIndex() int {
	return friedChicken.HappinessIndex.GetHappinessIndex() + 50
}
```



```go
decorator_test.go
package decorator

import (
	"fmt"
	"testing"
)

func Test(t *testing.T) {
	t.Run("decorator_strengthen: ", DecoratorStrengthen)
	t.Run("decorator_add: ", DecoratorAdd)
}

func DecoratorStrengthen(t *testing.T) {
	var myself, drinkCoffee, eatFriedChicken HappinessIndex
	myself = &MySelf{}
	fmt.Println(fmt.Sprintf("我的幸福指数是：%d", myself.GetHappinessIndex()))

	drinkCoffee = &DrinkCoffee{HappinessIndex: MySelf{}}
	fmt.Println(fmt.Sprintf("喝了咖啡后，我的幸福指数是：%d", drinkCoffee.GetHappinessIndex()))

	eatFriedChicken = &EatFriedChicken{HappinessIndex: drinkCoffee}
	fmt.Println(fmt.Sprintf("吃了炸鸡，喝了咖啡后，我的幸福指数是：%d", eatFriedChicken.GetHappinessIndex()))
	fmt.Println("============")
}

func DecoratorAdd(t *testing.T) {
	var book Booker
	book = &Book{}
	book.Reading()
	fmt.Println("============")

	var notesTake NotesTaker
	notesTake = &ConcreteNotesTake{Booker: book}
	notesTake.Reading()
	notesTake.TakeNotes()
	fmt.Println("============")

	var Underline Underliner
	Underline = &ConcreteUnderline{Booker: book}
	Underline.Reading()
	Underline.Underline()
}
```



## 8. 门面模式* *

```go
facade.go
// package facade 门面模式
package facade

import "fmt"

// RegisterFacade 注册门面
type RegisterFacade struct {
	// IUser 组合用户接口
	IUser
	// IVirtualAccount 组合虚拟账户接口
	IVirtualAccount
	// ICoupon 组合优惠券接口
	ICoupon
}

// Register 注册
func (facade RegisterFacade) Register(telephone string) {
	// 创建对象
	user := User{Telephone: telephone}
	virtualAccount := &VirtualAccount{user}
	coupon := &Coupon{user}
	// 创建用户
	user.CreateUser()
	// 创建虚拟账户
	virtualAccount.CreateVirtualAcount()
	// 对新账户发放优惠券
	coupon.IssueCoupons()
}

// IUser 用户接口
type IUser interface {
	// CreateUser 创建用户
	CreateUser()
}

// IVirtualAccount 虚拟账户接口
type IVirtualAccount interface {
	// CreateVirtualAcount 创建虚拟账户
	CreateVirtualAcount()
}

// ICoupon 优惠券接口
type ICoupon interface {
	// IssueCoupons 发放优惠券接口
	IssueCoupons()
}

// User 用户，实现用户接口
type User struct {
	// Telephone 电话号码
	Telephone string
}

// CreateUser 创建用户
func (user User) CreateUser() {
	fmt.Println(fmt.Sprintf("创建了手机号码为%s的账户", user.Telephone))
}

// VirtualAccount 虚拟账户，实现虚拟账户接口
type VirtualAccount struct {
	// User 虚拟账户嵌套用户
	User
}

func (virtualAccount VirtualAccount) CreateVirtualAcount() {
	fmt.Println(fmt.Sprintf("为手机号码为%s的用户创建虚拟钱包账户", virtualAccount.Telephone))
}

// Coupon 优惠券，实现优惠券接口
type Coupon struct {
	// User 优惠券嵌套用户
	User
}

func (coupon Coupon) IssueCoupons() {
	fmt.Println(fmt.Sprintf("为手机号码为%s的用户发放优惠券", coupon.Telephone))
}
```



```go
facade_test.go
package facade

import "testing"

func Test(t *testing.T) {
	t.Run("facade", RegisterUser)
}

func RegisterUser(t *testing.T) {
	facade := RegisterFacade{}
	facade.Register("18618193858")
}
```



## 9. 工厂方法模式

```go
factory.go
// package factory 工厂方法模式
package factory

import "fmt"

//AppleFactory 苹果工厂
type AppleFactory struct{}

//BananaFactory 香蕉工厂
type BananaFactory struct{}

//OrangeFactory 橘子工厂
type OrangeFactory struct{}

// CreateFruit 苹果工厂生产苹果
func (appleFactory AppleFactory) CreateFruit() Fruit {
	return &Apple{}
}

// CreateFruit 香蕉工厂生产香蕉
func (bananaFactory BananaFactory) CreateFruit() Fruit {
	return &Banana{}
}

// CreateFruit 橘子工厂生产橘子
func (orangeFactory OrangeFactory) CreateFruit() Fruit {
	return &Orange{}
}

// Fruit 水果接口
type Fruit interface {
	// Eat 吃水果
	Eat()
}

// Apple 苹果，实现Fruit接口
type Apple struct{}

// Banana 香蕉，实现Fruit接口
type Banana struct{}

// Orange 橘子，实现Fruit接口
type Orange struct{}

// Eat 吃苹果
func (apple Apple) Eat() {
	fmt.Println("吃苹果")
}

// Eat 吃香蕉
func (banana Banana) Eat() {
	fmt.Println("吃香蕉")
}

// Eat 吃橘子
func (orange Orange) Eat() {
	fmt.Println("吃橘子")
}
```



```go
factory_test.go
package factory

import "testing"

func Test(t *testing.T) {
	t.Run("factory: ", ProduceFruitAndEat)
}

func ProduceFruitAndEat(t *testing.T) {
	appleFactory := &AppleFactory{}
	bananaFactory := &BananaFactory{}
	orangeFactory := &OrangeFactory{}

	var apple, banana, orange Fruit

	apple = appleFactory.CreateFruit()
	apple.Eat()

	banana = bananaFactory.CreateFruit()
	banana.Eat()

	orange = orangeFactory.CreateFruit()
	orange.Eat()
}
```



## 10. 过滤器模式

```go
filter.go
/*
	package filter 过滤器模式，责任链模式的一种。
	每个filter都需要验证，完全验证通过才会放行，去执行业务逻辑
	一旦有filter验证不通过，则立刻退出
	适合处理参数校验，权限校验，记录日志，准备相关资源等场景
*/
package filter

import (
	"context"
	"fmt"
	"strings"
)

// Context 过滤器上下文
type Context struct {
	// context.Context 嵌套基础库
	context.Context
	// key 每个过滤器的key
	key string
}

// HandleFunc 定义别名：过滤器函数
type HandlerFunc func(ctx Context, params ...interface{}) error

// HandlersChain 过滤器责任链
type HandlersChain []HandlerFunc

// Filter 过滤器接口
type Filter interface {
	DoFilter() error
}

// FilterImpl 实际的过滤器，封装了过滤器上下文、过滤器责任链
type FilterImpl struct {
	// Ctx 过滤器上下文
	Ctx Context
	// Chain 过滤器责任链
	Chain HandlersChain
}

// DoFilter 执行过滤器，FilterImpl实现Filter接口
func (myFilter *FilterImpl) DoFilter() error {
	for _, handler := range myFilter.Chain {
		err := handler(myFilter.Ctx)
		if err != nil {
			fmt.Println(fmt.Sprintf("filter fail key=%s, err = %v", myFilter.Ctx.key, err))
			return err
		}
	}
	return nil
}

//NewFilter 创建过滤器
func NewFilter(ctx Context, handlers ...HandlerFunc) Filter {
	return &FilterImpl{
		Ctx:   ctx,
		Chain: handlers,
	}
}

/**********过滤器使用***********/

//ParamFilter 参数过滤器接口
type ParamFilter interface {
	// DoParamFilter 执行参数过滤
	DoParamFilter() error
}

// CommentParamFilter 评论参数过滤器，实现ParamFilter接口
type CommentParamFilter struct {
	CommentInfo
}

// UpParamFilter 点赞参数过滤器，实现ParamFilter接口
type UpParamFilter struct {
	UpInfo
}

// DoParamFilter 评论参数过滤器，实现参数过滤方法
func (cpf CommentParamFilter) DoParamFilter() error {
	myContext := Context{
		Context: context.TODO(),
		key:     "评论参数过滤器",
	}
	cf := NewFilter(
		myContext,
		validateComment1(myContext, cpf.CommentInfo),
		validateComment2(myContext, cpf.CommentInfo),
	)
	return cf.DoFilter()
}

// validateComment1 评论参数校验方法1
func validateComment1(ctx Context, info CommentInfo) HandlerFunc {
	return func(ctx Context, params ...interface{}) error {
		if info.ArticleID == 0 {
			return fmt.Errorf("当前评论无文章id，不允许评论")
		}
		return nil
	}
}

// validateComment2 评论参数校验方法2
func validateComment2(ctx Context, info CommentInfo) HandlerFunc {
	return func(ctx Context, params ...interface{}) error {
		if len(info.Content) == 0 || len(strings.Trim(info.Content, " ")) == 0 {
			return fmt.Errorf("当前评论无内容，不允许评论")
		}
		return nil
	}
}

// DoParamFilter 点赞参数过滤器，实现参数过滤方法
func (upf UpParamFilter) DoParamFilter() error {
	myContext := Context{
		Context: context.TODO(),
		key:     "点赞参数过滤器",
	}
	cf := NewFilter(
		myContext,
		validateUp1(myContext, upf.UpInfo),
		validateUp2(myContext, upf.UpInfo),
	)
	return cf.DoFilter()
}

// validateUp1 点赞参数校验方法1
func validateUp1(ctx Context, info UpInfo) HandlerFunc {
	return func(ctx Context, params ...interface{}) error {
		if info.CommenID == 0 {
			return fmt.Errorf("当前点赞无评论id，不允许点赞")
		}
		return nil
	}
}

// validateUp2 点赞参数校验方法2
func validateUp2(ctx Context, info UpInfo) HandlerFunc {
	return func(ctx Context, params ...interface{}) error {
		if info.UserID == 0 {
			return fmt.Errorf("当前点赞无用户id，不允许点赞")
		}
		return nil
	}
}

// CommentInfo 评论
type CommentInfo struct {
	// Content 评论内容
	Content string
	// ArticleID 文章ID
	ArticleID uint64
}

// UpInfo 点赞
type UpInfo struct {
	// CommenID 评论ID
	CommenID uint64
	// UserID 点赞人的用户ID
	UserID uint64
}
```



filter_test.go

```go
package filter

import "testing"

func Test(t *testing.T) {
	t.Run("comment_filter1: ", CommentFilter1)
	t.Run("comment_filter2: ", CommentFilter2)
	t.Run("comment_filter3: ", CommentFilter3)
	t.Run("up_filter1: ", UpFilter1)
	t.Run("up_filter2: ", UpFilter2)
}

func CommentFilter1(t *testing.T) {
	//构造过滤器信息
	info := CommentInfo{
		Content:   "",
		ArticleID: 1,
	}
	//执行过滤器
	var commentFilter CommentParamFilter
	commentFilter = CommentParamFilter{info}
	commentFilter.DoParamFilter()
}

func CommentFilter2(t *testing.T) {
	//构造过滤器信息
	info := CommentInfo{
		Content:   "太可怕了",
		ArticleID: 0,
	}
	//执行过滤器
	var commentFilter CommentParamFilter
	commentFilter = CommentParamFilter{info}
	commentFilter.DoParamFilter()
}

func CommentFilter3(t *testing.T) {
	//构造过滤器信息
	info := CommentInfo{
		Content:   "    ",
		ArticleID: 1234,
	}
	//执行过滤器
	var commentFilter CommentParamFilter
	commentFilter = CommentParamFilter{info}
	commentFilter.DoParamFilter()
}

func UpFilter1(t *testing.T) {
	//构造过滤器信息
	info := UpInfo{
		CommenID: 0,
		UserID:   6379,
	}
	//执行过滤器
	var upFilter UpParamFilter
	upFilter = UpParamFilter{info}
	upFilter.DoParamFilter()
}

func UpFilter2(t *testing.T) {
	//构造过滤器信息
	info := UpInfo{
		CommenID: 5678,
		UserID:   0,
	}
	//执行过滤器
	var upFilter UpParamFilter
	upFilter = UpParamFilter{info}
	upFilter.DoParamFilter()
}
```



## 11. 享元模式

```go
flyweight.go
// package flyweight 享元模式。通过抽离出被大量创建的对象中的不易变的属性，达到减少对象个数、降低内存的效果。
// 假设一个象棋游戏，游戏大厅中有百万棋局同时启动，那么会产生非常多的棋子和棋局对象，通过抽离出不变的棋子的属性，来降低内存。
package flyweight

import (
	"fmt"
	"sync"
)

// Color 棋子的颜色
type Color string

const (
	// ColorBlack 黑色
	ColorBlack Color = "黑色"
	// ColorRed 红色
	ColorRed Color = "红色"
)

var (
	// once 保证只执行一次
	once sync.Once

	// instance 单例对象
	PieceUnitFactory *pieceUnitFactory
)

// GetPieceUnitFactory 获取单例对象。 注：必须使用指针对象，否则会在返回时，拷贝一份，对象就不是同一个了。
func GetPieceUnitFactory() *pieceUnitFactory {
	once.Do(func() {
		PieceUnitFactory = &pieceUnitFactory{}
		PieceUnitFactory.PieceUnitMap = make(map[int]PieceUnit)
		PieceUnitFactory.init()
	})
	return PieceUnitFactory
}

// PieceUnit 棋子享元
type PieceUnit struct {
	// ID 唯一标识
	ID int
	// Text 棋子的文字
	Text string
	// Color 棋子的颜色
	Color Color
}

// pieceUnitFactory 棋子享元对象工厂
type pieceUnitFactory struct {
	//sync.RWMutex
	// PieceUnitMap 棋子id与享元对象的对应关系的map
	PieceUnitMap map[int]PieceUnit
}

// init 初始化工厂
func (factory pieceUnitFactory) init() {
	// 不需要锁，因为通过once.Do创建
	//factory.Lock()
	//defer factory.Unlock()

	factory.PieceUnitMap[1] = PieceUnit{1, "將", ColorBlack}
	factory.PieceUnitMap[2] = PieceUnit{2, "帥", ColorRed}
	factory.PieceUnitMap[3] = PieceUnit{3, "馬", ColorBlack}
	factory.PieceUnitMap[4] = PieceUnit{4, "傌", ColorRed}
	factory.PieceUnitMap[5] = PieceUnit{5, "馬", ColorBlack}
	factory.PieceUnitMap[6] = PieceUnit{6, "傌", ColorRed}
	factory.PieceUnitMap[7] = PieceUnit{7, "砲", ColorBlack}
	factory.PieceUnitMap[8] = PieceUnit{8, "炮", ColorRed}
	factory.PieceUnitMap[9] = PieceUnit{9, "砲", ColorBlack}
	factory.PieceUnitMap[10] = PieceUnit{10, "炮", ColorRed}
	factory.PieceUnitMap[11] = PieceUnit{11, "車", ColorBlack}
	factory.PieceUnitMap[12] = PieceUnit{12, "俥", ColorRed}
	factory.PieceUnitMap[13] = PieceUnit{13, "車", ColorBlack}
	factory.PieceUnitMap[14] = PieceUnit{14, "俥", ColorRed}
	factory.PieceUnitMap[15] = PieceUnit{15, "士", ColorBlack}
	factory.PieceUnitMap[16] = PieceUnit{16, "仕", ColorRed}
	factory.PieceUnitMap[17] = PieceUnit{17, "士", ColorBlack}
	factory.PieceUnitMap[18] = PieceUnit{18, "仕", ColorRed}
	factory.PieceUnitMap[19] = PieceUnit{19, "象", ColorBlack}
	factory.PieceUnitMap[20] = PieceUnit{20, "相", ColorRed}
	factory.PieceUnitMap[21] = PieceUnit{21, "象", ColorBlack}
	factory.PieceUnitMap[22] = PieceUnit{22, "相", ColorRed}
	factory.PieceUnitMap[23] = PieceUnit{23, "卒", ColorBlack}
	factory.PieceUnitMap[24] = PieceUnit{24, "兵", ColorRed}
	factory.PieceUnitMap[25] = PieceUnit{25, "卒", ColorBlack}
	factory.PieceUnitMap[26] = PieceUnit{26, "兵", ColorRed}
	factory.PieceUnitMap[27] = PieceUnit{27, "卒", ColorBlack}
	factory.PieceUnitMap[28] = PieceUnit{28, "兵", ColorRed}
	factory.PieceUnitMap[29] = PieceUnit{29, "卒", ColorBlack}
	factory.PieceUnitMap[30] = PieceUnit{30, "兵", ColorRed}
	factory.PieceUnitMap[31] = PieceUnit{31, "卒", ColorBlack}
	factory.PieceUnitMap[32] = PieceUnit{32, "兵", ColorRed}

}

// GetChessPieceUnit 根据id获取棋子享元对象
func (factory pieceUnitFactory) GetChessPieceUnit(id int) PieceUnit {
	return factory.PieceUnitMap[id]
}

// Piece 棋子
type Piece struct {
	// PieceUnit
	PieceUnit
	// PositionX
	PositionX int
	// PositionY
	PositionY int
}

// ChessBoard 棋盘
type ChessBoard struct {
	// sync.RWMutex 嵌套锁
	sync.RWMutex
	// ChessPieceMap 棋子id与棋子对象的对应关系的map
	ChessPieceMap map[int]*Piece
}

//init 初始化棋局。 You must call this。
func (board *ChessBoard) Init() {
	// 加锁
	board.Lock()
	defer board.Unlock()
	// 获取棋子享元工厂
	unitFactory := GetPieceUnitFactory()
	board.ChessPieceMap = make(map[int]*Piece)
	board.ChessPieceMap[1] = &Piece{
		PieceUnit: unitFactory.GetChessPieceUnit(1),
		PositionX: 0,
		PositionY: 1,
	}
	board.ChessPieceMap[2] = &Piece{
		PieceUnit: unitFactory.GetChessPieceUnit(2),
		PositionX: 5,
		PositionY: 1,
	}
	board.ChessPieceMap[3] = &Piece{
		PieceUnit: unitFactory.GetChessPieceUnit(3),
		PositionX: 2,
		PositionY: 8,
	}
	//......
}

// Move 棋盘上移动棋子
func (board *ChessBoard) Move(id int, positionX, positionY int) {
	board.ChessPieceMap[id].PositionX = positionX
	board.ChessPieceMap[id].PositionY = positionY
	fmt.Println(fmt.Sprintf("%s的%s被移动到了(%d，%d)的位置", board.ChessPieceMap[id].Color, board.ChessPieceMap[id].Text, positionX, positionY))
}
```



```go
flyweight_test.go
package flyweight

import (
	"testing"
)

func Test(t *testing.T) {
	t.Run("flyweight", Flyweight)
}

func Flyweight(t *testing.T) {
	board1 := ChessBoard{}
	board1.Init()
	board1.Move(1, 10, 20)

	board2 := ChessBoard{}
	board2.Init()
	board2.Move(2, 6, 6)
}
```



## 12. 备忘录模式

```go
memorandum.go
// package memorandum 备忘录模式
package memorandum

import "fmt"

// Originator 发起者，负责创建备忘录，可以记录和恢复本身的状态。
type Originator struct {
	// State 当前状态
	State string
}

// restoreMemento 将发起人的状态更新为备忘录的状态，即恢复至备忘录
func (originator *Originator) restoreMemento(memento Memento) {
	originator.State = memento.state
}

// createMemento 使用发起者当前的状态，创建备忘录
func (originator *Originator) createMemento() Memento {
	return Memento{originator.State}
}

// print 打印发起者的状态
func (originator *Originator) print() {
	fmt.Println(fmt.Sprintf("originator的状态为%s", originator.State))
}

// Memento 备忘录，存储发起者的状态，防止除发起者之外的人访问备忘录
type Memento struct {
	// state 当前状态，不可导出
	state string
}

// Caretaker 管理者，负责存储备忘录，但不对备忘录操作
type Caretaker struct {
	// memento 备忘录对象
	memento Memento
}
```



```go
memorandum_test.go
package memorandum

import "testing"

func Test(t *testing.T) {
	t.Run("memorandum", Memorandum)
}

func Memorandum(t *testing.T) {
	// 发起者初始状态
	originator := Originator{}
	originator.State = "start"
	originator.print()

	// 设置备忘录
	caretaker := Caretaker{}
	caretaker.memento = originator.createMemento()
	originator.State = "Stage Two"
	originator.print()

	// 恢复为备忘录
	originator.restoreMemento(caretaker.memento)
	originator.print()
}
```



## 13. 观察者模式

```go
observer.go
// package observer 观察者模式，以两种类型的观察者（可以有多个观察者）为例，向被观察者添加观察者，观察者完成操作后，通知观察者
package observer

import "fmt"

// Observer 观察者
type Observer interface {
	//notify 通知
	notify(sub Subject)
}

// BaseObserver 基础观察者
type BaseObserver struct {
	Name string
}

// TeacherObserver 老师观察者
type TeacherObserver struct {
	BaseObserver
}

// ParentObserver 家长观察者
type ParentObserver struct {
	BaseObserver
}

// notify 老师观察者，实现Observer接口
func (to TeacherObserver) notify(sub Subject) {
	fmt.Println(to.Name + "老师收到了作业")
}

// notify 家长观察者，实现Observer接口
func (to ParentObserver) notify(sub Subject) {
	fmt.Println(to.Name + "(家长)收到了作业")
}
```



```go
subject.go
package observer

import "fmt"

//Subject 被观察者接口
type Subject interface {
	//通知观察者
	NotifyObservers()

	//添加观察者
	AddObserver(observer Observer)

	//批量添加观察者
	AddObservers(observers ...Observer)

	//移除观察者
	RemoveObserver(observer Observer)

	//移除全部观察者
	RemoveAllObservers()
}

// StudentSubject 学生被观察者，实现Subject接口
type StudentSubject struct {
	// observers 观察者们
	observers []Observer
}

// NotifyObservers 通知观察者 注：需要使用指针接收者，不然s.observers的值会为空
func (ss *StudentSubject) NotifyObservers() {
	fmt.Println("学生写完了作业，通知观察者们")
	for _, o := range ss.observers {
		//通知观察者
		o.notify(ss)
	}
}

// AddObserver 添加观察者
func (ss *StudentSubject) AddObserver(observer Observer) {
	ss.observers = append(ss.observers, observer)
}

// AddObservers 添加观察者列表
func (ss *StudentSubject) AddObservers(observers ...Observer) {
	ss.observers = append(ss.observers, observers...)
}

// RemoveObserver 移除观察者
func (ss *StudentSubject) RemoveObserver(observer Observer) {
	for i := 0; i < len(ss.observers); i++ {
		if ss.observers[i] == observer {
			ss.observers = append(ss.observers[:i], ss.observers[i+1:]...)
		}
	}
}

// RemoveAllObservers 移除全部观察者
func (ss *StudentSubject) RemoveAllObservers() {
	ss.observers = ss.observers[:0]
}
```



```go
observer_test.go
package observer

import (
	"fmt"
	"testing"
)

func Test(t *testing.T) {
	t.Run("observer: ", NotifyObservers)
}

func NotifyObservers(t *testing.T) {
	var subject Subject
	subject = &StudentSubject{}

	var mathTeacherObserver, englishTeacherObserver, motherObserver, fatherObserver Observer
	mathTeacherObserver = &TeacherObserver{BaseObserver{"数学"}}
	englishTeacherObserver = &TeacherObserver{BaseObserver{"英语"}}
	motherObserver = &ParentObserver{BaseObserver{"妈妈"}}
	fatherObserver = &ParentObserver{BaseObserver{"爸爸"}}

	//只添加数学老师观察者，那么只有数学老师会收到作业完成通知
	fmt.Println("******只添加数学老师观察者，那么只有数学老师会收到作业完成通知******")
	subject.AddObserver(mathTeacherObserver)
	subject.NotifyObservers()
	fmt.Println()

	//又批量添加了英语老师、妈妈、爸爸，那么数学老师、英语老师、妈妈、爸爸，都会收到作业完成通知
	fmt.Println("******又批量添加了英语老师、妈妈、爸爸，那么数学老师、英语老师、妈妈、爸爸，都会收到作业完成通知******")
	subject.AddObservers(englishTeacherObserver, motherObserver, fatherObserver)
	subject.NotifyObservers()
	fmt.Println()

	//移除了妈妈观察者，那么只有数学老师、英语老师、爸爸，会收到作业完成通知
	fmt.Println("******移除了妈妈观察者，那么只有数学老师、英语老师、爸爸，会收到作业完成通知******")
	subject.RemoveObserver(motherObserver)
	subject.NotifyObservers()
	fmt.Println()

	//移除了所有观察者，那么不会有人收到作业完成通知
	fmt.Println("******移除了所有观察者，那么不会有人收到作业完成通知******")
	subject.RemoveAllObservers()
	subject.NotifyObservers()
	fmt.Println()
}
```



## 14. 代理模式

```go
proxy.go
package proxy

import "fmt"

// Downloader 下载器接口
type Downloader interface {
	// Download 下载
	Download()
}

// ConcreteDownload 具体的下载结构体，实现Downloader接口
type ConcreteDownload struct {
	Url string
}

// Download 下载
func (download ConcreteDownload) Download() {
	fmt.Println(fmt.Sprintf("%s 在下载中", download.Url))
}

// DownloadProxy，下载代理，实现Downloader接口
type DownloadProxy struct {
	Url        string
	downloader Downloader
}

// Download 下载
func (proxy DownloadProxy) Download() {
	fmt.Println(fmt.Sprintf("准备开始下载%s", proxy.Url))
	proxy.downloader.Download()
	fmt.Println(fmt.Sprintf("下载%s完成", proxy.Url))
}
```



```go
proxy_test.go
package proxy

import (
	"testing"
)

func Test(t *testing.T) {
	t.Run("proxy-statics: ", StaticsProxy)
}

func StaticsProxy(t *testing.T) {
	url := "http://baidu.com"
	download := &ConcreteDownload{Url: url}
	proxy := &DownloadProxy{
		Url:        url,
		downloader: download,
	}
	proxy.Download()
}
```



## 15. 责任链模式

```go
responsibility_chain.go
/*
    package responsibility_chain 责任链模式，把多个处理器串成链，然后让请求在链上传递。
    以财务审批为例：
    Leader 直接上级只能审核500元以下的报销
	Director 总监只能审核5000元以下的报销
	CFO 首席财务官可以审核任意金额的报销
*/
package responsibility_chain

import "fmt"

// Manager 管理者接口
type Manager interface {
	// Review 审核
	Review(request Request) bool
}

// Request 报销请求
type Request struct {
	// Name 姓名
	Name string
	// Amount 金额
	Amount int
}

// Leader 直接上级只能审核500元以下的报销，实现Manager接口
type Leader struct{}

// Director 总监只能审核5000元以下的报销，实现Manager接口
type Director struct{}

// CFO 首席财务官可以审核任意金额的报销，实现Manager接口
type CFO struct{}

// Review Leader实现报销方法
func (leader Leader) Review(request Request) bool {
	if request.Amount < 500 {
		fmt.Println(fmt.Sprintf("leader 处理了%s的%d元报销", request.Name, request.Amount))
		return true
	}
	return false
}

// Review Director实现报销方法
func (director Director) Review(request Request) bool {
	if request.Amount < 5000 {
		fmt.Println(fmt.Sprintf("director 处理了%s的%d元报销", request.Name, request.Amount))
		return true
	}
	return false
}

// Review CFO实现报销方法
func (cfo CFO) Review(request Request) bool {
	fmt.Println(fmt.Sprintf("cfo 处理了%s的%d元报销", request.Name, request.Amount))
	return true
}

// HandlerChain 责任链处理器
type HandlerChain struct {
	// managers 责任链切片，不可导出
	managers []Manager
}

// AddHandler 责任链处理器对象，添加处理器。注：使用指针接收者，需要更改切片
func (chain *HandlerChain) AddHandler(manager Manager) {
	chain.managers = append(chain.managers, manager)
}

// AddHandlers 责任链处理器对象，批量添加处理器。注：使用指针接收者，需要更改切片
func (chain *HandlerChain) AddHandlers(managers ...Manager) {
	chain.managers = append(chain.managers, managers...)
}

// HandleRequest 责任链处理器对象，处理请求
func (chain *HandlerChain) HandleRequest(request Request) error {
	for _, manager := range chain.managers {
		result := manager.Review(request)
		// 如果result为true，则说明已经处理完成
		if result {
			return nil
		}
	}
	return fmt.Errorf("请求未被处理")
}
```



```go
responsibility_chain_test.go
package responsibility_chain

import "testing"

func Test(t *testing.T) {
	t.Run("responsibility_chain_200: ", ChainOfResponsibility200)
	t.Run("responsibility_chain_3000: ", ChainOfResponsibility3000)
	t.Run("responsibility_chain_6000: ", ChainOfResponsibility6000)
}

// ChainOfResponsibility200 200元报销请求
func ChainOfResponsibility200(t *testing.T) {
	var leader, director, cfo Manager
	leader = &Leader{}
	director = &Director{}
	cfo = &CFO{}

	handlerChain := HandlerChain{}
	handlerChain.AddHandler(leader)
	handlerChain.AddHandler(director)
	handlerChain.AddHandler(cfo)

	request := Request{
		Name:   "kay",
		Amount: 200,
	}
	handlerChain.HandleRequest(request)
}

// ChainOfResponsibility3000 3000元报销请求
func ChainOfResponsibility3000(t *testing.T) {
	var leader, director, cfo Manager
	leader = &Leader{}
	director = &Director{}
	cfo = &CFO{}

	handlerChain := HandlerChain{}
	handlerChain.AddHandlers(leader, director, cfo)

	request := Request{
		Name:   "kay",
		Amount: 3000,
	}
	handlerChain.HandleRequest(request)
}

// ChainOfResponsibility6000 6000元报销请求
func ChainOfResponsibility6000(t *testing.T) {
	var leader, director, cfo Manager
	leader = &Leader{}
	director = &Director{}
	cfo = &CFO{}

	handlerChain := HandlerChain{}
	handlerChain.AddHandlers(leader, director, cfo)

	request := Request{
		Name:   "kay",
		Amount: 6000,
	}
	handlerChain.HandleRequest(request)
}
```



## 16. 简单工厂模式

```go
simple_factory.go
// package simple_factory 简单工厂模式。提供创建接口对象的工厂方法，将对象交由工厂创建，客户端只和工厂打交道，获得接口对象。
package simple_factory

import "fmt"

// Fruit 水果接口
type Fruit interface {
	// Peeling 削果皮
	Peeling()

	// Eat 吃水果
	Eat()
}

// Apple 苹果，实现Fruit接口
type Apple struct{}

// Banana 香蕉，实现Fruit接口
type Banana struct{}

// Orange 橘子，实现Fruit接口
type Orange struct{}

// Peeling 苹果削果皮
func (apple Apple) Peeling() {
	fmt.Println("削苹果果皮")
}

// Eat 吃苹果
func (apple Apple) Eat() {
	fmt.Println("吃苹果")
}

// Peeling 香蕉剥果皮
func (banana Banana) Peeling() {
	fmt.Println("剥开香蕉果皮")
}

// Eat 吃香蕉
func (banana Banana) Eat() {
	fmt.Println("吃香蕉")
}

// Peeling 橘子剥果皮
func (orange Orange) Peeling() {
	fmt.Println("剥开橘子果皮")
}

// Eat 吃橘子
func (orange Orange) Eat() {
	fmt.Println("吃橘子")
}

// FruitType 水果类型
type FruitType uint8

const (
	// FruitTypeApple 苹果
	FruitTypeApple FruitType = iota
	// FruitTypeBanana 香蕉
	FruitTypeBanana
	// FruitTypeOrange 橙子
	FruitTypeOrange
)

// FruitFuncMap 全局可导出变量，水果类型与创建水果对象的map，用于减小圈复杂度
var FruitFuncMap = map[FruitType]func() Fruit{
	FruitTypeApple:  produceApple,
	FruitTypeBanana: produceBanana,
	FruitTypeOrange: produceOrange,
}

// ProduceFruit 生产水果的简单工厂，根据水果类型，获取创建接口对象的func
func ProduceFruit(fruitType FruitType) Fruit {
	if handler, ok := FruitFuncMap[fruitType]; ok {
		return handler()
	}
	return nil
}

// produceApple 生产苹果
func produceApple() Fruit {
	return &Apple{}
}

// produceApple 生产香蕉
func produceBanana() Fruit {
	return &Banana{}
}

// produceApple 生产橘子
func produceOrange() Fruit {
	return &Orange{}
}
```



```go
simple_factory_test.go
package simple_factory

import "testing"

func Test(t *testing.T) {
	t.Run("simple_factory: ", ProduceFruitAndEat)
}

func ProduceFruitAndEat(t *testing.T) {
	var apple, banana, orange Fruit
	apple = ProduceFruit(FruitTypeApple)
	banana = ProduceFruit(FruitTypeBanana)
	orange = ProduceFruit(FruitTypeOrange)

	apple.Peeling()
	apple.Eat()

	banana.Peeling()
	banana.Eat()

	orange.Peeling()
	orange.Eat()
}
```



## 17. 单例模式

```go
singleton.go
// package singleton 单例模式
package singleton

import (
	"fmt"
	"sync"
)

var (
	// once 保证只执行一次
	once sync.Once

	// instance 单例对象
	instance *singleton
)

// singleton 单例类型结构体
type singleton struct {
	// Title 标题
	Title string
}

// GetInstance 获取单例对象。 注：必须使用指针对象，否则会在返回时，拷贝一份，对象就不是同一个了。
func GetInstance(title string) *singleton {
	once.Do(func() {
		fmt.Println("执行获取单例对象")
		instance = &singleton{Title: title}
	})
	return instance
}
```



```go
singleton_test.go
package singleton

import (
	"fmt"
	"testing"
)

func Test(t *testing.T) {
	t.Run("singleton: ", GetSingleton)
}

func GetSingleton(t *testing.T) {
	/* 执行结果：
	   执行获取单例对象
	   instance1 = 0xc0000584a0, name=标题1
	   instance2 = 0xc0000584a0, name=标题1
	*/

	instance1 := GetInstance("标题1")
	fmt.Println(fmt.Sprintf("instance1 = %p, name=%s", instance1, instance1.Title))

	instance2 := GetInstance("标题2")
	fmt.Println(fmt.Sprintf("instance2 = %p, name=%s", instance2, instance2.Title))
}
```



## 18. 状态模式

```go
state.go
// package state 状态模式
package state

import "fmt"

// State 用户状态
type State uint8

const (
	// StateNormal 普通用户
	StateNormal State = 1
	// StateVip 普通会员
	StateVip State = 2
)

// ISwitchState 转换状态接口
type ISwitchState interface {
	// PurchaseVip 购买会员
	PurchaseVip()
	// Expire 会员过期
	Expire()
}

// IUser 用户接口
type IUser interface {
	// WatchVideo 看视频
	WatchVideo()
}

// NormalUser 普通用户，实现IUser接口
type NormalUser struct{}

// WatchVideo 看视频
func (user NormalUser) WatchVideo() {
	fmt.Println("看广告中...")
}

// VipUser 会员用户，实现IUser接口
type VipUser struct{}

// WatchVideo 看视频
func (user VipUser) WatchVideo() {
	fmt.Println("您是尊敬的vip用户，已为您跳过120s广告")
}

// User 用户，实现ISwitchState、IUser接口
type User struct {
	// UserState 用户
	UserState IUser
}

// SetUser 设置用户状态
func (user *User) SetUser(userState IUser) {
	user.UserState = userState
}

// Expire 会员过期
func (user *User) Expire() {
	user.UserState = NormalUser{}
}

// PurchaseVip 购买vip会员
func (user *User) PurchaseVip() {
	user.UserState = VipUser{}
}

// WatchVideo 看视频
func (user *User) WatchVideo() {
	if user.UserState == nil {
		// 默认为普通用户
		user.SetUser(NormalUser{})
	}
	user.UserState.WatchVideo()
}
```



```go
state_test.go
package state

import (
	"testing"
)

func Test(t *testing.T) {
	t.Run("state", StateUser)
}

func StateUser(t *testing.T) {
	user := User{}
	user.WatchVideo()

	user.PurchaseVip()
	user.WatchVideo()

	user.Expire()
	user.WatchVideo()
}
```



## 19. 策略模式

```go
strategy.go
// package strategy 策略模式。以商场促销方案为例。
package strategy

import "fmt"

// Strategy 策略接口
type Strategy interface {
	// Promotion 促销
	Promotion()
}

// ConcreteStrategyA 促销策略A
type ConcreteStrategyA struct {
}

// ConcreteStrategyB 促销策略B
type ConcreteStrategyB struct {
}

// ConcreteStrategyC 促销策略C
type ConcreteStrategyC struct {
}

func (strategy ConcreteStrategyA) Promotion() {
	fmt.Println("618促销")
}

func (strategy ConcreteStrategyB) Promotion() {
	fmt.Println("99促销")
}

func (strategy ConcreteStrategyC) Promotion() {
	fmt.Println("双十一促销")
}
```



```go
context.go
package strategy

import "context"

// Context
type Context struct {
	// context.Context 嵌套上下文
	context.Context
	//Strategy 策略对象
	Strategy Strategy
}

// Sale 促销
func Sale(saleType SaleType) {
	strategy := NewSaleStrategy(saleType)
	strategy.Promotion()
}

// SaleType 促销类型
type SaleType uint8

const (
	// SaleTypeA 促销类型A
	SaleTypeA SaleType = iota
	// SaleTypeB 促销类型B
	SaleTypeB
	// SaleTypeC 促销类型C
	SaleTypeC
)

// SaleTypeFuncMap 全局可导出变量，咖啡额外添加类型与创建咖啡额外添加对象的map，用于减小圈复杂度
var SaleTypeFuncMap = map[SaleType]func() Strategy{
	SaleTypeA: NewStrategyA,
	SaleTypeB: NewStrategyB,
	SaleTypeC: NewStrategyC,
}

// NewSaleStrategy 创建促销类型接口对象的简单工厂，根据促销类型类型，获取创建接口对象的func
func NewSaleStrategy(saleType SaleType) Strategy {
	if handler, ok := SaleTypeFuncMap[saleType]; ok {
		return handler()
	}
	return nil
}

// NewStrategyA 创建促销A
func NewStrategyA() Strategy {
	return &ConcreteStrategyA{}
}

// NewStrategyB 创建促销B
func NewStrategyB() Strategy {
	return &ConcreteStrategyB{}
}

// NewStrategyC 创建促销C
func NewStrategyC() Strategy {
	return &ConcreteStrategyC{}
}
```



```go
strategy_test.go
// package strategy 策略模式。以商场促销方案为例。
package strategy

import "fmt"

// Strategy 策略接口
type Strategy interface {
	// Promotion 促销
	Promotion()
}

// ConcreteStrategyA 促销策略A
type ConcreteStrategyA struct {
}

// ConcreteStrategyB 促销策略B
type ConcreteStrategyB struct {
}

// ConcreteStrategyC 促销策略C
type ConcreteStrategyC struct {
}

func (strategy ConcreteStrategyA) Promotion() {
	fmt.Println("618促销")
}

func (strategy ConcreteStrategyB) Promotion() {
	fmt.Println("99促销")
}

func (strategy ConcreteStrategyC) Promotion() {
	fmt.Println("双十一促销")
}
```



## 20. 模版模式

```go
template.go
// package template 模版模式
package template

import "fmt"

// AskForLeaveRequest 请假单接口
type AskForLeaveRequest interface {
	GetName() string
	ReasonForLeave() string
	HowManyDaysForLeave() float32
}

// Company 公司
type Company struct {
	// AskForLeaveRequest 组合AskForLeaveRequest接口对象
	AskForLeaveRequest
}

// AskLeave 请假
func (company Company) AskLeave() {
	fmt.Println(fmt.Sprintf("%s 因为 %s，请假 %.1f 天",
		company.GetName(), company.ReasonForLeave(), company.HowManyDaysForLeave()))
}

// MyAskForLeaveRequest 我的请假单，实现AskForLeaveRequest接口
type MyAskForLeaveRequest struct {
}

// GetName 请假人姓名
func (request MyAskForLeaveRequest) GetName() string {
	return "kaysun"
}

// ReasonForLeave 请假事由
func (request MyAskForLeaveRequest) ReasonForLeave() string {
	return "给娃打疫苗"
}

// HowManyDaysForLeave 请假多少天
func (request MyAskForLeaveRequest) HowManyDaysForLeave() float32 {
	return 0.5
}

// MyAskForLeaveRequest 我的请假单，实现AskForLeaveRequest接口
type TomAskForLeaveRequest struct {
}

// GetName 请假人姓名
func (request TomAskForLeaveRequest) GetName() string {
	return "tom"
}

// ReasonForLeave 请假事由
func (request TomAskForLeaveRequest) ReasonForLeave() string {
	return "回家探亲"
}

// HowManyDaysForLeave 请假多少天
func (request TomAskForLeaveRequest) HowManyDaysForLeave() float32 {
	return 5
}
```



```go
template_test.go
package template

import (
	"testing"
)

func Test(t *testing.T) {
	t.Run("template: my", MyAskForLeaveTemplate)
	t.Run("template: tom", TomAskForLeaveTemplate)
}

func MyAskForLeaveTemplate(t *testing.T) {
	request := &MyAskForLeaveRequest{}
	company := Company{request}
	company.AskLeave()
}

func TomAskForLeaveTemplate(t *testing.T) {
	request := &TomAskForLeaveRequest{}
	company := Company{request}
	company.AskLeave()
}
```

