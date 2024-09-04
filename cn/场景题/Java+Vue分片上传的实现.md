# Java+Vue分片上传的实现

基本流程就是前端把文件分割上传，然后后端收到所有分片然后进行合并。

## 后端

```java
package com.bingzhi.quickfile.controller;

import com.bingzhi.quickfile.domain.UploadRequest;
import com.bingzhi.quickfile.util.FileMd5Calculator;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.file.Files;
import java.security.NoSuchAlgorithmException;
import java.util.*;
import java.util.stream.Collectors;

@CrossOrigin(origins = "http://localhost:8080") // 允许来自指定域名的请求
@RestController
@RequestMapping("/api/upload")
public class FileUploadControllerOut {

    private static final String UPLOAD_DIR = "/Users/yangyida/Documents/project/QuickFile/QuickFileServer/src/main/resources"; // 文件上传目录
    private static final long CHUNK_SIZE = 5 * 1024 * 1024; // 分片大小

    // 初始化上传，返回上传ID和已上传的分片信息
    @PostMapping("/init")
    public ResponseEntity<Map<String, Object>> initUpload(@RequestBody UploadRequest uploadRequest) {
        String fileMd5 = uploadRequest.getFileMd5(); // 获取文件的MD5值
        String uploadId = UUID.randomUUID().toString(); // 生成唯一的上传ID
        File file = new File(UPLOAD_DIR + uploadId); // 创建上传文件目录
        System.err.println("fileMd5"+fileMd5);
        file.mkdirs();

        // 获取已上传的分片信息
        File[] uploadedChunks = file.listFiles();
        List<Integer> uploadedChunkIndexes = Arrays.stream(uploadedChunks)
                .map(f -> Integer.parseInt(f.getName()))
                .collect(Collectors.toList());


        Map<String, Object> response = new HashMap<>();
        response.put("uploadId", uploadId); // 返回上传ID
        response.put("uploadedChunks", uploadedChunkIndexes); // 返回已上传的分片
        return ResponseEntity.ok(response);
    }

    // 上传单个分片
    @PostMapping("/chunk")
    public ResponseEntity<String> uploadChunk(@RequestParam String uploadId,
                                              @RequestParam int chunkIndex,
                                              @RequestParam MultipartFile chunk) throws IOException {
        String fileDir = UPLOAD_DIR + uploadId; // 获取上传目录路径
        File dir = new File(fileDir);
        System.err.println(uploadId);
        // 确保目录存在，如果不存在则创建
        if (!dir.exists()) {
            dir.mkdirs(); // 创建目录，包括必要的但不存在的父目录
        }

        File chunkFile = new File(fileDir, String.valueOf(chunkIndex)); // 创建分片文件
        chunk.transferTo(chunkFile); // 保存分片数据到服务器

        return ResponseEntity.ok("Chunk uploaded successfully.");
    }


    // 完成上传，合并所有分片
    @PostMapping("/complete")
    public ResponseEntity<HashMap> completeUpload(@RequestParam("uploadId") String uploadId , @RequestParam("fileName")String fileName , @RequestParam("md5")String md5) throws IOException, NoSuchAlgorithmException {
        System.err.println("uploadId is "+uploadId);
        File dir = new File(UPLOAD_DIR + uploadId); // 获取上传目录
        File[] chunks = dir.listFiles(); // 获取所有分片文件
        Arrays.sort(chunks, Comparator.comparingInt(f -> Integer.parseInt(f.getName()))); // 按分片索引排序

        File aaa = new File(UPLOAD_DIR + "/res/"+ uploadId );
        aaa.mkdir();

        File completeFile = new File(UPLOAD_DIR +"/res/"+ uploadId +"/"+ fileName); // 创建完整文件


        try (RandomAccessFile raf = new RandomAccessFile(completeFile, "rw")) {
            // 依次写入所有分片到完整文件
            for (File chunk : chunks) {
                byte[] chunkData = Files.readAllBytes(chunk.toPath());
                raf.write(chunkData);
                chunk.delete();
            }
        }

        dir.delete();

        String againPassWord = FileMd5Calculator.calculateFileMd5(completeFile);
        if (!md5.equals(againPassWord)) {
            //todo 文件不一致

        }

        HashMap<String,String> res=  new HashMap<>();
        res.put("uploadId",uploadId);
        res.put("fileName",fileName);

        return ResponseEntity.ok(res);
    }
}
```

