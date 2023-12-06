ServletRequestAttributes 和 ThreadLocal  的使用场景

- ServletRequestAttributes 在spring中是封装好了的，在接口请求结束后会自动清理里面的数据，只有在接口请求时才可以使用，在非接口请求时，获取不到。
> `ServletRequestAttributes` 是 Spring 框架中用于封装 HTTP 请求的类，它的优缺点如下：
> 
> 优点：
> 
> 1. 封装了 HTTP 请求的相关信息
> 
> `ServletRequestAttributes` 封装了 HTTP 请求的相关信息，包括请求头、请求参数、请求方法、请求路径等等。这样可以方便地在 Spring 应用中获取和处理这些信息。
> 
> 2. 可以方便地和其他 Spring 组件集成
> 
> 通过 `ServletRequestAttributes`，可以方便地获取当前请求的上下文信息，并将其传递给其他 Spring 组件进行处理。例如，在使用 Spring MVC 框架时，可以自动将请求参数绑定到 Controller 方法的入参中，或者使用 Spring Security 框架来进行身份认证和授权等操作。
> 
> 3. 可以方便地进行单元测试
> 
> 通过模拟 `ServletRequestAttributes` 对象，可以方便地进行单元测试。在单元测试中，可以通过创建模拟的 `ServletRequestAttributes` 对象，来模拟 HTTP 请求的相关信息，从而进行测试。
> 
> 缺点：
> 
> 1. 依赖于 Servlet API
> 
> `ServletRequestAttributes` 是基于 Servlet API 实现的，因此只能在基于 Servlet 的 Web 应用中使用。在非 Servlet 环境中，例如使用 Netty 或 Undertow 等非 Servlet 容器的环境中，无法使用 `ServletRequestAttributes`。
> 
> 2. 可能会增加代码复杂度
> 
> 在某些情况下，使用 `ServletRequestAttributes` 来封装请求上下文信息可能会增加代码的复杂度，例如在使用异步请求或者跨线程调用的场景中，需要注意线程安全问题。
> 
> 总之，`ServletRequestAttributes` 提供了一种方便、统一的方式来封装和处理 HTTP 请求的相关信息，能够简化 Spring 应用的开发和测试，并且可以方便地和其他 Spring 组件集成，提高应用的可维护性和可扩展性。但是，需要注意其依赖于 Servlet API 的限制，以及在某些情况下可能会增加代码复杂度的问题。

- ThreadLocal  需要手动清理里面的数据，如果不清理可能会存在内存泄露的风险。但是无论是在接口请求时，还是非接口请求时都可以使用。目前开源框架都会使用这种方式保存数据
> `ThreadLocal` 是 Java 中的一个线程本地变量，它可以让每个线程独立地访问自己的变量副本，从而避免线程安全问题。`ThreadLocal` 的优缺点如下：
> 
> 优点：
> 
> 1. 线程安全
> 
> `ThreadLocal` 可以避免多线程访问共享资源时的线程安全问题。每个线程都拥有自己的变量副本，互不干扰。
> 
> 2. 高效性
> 
> `ThreadLocal` 可以提高程序的执行效率。由于每个线程都拥有自己的变量副本，在访问变量时不需要进行任何的同步操作，避免了多线程竞争锁的开销，从而提高了程序的执行效率。
> 
> 3. 适用范围广
> 
> `ThreadLocal` 可以适用于多种场景，例如在 Web 应用中存储请求上下文信息，或者在单元测试中模拟多线程并发操作等。
> 
> 缺点：
> 
> 1. 容易引起内存泄漏
> 
> `ThreadLocal` 中存储的变量副本是与线程绑定的，如果线程不及时释放，则可能导致内存泄漏。需要注意在使用 `ThreadLocal` 时，及时清理线程的变量副本，避免内存泄漏问题。
> 
> 2. 可能引起上下文切换开销
> 
> 在多线程并发操作时，由于每个线程都拥有自己的变量副本，可能会导致上下文切换的开销增加，从而影响程序的执行效率。
> 
> 总之，`ThreadLocal` 是 Java 中一种非常有用的线程安全技术，能够解决多线程并发访问共享资源的问题。但是，在使用 `ThreadLocal` 时，需要注意内存泄漏和上下文切换等问题，避免影响程序的执行效率和稳定性。


