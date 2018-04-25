```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
    if ([super methodSignatureForSelector:sel]) {
        return [super methodSignatureForSelector:sel];
    }
    return [super methodSignatureForSelector:@selector(slk_format:)];
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
    NSString *title = [self slk_formattingTitleFromSelector:[invocation selector]];
    
    if (title.length > 0) {
        [self slk_format:title];
    }
    else {
        [super forwardInvocation:invocation];
    }
}
```

我们在objc中会看到`NSMethodSignature`和`NSInvocation`的使用，比如上面的消息转发，还有运行时的类型检查，等。

上面的代码是什么意思？

**我们一起来理解一下**

首先看官方说明：

`- (void)forwardInvocation:(NSInvocation *)invocation`

> Overridden by subclasses to forward messages to other objects.
When an object is sent a message for which it has no corresponding method, the runtime system gives the receiver an opportunity to delegate the message to another receiver. It delegates the message by creating an `NSInvocation` object representing the message and sending the receiver a `forwardInvocation:` message containing this `NSInvocation` object as the argument. The receiver’s `forwardInvocation:` method can then choose to forward the message to another object. (If that object can’t respond to the message either, it too will be given a chance to forward it.)
The `forwardInvocation:` message thus allows an object to establish relationships with other objects that will, for certain messages, act on its behalf. The forwarding object is, in a sense, able to “inherit” some of the characteristics of the object it forwards the message to.

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
The return value of the forwarded message is returned to the original sender. All types of return values can be delivered to the sender: id types, structures, double-precision floating-point numbers.
Implementations of the forwardInvocation: method can do more than just forward messages. forwardInvocation: can, for example, be used to consolidate code that responds to a variety of different messages, thus avoiding the necessity of having to write a separate method for each selector. A forwardInvocation: method might also involve several other objects in the response to a given message, rather than forward it to just one.
NSObject’s implementation of forwardInvocation: simply invokes the doesNotRecognizeSelector: method; it doesn’t forward any messages. Thus, if you choose not to implement forwardInvocation:, sending unrecognized messages to objects will raise exceptions.
anInvocation
The invocation to forward.

`- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel`

Returns an NSMethodSignature object that contains a description of the method identified by a given selector.
This method is used in the implementation of protocols. This method is also used in situations where an NSInvocation object must be created, such as during message forwarding. If your object maintains a delegate or is capable of handling messages that it does not directly implement, you should override this method to return an appropriate method signature.
aSelector
A Selector that identifies the method for which to return the implementation address. When the receiver is an instance, aSelector should identify an instance method; when the receiver is a class, it should identify a class method.
returns:
An NSMethodSignature object that contains a description of the method identified by aSelector, or nil if the method can’t be found.
