
## setter, getter
- MRC
```
@property (strong, nonatomic) NSString *name;
```
setter
```
// release 旧对象(旧对象计数器 -1),retain 新对象(新对象计数器 +1),然后指向新对象.
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
-(NSString *)name 
{
    //将实例变量的引用计数加1后,添加自动减1
    //作用,保证调用getter方法取值时可以取到值的同时在完全不需要使用后释放
    return [[_name retain] autorelease];
}
```
dealloc
```
//MRC下 手动释放内存 可重写dealloc但不要调用dealloc  会崩溃
-(void)dealloc 
{
    [_name release];//self.name = nil;更好
    //必须最后调用super dealloc
    [super  dealloc];
}
```
- ARC
```
@property (strong, nonatomic) NSString *name;
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