### **初始化上传 (`/init`接口)**

- 客户端发起文件上传请求时，首先调用此接口。
- 通过上传请求中的文件MD5值，生成一个唯一的`uploadId`，并为该上传会话创建对应的上传目录，用于存储上传的分片。
- 服务器返回给客户端`uploadId`，同时返回当前已经上传的分片索引信息，客户端可以基于此信息续传未上传的分片。

### 2. **上传分片 (`/chunk`接口)**

- 每个文件被分为多个分片，客户端逐个上传分片时，携带`uploadId`和分片索引（`chunkIndex`）。
- 服务器接收到分片后，根据`uploadId`找到相应的目录，将分片数据保存到该目录下，以分片的索引（`chunkIndex`）为文件名。
- 分片上传成功后，服务器返回“上传成功”的响应。

### 3. **合并分片 (`/complete`接口)**

- 当所有分片都上传完成后，客户端调用此接口来请求合并分片。
- 服务器获取到`uploadId`后，找到该上传的分片目录，并按分片的索引顺序依次读取所有分片文件。
- 通过`RandomAccessFile`的方式，将所有分片的内容依次写入到一个完整文件中。
- 合并完成后，删除各个分片文件。
- 服务器对合并后的文件计算其MD5值，并与上传时客户端传递的MD5值进行比对，确保文件的完整性。
- 返回合并后的文件信息，如`uploadId`和文件名。

### 4. **MD5校验**

- `FileMd5Calculator.calculateFileMd5(completeFile)`用于在合并文件后重新计算文件的MD5值，与上传时客户端提供的MD5进行对比。
- 若MD5值不一致，说明文件在传输或合并过程中出现了问题，需要处理文件不一致的情况。

### **关键点**：

- **分片上传**：为了避免单次上传大文件的负载和失败风险，将文件切分为多个分片分别上传。
- **断点续传**：通过`/init`接口返回已上传的分片索引列表，客户端可只上传尚未上传的部分，从而实现断点续传功能。
- **文件校验**：使用MD5值校验确保文件上传的完整性，保证分片合并后的文件与源文件一致。

## 前端

~~~vue
<template>
  <div>
    <!-- 文件选择输入框 -->
    <input type="file" @change="onFileChange" />
    
    <!-- 显示上传进度 -->
    <div v-if="uploadProgress !== null">
      Upload Progress: {{ uploadProgress }}%
    </div>
  </div>
</template>

<script>
import axios from 'axios'; // 引入axios用于HTTP请求
import SparkMD5 from 'spark-md5'; // 引入SparkMD5用于计算文件的MD5哈希值

