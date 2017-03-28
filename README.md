# Block整理
> 对象与对象之间的通信方式:
- Protocol-delegate （一对一）
- NSNotification （一对多）
- Block （一对一）

>三种通信方式都实现了对象之间的解耦合。


[TOC]

## 定义
Block作为C语言的扩展，并不是高新技术，和其他语言的闭包或lambda表达式是一回事。iOS4.0系统已开始支持block，在编程过程中，blocks被Obj-C看成是对象，它封装了一段代码，这段代码可以在任何时候执行。Blocks可以作为函数参数或者函数的返回值，而其本身又可以带输入参数或返回值。
Block的使用很像函数指针，不过与函数最大的不同是：**Block可以访问函数以外、词法作用域以内的外部变量的值。换句话说，Block不仅 实现函数的功能，还能携带函数的执行环境。**
```
// 声明和实现写在一起，就像变量的声明实现 int a = 10;
int (^aBlock)(int, int) = ^(int num1, int num2) {
	return num1 * num2;
};
```
```
// 声明和实现分开，就像变量先声明后实现 int a; a = 10;
int (^cBlock)(int,int);
cBlock = ^(int num1,int num2){
	return num1 * num2;
};
```
其中，定义了一个名字为aBlock的blocks对象，并携带了相关信息：

　　1、aBlock 有两个形式参数，分别为int类型；

　　2、aBlock 的返回值为int 类型；

　　3、等式右边就是blocks的具体实现；

　　4、^ 带边blocks声明和实现的标示（关键字）；

当然，你可以定义其他形式的block。e.g:无返回值，无形式参数等；
```
void (^bBlock)() = ^(){
	int a = 10;
	printf(num = %d,a);
	};
```
## 类型
> 一个由C/C++编译的程序占用的内存分为以下几个部分
* **栈**（stack）:由编译器自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。
* **堆**（heap）: 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收 。注意它与数据结构中的堆是两回事，分配方式类似于链表。
* **全局区**（静态区static）:全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域，程序结束后由系统释放。
* **常量区**：常量字符串放在这里， 程序结束后由系统释放
* **程序代码区**：存放函数体的二进制代码。
		
	根据isa指针，block一共有3种类型的block：
- _NSConcreteGlobalBlock 全局静态 类似函数，位于text段；
- _NSConcreteStackBlock 保存在栈中，出函数作用域就销毁
- _NSConcreteMallocBlock 保存在堆中，retainCount == 0销毁

而ARC和MRC中，还略有不同:
1. 不管在ARC还是MRC环境下，block内部如果没有访问外部变量，这个block是全局block__NSGlobalBlock__，形式类似函数，存储在内存中的代码区。
2. 在MRC下，block内部如果访问外部变量，这个block是栈block__NSStackBlock__，存储在内存中的栈上。
3. 在MRC下，block内部访问外部变量，同时对该block做一次copy操作，这个block是堆block__NSMallocBlock__，存储在内存中的堆上。
4. 在ARC下，block内部如果访问外部变量，这个block是堆block__NSMallocBlock__，存储在内存中的堆上，因为在ARC下，默认对block做了一次copy操作。
## 用途
### 作为方法的参数

    (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
	    [self blockAsPara:^{
		    NSLog("block作为参数");
	    }];
	}
	-(void)blockAsPara:(void(^)())block{
		block(); //执行block内部的代码
	}
    
### 作为属性

    @property (nonatomic,copy) void (^myBlock)();//block作为属性
    -（void）viewDidLoad{
	   [super viewDidLoad];
	   [self setMyBlock:^{
		   NSLog("block作为属性");
	   }];
	   NSLog("%@",self.myBlock);
	   self.myBlock()；//myBlock作为属性时的调用
    }

## 本质
Block 的本质是可以截取自动变量的匿名函数。
__匿名函数__：顾名思义就是不带名字的函数，在C语言中不允许这样的方法存在，至少要知道函数的名字来对函数进行引用，而在OC中的Block则可以用指针来直接调用一个函数，但虽说如此我们还是需要知道指针的名称。
__截取自动变量__：

    int b = 0; 
	void (^blo)() = ^{ 
	   NSLog(@"Input:b=%d",b); 
	}; 
	b = 3; 
	blo(); 
	/* Output:b=0 */
