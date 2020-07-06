# New SwiftUI APP Structure
## APP Entrance
```swift
@main
struct MyApp: App {
	@SceneBuilder var body: some Scene {
		// 对于不基于文档的应用程序
		WindowGroup {
			<#some View#>
		}
		// 对于基于文档的应用程序
		DocumentGroup(newDocument: TextFile()) { textFile in
			TextEditor(textFile.$document.text)
		}

		#if os(macOS)
		Settings {
			SettingsView()
		}
		#endif
	}
}
```

## Code Structure
在使用新的“多平台应用程序”模版创建应用程序时，会自动生成新的代码整理结构。shared用于存放与特定平台无关的代码，而针对不同平台的不同功能和API调用的代码，则分别存放于对应的平台名称的文件夹下。