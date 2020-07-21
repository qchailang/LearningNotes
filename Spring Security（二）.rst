
https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/

FilterInvocationSecurityMetadataSource (DefaultFilterInvocationSecurityMetadataSource)

SecurityMetadataSource

SecurityMetadataSource是一个接口，同时还有一个接口FilterInvocationSecurityMetadataSource继承于它，但FilterInvocationSecurityMetadataSource只是一个标识接口，对应于FilterInvocation，本身并无任何内容：

因为我们做的一般都是web项目，所以实际需要实现的接口是FilterInvocationSecurityMetadataSource，这是因为Spring Security中很多web才使用的类参数类型都是FilterInvocationSecurityMetadataSource。

getAttributes方法返回本次访问需要的权限，可以有多个权限。在上面的实现中如果没有匹配的url直接返回null，也就是没有配置权限的url默认都为白名单，想要换成默认是黑名单只要修改这里即可。

getAllConfigAttributes方法如果返回了所有定义的权限资源，Spring Security会在启动时校验每个ConfigAttribute是否配置正确，不需要校验直接返回null。

supports方法返回类对象是否支持校验，web项目一般使用FilterInvocation来判断，或者直接返回true。

AccessDecisionManager
decide方法的三个参数中
authentication包含了当前的用户信息，包括拥有的权限。这里的权限来源就是前面登录时UserDetailsService中设置的authorities。
object就是FilterInvocation对象，可以得到request等web资源。
configAttributes是本次访问需要的权限。

SpringSecurity的Java配置方式，没有对外暴露出其是如何配置每一个对象的每一个属性。对于大部分用户来说，这简化了他们的配置工作，
虽不对外直接暴露属性有很多理由，但是用户可能需要更高级的配置方式。为了处理这个问题，SpringSecurity引入了 ObjectPostProcessor 的概念，其可以用来修改或者替代通过Java配置方式创建的对象实例，让用户实现更多想要的高级配置。

AbstractAuthenticationProcessingFilter
用于用户表单认证，内部调用了authenticationManager完成认证，根据认证结果执行successfulAuthentication或者unsuccessfulAuthentication，无论成功失败，一般的实现都是转发或者重定向等处理。
AuthenticationManager
AuthenticationManager是一个用来处理认证（Authentication）请求的接口。在其中只定义了一个方法authenticate()，该方法只接收一个代表认证请求的Authentication对象作为参数，如果认证成功，则会返回一个封装了当前用户权限等信息的Authentication对象进行返回。
AuthenticationManager的默认实现是ProviderManager，而且它不直接自己处理认证请求，而是委托给其所配置的AuthenticationProvider列表，然后会依次使用每一个AuthenticationProvider进行认证，如果有一个AuthenticationProvider认证后的结果不为null，则表示该AuthenticationProvider已经认证成功，之后的AuthenticationProvider将不再继续认证。然后直接以该AuthenticationProvider的认证结果作为ProviderManager的认证结果。如果所有的AuthenticationProvider的认证结果都为null，则表示认证失败，将抛出一个ProviderNotFoundException。
校验认证请求最常用的方法是根据请求的用户名加载对应的UserDetails，然后比对UserDetails的密码与认证请求的密码是否一致，一致则表示认证通过。
Spring Security内部的DaoAuthenticationProvider就是使用的这种方式。其内部使用UserDetailsService来负责加载UserDetails。在认证成功以后会使用加载的UserDetails来封装要返回的Authentication对象，加载的UserDetails对象是包含用户权限等信息的。认证成功返回的Authentication对象将会保存在当前的SecurityContext中。
UserDetailsService
UserDetailsService只定义了一个方法 loadUserByUsername，根据用户名可以查到用户并返回的方法
AbstractSecurityInterceptor
访问url时，会被AbstractSecurityInterceptor拦截器拦截，然后调用FilterInvocationSecurityMetadataSource的方法来获取被拦截url所需的全部权限，再调用授权管理器AccessDecisionManager鉴权。
FilterInvocationSecurityMetadataSource 获取所需权限
AccessDecisionManager 鉴权