虽然我们在调用blo之前改变了b的值，但是输出的还是Block编译时候b的值，所以截获瞬间自动变量就是：在Block中会保存变量的值，而不会随变量的值的改变而改变。

## Block 为什么能实现神奇的回调?
为什么在 A 中的 block 块能调用到 B 中的数据？

回顾一下我们在 B 中所实现的代码，不外乎定义了一个 Block 变量，并在适当的时候传入参数，那么为什么在调用了 self.callBackBlock(_textField.text) 之后，值就神奇传到了 A 中的 Block 块了呢？

事实是，通过简单的整理我们可以发现完整的回调流程应该是这样的：


回调流程
1. block 代码块赋值给 bVC.callBackBlock，此时 callBackBlock 的指针就指向这个代码块。
2. 调用 callBackBlock(NSString *text)
3. 由于 callBackBlock 的指针是指向 A 中的 block 代码块，因此执行代码块的代码，实现回调。

## Block的修饰
**ARC情况下**：

- 如果用copy修饰Block，该Block就会存储在堆空间。则会对Block的内部对象进行强引用，导致循环引用。内存无法释放。
解决方法：
新建一个指针(__weak typeof(Target) weakTarget = Target )指向Block代码块里的对象，然后用weakTarget进行操作。就可以解决循环引用问题。

- 如果用weak修饰Block，该Block就会存放在栈空间。不会出现循环引用问题。		
-  情形一：如果block没有访问外部的局部变量，或者访问的外部局部变量被static修饰过，那么block默认存在于全局区(Global)。可以weak，可以strong。此后的block依然存在于全局区中。
* 情形二：如果block访问外部的局部变量，那么block存放在堆(Malloc)中。此时，如果block声明为成员变量，不能使用weak，因为此时没有强指针指向,会自动销毁。strong后重新调用的block存在于堆(Malloc)中。
* 所以，不要用weak修饰ARC下的block

**MRC情况下**：

- 用copy修饰后，如果要在Block内部使用对象，则需要进行

```
(__block typeof(Target) blockTarget = Target )
```

  处理。在Block里面用blockTarget进行操作。

* 情形一：如果block没有访问外部的局部变量，或者访问的外部局部变量被static修饰过，那么block默认存在于全局区(Global)。可以retain，可以copy。copy后依然存在全局区中。
* 情形二：如果block访问外部的局部变量，那么block存放在栈(Stack)中。此时，如果block声明为成员变量，不能使用retain，因为此时依然放在栈里面,会自动销毁。需要用copy声明。此种情况下copy后的block，放在堆(Malloc)里面。

MRC小常识:
1.block是不是对象?是的。在文档working with blocks中可以看到。 

2.MRC开发习惯:访问属性或者设置属性,必须使用点语法,不要使用下划线.

3.MRC没有强指针,默认一个对象就是基本类型

## Block的使用注意
**block访问外部变量**
1. block内部可以访问外部的变量，block默认是将其复制到其数据结构中来实现访问的。
2. 默认情况下，block内部不能修改外面的局部变量，因为通过block进行闭包的变量是const的。
3. 给局部变量加上__block关键字，这个局部变量就可以在block内部修改，block是复制其引用地址来实现访问的。

**block作为属性应该用copy修饰**
1. 当用weak、assign修饰block属性时，block访问外部变量，此时block的类型是栈block。保存在栈中的block，当block所在函数\方法返回\结束，该block就会被销毁。在其他方法内部调用访问该block，就会引发野指针错误EXC_BAD_ACCESS。
2. 当用copy、strong修饰block属性时，block访问外部变量，此时block的类型是堆block。保存在堆中的block，当引用计数器为0时被销毁，该类型block是由栈类型的block从栈中复制到堆中形成的，因此可以在其他方法内部调用该block。在ARC下，strong和copy都可以用来修饰block，但是建议修饰block属性使用copy。

