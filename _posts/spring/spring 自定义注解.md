---
layout: post
title:  "自定义注解"
date:   2018-04-17
categories: spring
---

# spring 自定义注解

## 代码

LogConfiguration.java

```
@Configuration
public class LogConfiguration extends AbstractPointcutAdvisor implements IntroductionAdvisor,BeanFactoryAware{
    private Advice advice;
    private Pointcut pointcut;
    private BeanFactory beanFactory;

    @PostConstruct
    public void init(){
        Set<Class<? extends Annotation>> annotationTypes=new LinkedHashSet<>(1);
        annotationTypes.add(Log.class);
        this.pointcut=buildPointcut(annotationTypes);
        this.advice=buildAdvice();
        if(this.advice instanceof BeanFactoryAware){
            ((BeanFactoryAware)this.advice).setBeanFactory(beanFactory);
        }
    }

    protected Advice buildAdvice() {
        LogInterceptor interceptor = new LogInterceptor();
        return interceptor;
    }

    protected Pointcut buildPointcut(Set<Class<? extends Annotation>> retryAnnotationTypes) {
        ComposablePointcut result = null;
        for (Class<? extends Annotation> retryAnnotationType : retryAnnotationTypes) {
            Pointcut filter = new AnnotationClassOrMethodPointcut(retryAnnotationType);
            if (result == null) {
                result = new ComposablePointcut(filter);
            }
            else {
                result.union(filter);
            }
        }
        return result;
    }

    private final class AnnotationClassOrMethodPointcut extends StaticMethodMatcherPointcut {

        private final MethodMatcher methodResolver;

        AnnotationClassOrMethodPointcut(Class<? extends Annotation> annotationType) {
            this.methodResolver = new AnnotationMethodMatcher(annotationType);
            setClassFilter(new AnnotationClassOrMethodFilter(annotationType));
        }

        @Override
        public boolean matches(Method method, Class<?> targetClass) {
            return getClassFilter().matches(targetClass) || this.methodResolver.matches(method, targetClass);
        }

    }

    private final class AnnotationClassOrMethodFilter extends AnnotationClassFilter {

        private final AnnotationMethodsResolver methodResolver;

        AnnotationClassOrMethodFilter(Class<? extends Annotation> annotationType) {
            super(annotationType, true);
            this.methodResolver = new AnnotationMethodsResolver(annotationType);
        }

        @Override
        public boolean matches(Class<?> clazz) {
            return super.matches(clazz) || this.methodResolver.hasAnnotatedMethods(clazz);
        }

    }

    private static class AnnotationMethodsResolver {

        private Class<? extends Annotation> annotationType;

        public AnnotationMethodsResolver(Class<? extends Annotation> annotationType) {
            this.annotationType = annotationType;
        }

        public boolean hasAnnotatedMethods(Class<?> clazz) {
            final AtomicBoolean found = new AtomicBoolean(false);
            ReflectionUtils.doWithMethods(clazz,
                    new ReflectionUtils.MethodCallback() {
                        @Override
                        public void doWith(Method method) throws IllegalArgumentException,
                                IllegalAccessException {
                            if (found.get()) {
                                return;
                            }
                            Annotation annotation = AnnotationUtils.findAnnotation(method,
                                    annotationType);
                            if (annotation != null) { found.set(true); }
                        }
                    });
            return found.get();
        }

    }

    @Override
    public ClassFilter getClassFilter() {
        return pointcut.getClassFilter();
    }

    @Override
    public void validateInterfaces() throws IllegalArgumentException {

    }

    @Override
    public Advice getAdvice() {
        return advice;
    }

    @Override
    public Class<?>[] getInterfaces() {
        return new Class<?>[]{Log.class};
    }

    @Override
    public Pointcut getPointcut() {
        return pointcut;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory=beanFactory;
    }
}

```

Log.java

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD,ElementType.TYPE})
@Documented
public @interface Log {

}

```

LogInterceptor.java

```

public class LogInterceptor implements IntroductionInterceptor,BeanFactoryAware{
    private  BeanFactory beanFactory;
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Method method=invocation.getMethod();
        Log log = AnnotationUtils.findAnnotation(method, Log.class);
        if (log == null) {
            log = AnnotationUtils.findAnnotation(method.getDeclaringClass(), Log.class);
        }
        if (log == null) {
            log = findAnnotationOnTarget(invocation.getThis(), method);
        }
        if(log!=null) {
            System.out.println("invoke:" + invocation.getMethod().getName());
        }
        return invocation.proceed();
    }

    private Log findAnnotationOnTarget(Object target, Method method) {
        try {
            Method targetMethod = target.getClass().getMethod(method.getName(), method.getParameterTypes());
            return AnnotationUtils.findAnnotation(targetMethod, Log.class);
        }
        catch (Exception e) {
            return null;
        }
    }

    @Override
    public boolean implementsInterface(Class<?> intf) {
        return Log.class.isAssignableFrom(intf);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory=beanFactory;
    }
}


```