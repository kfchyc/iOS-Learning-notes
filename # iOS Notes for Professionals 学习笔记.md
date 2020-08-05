# iOS Notes for Professionals 学习笔记

## Chapter 1 Getting start with iOS Development

略

## Chapter 2 UILabel

### Section 2.1 Create a UILabel

#### 2.1.1 创建和布局

Swift:

```swift
let label = UILabel()
```

Objective-C:

```objective-c
UILabel *label = [[UILabel alloc] init];
//or
UILabel *label = [UILabel new]; // convenience method for calling alloc-init
```



两种布局方式：创建时给定frame / AutoLayout

Swift:

```swift
let label = UILabel()
label.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(label)
```

Objective-C:

```objective-c

```



使用 AutoLayout 布局时要额外添加下列语句：

```swift
label.translatesAutoresizingMaskIntoConstraints = false
```

在 Swift 中，使用：

```swift
NSLayoutConstraint.activate([
  label.topAnchor.constraint(equalTo: view.topAnchor)
  // ...
])
```

在 Objective-C 中，使用 VFL ( Visual Format Language ) ：

```objective-c
[self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-5-[labelName]-5-|" options:0 metrics:nil views:@{@"labelName":label}]];
```

上述代码在垂直方向上，在 label 的上下各添加了距离它的 superView（即 self.view ) 各 5pt 的 padding。

```objective-c
[self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|[labelName]|" options:0 metrics:nil views:@{@"labelName":label}]];
```

上述代码在水平方向上，直接对齐其父 View （即 self.view ）。

附：VFL 文档：https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH27-SW1

2.1.2 声明接口（ IBOutlet ）：

Swift:

```swift
@IBOutlet weak var nameLabel: UILabel!
```

Objective-C:

```objective-c
@property (nonatomic, weak) IBOutlet UILabel *nameLabel;
```



### Section 2.2 Number of Lines 标签行数

> ⚠️ 注意1：当 numberOfLines 属性设置为 0 时，也不意味着 UILabel 就可以有无限多行（ .infinity )。如果有高度约束，会优先遵从高度约束。
>
> ⚠️ 注意2：对于复杂的多行文本，UITextView 会是更合适的选择。

### Section 2.3 Set Font 设置字体

#### 设置字体大小：

Swift:

```swift
UIFont.systemFont(ofSize: 17)
```

Objective-C:

```objective-c
[UIFont systemFontOfSize:17];k
```

#### 设置字体大小和字重：

Swift:

```swift
UIFont.system(ofSize: 17, weight: .bold)
```

Objective-C:

```objective-c
[UIFont systemFontOfSize:17 weight:UIFontWeightBold];
```

#### 动态字体：

Swift:

```swift
UIFont.preferredFont(forTextStyle: .body)
```

Objective-C:

```objective-c
[UIFont preferredFontForTextStyle:UIFontTextStyleBody];
```

#### 自定义字体：

Swift:

```swift
Font(name: "Avenir", size: 15)
```

Objective-C:

```objective-c
[UIFont fontWithName:@"Avenir" size:15];
```

#### 重新设置字体大小（不改变字体和字重等其他属性）

Swift:

```swift
label.font = label.font.fontWithSize(15)
```

Objective-C:

```objective-c
label.font = [label.font fontWithSize:15];
```

### 设置文字颜色

#### 使用`NSAttributedString`设置部分文字的颜色：

Swift:

```swift
let attributedString = NSMutableAttributedString(string: "The grass is green; the sky is blue.") attributedString.addAttribute(NSForegroundColorAttributeName, value: UIColor.green(), range: NSRange(location: 13, length: 5))
attributedString.addAttribute(NSForegroundColorAttributeName, value: UIColor.blue(), range: NSRange(location: 31, length: 4))
label.attributedText = attributedString
```

Objective-C:

```objective-c
NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:@"The grass is green; the sky is blue."];
[attributedString addAttribute: NSForegroundColorAttributeName value:[UIColor greenColor] range:NSMakeRange(13, 5)];
[attributedString addAttribute: NSForegroundColorAttributeName value:[UIColor blueColor] range:NSMakeRange(31, 4)];
label.attributedText = attributesString;
```

### Size to fit

> `sizeToFit` 方法被用来使 label 根据 content 中存储的内容来重新改变自己的大小 (resize) 。

假设 storyboard 上有个 `UILabel`，并且在代码中创建了对应的 IBOutlet。

Swift:

```swift
label.text = "Hello, World!"
label.sizeToFit()
```

Objective-C:

```objective-c
label.text = @"Hello, World!";
[label sizeToFit];
```

注意⚠️：`UILabel` 的 `numberOfLines` 属性如果设置为非0（即限制了固定行数），则 `UILabel` 会首先遵循行数限制。因此如果设置内容过长，超过了行数限制，依然会被截断。

### 根据文本内容自适应高度