相关术语、核心类、接口功能说明
============
* secure object 安全对象 指一切可以加上安全配置的对象，最常见的例子是方法调用和web请求，如：一个URL地址。
* UserDetails 包含了构建Authentication对象需要的必要信息，这些信息来自于应用的DAO或其他数据源。
* UserDetailsService 根据传入用户名字符串构建一个UserDetails对象。
* Authentication 表示一个认证的用户信息，封装了当前用户所有的细节信息，包括角色、拥有的权限等。
* SecurityContextHolder 提供对SecurityContext的访问，默认使用ThreadLocal存储当前的安全上下文SecurityContext，因此在同一个线程内调用的方法都是可用的。相当于一个线程内的全局对象。
* SecurityContext 提供了Authentication以及其他一些请求相关的安全信息。
* GrantedAuthority接口 表示授予用户的权限信息。实现类：SimpleGrantedAuthority。Authentication的getAuthorities()方法返回GrantedAuthority的对象数组。GrantAuthority对象一般在UserDetailsService中加载。
* ConfigAttribute接口 配置属性（Configuration Attributes）。配置属性只是一个用来表示具有特殊含义的字符串，比如简单的表示一个角色名。或者更复杂的意义，取决于AccessDecisionManager实现的复杂性。实现类：SecurityConfig。
* SecurityContextPersistenceFilter 每当请求来时它都会通过SecurityContextHolder来获取认证信息（存储在SecurityContext中），并在请求结束后通过SecurityContextHolder存储当前认证的信息。
* SecurityMetadataSource接口 子接口：FilterInvocationSecurityMetadataSource。
* FilterInvocationSecurityMetadataSource接口 继承自SecurityMetadataSource，此接口本身并无任何内容。因为Spring Security中很多web使用的类参数类型都是FilterInvocationSecurityMetadataSource。而我们做的一般都是web项目，所以实际需要实现的接口是FilterInvocationSecurityMetadataSource，
* AccessDecisionManager 访问决策管理器
* AbstractAuthenticationProcessingFilter 登陆认证拦截器，
* AuthenticationManager接口 是一个用来处理认证请求的接口。在其中只定义了一个方法authenticate()，该方法只接收一个代表认证请求的Authentication对象作为参数，如果认证成功，则会返回一个封装了当前用户权限等信息的Authentication对象。
* AuthenticationProvider接口 主要用来进行认证的操作，调用其中的authenticate()方法进行认证操作．
* DaoAuthenticationProvider AuthenticationProvider接口的一个实现类．
* ProviderManager AuthenticationManager接口的实现类．内部维护了一个AuthenticationProvider实现类的列表．ProviderManager不直接自己处理认证请求，而是委托给其所配置的AuthenticationProvider列表，然后会依次使用每一个AuthenticationProvider进行认证，如果有一个AuthenticationProvider认证后的结果不为null，则表示该AuthenticationProvider已经认证成功，之后的AuthenticationProvider将不再继续认证。然后直接以该AuthenticationProvider的认证结果作为ProviderManager的认证结果。如果所有的AuthenticationProvider的认证结果都为null，则表示认证失败，将抛出一个ProviderNotFoundException。
* AbstractSecurityInterceptor接口　资源管理拦截器
* FilterSecurityInterceptor AbstractSecurityInterceptor接口的实现类．

Spring Security Web表单方式登录与授权主要流程
===================================

标准身份验证方案。

    提示用户使用用户名和密码登录。
    
    系统验证密码对用户名是否正确。
    
    获取该用户的上下文信息（其角色列表等）。
    
    为用户建立安全上下文
    
    用户可能会继续执行某些操作，该操作可能受访问控制机制保护，访问控制机制会针对当前安全上下文信息检查操作所需的权限。

