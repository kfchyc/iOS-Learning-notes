# MRC与ARC区别以及五大内存区
>MRC: 手动内存管理
>ARC: 自动内存管理

>栈区，堆区，全局区，常量区，代码区
五大内存区域之外还有 自由存储区也称之五大区域之外区

## 五大内存区
1. 栈区：
	创建临时变量时由编译器自动分配，在不需要的时候自动清除的变量的存储区。
        里面的变量通常是局部变量、函数参数等。在一个进程中，位于用户虚拟地址空间顶部的是用户栈，编译器用它来实现函数的调用。和堆一样，用户栈在程序执行期间可以动态地扩展和收缩。

```Objective-C
@interface TestObject()

@end
@implementation TestObject
- (void)testMethodWithName:(NSString *)name
{
   //方法参数name是一个指针，指向传入的参数指针所指向的对象内存地址。name是在栈中
  //通过打印地址可以看出来，传入参数的对象内存地址与方法参数的对象内存地址是一样的。但是指针地址不一样。
  NSLog(@"name指针地址:%p,name指针指向的对象内存地址:%p",&name,name);


  //*person 是指针变量,在栈中, [Person new]是创建的对象,放在堆中。
  //person指针指向了[Person new]所创建的对象。
  //那么[Person new]所创建的对象的引用计数器就被+1了,此时[Person new]对象的retainCount为1
  Person *person = [Person new];
}
```

2. 堆区：
就是那些由 new alloc 创建的对象所分配的内存块，它们的释放系统不会主动去管，由我们的开发者去告诉系统什么时候释放这块内存(一个对象引用计数为0是系统就会回销毁该内存区域对象)。一般一个 new 就要对应一个 release。在ARC下编译器会自动在合适位置为OC对象添加release操作。会在当前线程Runloop退出或休眠时销毁这些对象，MRC则需程序员手动释放。
堆可以动态地扩展和收缩。

```Objective-C
//alloc是为Person对象分配内存,init是初始化Person对象。本质上跟[Person new]一样。
Person *person = [[Person alloc] init];
```

3. 全局变量和静态变量存储区
全局变量和静态变量被分配到同一块内存中，在以前的 C 语言中，全局变量又分为初始化的和未初始化的（初始化的全局变量和静态变量在一块区域，
未初始化的全局变量与静态变量在相邻的另一块区域，
同时未被初始化的对象存储区可以通过 void* 来访问和操纵，
程序结束后由系统自行释放），在 C++ 里面没有这个区分了，
他们共同占用同一块内存区。

4. 常量存储区
全局变量和静态变量被分配到同一块内存中，在以前的 C 语言中，全局变量又分为初始化的和未初始化的（初始化的全局变量和静态变量在一块区域，
未初始化的全局变量与静态变量在相邻的另一块区域，
同时未被初始化的对象存储区可以通过 void* 来访问和操纵，
程序结束后由系统自行释放），在 C++ 里面没有这个区分了，
他们共同占用同一块内存区。

5. 代码区
存放函数的二进制代码。

6. 自由存储区
就是那些由 malloc 等分配的内存块，他和堆是十分相似的，不过它是用 free 来结束自己的生命的。

```Objective-C
NSString *string1;//string1 这个NSString 类型的指针,未初始化存在于<全局区>的<BBS区>

NSString *string2 = @"1234";//string2 这个NSString类型的指针，已初始化存在与<全局区>的<data数据区>，@“1234”存在与堆区，因为@代表了对象。 

static NSString *string3;//string3 这个NSString 类型的指针存在于<全局区>的<BBS区>

static NSString *string4 = @"1234";//string4 这个NSString类型的指针存在与<全局区>的<data数据区>，@“1234”存在与堆区，因为@代表了对象。stiring2和string4的值地址是一样的

static const NSString *string5 = @"654321";//const 修饰后  string5不能修改值。 其他的与string4一样

- (void)test
{
int  a;//a这个int类型的变量 是存在与<栈区>的
a = 10;//10这个值是存在与 <常量区>的

NSStirng *str；//str这个NSString类型的指针 存在于<栈区>
str = @“1234”；//@“1234”这个@对象存在于 <堆区>

static NSString *str1;//str1这个NSString类型的指针 存在于<全局区>的<BBS区>
static NSString *str2 = @"4321';//str2这个NSString类型的指针 存在于<全局区>的<data区>

NSString *str3;//str3这个NSString类型的指针 存在于<栈区>
str3 = [[NSString alloc]initWithString:@"1234"];//[[NSString alloc]initWithString:@"1234"]这个对象 存在于<堆区>

}
```

## 全局变量与全局静态变量
全局静态变量与全局变量 其实本质上是没有区别的，只是存在修饰区别，一个static让其只能内部使用，一个extern让其可以外部使用。


## MRC与ARC区别
### MRC手动内存管理
引用计数器:在MRC时代，系统判定一个对象是否销毁是根据这个对象的引用计数器来判断的。
1. 每个对象被创建时引用计数都为1
2. 每当对象被其他指针引用时，需要手动使用[obj retain];让该对象引用计数+1。
3. 当指针变量不在使用这个对象的时候，需要手动释放release这个对象。 让其的引用计数-1.
4. 当一个对象的引用计数为0的时候，系统就会销毁这个对象。

```Objective-C
    NSMutableArray *array = [NSMutableArray array];//[NSMutableArray array]创建后引用计数器为1
    NSLog(@"array的对象地址:%p,array的retainCount:%zd",array,[array retainCount]);
    [array release];//调用release后[NSMutableArray array]创建的对象引用计数-1.

     //当程序执行到[array addObject:@"1234"];这里是就会崩溃。因为此时array指针指向的内存地址中没有任何对象，该指针是一个野指针。
     //因为release后[NSMutableArray array]创建的对象引用计数变为了0.系统就会销毁这个内存地址的对象。
    [array addObject:@"1234"];
    NSLog(@"array的对象地址:%p,array的retainCount:%zd",array,[array retainCount]);
    NSLog(@"%@",array);
```

### ARC自动内存管理
