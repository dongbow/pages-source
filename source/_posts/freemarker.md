---
title: 模板渲染 - Freemarker
date: 2019-07-19 15:32:44
tags: 
	- 模板渲染
	- Freemarker
categories: 模板渲染
---
模板引擎除了去做后端应用的页面渲染，很多时候也可以用来生成一些代码，例如利用sql生成pojo对象，生成模板页面等  
这里简单记录一下FreeMarker渲染模板时最基本的代码  
1. 引入FreeMarker Maven信息
``` xml
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.26-incubating</version>
</dependency>
```
2. 实现代码
``` java
import com.google.common.base.Charsets;
import com.github.pdf.tools.exception.FreeMarkerException;
import freemarker.cache.FileTemplateLoader;
import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateExceptionHandler;

import java.io.File;
import java.io.IOException;
import java.io.StringWriter;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author wangdongbo
 * @since 2019/7/3.
 */
public class FreeMarkerTemplate extends AbstractTemplate {

    private static Map<String, FileTemplateLoader> FILE_TEMPLATE_LOADER_CACHE = new ConcurrentHashMap<>(16);

    private static Map<String, Configuration> CONFIGURATION_CACHE = new ConcurrentHashMap<>(16);

    private static Configuration getConfiguration(String templatePath) {
        if (CONFIGURATION_CACHE.get(templatePath) != null) {
            return CONFIGURATION_CACHE.get(templatePath);
        }
        Configuration config = new Configuration(Configuration.VERSION_2_3_25);
        config.setDefaultEncoding(Charsets.UTF_8.toString());
        config.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
        config.setLogTemplateExceptions(false);
        FileTemplateLoader fileTemplateLoader;
        try {
            if (FILE_TEMPLATE_LOADER_CACHE.containsKey(templatePath)) {
                fileTemplateLoader = FILE_TEMPLATE_LOADER_CACHE.get(templatePath);
            } else {
                fileTemplateLoader = new FileTemplateLoader(new File(templatePath));
            }
        } catch (IOException e) {
            throw new FreeMarkerException("fileTemplateLoader init error!", e);
        }
        config.setTemplateLoader(fileTemplateLoader);
        CONFIGURATION_CACHE.put(templatePath, config);
        return config;
    }

    @Override
    protected String getContent(String templatePath, String templateName, Object data) {
        try {
            Template template = getConfiguration(templatePath).getTemplate(templateName);
            StringWriter writer = new StringWriter();
            template.process(data, writer);
            writer.flush();
            return writer.toString();
        } catch (Exception ex) {
            throw new FreeMarkerException("FreeMarkerUtil process fail", ex);
        }
    }

}
```