前三项构成了认证过程，因此我们将在Spring Security中看看这些是如何发生的。

    获取用户名和密码并将其合并到UsernamePasswordAuthenticationToken（Authentication接口的一个实例，我们之前看到）的一个实例中。
    
    将令牌传递给AuthenticationManager的实例进行验证。
    
    AuthenticationManager在成功验证时返回完全填充的Authentication实例。
    
    通过调用SecurityContextHolder.getContext().setAuthentication(…​)传入返回的认证对象来建立安全上下文。

从那时起，用户被认为是被认证的。

SavedRequest和RequestCache接口

认证
-------
用户登录请求首先会被UsernamePasswordAuthenticationFilter(AuthenticationProcessingFilter的子类)拦截，通过ProviderManager（AuthenticationManager接口的实现类）来获取用户认证信息，如果认证通过后会将用户的权限信息封装一个UserDetails对象中，然后放到spring的全局缓存SecurityContextHolder中，以备后面访问资源时使用。

授权
-------
每次访问资源都会被FilterSecurityInterceptor（AbstractSecurityInterceptor接口的实现类）过滤器拦截，通过FilterInvocationSecurityMetadataSource（SecurityMetadataSource接口的实现类）获取被拦截url所需的全部权限，再调用授权管理器AccessDecisionManager，这个授权管理器会通过spring的全局缓存SecurityContextHolder获取用户的权限信息，还会获取被拦截的url和被拦截url所需的全部权限，然后根据所配的策略（有：一票肯定，一票否决，少数服从多数等），如果权限足够，则返回，权限不够则报错并调用权限不足页面。


Spring Security 5.xx 表单登录流程源码分析
==========================
认证
----
#. 登录请求首先会被UsernamePasswordAuthenticationFilter(从AbstractAuthenticationProcessingFilter继承，这是一个抽象类)拦截．AbstractAuthenticationProcessingFilter过滤器的doFilter方法主要内容如下：
    .. code:: java

       	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
    		throws IOException, ServletException {
    	HttpServletRequest request = (HttpServletRequest) req;
    	HttpServletResponse response = (HttpServletResponse) res;
    	// requiresAuthentication方法内部通过调用一个AntPathRequestMatcher类的实例，来判断当前请求的地址
    	// 与配置的地址是否相等匹配，如果不匹配就放行，进入下一个过滤器，否则就使用当前过滤器进行认证．
    	// 这个AntPathRequestMatcher类的实例应该是在创建UsernamePasswordAuthenticationFilter实例时通过
    	// 构造方法传入．参数应该是来自Security的配置类的protected void configure(HttpSecurity http)
    	// 中的http.formLogin().loginProcessingUrl("/user/login")方法．
    	if (!requiresAuthentication(request, response)) {
    		chain.doFilter(request, response);
    		return;
    	}
    	Authentication authResult;
    	try {
    	// 抽象方法由子类UsernamePasswordAuthenticationFilter实现．
    	// 认证成功就返回一个包含用户权限等信息的Authentication对象。
    		authResult = attemptAuthentication(request, response);
    		if (authResult == null) {
    			// 如果为null，表示认证没完成，立即返回．
    			return;
    		}
    		// 认证成功后，处理一些与session相关的方法
    		sessionStrategy.onAuthentication(authResult, request, response);
    	}
    	catch (InternalAuthenticationServiceException failed) {
    		logger.error(
    				"An internal error occurred while trying to authenticate the user.",
    				failed);
    		// 认证失败后的的一些操作
    		unsuccessfulAuthentication(request, response, failed);
    		return;
    	}
    	catch (AuthenticationException failed) {
    		// Authentication failed
    		unsuccessfulAuthentication(request, response, failed);
    		return;
    	}
    	// Authentication success
    	if (continueChainBeforeSuccessfulAuthentication) {
    		chain.doFilter(request, response);
    	}
    	// 认证成功后调用的方法，主要是将认证成功的authentication放入SecurityContext．
    	successfulAuthentication(request, response, chain, authResult);
     }
    
      protected void unsuccessfulAuthentication(HttpServletRequest request,
    		HttpServletResponse response, AuthenticationException failed)
    		throws IOException, ServletException {
    	SecurityContextHolder.clearContext();
    	if (logger.isDebugEnabled()) {
    		logger.debug("Authentication request failed: " + failed.toString(), failed);
    		logger.debug("Updated SecurityContextHolder to contain null Authentication");
    		logger.debug("Delegating to authentication failure handler " + failureHandler);
    	}
    	// 认证失败，调用remember-me的登录失败功能，删除名为"remember-me"的Cookie．
    	rememberMeServices.loginFail(request, response);
    	// 调用失败处理器．实现AuthenticationFailureHandler接口即可．
    	failureHandler.onAuthenticationFailure(request, response, failed);
      }
    
      protected void successfulAuthentication(HttpServletRequest request,
    		HttpServletResponse response, FilterChain chain, Authentication authResult)
    		throws IOException, ServletException {
    
    	if (logger.isDebugEnabled()) {
    		logger.debug("Authentication success. Updating SecurityContextHolder to contain: "
    				+ authResult);
    	}
        // 将认证成功的authentication放入SecurityContext．
    	SecurityContextHolder.getContext().setAuthentication(authResult);
        // 调用remember-me功能．进入remember-me的流程．
    	rememberMeServices.loginSuccess(request, response, authResult);
    
    	// Fire event
    	if (this.eventPublisher != null) {
    		eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(
    				authResult, this.getClass()));
    	}
        // 调用认证成功处理器．实现AuthenticationSuccessHandler接口即可．
    	successHandler.onAuthenticationSuccess(request, response, authResult);
     }
