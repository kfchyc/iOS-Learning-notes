# Result<Success, Error> & throws
## 将throws转换为Result
```swift
extension Result where Failure == Swift.Error {
	public init(catching body: () throws -> Success) {
		do {
			self = .success(try body())
		} catch {
			self = .failure(error)
		}
	}
}
```

使用方法：
```swift
let config = Result { try String(contentsOfFile: configuration) }
// do something with config later
```

## Result转换成throws
利用Result自带的`get`方法：
```swift
public func get() throws -> Success {
	switch self {
		case let .success(success):
			return success
		case let .failure(failure):
			throw failure
	}
}
```


>一般情况下：推荐设计同步API的时候仍旧使用throws，在有使用需要的时候再转换成Result。

