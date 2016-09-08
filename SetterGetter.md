## assign,retain,weak,strong,copy
- assign (ARC/MRC)
1.这个修饰词是直接赋值的意思,整型/浮点型等数据类型都用这个词修饰
2.如果没有使用 weak strong retain copy 修饰,那么默认就是使用assign了(它们之间是有你没我的关系)
3.当然其实对象也可以用 assign 修饰,只是对象的计数器不会+1.(与 strong 的区别)
4.如果用来修饰对象属性,那么当对象被销毁后指针是不会指向 nil 的.所以会出现野指针错误.(与weak的区别)

- weak (ARC)
1.弱指针是针对对象的修饰词,就是说它不能修饰基本数据类型.
2.weak 修饰的对象计数器不会+1,也就是直接赋值.
3.弱引用是为打破循环引用而生的.
4.它最被人所喜欢的原因是 它所指向的对象如果被销毁,它会指向 nil.而nil访问什么鬼都不会报野指针错误.

- strong(ARC)
1.直接赋值并且计数器 +1.
2.在ARC里替代了retain的作用.

- retain (MRC)
1.release旧对象(旧对象计数器 -1),retain新对象(新对象计数器 +1),然后指向新对象.


- copy (ARC/MRC)
1.copy在MRC时是这样做的release旧对象(旧对象计数器 -1),copy新对象(新对象计数器 +1),然后指向新对象.（新对象是指最终指向的那个对象，不管深拷贝还是浅拷贝）
在set方法里面是这样的:
```
  if (_dog) {
      [_dog release];
  }
  _dog = [dog copy];
```
2.copy在ARC时是这么干的 copy 新对象(新对象计数器 +1),然后指向新对象.
在set方法里面是这样的:
```
  _dog = [dog copy];
```
3.使用注意:修饰的属性本身要不可变的.例如 NSMutableArray 采用 copy 修饰,添加元素表面上可以 一到运行就崩溃了, 因为 copy 过后实际上成了NSArray了.
遵守 NSCopying 协议的对象使用.

- nonatomic (ARC/MRC)
1.不对set方法加锁
2.性能好
3.线程不安全

- atomic (ARC/MRC)
1.原子属性就是对生成的 set 方法加互斥锁 @synchronized(锁对象).
```
@synchronized(self) {
  _delegate = delegate;
}
```
2.需要消耗系统资源.
3.互斥锁是利用线程同步实现的,意在保证同一时间只有一个线程调用 set 方法.
4.其实还有 get 方法,要是同时 set 和 get 一起调用还是会有问题的.所以即使用了 atomic 修饰 还是不够安全.

- readonly
1.让 Xcode 只生成get方法.
2.不想把暴露的属性被人随便替换时,可以使用.

- readwrite
1.让 Xcode 生成get/set方法.
2.不用 readonly 修饰时,默认就是 readwrite.

- getter/setter 的自定义方法名
1.一般对于 有/无 是/否 等这样的属性,getter 方法名前面加个 is 会显得通俗易懂.
```
@property (nonatomic, getter = isFinish, setter = setFinish) BOOL finish;
```





## setter, getter
- MRC
```
@property (strong,nonatomic) NSString *name;
```
setter
```
// release 旧对象( 旧对象计数器 -1 ) , retain 新对象( 新对象计数器 +1 ) , 然后指向新对象.
  - (void)setName:(NSString *)name
{
    if (_name != name) {
        [_name release];
        [name retain];
        _name = name;
    }
}
```
```
//假设有一个实例变量value，它指向了一个内存对象，这个内存对象的引用计数是1吧。那么对这个value重新赋值的时候，value就会指向另一个新内存对象了，所以新内存对象需要将引用计数+1，旧的内存对象当然要释放一次拉，因为该类已经不再拥有这块内存的拥有权。
  - (void)setName:(NSString *)name
{
    //如果实例变量指向的地址和参数指向的地址不同
    if (_name != name) {
        //将实例变量的引用计数减一
        [_name release];
        //将参数变量的引用计数加一,并赋值给实例变量
        _name = [name retain];
    }
}
```
getter
```
//用@property声明的成员属性,相当于自动生成了setter getter方法,重写了set和get方法,与@property声明的成员属性就不是一个成员属性了,是另外一个实例变量,而这个实例变量需要手动声明
@synthesize name = _name;
-(NSString *)name{
    //将实例变量的引用计数加1后,添加自动减1
    //作用,保证调用getter方法取值时可以取到值的同时在完全不需要使用后释放
    return [[_name retain] autorelease];
}
```
dealloc
```
//MRC下 手动释放内存 可重写dealloc但不要调用dealloc  会崩溃
-(void)dealloc{
    [_name release];//self.name = nil;更好
    //必须最后调用super dealloc
    [super  dealloc];
}
```
- ARC
```
@property (strong,nonatomic) NSString *name;
```
setter

  ```
  - (void)setName:(NSString *)name
  {
    if (_name != name) {
        _name = name;
    }
  }

  ```
getter
  ```
  @synthesize name = _name;
  - (NSString *)name
  {
      return _name;
  }
  ```
