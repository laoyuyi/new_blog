##### 模块和文件
   - `module`：定义模块。
   - `end`：结束命名空间、代码段或模块。
   - `import`：导入模块。
   - `export`：导出模块内容。
   - `open`：打开命名空间或模块。
   - `prelude`：预加载模块。
   - `init`：初始化代码。

##### 调试和元信息
   - `#check`：检查类型。
   - `#eval`：求值表达式。
   - `#print`：打印信息。
   - `#exit`：退出程序。
   - `#set_option`：设置选项。
   - `#show`：显示信息。
   - `#trace`：跟踪信息。
   - `#time`：计时信息。
   - `#reduce`：减少表达式。
   - `#compile`：编译表达式。
   - `#compile_module`：编译模块。
   - `#compile_file`：编译文件。
   - `#print_environment`：打印环境。
   - `#print_all`：打印所有信息。
   - `#print_axioms`：打印公理。
   - `#print_notation`：打印符号。
   - `#print_rules`：打印规则。
   - `#print_instances`：打印实例。

##### 注解
- 单行注解：`-- note`
- 多行注解：`\- note -\`
- 模块注解：`library_note`关键字
##### 定义：`def`
- 隐式参数：使用`{}`包裹
- 类型约束：使用`[]`包裹
- 显式参数：使用`()`包裹
##### 类型
`inductive`关键字用于定义归纳类型
```java
inductive Nat where
  | zero : Nat
  | succ (n : Nat) : Nat
```
##### 别名：`alias`
##### 引理：`lemma`
```java
lemma name : type := proof
```
##### 定理：`theorem`
##### 类型类
`class`
#####  实例化：`instance`
##### 属性注解
   - `@[... ]`：属性注解，用于标记定义或声明。
   - `@[reducible]`：标记定义为可约简的。
   - `@[irreducible]`：标记定义为不可约简的。
   - `@[inline]`：标记定义为内联的。
   - `@[simp]`：标记定义为简化规则。
   - `@[derive]`：自动派生实例。
   - `@[specialize]`：标记定义为特化的。
   - `@[extern]`：外部定义。
   - `@[extern "C"]`：外部 C 语言定义。
   - `@[extern "lean"]`：外部 Lean 定义。
   - `@[extern "cpp"]`：外部 C++ 定义。
   - `@[export]`：导出定义。
   - `@[inline]`：内联定义。
   - `@[opaque]`：不透明定义。
   - `@[abbrev]`：定义缩写。
   - `@[notation]`：定义自定义符号。
   - `@[reserve]`：保留符号。
   - `@[scoped]`：作用域内定义。

---

#####
`attribute`
`structure`
`match`
`private`
`namespace`
`end`
`opaque`
`abbrev`
`fun`
`deriving`
`unsafe`
`extern`
`set_option`
`show`
`protected`

---

1. **元编程和宏**
   - `macro_rules`：定义宏规则。
   - `macro`：定义宏。
   - `elab`：定义解析器。
   - `syntax`：定义语法。
   - `precedence`：定义优先级。
   - `category`：定义语法类别。
   - `command`：定义命令。
   - `environment`：环境操作。
   - `tactic`：定义战术。
   - `meta`：元编程相关。
   - `eval`：评估表达式。
   - `quote`：引用表达式。
   - `unquote`：取消引用表达式。
   - `stx`：语法树节点。
   - `mk_syntax`：创建语法树节点。
   - `mk_ident`：创建标识符。
   - `mk_numLit`：创建数字字面量。
   - `mk_strLit`：创建字符串字面量。

2. **语法和解析**
   - `infix`：定义中缀操作符。
   - `infixl`：定义左结合中缀操作符。
   - `infixr`：定义右结合中缀操作符。
   - `postfix`：定义后缀操作符。
   - `prefix`：定义前缀操作符。
   - `notation`：定义自定义符号。
   - `reserve`：保留符号。
   - `local`：局部定义。
   - `scoped`：作用域内定义。

5. **控制流和模式匹配**
   - `if`：条件分支。
   - `then`：条件分支的一部分。
   - `else`：条件分支的一部分。
   - `match`：模式匹配。
   - `with`：模式匹配的一部分。
   - `|`：模式匹配中的分支分隔符。
   - `=>`：模式匹配中的箭头。
   - `←`：绑定操作符，用于 `do` 表达式中。
   - `<-`：绑定操作符的 ASCII 版本。
   - `do`：顺序执行操作。
   - `let`：局部绑定。
   - `in`：局部绑定的一部分。
   - `for`：循环。
   - `while`：循环。
   - `break`：跳出循环。
   - `continue`：继续下一次循环。
   - `return`：返回值。

6. **类型和约束**
   - `Type`：类型宇宙。
   - `Prop`：命题类型。
   - `universe`：定义宇宙变量。
   - `u`、`v`、`w` 等：宇宙变量。
   - `outParam`：输出参数。
   - `by`：开始证明块。
   - `rw`：重写战术。
   - `simp`：简化战术。
   - `refl`：反射战术。
   - `exact`：精确匹配战术。
   - `sorry`：暂时跳过证明。
   - `as`：类型转换。
   - `:>`：显式类型注释。
   - `:>`：显式类型注释。
   - `:=`：定义赋值。

8. **其他**
   - `private`：私有声明。
   - `protected`：受保护声明。
   - `noncomputable`：非计算定义。
   - `deriving`：自动派生实例。
   - `default`：默认值。
   - `mutable`：可变字段。
   - `immutable`：不可变字段。
   - `inline`：内联定义。
   - `opaque`：不透明定义。
   - `abbrev`：定义缩写。
   - `notation`：定义自定义符号。
   - `reserve`：保留符号。
   - `infix`：定义中缀操作符。
   - `infixl`：定义左结合中缀操作符。
   - `infixr`：定义右结合中缀操作符。
   - `postfix`：定义后缀操作符。
   - `prefix`：定义前缀操作符。