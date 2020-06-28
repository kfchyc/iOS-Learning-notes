# Block
## 问题列表
1. block的内部实现，结构体是什么样的
2. block是类吗，有哪些类型
3. 一个int变量被 __block 修饰与否的区别？block的变量截获
4. block在修改NSMutableArray，需不需要添加__block
5. 怎么进行内存管理的
6. block可以用strong修饰吗
7. 解决循环引用时为什么要用__strong、__weak修饰
8. block发生copy时机
9. Block访问对象类型的auto变量时，在ARC和MRC下有什么区别

### 1. block的内部实现，结构体是什么样的
block一个OC对象，享有所有OC对象的待遇。
它的本质是一个C++结构体，一般名为`__<block名字>_block_impl_0`。
普通OC对象用来封装数据，而block用来封装函数以及函数调用环境。
block实际上是指向结构体的指针。

下面是示例 block：
```Objective-C
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        void (^block)() = ^{
            NSLog(@"block");
        };
        block();
    }
    return 0;
}
```

使用 clang 将其转换为C++代码，下面是 block 具体实现的C++代码：
```c++
#include <iostream>
using namespace std;

/// 封装了 block 函数的实现的信息
struct __block_impl {
    void *isa; //指向block所属的类
    int Flags;
    int Reserved;
    void *FuncPtr; //指向block对应的函数
};

/// block 底层数据结构，block 的本质是一个C++结构体
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    /**
     * block构造函数
     *
     * @param fp block函数对应的内存地址
     * @param desc block的描述信息的结构体的内存地址
     *
     * @return 返回一个当前类型的结构体，即返回一个 __main_block_impl_0 类型的结构体
     */
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
        impl.isa = &_NSConcreteStackBlock; // block 所属的类
        impl.Flags = flags;
        impl.FuncPtr = fp; // block 对应函数的内存地址存储在 block 实现的内部
        Desc = desc; // block 的描述信息的结构体的内存地址也存储在 block 实现的内部
    }
};

/// block 语句块
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_ff_w6_lxpm949b5cjx4l2x594bc0000gn_T_main_9b1e1b_mi_0);
}

/// 描述了 block 描述信息（内存占用）
static struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size; // block 的实际大小
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        // 下面这行声明了block，是一个指向__main_block_impl_0结构体的指针，将__main_block_func_0和__main_block_desc_0_DATA的指针传入，作为参数
        void (*block)() = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    }
    return 0;
}
```

### 2. block是类吗？有哪些类型？
block就是指向结构体的指针。
block有三种类型：全局block、栈block、堆block。他们都继承自NSBlock。
1. 全局block：指存储在全局区的block；
2. 栈block：指存储在栈区的block；
3. 堆block：指存储在堆区的block。

>看一个block是什么类型，不是看他在代码的什么位置定义的，而是看它被系统存储在哪块内存分区中。

1. 全局block：
没有访问外界的普通局部变量的block就是全局block，系统会把这样的block放在全局区。

```Objective-C
// 普通局部变量
// int age = 25;
// 静态全局变量
// static int age = 25;

int main(int argc, const char * argv[]) {
	@autoreleasepool {
		// 静态局部变量
		static int age = 25;

		void (^block)(void) = ^{
			// 访问外界的变量
			NSLog(@"%d", age);
		};

		NSLog(@"%@", [block class]); // __NSGlobalBlock__
		NSLog(@"%@", [[block class] superclass]; // __NSGlobalBlock
		NSLog(@"%@", [[[block class] superclass] superclass)]; // NSBlock
		NSLog(@"%@", [[[[block class] superclass] superclass] superclass]); // NSObject
	}
	return 0;
}
```

2. 栈block：
访问了外界普通局部变量的block就是栈block，系统会把这样的block放在栈区。可见栈block和全局block是完全对立的。
```Objective-C
int main(int argc, const char * argv[]) {
	@autoreleasepool {
		// 普通局部变量
		int age = 25;

		void (^block)(void) = ^{
			// 访问外界的变量
			NSLog(@"%d", age);
		};

		NSLog(@"%@", [block class]); // __NSStackBlock__
		NSLog(@"%@", [[block class] superclass]); // __NSStackBlock
		NSLog(@"%@", [[[block class] superclass] superclass]); // NSBlock
		NSLog(@"%@", [[[[block class] superclass] superclass] superclass]); // NSObject
	}
	return 0;
}
```

### 3. 一个int变量被__block修饰与否的区别？block的变量截获


### 4. block在修改NSMutableArray，需不需要添加__block


### 5. 怎么进行内存管理的
系统把栈block copy到堆区市，也会把栈block内部使用的__block变量复制一份到堆区，并让堆block持有它。并且让栈__block变量的forwarding指针指向堆上面的__block变量。
ARC下系统会在某些情况下自动copy一份栈block到堆区：
	1. block赋值给一个强指针时（即__strong修饰的指针）；
	2. block作为函数的返回值时；
	3. GCD方法里的block，系统都会自动复制到堆区；
	4. Foundation框架usingBlock方法里的block，系统都会自动复制到堆区。


### 6. block可以用strong修饰吗？
不可以，`__strong`只可以用来修饰非block类型。

### 7. 解决循环引用时为什么要用__strong、__weak修饰
`__weak`: 
主要用于解决循环引用，用__weak修饰的变量 当对象释放后，指针自动设置为nil，当后面继续使用该指针变量的时候不会造成crash，更不会造成强引用使该释放的对象无法释放，造成内存泄露。
```Objective-C
__weak typeof(self) weakSelf = self;
```

`__strong`:
相反与__weak,主要用于当使用某个对象是，希望它没有提前被释放。强引用该对象使其无法释放。例如在block内部，希望block调用时该对象不会被提前释放造成错误。可以使用强引用。
```Objective-C
TestAlertView *alertView = [TestAlertView new];
alertView = ^()
{
  //当block内部需要使用本身这个局部对象时，需要用强引用方式，让alertView在传递完block后不会被释放依然可以执行setTitle操作
   __strong typeof(alertView) strongAlertView = alertView;
  [strongAlertView setTitle:@"1234"];

}
[alertView show];
```

### 8. block发生copy的时机
对栈block执行一下copy操作，copy方法返回的就是一个堆block，所以说堆block就是把栈block copy了一份到堆区。
在平常的开发中，用到的总是堆block，而不是栈block。这是因为ARC下，系统会自动复制一份栈block到堆区，而MRC下则需要我们手动调用copy方法复制一份栈block到堆区。
>为什么要把栈block copy到堆区？
>原因：block刚被创建出来时，若不是全局block（没有访问block外部的局部变量）就是栈block（访问了block之外的局部变量）。而栈内存是系统自动管理的，一旦超出变量的作用域，变量对应的内存就会被释放，所以如果不把栈block复制到堆区，就很有可能在调用栈block的时候已经被销毁了，那就是瞎调用了，会导致数据错乱。

| block类型 | 执行copy操作后的效果                |
|---------|-----------------------------|
| 全部block | 什么也不做，不会产生新的block           |
| 栈block  | 复制一份栈block到堆区               |
| 堆block  | 仅仅是block的引用计数加1，不会产生新的block |

### 9. block访问对象类型的auto变量时，在ARC和MRC下有什么区别？
auto变量 随用随开，用完即消。
因此block在捕获auto变量时，只把值传递进去保存。当auto变量销毁的时候，也不影响block内部这个auto变量的使用。