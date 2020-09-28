# 二、JavaWeb

## 1、Tomcat

## 2、Servlet 和 Http 请求协议

## 3、request 和 response

## 4、Cookie 和 Session

## 5、Filter 和 Listener

### Filter: 过滤器

#### 1、概念

* 生活中的过滤器： 净水器，空气净化器，土匪
* web中的过滤器： 当访问服务器资源时，过滤器可以将请求拦截下来，完成一些特殊的功能
* 过滤器的作用 ：
  * 一般用于完成通过的操作。如：登录验证、统一编码处理、敏感字符过滤...

#### 2、快速入门

 1. 配置：

     	1. 定义一个类，实现接口 Filter
          	2. 复写方法
               	3. 配置拦截路径
                  	1. web.xml
                            	2. 注解

    ```java
    @WebFilter("/*") // 访问所有资源之前，都会执行该过滤器
    public class FilterDemo1 implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
    
        }
    
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            System.out.println("FilterDemo1 被执行了000");
            // 放行
            filterChain.doFilter(servletRequest, servletResponse);
        }
    
        @Override
        public void destroy() {
    
        }
    }
    ```

    
       3. 过滤器细节:
    
    
             1. web.xml 配置
    
             2. 过滤器执行流程
    
             3. 过滤器生命周期方法
    
    
                             	1. init
                             	2. doFilter
                             	3. destroy
    
             4. 过滤器配置详解
    
    
                   1. 拦截路径配置
    
    
                                   	1. 具体资源路径： /index.jsp 只有访问 index.jsp资源时
                                   	2. 拦截目录： /user/* 访问 /user 下的所有资源时
                                   	3. 后缀名拦截： *.jsp 访问所有后缀名为 jsp 资源
                                   	4. 拦截所有资源： /*
    
                   2. 拦截方式配置：配置访问方式
    
    
                         1. 注解配置
    
    
                               - 设置 dispatcherTypes 属性
    
    
                                    - REQUEST：默认值，浏览器直接请求
                                    - FORWARD： 转发访问资源
                                    - INCLUDE：包含访问资源
                                    - ERROR：错误跳转资源
                                    - ASYNC：异步访问资源
    
                                 例：可以多种方式@WebFilter(value="/index.jsp", dispatcherTypes = { DispatcherType.FORWARD, DispatcherType.REQUEST })
    
                                	2. web.xml配置
    
             5. 过滤器链（配置多个过滤器）
    
                执行顺序：如何有两个过滤器：过滤器1和过滤器2
    
    
                           	1. 过滤器1
                           	2. 过滤器2
                           	3. 资源执行
                           	4. 过滤器2
                           	5. 过滤器1
    
                过滤器先后顺序问题
    
                 1. 注解配置：按照类名的字符串比较规则比较，值小的先执行
    
                    如： AFilter 和 BFilter, AFilter 就先执行了
    
                 2. web.xml配置： <filter-mapping>谁定义在上边，谁先执行

