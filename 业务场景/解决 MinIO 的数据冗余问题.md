解决使用文件存储服务器 MinIO 时的数据冗余问题

当使用 MinIO 存储数据对象时，如果对象在数据库中的引用被删除，MinIO 服务中会存在冗余文件。

#### 定时清理 MinIO 中的冗余文件

1. 使用 MinIO 的 `listObjects` 方法可以列出存储桶中的所有文件
2. 根据业务逻辑，检查这些文件是否仍然被数据库或其他系统引用
3. 如果某个文件不再被引用，则调用 `removeObject` 方法将其删除

```java
MinioClient minioClient = MinioClient.builder()
    .endpoint("http://minio-server:9000")
    .credential("accessKey", "secretKey")
    .build();
// 获取存储桶中的所有文件
Iterable<Result<Item>> results = minioClient.listObjects(
	ListObjectsArgs.builder().bucket("your-bucket-name").recursive(true).build());

for (Result<Item> result : results) {
    Item item = result.get();
    String obejctName = item.objectName();
    
    // 检查文件是否被引用（根据业务逻辑实现）
    boolean isReferenced = checkIfFieldIsReference(objectName);
    
    if (!isReferenced) {
        // 删除未引用的文件
        minioClient.removeObject(
        	RemoveObjectArgs.builder().bucket("your-bucket-name").object(objectName).build());
        System.out.println("Deleted reduntant file: " + objectName);
    }
}
```

#### 使用 MinIO 的生命周期管理功能

MinIO 支持生命周期管理功能，可以自动删除满足特定条件的文件。例如，可以设置规则，自动删除超过一定时间未被访问的文件。

#### 同步删除

在业务代码中删除文件引用的时候，同步删除 MinIO 中的文件。

```java
public void deleteImage(String objectName) {
    try {
        minioClient.removeObject(
        	RemoveObjectArgs.builder().bucket("your-bucket-name").object(objectName).build());
        System.out.println("Image deleted from MinIO:" + objectName);
    } catch (Exception e) {
        System.err.println("Error deleting image: " + e.getMessage());
    }
}
```





