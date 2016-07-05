# auto-weave

<!--- All the badges! --->
[![Maven Central](https://img.shields.io/maven-central/v/com.bnorm.auto.weave/auto-weave.svg?maxAge=7200&style=flat-square)](http://mvnrepository.com/artifact/com.bnorm.auto.weave/auto-weave)
[![Travis](https://img.shields.io/travis/bnorm/auto-weave.svg?maxAge=7200&style=flat-square)](https://travis-ci.org/bnorm/auto-weave)
[![Codecov](https://img.shields.io/codecov/c/github/bnorm/auto-weave.svg?maxAge=7200&style=flat-square)](https://codecov.io/gh/bnorm/auto-weave)
[![Codacy](https://img.shields.io/codacy/ba632994831044a2b3049a5a17e26c45.svg?maxAge=7200&style=flat-square)](https://www.codacy.com/app/bnorm/auto-weave)
[![VersionEye](https://img.shields.io/versioneye/d/user/projects/57489055ce8d0e00360be076.svg?maxAge=7200&style=flat-square)](https://www.versioneye.com/user/projects/57489055ce8d0e00360be076)

AutoWeave generates AOP weaved classes using Java Annotation Processing.

Now available on Maven Central, see maven central badge for the latest version.

```xml
<dependency>
    <groupId>com.bnorm.auto.weave</groupId>
    <artifactId>auto-weave</artifactId>
    <version>${auto-weave.version}</version>
</dependency>
```

It works something like this...

```java
@AutoWeave
public abstract class Target {
    public static Target create() { return new AutoWeave_Target(); }
    @Trace String method() {...}
}

@interface Trace {...}

public class TraceAspect {
    @AutoAdvice(Trace.class)
    public Object around(AroundJoinPoint point) {
        System.out.println("Starting method " + point.method());
        Object result = point.proceed();
        System.out.println("Completed method " + point.method() + " with a result of " + result);
        return result;
    }
}

// Generated by auto-weave
final class AutoWeave_Target extends Target {
    private static final StaticPointcut methodPointcut = StaticPointcut.create("method", Target.class, String.class, Arrays.<Class<?>>asList());
    private final TraceAspect traceAspect = new TraceAspect();
    private final Advice[] methodAdvice = new Advice[] {
    new AroundAdvice() {
        @Override
        public Object around(AroundJoinPoint joinPoint) {
            return traceAspect.around(joinPoint);
        }
    }
    };

    @Override
    @Trace
    public String method() {
        return (String) new Chain(methodAdvice, this, methodPointcut, Arrays.<Object>asList()) {
            @Override
            public Object call() throws Throwable {
                return AutoWeave_Target.super.method();
            }
        }.proceed();
    }
}
```

## Advice

There are 5 different types of Advice available in this library:
 - Before - Called before the method is run
 - Around - Weaved into the call stack
 - After - Called after the method is run regardless of result
 - AfterReturning - Called after the method returns
 - AfterThrowing - Called after the method throws an exception

To create Advice, create a class with a default constructor.  Then add
methods annotated with @AutoAdvice and the following signatures:
 - Before - Returns void with a single parameter: BeforeJoinPoint
 - Around - Returns Object with a single parameter: AroundJoinPoint
 - After - Returns void with a single parameter: AfterJoinPoint
 - AfterReturning - Returns void with a single parameter: AfterReturningJoinPoint
 - AfterThrowing - Returns void with a single parameter: AfterThrowingJoinPoint

Note: for around advice, AroundJoinPoint.proceed() needs to be called
at some point because this will continue the call stack and return the
result of the weaved method.

To control how the advice classes are instantiated, use the @AutoAspect
annotation.  This annotation allows 3 different instantiation stategies:
 - Instance - New class with each new instance of weaved class (member)
 - Class - New class for each weaved class (static member)
 - Singleton - Class must provide an instance of itself

Note: for singleton aspects, use an enum with a single value (Effective
Java - Item 3) or have a single `public static final` field with the same
type as the aspect class.  This instance of the class will be used when
calling advice.

## Android

This type of AOP can be quite useful in Android because of build
flavors.  Place the annotations and @AutoWeave annotation on any
classes you would want to be weaved.  Then always use the AutoWave_*
generated class when an instance is required.  Even if no aspects are
applied to the weaved class, an AutoWeave_* class will always be
generated.

Then, in a build flavor directory - debug for example - add the aspect
classes with the advice methods.  Now the advice will be applied to the
weaved classes in the debug build.

## License

    Copyright 2016 Brian Norman

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
