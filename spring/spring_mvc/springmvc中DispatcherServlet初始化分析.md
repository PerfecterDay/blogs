# Spring MVC 中DispatcherServlet初始化分析
{docsify-updated}


首先看 `DispatcherServlet` 继承关系：
`DispatcherServlet` -> `FrameworkServlet` --> `HttpServletBean` -> `HttpServlet`

找到 `HttpServletBean` 的 `init` 方法：

    @Override
	public final void init() throws ServletException {
		// Set bean properties from init parameters.
		PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
		if (!pvs.isEmpty()) {
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
            initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
		}
        // Let subclasses do whatever initialization they like.
		initServletBean();
	}

跟到 `FrameworkServlet` 的 `initServletBean` 方法：

    @Override
	protected final void initServletBean() throws ServletException {
        this.webApplicationContext = initWebApplicationContext();
        initFrameworkServlet();	
	}
继续跟到 `initWebApplicationContext` 方法：

    protected WebApplicationContext initWebApplicationContext() {
		WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
		WebApplicationContext wac = null;
		if (this.webApplicationContext != null) {
			wac = this.webApplicationContext;
			if (wac instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
				if (!cwac.isActive()) {
						if (cwac.getParent() == null) {
						cwac.setParent(rootContext);
					}
					configureAndRefreshWebApplicationContext(cwac);
				}
			}
		}
		if (wac == null) {
			wac = findWebApplicationContext();
		}
		if (wac == null) {
			wac = createWebApplicationContext(rootContext);
		}
		if (!this.refreshEventReceived) {
			onRefresh(wac);
		}
		return wac;
	}
一路跟到 `FrameworkServlet` 的 `createWebApplicationContext` 方法：

    protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
		ConfigurableWebApplicationContext wac =
				(ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
		wac.setEnvironment(getEnvironment());
		//设置 parent 上下文
		wac.setParent(parent);
		String configLocation = getContextConfigLocation();
		if (configLocation != null) {
			wac.setConfigLocation(configLocation);
		}
		configureAndRefreshWebApplicationContext(wac);
		return wac;
	}

跟着看 `configureAndRefreshWebApplicationContext` 方法：

    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
		wac.setServletContext(getServletContext());
		wac.setServletConfig(getServletConfig());
		wac.setNamespace(getNamespace());
		wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));
		ConfigurableEnvironment env = wac.getEnvironment();
		if (env instanceof ConfigurableWebEnvironment) {
			((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());
		}
		postProcessWebApplicationContext(wac);
		applyInitializers(wac);
		wac.refresh();
	}

于是可以看出， `DispatcherServlet` 初始化了 ApplicationContext。

再看 `DispatcherServlet` 的 `onRefresh` 方法：

	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}

	/**
	 * Initialize the strategy objects that this servlet uses.
	 * <p>May be overridden in subclasses in order to initialize further strategy objects.
	 */
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
到此， DispatcherServlet 的初始化流程大致清晰了。