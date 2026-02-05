# go mock

## 一、参考

>[go mock](https://github.com/golang/mock)


## 二、基本使用

gomock is a mocking framework for the Go programming language. It integrates well with Go's built-in testing package, but can be used in other contexts too.

gomock是针对Go编程语言的模拟框架。它与Go的内置测试包很好地集成在一起，但也可以在其他上下文中使用。


### 2.1 Installation



```go

// Go 1.16+
go install github.com/golang/mock/mockgen@v1.6.0

```

If you use mockgen in your CI pipeline, it may be more appropriate to fixate on a specific mockgen version. You should try to keep the library in sync with the version of mockgen used to generate your mocks.

如果您在CI 中使用mockgen，固定特定的mockgen版本可能更为合适。您应该尝试让库与用于生成模拟的mockgen版本保持同步。

### 2.2 Running mockgen

mockgen has two modes of operation: source and reflect.

mockgen有两种操作模式：源代码模式和反射模式。

#### 2.2.1 Source mode


Source mode generates mock interfaces from a source file. It is enabled by using the -source flag. Other flags that may be useful in this mode are -imports and -aux_files.

源代码模式从源文件生成模拟接口。它通过使用-source标志来启用。在此模式下可能有用的其他标志是-imports和-aux_files。

Example:


```go

mockgen -source=foo.go [other options]

```

#### 2.2.2 Reflect mode

Reflect mode generates mock interfaces by building a program that uses reflection to understand interfaces. It is enabled by passing two non-flag arguments: an import path, and a comma-separated list of symbols.

You can use "." to refer to the current path's package.

反射模式通过构建一个使用反射理解接口的程序来生成模拟接口。它通过传递两个非标志参数来启用：一个导入路径和一个逗号分隔的符号列表。

您可以使用“.”来引用当前路径的包。

Example:

```go

mockgen database/sql/driver Conn,Driver

# Convenient for `go:generate`.
mockgen . Conn,Driver

```

#### 2.2.3 Flags

The mockgen command is used to generate source code for a mock class given a Go source file containing interfaces to be mocked. It supports the following flags:

* -source: A file containing interfaces to be mocked.

* -destination: A file to which to write the resulting source code. If you don't set this, the code is printed to standard output.

* -package: The package to use for the resulting mock class source code. If you don't set this, the package name is mock_ concatenated with the package of the input file.

* -imports: A list of explicit imports that should be used in the resulting source code, specified as a comma-separated list of elements of the form foo=bar/baz, where bar/baz is the package being imported and foo is the identifier to use for the package in the generated source code.

* -aux_files: A list of additional files that should be consulted to resolve e.g. embedded interfaces defined in a different file. This is specified as a comma-separated list of elements of the form foo=bar/baz.go, where bar/baz.go is the source file and foo is the package name of that file used by the -source file.

* -build_flags: (reflect mode only) Flags passed verbatim to go build.

* -mock_names: A list of custom names for generated mocks. This is specified as a comma-separated list of elements of the form Repository=MockSensorRepository,Endpoint=MockSensorEndpoint, where Repository is the interface name and MockSensorRepository is the desired mock name (mock factory method and mock recorder will be named after the mock). If one of the interfaces has no custom name specified, then default naming convention will be used.

* -self_package: The full package import path for the generated code. The purpose of this flag is to prevent import cycles in the generated code by trying to include its own package. This can happen if the mock's package is set to one of its inputs (usually the main one) and the output is stdio so mockgen cannot detect the final output package. Setting this flag will then tell mockgen which import to exclude.

* -copyright_file: Copyright file used to add copyright header to the resulting source code.

* -debug_parser: Print out parser results only.

* -exec_only: (reflect mode) If set, execute this reflection program.

* -prog_only: (reflect mode) Only generate the reflection program; write it to stdout and exit.

* -write_package_comment: Writes package documentation comment (godoc) if true. (default true)

For an example of the use of mockgen, see the sample/ directory. In simple cases, you will need only the -source flag.


mockgen命令用于根据包含要模拟的接口的Go源文件生成模拟类的源代码。它支持以下标志：

* -source: 包含要模拟的接口的文件

* -destination: 要写入生成源代码的文件。如果不设置此标志，则代码将打印到标准输出

* -package: 用于生成模拟类源代码的包名称。如果不设置此标志，则包名称为mock_加上输入文件的包名

* -imports: 显式导入列表，应用于生成的源代码，指定为foo=bar/baz形式的元素的逗号分隔列表，其中bar/baz是要导入的包，foo是要在生成的源代码中用于该包的标识符

* -aux_files: 需要查询的其他文件列表，以解决例如在不同文件中定义的嵌入接口。这是指定为元素foo=bar/baz.go的逗号分隔列表的形式，其中bar/baz.go是源文件，foo是-source文件中使用的该文件的包名称。

* -build_flags: （仅反射模式）传递给go build的标志。

* -mock_names: 生成模拟的自定义名称列表。这是指定为元素Repository=MockSensorRepository,Endpoint=MockSensorEndpoint的逗号分隔列表的形式，其中Repository是接口名称，MockSensorRepository是所需的模拟名称（模拟工厂方法和模拟记录器将以该模拟命名）。如果其中一个接口未指定自定义名称，则将使用默认命名约定。

* -self_package: 生成代码的完整包导入路径。此标志的目的是通过尝试包含自己的包来防止生成的代码中出现import循环。如果模拟的包设置为其输入之一（通常是主要输入），并且输出是stdio，因此mockgen无法检测到最终输出包，则可能会发生这种情况。设置此标志将告诉mockgen要排除哪个导入。

* -copyright_file: 用于向生成的源代码添加版权头的版权文件。

* -debug_parser: 仅打印解析器结果。

* -exec_only: （反射模式）如果设置，则执行此反射程序

* -prog_only: （反射模式）仅生成反射程序；将其写入stdout并退出

* -write_package_comment: 如果为true，则编写软件包文档注释（godoc）。 （默认为true）

有关mockgen的使用示例，请参见sample/directory。在简单情况下，您只需要-source标志。


### 2.3 Building Mocks

```go

type Foo interface {
  Bar(x int) int
}

func SUT(f Foo) {
 // ...
}

```

```go

func TestFoo(t *testing.T) {
  ctrl := gomock.NewController(t)

  // Assert that Bar() is invoked.
  defer ctrl.Finish()

  m := NewMockFoo(ctrl)

  // Asserts that the first and only call to Bar() is passed 99.
  // Anything else will fail.
  m.
    EXPECT().
    Bar(gomock.Eq(99)).
    Return(101)

  SUT(m)
}


```

If you are using a Go version of 1.14+, a mockgen version of 1.5.0+, and are passing a *testing.T into gomock.NewController(t) you no longer need to call ctrl.Finish() explicitly. It will be called for you automatically from a self registered Cleanup function.

如果您正在使用Go 1.14+版本、mockgen 1.5.0+版本，并将*testing.T传递给gomock.NewController(t)，则不再需要显式调用ctrl.Finish()。它将自动从自注册的Cleanup函数中为您调用。


```go

// (1) 创建 foo.go

package yzmock

type Foo interface {
	Bar(x int) int
}

func SUT(f Foo) {
	// ...
}

// (2) 生成mock , foo_mock.go
// mockgen -source=foo.go -destination=foo_mock.go -package=yzmock

// Code generated by MockGen. DO NOT EDIT.
// Source: foo.go

// Package yzmock is a generated GoMock package.
package yzmock

import (
	reflect "reflect"

	gomock "github.com/golang/mock/gomock"
)

// MockFoo is a mock of Foo interface.
type MockFoo struct {
	ctrl     *gomock.Controller
	recorder *MockFooMockRecorder
}

// MockFooMockRecorder is the mock recorder for MockFoo.
type MockFooMockRecorder struct {
	mock *MockFoo
}

// NewMockFoo creates a new mock instance.
func NewMockFoo(ctrl *gomock.Controller) *MockFoo {
	mock := &MockFoo{ctrl: ctrl}
	mock.recorder = &MockFooMockRecorder{mock}
	return mock
}

// EXPECT returns an object that allows the caller to indicate expected use.
func (m *MockFoo) EXPECT() *MockFooMockRecorder {
	return m.recorder
}

// Bar mocks base method.
func (m *MockFoo) Bar(x int) int {
	m.ctrl.T.Helper()
	ret := m.ctrl.Call(m, "Bar", x)
	ret0, _ := ret[0].(int)
	return ret0
}

// Bar indicates an expected call of Bar.
func (mr *MockFooMockRecorder) Bar(x interface{}) *gomock.Call {
	mr.mock.ctrl.T.Helper()
	return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "Bar", reflect.TypeOf((*MockFoo)(nil).Bar), x)
}


```

### 2.4 Building Stubs

```go

type Foo interface {
  Bar(x int) int
}

func SUT(f Foo) {
 // ...
}

```

```go

func TestFoo(t *testing.T) {
  ctrl := gomock.NewController(t)
  defer ctrl.Finish()

  m := NewMockFoo(ctrl)

  // Does not make any assertions. Executes the anonymous functions and returns
  // its result when Bar is invoked with 99.
  m.
    EXPECT().
    Bar(gomock.Eq(99)).
    DoAndReturn(func(_ int) int {
      time.Sleep(1*time.Second)
      return 101
    }).
    AnyTimes()

  // Does not make any assertions. Returns 103 when Bar is invoked with 101.
  m.
    EXPECT().
    Bar(gomock.Eq(101)).
    Return(103).
    AnyTimes()

  SUT(m)
}

```


### 2.5 Modifying Failure Messages

When a matcher reports a failure, it prints the received (Got) vs the expected (Want) value.

```go


Got: [3]
Want: is equal to 2
Expected call at user_test.go:33 doesn't match the argument at index 1.
Got: [0 1 1 2 3]
Want: is equal to 1

```

#### 2.5.1 Modifying Want

The Want value comes from the matcher's String() method. If the matcher's default output doesn't meet your needs, then it can be modified as follows:

```go


gomock.WantFormatter(
  gomock.StringerFunc(func() string { return "is equal to fifteen" }),
  gomock.Eq(15),
)

```

#### 2.5.2 Modifying Got

The Got value comes from the object's String() method if it is available. In some cases the output of an object is difficult to read (e.g., []byte) and it would be helpful for the test to print it differently. The following modifies how the Got value is formatted:

```go

gomock.GotFormatterAdapter(
  gomock.GotFormatterFunc(func(i interface{}) string {
    // Leading 0s
    return fmt.Sprintf("%02d", i)
  }),
  gomock.Eq(15),
)

```

If the received value is 3, then it will be printed as 03.

### 2.6 Debugging Errors

#### 2.6.1 reflect vendoring error

```go

cannot find package "."
... github.com/golang/mock/mockgen/model

```

If you come across this error while using reflect mode and vendoring dependencies there are three workarounds you can choose from:

1. Use source mode.

1. Include an empty import import _ "github.com/golang/mock/mockgen/model".

1. Add --build_flags=--mod=mod to your mockgen command.

This error is due to changes in default behavior of the go command in more recent versions. More details can be found in [#494](https://github.com/golang/mock/issues/494).


