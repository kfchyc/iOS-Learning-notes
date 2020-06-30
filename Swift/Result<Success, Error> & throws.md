# `Result<Success, Error>` & `throws`
## 将`throws`转换为`Result`
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

## `Result`转换成`throws`
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


>一般情况下：推荐设计同步API的时候仍旧使用`throws`，在有使用需要的时候再转换成`Result`。

## `Result`的`flatMap`与`do/try/catch`
> 假设有多个同步返回`Result`的函数进行连续调用，如果每个结果都直接用pattern matching来解，很容易形成pattern matching的多层嵌套。

### `Result.flatMap`如何帮助解决这个问题？
```swift
func fetchImageData(from url: URL) -> Result<Data, Error> {
	return Result(catching: { try Data(contentsOf: url) })
}

func process(imageData: Data) -> Result<UIImage, Error> {
	if let image = UIImage(data: imageData) {
		return .success(image)
	} else {
		return .failure(ImageProcessingError.corruptedData)
	}
}

func persist(image: UIImage) -> Result<Void, Error> {
	return .success(())
}

let result = fetchImageData(from: url)
				.flatMap(process)
				.flatMap(persist)

switch result {
	case .success: 
		// do something
		break
	case .failure(ImageProcessingError.corruptedData):
		// do something
		break
	case .failure(CocoaError.fileNoSuchFile):
		// do something
		break
	default:
		// do something
		break
}
 ```

### `do/try/catch`如何解决这个问题？
```swift
func fetchImageData(from url: URL) throws -> Data {
	return try Data(contentsOf: url)
}

func process(imageData: Data) throws -> UIImage {
	if let image = UIImage(data: image) {
		return image
	} else {
		throw ImageProcessingError.corruptedData
	}
}

func persist(image: UIImage) throws {

}

do {
	let data = try fetchImageData(from: url)
	let image = try process(image: data)
	try persist(image: image)
} catch ImageProcessingError.corruptedData {

} catch CocoaError.fileNoSuchFile {

} catch {

}
```

>同样的功能，`do/try/catch`比使用`Result`更简练灵活。因此一般情况下仍推荐使用`throw/throws`的形式。

## 关于`Error`协议
>原本在Swift中，对某个协议的实现必须是具体的实际类型，但是Error具有特殊性，被认为实现了协议本身。

```swift
struct A<T: K> {}

protocol K {
	func doIt()
}

// 编译错误 Protocol type 'K' cannot conform to 'K' because only concrete types can conform to protocols
let a = A<K>()

struct B<T: Error> {}
// 编译通过
let b = B<Error>()
```

编译错误是：`K`协议没有实现`struct A`中，对T的类型K的实现。仅有实际类型可以实现接口；但如果`K`改成`Error`的话，则可以编译通过。证明了`Error`具有特殊性：他被认为实现了协议本身。