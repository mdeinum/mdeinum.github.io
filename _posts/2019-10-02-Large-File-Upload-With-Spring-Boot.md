---
layout: post
title: "Large/Streaming File Uploads with Spring Boot"
date: 2019-10-02
categories:
- Java
- Spring
tags:
- web
- file-upload
- spring
published: true
---
When uploading files to a Spring (Boot) based web application one generally uses a `MultipartFile` and `MultipartResolver` to handle the file upload. Although this works in a lot of cases it has one drawback that it writes to memory and/or disk. Memory is limited and diskspace might be as well, especially when uploading large files of many GBs. 



```java
@Controller
public class SimpleFileUploadController {

  @RequestMapping
  public void upload(@RequestParam("file") MultipartFile upload) throws Exception {
    // @TODO implement
    
  }
}
```

Enter streaming file uploads. When using [Apache Commons FileUpload][1] you can also use a a streaming way of handling the file uploads. This will save on resources and depending on what you want to do with the file allows you to directly process the file while it is uploading. 

```java
@Controller
public class StreamingFileUploadController {

  @RequestMapping
  public void upload(HttpServletRequest request) {
    // Check that we have a file upload request
    boolean isMultipart = ServletFileUpload.isMultipartContent(request);
    if (!isMultipart) {
      throw new IllegalStateException("Expected a Multipart request");
    }
    // Create a new file upload handler
    ServletFileUpload upload = new ServletFileUpload();

   // Parse the request
   FileItemIterator iter = upload.getItemIterator(request);
   while (iter.hasNext()) {
     FileItemStream item = iter.next();
     String name = item.getFieldName();
     
     if (item.isFormField()) {
        System.out.println("Form field " + name + " with value " + Streams.asString(stream) + " detected.");
    } else {
        InputStream stream = item.getInputStream();
        System.out.println("File field " + name + " with file name " + item.getName() + " detected.");
        // Process the input stream
        ...
    }
}
    // @TODO implement.
  }

}
```

[1]: https://commons.apache.org/proper/commons-fileupload
