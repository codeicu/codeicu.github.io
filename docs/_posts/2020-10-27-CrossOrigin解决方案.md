---
layout: post
title: "CrossOrigin解决方案"
date: 2020-10-27T22:20:35+08:00
draft: false
tags: Java
categories:
author: Zink
---
# 简单重现Cors问题

启动两个Spring Application, 分别为8080,8081端口.  
8080提供API接口, 8081提供页面去访问8080端口的API.

8080端口的API
```

@RestController
public class Controller {
    @GetMapping("/hello")
    //@CrossOrigin
    public String index(){
        return "8080";
    }

    @PostMapping("/hello")
    //@CrossOrigin
    public String post(){
        return "8080";
    }
}

```


8081端口的页面
```
<html>
<head>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.js"></script>
</head>
<body>
<div id="app"></div>
<input type="button" onclick="btnClick()" value="get_button">
<input type="button" onclick="btnClick2()" value="post_button">
<script>
    function btnClick() {
        $.get('http://localhost:8080/hello', function (msg) {
            $("#app").html(msg);
        });
    }

    function btnClick2() {
        $.post('http://localhost:8080/hello', function (msg) {
            $("#app").html(msg);
        });
    }
</script>
</body>
</html>

```

直接访问即可出现has been blocked by CORS policy: No 'Access-Control-Allow-Origin' 
header is present on the requested resource.错误.

解决方法很简单, 只需加上CrossOrigin注解.

#源码阅读

Annotation for permitting cross-origin requests on specific handler classes 
and/or handler methods. Processed if an appropriate {@code HandlerMapping} is 
configured.
可以用在类和方法上, 如果配置正确就会处理.  
Both Spring Web MVC and Spring WebFlux support this annotation through the 
{@code RequestMappingHandlerMapping} in their respective modules.The values
 from each type and method level pair of annotations are added to annotations are 
 added to a CorsConfiguration and then default values are applied via CorsConfiguration
 applyPermitDefaultValues.  
 Web/WebFlux都可以使用该注解, 注解中的值会被添加到CorsConfiguration中, 并被设为默认值.

注解的调用位置: RequestMappingHandlerMapping.class 的 initCorsConfiguration方法
```
@Override
	protected CorsConfiguration initCorsConfiguration(Object handler, Method method, RequestMappingInfo mappingInfo) {
		HandlerMethod handlerMethod = createHandlerMethod(handler, method);
		Class<?> beanType = handlerMethod.getBeanType();
		CrossOrigin typeAnnotation = AnnotatedElementUtils.findMergedAnnotation(beanType, CrossOrigin.class);
		CrossOrigin methodAnnotation = AnnotatedElementUtils.findMergedAnnotation(method, CrossOrigin.class);

		if (typeAnnotation == null && methodAnnotation == null) {
			return null;
		}

		CorsConfiguration config = new CorsConfiguration();
		updateCorsConfig(config, typeAnnotation);
		updateCorsConfig(config, methodAnnotation);

		if (CollectionUtils.isEmpty(config.getAllowedMethods())) {
			for (RequestMethod allowedMethod : mappingInfo.getMethodsCondition().getMethods()) {
				config.addAllowedMethod(allowedMethod.name());
			}
		}
		return config.applyPermitDefaultValues();
	}

```

配置完成, 在DefaultCorsProcessor方法会往response中增加header
```
@Override
	@SuppressWarnings("resource")
	public boolean processRequest(@Nullable CorsConfiguration config, HttpServletRequest request,
			HttpServletResponse response) throws IOException {

		Collection<String> varyHeaders = response.getHeaders(HttpHeaders.VARY);
		if (!varyHeaders.contains(HttpHeaders.ORIGIN)) {
			response.addHeader(HttpHeaders.VARY, HttpHeaders.ORIGIN);
		}
		if (!varyHeaders.contains(HttpHeaders.ACCESS_CONTROL_REQUEST_METHOD)) {
			response.addHeader(HttpHeaders.VARY, HttpHeaders.ACCESS_CONTROL_REQUEST_METHOD);
		}
		if (!varyHeaders.contains(HttpHeaders.ACCESS_CONTROL_REQUEST_HEADERS)) {
			response.addHeader(HttpHeaders.VARY, HttpHeaders.ACCESS_CONTROL_REQUEST_HEADERS);
		}

```


