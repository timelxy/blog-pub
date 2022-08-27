# Maven核心概念理解：Build Lifecycle
## 序
在偶尔接触java项目的过程中，经常在maven编译环节报错，每次都要各种搜索才能勉强跑通。 反省了下，主要原因是对maven缺乏体系理解。中间蜻蜓点水看了官方文档多次，过一段时间不使用又忘了。用diigo做的标记，也不怎么利于唤醒记忆。这次，决定把lifecycle涉及概念之间的关系，简单记录下来，作为下次的记忆装载快照。

## Phase
1. lifecycle由phase list组成
2. 不同的lifecycle，phase list不同
3. 一个phase代表一个lifecycle stage。所有的phase见[链接](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)
4. phase串行顺序执行
5. commad line里指定执行某个phase，其之前的phase也会被顺序的先执行。如执行"mvn package"，之前的phase都会被执行
6. 有些phase不能直接在commad line里单独执行，因为它们和其他phase强关联，单独执行会导致一种中间态

## Goal
1. 每个phase实际的执行内容，总体符合phase定义。 实际细节会有所不同。 这种“不同”，通过plugin goal实现
2. goal代表一个具体的task。 粒度比phase更细
3. goal可以与0或多个phase绑定。 绑定0个：可以在lifecycle之外单独执行。如：
   >dependency:copy-dependencies，其中dependency是plugin，copy-dependencies是goal
4. 绑定大于1个phase，每个phase执行都会call一次
5. phase如果没有绑定任何goal，不会被执行

## 如何在project中，把各种goal（task）加入到build phases中？
1. 第零种：一些phase 本身就带有默认的goal。见[链接](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#built-in-lifecycle-bindings)
2. 第一种：设定project的packaging方式。默认是jar。每种packaging方式，会把对应的一些goal绑定到default lifecycle中
3. 第二种：使用Plugins
   - 定义：plugin are artifacts that provide goals to Maven
   - 一个phase多个goal的执行顺序：先packaging相关，后plugin相关。使用 **\<executions\>** 标签可以改变默认的顺序
   - 有些goal可以在多个phase中使用。 可以在 **\<execution\>** 标签中，使用 **\<phase>phasename\<phase>** 指明phase
## 参考
https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
