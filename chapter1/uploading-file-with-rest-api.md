# Uploading File with REST API

There are multiple ways to upload files using REST API in Spring. Due to client limitations, it can happen that one ways is not enough and we need to provide multiple ways to upload files.

We are going to create REST API that reads file which has been upload and returns its content.

### Using `multipart/form-data`

 The most standard way to upload way is to use multipart.

```
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

@RestController
public class UploadController {

  @PostMapping("/upload/multipart/form-data")
  public ResponseEntity<String> upload(@RequestParam("file") MultipartFile file) throws IOException {
    byte[] bytes = file.getBytes();
    String fileContent = new String(bytes);
    return ResponseEntity.status(HttpStatus.CREATED).body(fileContent);
  }
}
```

Here is curl command that will upload a file.

```
curl -X "POST" "http://localhost:8080/upload/multipart/form-data" \
     -H 'Content-Type: multipart/form-data; charset=utf-8' \
     -F "file=@/Users/ondrej/testfile.txt"
```

Here is how we could test the code to verify functionality.

```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.springframework.http.MediaType.TEXT_PLAIN_VALUE;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(UploadController.class)
public class UploadControllerTest {

  @Autowired
  private MockMvc mvc;

  @Test
  public void uploadsMultipartFormData() throws Exception {
    byte[] fileContent = "content of a file".getBytes();
    MockMultipartFile firstFile = new MockMultipartFile("file", "file.txt", TEXT_PLAIN_VALUE, fileContent);

    mvc.perform(MockMvcRequestBuilders.multipart("/upload/multipart/form-data").file(firstFile))
        .andExpect(status().is(201))
        .andExpect(content().string("content of a file"));
  }
}
```

Here is the configuration file that will increase max allowed size of uploaded file, changes port and context path, configures logging output file and level.

```
---
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB

server:
  port: 8080
  servlet:
    context-path: /v1

logging:
  file: logs/logs.log
  level: 
    org.springframework: INFO
```

### Using `application/x-www-form-urlencoded`

 Other way is to use `application/x-www-form-urlencoded`, usually  sent from web clients.

```
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

@RestController
public class UploadController {

  @PostMapping("/upload/application/x-www-form-urlencoded")
  public ResponseEntity<String> upload(HttpServletRequest request) {
    String file = request.getParameter("file");
    return ResponseEntity.status(HttpStatus.CREATED).body(file);
  }
}
```

Here is curl command that will upload a file using this REST API endpoint.

```
curl -X "POST" "http://localhost:8080/upload/application/x-www-form-urlencoded" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "file=content of a test file"
```

Here is how we could test the code to verify functionality.

```
package com.example.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(UploadController.class)
public class UploadControllerTest {

  @Autowired
  private MockMvc mvc;

  @Test
  public void uploadsFileUsingApplicationXwwwFormUrlEncoded() throws Exception {
    String fileContent = "content of a file";

    mvc.perform(MockMvcRequestBuilders.post("/upload/application/x-www-form-urlencoded").param("file", fileContent))
        .andExpect(status().is(201))
        .andExpect(content().string("content of a file"));
  }
}
```

### Using `application/octet-stream`

The file is set into body and needs to be obtained from it.

```
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import java.io.BufferedReader;
import java.io.IOException;
import java.util.stream.Collectors;

@RestController
public class UploadController {

  @PostMapping("/upload/application/octet-stream")
  public ResponseEntity<String> upload(HttpServletRequest request) throws IOException {
    BufferedReader reader = request.getReader();
    String file = reader.lines().collect(Collectors.joining("\n"));
    return ResponseEntity.status(HttpStatus.CREATED).body(file);
  }
}
```

Here is curl command that will upload a file using this REST API endpoint.

```
curl -X "POST" "http://localhost:8080/upload/application/octet-stream" \
     -H 'Content-Type: application/octet-stream' \
     -d $'content of a test file'
```

Here is how we could test the code to verify functionality.

```
package com.example.demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(UploadController.class)
public class UploadControllerTest {

  @Autowired
  private MockMvc mvc;

  @Test
  public void uploadsFileUsingOctetStream() throws Exception {
    byte[] fileContent = "content of a file".getBytes();

    mvc.perform(MockMvcRequestBuilders.post("/upload/application/octet-stream").content(fileContent))
        .andExpect(status().is(201))
        .andExpect(content().string("content of a file"));
  }
}
```



