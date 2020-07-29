# iOS Notes for Professionals 学习笔记

## Chapter 1 Getting start with iOS Development

略

## Chapter 2 UILabel

### Section 2.1 Create a UILabel

#### 2.1.1 创建

两种创建方式：给定frame / AutoLayout

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