**block为空**
代码存放在堆区时，就需要特别注意，因为堆区不像代码区不变化，堆区是不断变化的（不断创建销毁）。因此代码有可能会被销毁（当没有强指针指向时），如果这时再访问此段代码则会程序崩溃。因此，对于这种情况，我们在定义一个block属性时应指定为strong，或copy:

```
property (nonatomic, strong) void (^myBlock)(void); 
// 这样就有强指针指向它
@property (nonatomic, copy) void (^myBlock)(void); 
// 并不会在堆区copy一份，原因见四
```

而对于block代码存在代码区，使用strong，copy(不会复制一份到堆区)也可以。因此定义block时最好指定为strong(推荐)或copy。我们在使用时最后判断下block是否为空，例如：

```
- (void)blockTest {
    // 如果为空则返回
    if (!block) {
        NSLog(@"block is nil");
        return;
    }
    block();
}
```
**当不再使用指向block的指针时，将其置空**
当有类对象的成员变量pBlock指向block时，一方面是调用方，调用pBlock调用完成后，应将pBlock置为nil;另一方面是被调用方即block函数内部使用到self时要__weak声明。其实__weak声明有很多注意事项，下面是一个经典例子（是正确的写法）：

	// 弱声明，防止block强引用self，造成循环引用     
	__weak __typeof(self) weakSelf = self;     
	self.observer = [[NSNotificationCenter defaultCenter]addObserverForName:@"blockTest" object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {        
	// 多线程情况下（假设发出通知的代码在另一线程下），strong强引用防止后面调用strongSelf时：前面的strongSelf正常，后面的strongSelf已在其它线程被释放，造成很奇怪的结果，虽然这种情况很少发生        
	 __strong __typeof(self) strongSelf = weakSelf;         
	 //if (strongSelf == nil) {         
	 //    return;         
	 //}         
	 // 下面再对strongSelf进行访问         
	 // 防止block为空         
	 if (!strongSelf.block) {             
	 return;         
	 }         
	 strongSelf.block();         // 如果不用应置空，养成好习惯        
	 strongSelf.block = nil;         
	 NSLog(@"%@",strongSelf);     
	 }];

1）我们都知道在使用通知中心时，应在dealloc函数中释放通知，如果上面没有使用__weak声明，那么：通知中心持有self.observer，observer又强引用 usingBlock，usingBlock又强引用self，self就不会被释放，那么dealloc就不会被调用(即使在dealloc中写了[[NSNotificationCenter defaultCenter] removeObserver:self.observer]也不会调用，因为dealloc没有被调用)，就造成内存泄露；


2）另外，我们在第5行看到又使用了__strong声明，是否瞬间凌乱？下面给出解释：在多线程情况下，有可能在usingBlock调用时，执行if (!strongSelf.block)时strongSelf还没有释放，而执行到strongSelf.block()的时候strongSelf就被释放(现在没有强引用了，又开始担心self被释放，真是操碎了心。。。)，造成调用失败（最大的问题是不统一，造成不可预知的错误。用__strong操作后保证要么都访问成功，要么都访问失败或者判断为空后直接return退出）。

而使用了__strong声明后：

如果执行usingBlock时self已经被释放则后面的strongSelf均为nil，因为对weakSelf引用计数为0再retain一次也不会有变化；

如果执行usingBlock时self没有释放，则strongSelf会使self引用计数+1，那么self在其它线程被release -1也不会有影响，只有到usingBlock全部执行完毕后，strongSelf释放，然后self引用计数-1，self才会释放（weak–strong dance）。

上面的例子是通知中心可能造成的内存泄露，而使用block还经常出现循环引用，如下：

    @interface BlockViewController () 
    @property (nonatomic, strong) void (^block)(void); 
    @property (nonatomic, copy) NSString *str; 
    @end 
    @implementation BlockViewController 
    - (void)viewDidLoad {     
		[super viewDidLoad];     
			self.block = ^{         
		        self.str = @"123";    
		    }; 
        } 
    @end
上面的代码，self.block强引用block，而block中又使用了self.str，所以block强引用self，造成强引用，解决方法使用2中所说即可。