#. UsernamePasswordAuthenticationFilter过滤器的核心方法attemptAuthentication的主要代码：
    .. code:: java

       public Authentication attemptAuthentication(HttpServletRequest request,HttpServletResponse response) {
           // ......　略
           // 根据从请求里获取的　username password 生成 UsernamePasswordAuthenticationToken 对象
           UsernamePasswordAuthenticationToken authRequest =
               new UsernamePasswordAuthenticationToken(username, password);
    	   // 将当前请求的 ip sessionId 封装成一个WebAuthenticationDetails对象，放在 authRequest 里．
           setDetails(request, authRequest);
           // 调用 AuthenticationManager 的 authenticate 方法进行认证．AuthenticationManager是一个接口，
           // 只有一个方法 Authentication authenticate(Authentication authentication)．实现类为：ProviderManager，
           // 实际调用的是这个实现类中的authenticate方法．
           return this.getAuthenticationManager().authenticate(authRequest);
       }
#. ProviderManager类中有一个属性：private List<AuthenticationProvider> providers，这是一个AuthenticationProvider接口的实现类的集合．ProviderManager类中的authenticate方法主要代码：
    .. code:: java

       public Authentication authenticate(Authentication authentication)
    		throws AuthenticationException {
           Class<? extends Authentication> toTest = authentication.getClass();
           Authentication result = null;
    	AuthenticationException lastException = null;
    	// 循环调用provider的supports方法，找到支持当前的authentication的provider．然后调用这个provider的
    	// authenticate方法．provider的authenticate方法中才是真正认证逻辑．
    	for (AuthenticationProvider provider : getProviders()) {
    		if (!provider.supports(toTest)) {
    			continue;
    		}
    		// 找到匹配的provider，调用provider的authenticate方法．返回一个Authentication对象．
    		try {
    			result = provider.authenticate(authentication);
    			if (result != null) {
    			// 如果 result 不为 null 表示认证通过，
    			// 然后将上一步封装的当前请求的ip sessionId(WebAuthenticationDetails对象)
    			// 拷贝到result中．
    				copyDetails(authentication, result);
    				break;
    			}
    		}
    	}
    	if (result != null) {
    		if (eraseCredentialsAfterAuthentication
    				&& (result instanceof CredentialsContainer)) {
    			// Authentication is complete. Remove credentials and other secret data
    			// from authentication
    			((CredentialsContainer) result).eraseCredentials();
    		}
    		return result;
    	}
    	// 代码运行到这表示在AuthenticationManager里没找到对应的provider，抛出异常，方法结束．
    	if (lastException == null) {
    		lastException = new ProviderNotFoundException(messages.getMessage(
    				"ProviderManager.providerNotFound",
    				new Object[] { toTest.getName() },
    				"No AuthenticationProvider found for {0}"));
    	throw lastException;
       }
#. Spring Security缺省的AuthenticationProvider实现是DaoAuthenticationProvider，而这个DaoAuthenticationProvider又是从AbstractUserDetailsAuthenticationProvider这个抽象类继承的．AuthenticationProvider的authentication方法具体实现是在AbstractUserDetailsAuthenticationProvider类中，主要代码如下：
    .. code:: java

      	 public Authentication authenticate(Authentication authentication)
    		throws AuthenticationException {
    	// Determine username
    	String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
    			: authentication.getName();
    	boolean cacheWasUsed = true;
    	UserDetails user = this.userCache.getUserFromCache(username);
    	if (user == null) {
    		cacheWasUsed = false;
    		// 从Spring Security的缓存中拿UserDetails对象，如果没有就调用 retrieveUser 方法，
    		// 返回一个UserDetails对象，retrieveUser方法中是调用UserDetailsService的loadUserByUsername
    		// 方法拿到UserDetails对象．这个loadUserByUsername方法就是要我们自己实现的那个．
    		try {
    			user = retrieveUser(username,
    					(UsernamePasswordAuthenticationToken) authentication);
    		}
    		catch (UsernameNotFoundException notFound) {
    			logger.debug("User '" + username + "' not found");
    			if (hideUserNotFoundExceptions) {
    				throw new BadCredentialsException(messages.getMessage(
    						"AbstractUserDetailsAuthenticationProvider.badCredentials",
    						"Bad credentials"));
    			}
    			else {
    				throw notFound;
    			}
    		}
    	}
    	try {
    	    // 检查帐号是否锁定，帐号是否删除，帐号是否到期．
    	    preAuthenticationChecks.check(user);
    	    // 检查用户登录输入的密码是否正确．
    	    additionalAuthenticationChecks(user,(UsernamePasswordAuthenticationToken) authentication);
    	}
    	catch (AuthenticationException exception) {
    	    // 上面的４个状态检查出现异常时，如果用户信息是从Spring Security的缓存中取的，
    	    // 就重新调用retrieveUser方法，获得最新的数据，再重新检查．
    		if (cacheWasUsed) {
    			cacheWasUsed = false;
    			user = retrieveUser(username,
    					(UsernamePasswordAuthenticationToken) authentication);
    			// 检查帐号是否锁定，帐号是否删除，帐号是否到期
    			preAuthenticationChecks.check(user);
    			// 检查用户登录输入的密码是否正确．
    			additionalAuthenticationChecks(user,
    					(UsernamePasswordAuthenticationToken) authentication);
    		}
    		else {
    			throw exception;
    		}
    	}
    	// 检查密码是否到期
    	postAuthenticationChecks.check(user);
    	// 将认证通过的UserDetails存入Spring Security缓存．
    	if (!cacheWasUsed) {
    		this.userCache.putUserInCache(user);
    	}
    	Object principalToReturn = user;
    	if (forcePrincipalAsString) {
    		principalToReturn = user.getUsername();
    	}
    	// 返回一个新的UsernamePasswordAuthenticationToken对象．
    	return createSuccessAuthentication(principalToReturn, authentication, user);
     }
    
     protected Authentication createSuccessAuthentication(Object principal,
    		Authentication authentication, UserDetails user) {
    	// 使用UserDetails对象重新创建一个UsernamePasswordAuthenticationToken对象，
    	// 此时创建的对象中的principal属性是一个Userdetails对象，以前存的是用户输入的username，String类型的．
    	// authentication.getCredentials()返回的是用户输入的未加密的密码．
    	UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(
    			principal, authentication.getCredentials(),
    			authoritiesMapper.mapAuthorities(user.getAuthorities()));
    	// 更新result中的WebAuthenticationDetails对象(封装的是当前请求ip，sessionId)．
    	result.setDetails(authentication.getDetails());
    	return result;
    }
#. Remember-Me功能


授权
----
#.

增加自定义的认证-手机认证码登录。
--------------
#. 实现一个自己的AuthenticationProvider。参考AbstractUserDetailsAuthenticationProvider类。
    .. code:: java

     public class SmsAuthenticationProvider implements
        AuthenticationProvider, InitializingBean, MessageSourceAware {
        // 略...
     }

#. 实现一个自己的认证过滤器。参考UsernamePasswordAuthenticationFilter类。
    .. code:: java

     public class SmsAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    
        public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "mobile";
        public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "vcode";
    
        private String mobileParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
        private String vcodeParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
    
        public SmsAuthenticationFilter() {
            super(new AntPathRequestMatcher("/user/sms", "POST"));
        }
    
        @Override
        public Authentication attemptAuthentication(HttpServletRequest request,
                                                    HttpServletResponse response) throws AuthenticationException {
            if (!"POST".equals(request.getMethod())) {
                throw new AuthenticationServiceException(
                        "Authentication method not supported: " + request.getMethod());
            }
    
            String mobile = obtainMobile(request);
            String vcode = obtainVcode(request);
    
            if (mobile == null) {
                mobile = "";
            }
    
            if (vcode == null) {
                vcode = "";
            }
    
            mobile = mobile.trim();
    
            SmsAuthenticationToken authRequest = new SmsAuthenticationToken(
                    mobile, vcode);
    
            setDetails(request, authRequest);
    
            return this.getAuthenticationManager().authenticate(authRequest);
        }
    
        private String obtainVcode(HttpServletRequest request) {
            return request.getParameter(vcodeParameter);
        }
    
        private String obtainMobile(HttpServletRequest request) {
            return request.getParameter(mobileParameter);
        }
    
        protected void setDetails(HttpServletRequest request,
                                  SmsAuthenticationToken authRequest) {
            authRequest.setDetails(authenticationDetailsSource.buildDetails(request));
        }
    
        public String getMobileParameter() {
            return mobileParameter;
        }
    
        public void setMobileParameter(String mobileParameter) {
            this.mobileParameter = mobileParameter;
        }
    
        public String getVcodeParameter() {
            return vcodeParameter;
        }
    
        public void setVcodeParameter(String vcodeParameter) {
            this.vcodeParameter = vcodeParameter;
        }
     }
#. 在Security配置类中加入以下内容：
    .. code:: java

        @Autowired
        private UserDetailsService myUserDetailsService;
    
        @Bean
        public SmsAuthenticationFilter smsAuthenticationFilter() {
            SmsAuthenticationFilter filter = new SmsAuthenticationFilter();
            filter.setAuthenticationManager(this.authenticationManagerBean());
            filter.setAuthenticationSuccessHandler(authenticationSuccessHandler);
            return filter;
        }
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
        @Bean
        public AuthenticationProvider smsAuthenticationProvider() {
            return new SmsAuthenticationProvider();
        }
    
        // Security缺省提供的用户名密码的认证的Provider。
        @Bean
        public AuthenticationProvider daoAuthenticationProvider() {
            DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
            provider.setUserDetailsService(myUserDetailsService);
            provider.setPasswordEncoder(passwordEncoder());
            return provider;
        }
    
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                .authenticationProvider(smsAuthenticationProvider())
                .authenticationProvider(daoAuthenticationProvider())
                .addFilterAfter(smsAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
                // ...略
        }