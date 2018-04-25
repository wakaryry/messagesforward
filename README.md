一开始就使用swift而没有使用过objc的同学，可能有点不太理解下面这个东东：

```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
    if ([super methodSignatureForSelector:sel]) {
        return [super methodSignatureForSelector:sel];
    }
    return [super methodSignatureForSelector:@selector(myp_format:)];
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
    NSString *title = [self myp_formattingTitleFromSelector:[invocation selector]];
    
    if (title.length > 0) {
        [self myp_format:title];
    }
    else {
        [super forwardInvocation:invocation];
    }
}
```

上面的代码是什么意思？

**我们一起来理解一下**

首先看官方说明：

`- (void)forwardInvocation:(NSInvocation *)invocation`

> Overridden by subclasses to forward messages to other objects.

> When an object is sent a message for which it has no corresponding method, the runtime system gives the receiver an opportunity to delegate the message to another receiver. It delegates the message by creating an `NSInvocation` object representing the message and sending the receiver a `forwardInvocation:` message containing this `NSInvocation` object as the argument. The receiver’s `forwardInvocation:` method can then choose to forward the message to another object. (If that object can’t respond to the message either, it too will be given a chance to forward it.)

> The `forwardInvocation:` message thus allows an object to establish relationships with other objects that will, for certain messages, act on its behalf. The forwarding object is, in a sense, able to “inherit” some of the characteristics of the object it forwards the message to.

**Important**
> To respond to methods that your object does not itself recognize, you must override `methodSignatureForSelector:` in addition to `forwardInvocation:`. The mechanism for forwarding messages uses information obtained from `methodSignatureForSelector:` to create the `NSInvocation` object to be forwarded. Your overriding method must provide an appropriate method signature for the given selector, either by pre formulating one or by asking another object for one.

> **An implementation of the `forwardInvocation:` method has two tasks:**
To locate an object that can respond to the message encoded in anInvocation. This object need not be the same for all messages.
To send the message to that object using anInvocation. anInvocation will hold the result, and the runtime system will extract and deliver this result to the original sender.

> In the simple case, in which an object forwards messages to just one destination (such as the hypothetical friend instance variable in the example below), a `forwardInvocation:` method could be as simple as this:

Listing 1
```objc
- (void)forwardInvocation:(NSInvocation *)invocation
{
    SEL aSelector = [invocation selector];
 
    if ([friend respondsToSelector:aSelector])
        [invocation invokeWithTarget:friend];
    else
        [super forwardInvocation:invocation];
}
```

> The message that’s forwarded must have a fixed number of arguments; variable numbers of arguments (in the style of `printf()`) are not supported.

> The return value of the forwarded message is returned to the original sender. All types of return values can be delivered to the `sender:` id types, structures, double-precision floating-point numbers.

> Implementations of the `forwardInvocation:` method can do more than just forward messages. `forwardInvocation:` can, for example, be used to consolidate code that responds to a variety of different messages, thus avoiding the necessity of having to write a separate method for each selector. A `forwardInvocation:` method might also involve several other objects in the response to a given message, rather than forward it to just one.

> NSObject’s implementation of `forwardInvocation:` simply invokes the `doesNotRecognizeSelector:` method; it doesn’t forward any messages. Thus, if you choose not to implement `forwardInvocation:`, sending unrecognized messages to objects will raise exceptions.

> para: `anInvocation` - 
The invocation to forward.

`- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel`

> Returns an `NSMethodSignature` object that contains a description of the method identified by a given selector.

> This method is used in the implementation of protocols. This method is also used in situations where an `NSInvocation` object must be created, such as during message forwarding. If your object maintains a delegate or is capable of handling messages that it does not directly implement, you should override this method to return an appropriate method signature.

> para: `aSelector` - 
A `Selector` that identifies the method for which to return the implementation address. When the receiver is an instance, `aSelector` should identify an instance method; when the receiver is a class, it should identify a class method.

> returns:
An `NSMethodSignature` object that contains a description of the method identified by `aSelector`, or `nil` if the method can’t be found.

翻译一下是什么意思了？
`- (void)forwardInvocation:(NSInvocation *)invocation` 就是在子类中重写将消息转发给其他的对象。

当一个对象发出一条消息，但却没有相应的方法的时候，运行时系统将给这个接收者一个把消息代理给其它接收者的机会。

就是通过`NSInvocation`对象来表示这个消息的，通过`forwardInvocation:`来发送这个包含`NSInvocation`对象的消息。

接收者的`forwardInvocation:`继续处理这个消息。

这里我们需要使用`methodSignatureForSelector:`来为转发的消息构建方法签名。

`forwardInvocation:`方法不仅仅是转发消息，还可以用来合并响应各种不同消息的代码，从而避免了必须为每个方法编写单独代码的必要性。它也可能涉及到几个对象，而不是将其转发到一个。

**我们来理解一下**

`class A`：没有`testMethod`这个方法

```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector { 
  if(aSelector == @selector(testMethod)) 
  { 
    return [NSMethodSignature signatureWithObjCTypes:"v@:"]; 
  } 
  return nil; 
} 

- (void)forwardInvocation:(NSInvocation *)anInvocation 
{ 
  if (anInvocation.selector == @selector(testMethod)) 
  { 
    B *b = [[B alloc] init]; 
    C *c = [[C alloc] init]; 
    [anInvocation invokeWithTarget:b]; 
    [anInvocation invokeWithTarget:c]; 
  } 
} 
```

`Class B`

```objc
-(void)testMethod 
{ 
  NSLog(@"I am B"); 
} 
```

`Class C`

```objc
-(void)testMethod 
{ 
  NSLog(@"I am C"); 
} 
```

然后我们运行：
```objc
A *test = [[A alloc] init]; 
[test testMethod];
```

我们发现A中没有`testMethod`这个方法，但是持续却没有报错，而且成功运行了，`B`和`C`中的方法得到了运行。

现在我们应该可以理解了吧。

当一条消息找不到对应的方法的时候，会进行一次消息转发，消息转发在`- (void)forwardInvocation:(NSInvocation *)invocation`定义，你想干什么在这里写就行了。

**那么我们如何将这种代码翻译成swift？**

我们需要知道为什么！

为什么会有这种代码？这样写的作用是什么？为什么要这样写？