export default {
  data() {
    return {
      uploadProgress: null, // 上传进度百分比
      file: null, // 用户选择的文件
      chunkSize: 5 * 1024 * 1024, // 每个分片的大小，设定为5MB
      uploadedChunks: [], // 已上传的分片索引
      uploadId: '', // 上传ID，用于标识一次上传过程
      extension: '', //file name
    };
  },
  methods: {
    // 处理文件选择事件
    async onFileChange(event) {
      this.file = event.target.files[0]; // 获取用户选择的文件
      if (!this.file) return;

      // 计算文件的MD5值
      const fileMd5 = await this.calculateFileMd5(this.file);

      // 初始化上传过程，获取uploadId和已上传的分片信息
      this.extension = this.file.name.split('.').pop();
      

      const { data: uploadInfo } = await axios.post('http://localhost:9999/api/upload/init', {
        fileName: this.file.name,
        fileSize: this.file.size,
        fileMd5,
      });

      this.fileName = this.file.name;
      this.uploadId = uploadInfo.uploadId; // 记录上传ID
      this.uploadedChunks = uploadInfo.uploadedChunks; // 记录已上传的分片

      // 开始上传文件
      await this.uploadFile();
    },

    // 计算文件的MD5值，用于秒传和断点续传
    async calculateFileMd5(file) {
      return new Promise((resolve, reject) => {
        const chunkSize = this.chunkSize; // 分片大小
        const chunks = Math.ceil(file.size / chunkSize); // 计算分片数量
        const spark = new SparkMD5.ArrayBuffer(); // 创建SparkMD5对象
        const fileReader = new FileReader(); // 创建文件读取对象
        let currentChunk = 0;

        // 当文件读取完一个分片后触发
        fileReader.onload = e => {
          spark.append(e.target.result); // 将分片数据加入到SparkMD5对象中
          currentChunk++;

          if (currentChunk < chunks) {
            loadNextChunk(); // 加载下一个分片
          } else {
            resolve(spark.end()); // 所有分片加载完成后返回MD5值
          }
        };

        fileReader.onerror = () => reject('MD5 calculation failed.'); // 处理文件读取错误

        // 加载文件的下一个分片
        const loadNextChunk = () => {
          const start = currentChunk * chunkSize; // 分片的开始字节位置
          const end = Math.min(start + chunkSize, file.size); // 分片的结束字节位置
          fileReader.readAsArrayBuffer(file.slice(start, end)); // 读取当前分片
        };

        loadNextChunk(); // 启动第一个分片的加载
      });
    },

    // 上传文件的分片
    async uploadFile() {
      const chunkSize = this.chunkSize; // 分片大小
      const chunks = Math.ceil(this.file.size / chunkSize); // 分片总数

      // 循环上传每个分片
      for (let chunkIndex = 0; chunkIndex < chunks; chunkIndex++) {
        // 如果分片已经上传过，跳过该分片
        if (this.uploadedChunks.includes(chunkIndex)) {
          this.updateProgress(chunkIndex, chunks); // 更新上传进度
          continue;
        }

        const start = chunkIndex * chunkSize; // 分片开始字节位置
        const end = Math.min(start + chunkSize, this.file.size); // 分片结束字节位置
        const chunk = this.file.slice(start, end); // 获取分片数据
        const formData = new FormData(); // 创建FormData对象用于上传
        formData.append('uploadId', this.uploadId); // 添加上传ID
        formData.append('chunkIndex', chunkIndex); // 添加分片索引
        formData.append('chunk', chunk); // 添加分片数据

        await axios.post('http://localhost:9999/api/upload/chunk', formData); // 上传当前分片
        this.updateProgress(chunkIndex, chunks); // 更新上传进度
      }

      // 所有分片上传完成后通知服务器完成上传
      await axios.post('http://localhost:9999/api/upload/complete?uploadId='+this.uploadId+"&fileName="+this.file.name);
      this.uploadProgress = 100; // 上传进度设置为100%
    },

    // 更新上传进度百分比
    updateProgress(chunkIndex, totalChunks) {
      this.uploadProgress = Math.round(((chunkIndex + 1) / totalChunks) * 100);
    },
  },
};
</script>
~~~



### **MD5值计算 (`calculateFileMd5` 方法)**

- 使用 `SparkMD5` 库来计算文件的MD5值。这个过程是通过将文件切成多个分片，逐个读取分片，然后计算出整个文件的MD5哈希值。
- 计算完成后，将MD5值返回，用于之后的秒传或断点续传功能。

### 3. **上传初始化 (`/init` API)**

- 在MD5值计算完成后，发送一个请求到服务器的 `/init` 接口，携带文件名、文件大小和MD5值。
- 服务器会返回 `uploadId` 和已经上传的分片索引列表（如果是断点续传的情况）。
- 将这些信息保存在组件的状态中，用于后续上传。

### 4. **分片上传 (`uploadFile` 方法)**

- `uploadFile` 方法根据文件大小，将文件按每5MB切成多个分片，然后逐个上传。
- 每次上传时，先检查当前分片是否已经上传（通过 `uploadedChunks` 数组判断）。如果已经上传，则跳过该分片。
- 对于未上传的分片，通过 `POST` 请求发送到 `/chunk` 接口，上传当前分片。
- 每成功上传一个分片，都会调用 `updateProgress` 方法更新上传进度。

### 5. **完成上传 (`/complete` API)**

- 当所有分片上传完毕后，客户端会向服务器的 `/complete` 接口发送一个请求，通知服务器进行分片合并。
- 合并完成后，上传进度设置为100%。

### 6. **上传进度**

- `updateProgress` 方法根据当前上传的分片索引与总分片数的比例来计算当前上传的百分比，并在页面中动态显示上传进度。

### **其他功能:**

- **断点续传:** 通过 `/init` 接口返回的已上传分片信息，客户端可以跳过已经上传的分片，支持断点续传功能。
- **MD5校验:** MD5值确保文件完整性，支持秒传和断点续传，如果文件已经上传过，服务器可以直接跳过无需重新上传。