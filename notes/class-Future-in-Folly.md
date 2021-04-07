# Folly 库与 STL 中的 Future 实现对比



### Folly 库的 Future 实现

在 folly/futures/Future.h 中先通过继承 std::logic_error 来定义了一堆跟 Future 有关的 Exception，其中 `FutureAlreadyContinued` 这个异常的注释中写到每一个 Future 对象只能绑定一个 continuation ，即 chained callback，在 Future 对象的值被设置之后会链式执行这个 callback：

```cpp
class FOLLY_EXPORT FutureException : public std::logic_error {
 public:
  using std::logic_error::logic_error;
};

class FOLLY_EXPORT FutureInvalid : public FutureException { ... };

/// At most one continuation may be attached to any given Future.
///
/// If a continuation is attached to a future to which another continuation has
/// already been attached, then an instance of FutureAlreadyContinued will be
/// thrown instead.
class FOLLY_EXPORT FutureAlreadyContinued : public FutureException { ... };

class FOLLY_EXPORT FutureNotReady : public FutureException { ... };

class FOLLY_EXPORT FutureCancellation : public FutureException { ... };

class FOLLY_EXPORT FutureTimeout : public FutureException { ... };

class FOLLY_EXPORT FuturePredicateDoesNotObtain : public FutureException { ... };

class FOLLY_EXPORT FutureNoTimekeeper : public FutureException { ... };

class FOLLY_EXPORT FutureNoExecutor : public FutureException { ... };
```

