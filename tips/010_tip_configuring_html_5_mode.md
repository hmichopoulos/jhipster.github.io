---
layout: default
title: Configuring HTML 5 mode
sitemap:
priority: 0.5
lastmod: 2015-11-04T23:23:00-00:00
---

# Configuring HTML 5 mode

__Tip submitted by [@brevleq](https://github.com/brevleq)__

As you may noticed, AngularJS uses a "#" in it's urls. HTML5Mode of AngularJS removes these "#" from URL.

## Activate HTML 5 Mode

Open the `app.js` file and add this line in `config` method:

    $locationProvider.html5Mode(true);

Then open `index.html` and add this line in `head` tag:

    <base href="/">
    
## Redirection filter     
    
Now, to have relative paths links working correctly (ex. activation link sent to user e-mail) we need create a filter to redirect the URI without "#" to a url with "#", so AngularJS can work correctly:
    
    public class Html5ModeFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
            // Nothing to initialize
        }
    
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
             HttpServletRequest httpRequest = (HttpServletRequest) request;
             HttpServletResponse httpResponse = (HttpServletResponse) response;
             if (httpResponse.getStatus() == HttpServletResponse.SC_NOT_FOUND) {
                 String errorRequestURI = (String) httpRequest.getAttribute("javax.servlet.error.request_uri");
                 if (errorRequestURI.equals(requestURI)) {
                     StringBuilder newURL = new StringBuilder("/#").append(requestURI).append(parameters);
                     httpResponse.sendRedirect(newURL.toString());
                     return;
                 }
             } else {
                parameters = new StringBuilder("?");
                requestURI = httpRequest.getRequestURI();
                Enumeration<String> parameterNames = httpRequest.getParameterNames();
                while (parameterNames.hasMoreElements()) {
                    String paramName = parameterNames.nextElement();
                    parameters.append(paramName);
                    parameters.append("=");
                    for (String value : httpRequest.getParameterValues(paramName))
                        parameters.append(value);
                    parameters.append("&");
                }
                int index = parameters.length() - 1;
                if (index >= 0)
                    parameters.delete(index, parameters.length());
             }
             chain.doFilter(request, response);
        }    
        
        @Override
        public void destroy() {
            // Nothing to destroy
        }
    }
    
Finally, init the filter in `WebConfigurer` class:
    
    public void onStartup(ServletContext servletContext) throws ServletException {
        ...
        initHtml5ModeFilter(servletContext, disps);
        ...
    }
    
    private void initHtml5ModeFilter(ServletContext servletContext, EnumSet<DispatcherType> disps){
        log.debug("Registering HTML5Mode Filter");
        FilterRegistration.Dynamic html5ModeFilter = servletContext.addFilter("html5Mode", new Html5ModeFilter());
        html5ModeFilter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST, DispatcherType.ERROR), true, "/*");
    }