- 1、在使用block前需要对block指针做判空处理。不判空直接使用，一旦指针为空直接产生崩溃。
- 2、在MRC的编译环境下，block如果作为成员参数要copy一下将栈上的block拷贝到堆上。
- 3、在block使用之后要对，block指针做赋空值处理，如果是MRC的编译环境下，要先release掉block对象。block作为类对象的成员变量，使用block的人有可能用类对象参与block中的运算而产生循环引用。将block赋值为空，是解掉循环引用的重要方法。（不能只在dealloc里面做赋空值操作，这样已经产生的循环引用不会被破坏掉）。
- 4、使用方将self或成员变量加入block之前要先将self变为__weak。
- 5、在多线程环境下（block中的weakSelf有可能被析构的情况下），需要先将self转为strong指针，避免在运行到某个关键步骤时self对象被析构。

**block指定为copy后是否会拷贝一份呢？（或者说是浅拷贝还是深拷贝）**
- copy可变变量：在赋值指针的同时也会复制指针指向的内存区域。深拷贝，例如NSMutableString对象。

- copy不可变变量：等同于strong，还是浅拷贝，例如NSString对象。

因为block是一段代码，即不可变的，所以并不会深拷贝。

**循环引用**
对于非ARC下, 为了防止循环引用, 我们使用__block来修饰在Block中使用的对象:
对于ARC下, 为了防止循环引用, 我们使用_weak来修饰在Block中使用的对象。原理就是:ARC中，Block中如果引用了_strong修饰符的自动变量，则相当于Block对该变量的引用计数+1。

**weakSelf和strongSelf**
需要注意的是由于Objective-C在iOS中不支持GC机制，使用Block必须自己管理内存，而内存管理正是使用Block坑最多的地方，错误的内存管理 要么导致return cycle内存泄漏要么内存被提前释放导致crash。
现在我们用 Objective-C 写代码时已经越来越多地用到了block，相比delegate的回调方式，block更直观易用。相信每个使用过block的人都遇到过block中使用self时需要weakself的情况，以下就是非常典型的一段代码：

```
__weak __typeof(self)weakSelf = self;
[self.context performBlock:^{
    __strong __typeof(weakSelf)strongSelf = weakSelf;
    [strongSelf doSomething];
}];
```
```
[UIView animateWithDuration:0.2 animations:^{
        self.myView.alpha=1.0;
}];
```
在ARC环境下的，每个block在创建时，编译器会对里面用到的所有对象自动增加一个reference count，然后当block执行完毕时，再释放这些reference。针对上面的代码，在animations block执行期间，self（假设这里的self是个view controller）的引用数会被加1，执行完后再次减1。但这种情况下为什么我们一般不会去weakify self呢？因为这个block的生命周期是明确可知的，在这个block执行期间当前的view controller一般是不会被销毁的，所以不存在什么风险。

**weakSelf**

```
NSBlockOperation *op = [[[NSBlockOperation alloc] init] autorelease]; 
    [op addExecutionBlock:^ { 
	    [self doSomething]; 
	    [self doMoreThing]; 
    }];
```
在这种情况下，我们并不知道这个execution block什么时候会执行完毕，所以很有可能发生的情况是，我在block还没执行完毕时就想销毁当前对象（比方说用户关闭了当前页面），这时就会因为block还对self有一个reference而没有立即销毁，这会引起很多问题，比方说你写在 - (void)dealloc {} 中的代码并不能马上得到执行。所以为了避免这种情况，我们会在block前加上 __weak __typeof(self)weakSelf = self; 的定义来使block对self获取一个弱引用（也就是refrence count不会加1）。

**strongSelf**
那block中的StrongSelf又是做什么的呢？还是上面的例子，当你加了WeakSelf后，block中的self随时都会有被释放的可能，所以会出现一种情况，在调用doSomething的时候self还存在，在doMoreThing的时候self就变成nil了，所以为了避免这种情况发生，我们会重新strongify self。一般情况下，我们都建议这么做，这没什么风险，除非你不关心self在执行过程中变成nil，或者你确定它不会变成nil（比方说所以block都在main thread执行）。
