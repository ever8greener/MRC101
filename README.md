# MRC101


Autorelease 메모리관련 정리 ( NSAutoreleasePool ) - run by Xocde 7.2 

사전준비사항.
*빈프로젝트 하나만들고나서*  RootViewController.m 에서 아래처럼 AutoreleasePool Test 위한 클래스 하나 정의한다.
```objc
#pragma mark - Autorelease pool test
@interface AutoreleasePoolTester : NSObject{
}
@end

@implementation AutoreleasePoolTester
-(void)dealloc{
    [super dealloc];
    NSLog(@"dealloc 수행");
}
@end
```

시험시작!!
autorelease 를 nesting 해서 구현하는 예제
```OBJC
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {

NSAutoreleasePool* pool = [[NSAutoreleasePool alloc]init];
AutoreleasePoolTester *test = [[AutoreleasePoolTester alloc]init];
[test autorelease];
[pool release];

// 이렇게 하면  임시로 pool을 먼저 비울 수 있다.
// 모든 변수의 릴리즈가 한꺼번에 처리되려면 시간이 걸리기 때문에
// 우선적으로 대량으로  auto release 가 필요하다면 이런식으로 하면 된다.
// nest 형식으로 autorelease 를 구현하면 됨.( ios debug and 최적화기법,p99 )

```

아래의 경우는 해제되지 않는다. 
```objc
NSAutoreleasePool* pool = [[NSAutoreleasePool alloc]init];
AutoreleasePoolTester *test = [[AutoreleasePoolTester alloc]init];
[test autorelease];

[test retain];
 
[pool release];
```
왜냐하면 retain 을 내부에 한번 써주었기 때문이다.

아래처럼 하면 어떻게 되나?
```objc
NSAutoreleasePool* pool = [[NSAutoreleasePool alloc]init];
AutoreleasePoolTester *test = [[AutoreleasePoolTester alloc]init];
[test autorelease];

[test release];
[test retain];

[pool release];
```
EXC_BAD ACCESS 오류나면서 crash 됨.

그럼 아래는?
```objc
NSAutoreleasePool* pool = [[NSAutoreleasePool alloc]init];
AutoreleasePoolTester *test = [[AutoreleasePoolTester alloc]init];
[test autorelease];

[test retain];
[test release];

[pool release];
```

retain 을하고 release 하면 아무런 이상없음. +1,  -1 이므로 더하기빼기해서 변화가 없는 셈이다.

### 두번째 예제위한 함수
```objc
//-----------------------------
//      썸 클래스
//-----------------------------
#pragma mark - 썸 클래스
@interface Sumsum : NSObject{
    int sum;
    NSString* name;
}
@property (retain) NSString* name;
- (int)sum;
- (void)setSum:(int)number;
@end


@implementation Sumsum
@synthesize name;

- (int)sum{
    return sum;
}
- (void)setSum:(int)number{
    sum = number;
}
@end
```

