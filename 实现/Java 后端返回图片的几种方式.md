### Java 后端返回图片的几种方式

场景：用户头像，社交平台图片展示等。

#### URL 直接访问

后端将图片存储在 Web 服务器的某个可访问目录下，并直接提供图片的 URL 给前端访问。

```java
@GetMapping("/image/{imageName}")
public ResponseEntity<Resource> getImageAsResource(@PathVariable String imageName) {
    try {
        Path filePath = Paths.get("images", imageName);
        Resource resource = new UrlResource(filePath.toUri());
        if (resource.exists() || resource.isReadable()) {
            return ResponseEntity
                	.ok()
                	.contentType(MediaType.IMAGE_JPEG)
                	.body(resource);
        } else {
            throw new RuntimeException("Could not read the file");
        }
    } catch (MalformedURLException e) {
        throw new RuntimeException("Error: " + e.getMessage());
    }
}
```

#### Base64 编码传输

对于小型图片，可以将图片转为 Base64 编码的字符串，嵌入到 JSON 或 HTML 中直接传输。这样可以减少 HTTP 请求，但会增加图片大小约33%。

```java
@GetMapping("/base64Image/{imageName}")
public ResponseEntity<String> getImageAsBase64(@PathVariable String imageName) {
    try {
        Path filePath = Paths.get("images", imageName);
        byte[] fileContent = Files.readAllBytes(filePath);
        String encodeString = Base64.getEncoder().encodeToString(fileContent);
        
        return ResponseEntity.ok().body(encodeString);
    } catch (IOException e) {
        throw new RuntimeException("Error: " + e.getMessage());
    }
}
```

#### 二进制流传输

对于需要控制访问权限或大型图片的场景，可以通过将图片作为二进制流传输。既可以保证图片安全传输，又可以减少数据传输的大小。

```java
@GetMapping("/binaryImage/{imageName}")
public void getImageAsBinary(HttpServletResponse response, @PathVariable String imageName) {
    Path filePath = Paths.get("images", imageName);
    byte[] imageContent = Files.readAllBytes(filePath);
    response.setContentType(MediaType.IMAGE_JPEG_VALUE);
    StreamUtils.copy(imageContent, response.getOutputStream());
}
```

URL 直接访问简单方便，而且可以利用缓存减轻服务器压力。缺点是资源无法控制访问权限。

Base64 适合小图片传输，复用性高且基本不会被更新的图片。网页上的图片都是需要消耗一个 http 请求下载而来的，Base64的数据可以随着 HTML下载，节省一个 http 请求。Base64 还能完美兼容几乎所有系统。

然而 Base64 付出的代价是图片大小的增长，以及 CSS 文件体积的增大。

二进制流传输可以控制访问权限，同时又能减少数据传输的大小，适合大型图片的场景。

#### 优化思路

- 缓存策略：对于不经常变更的图片资源，应用合理的HTTP缓存策略，减少服务器的负担和响应时间。
- 压缩：在保证图片质量的前提下，通过压缩图片的大小，加快传输速度。
- CDN：使用 CDN 将图片缓存于离用户更近的服务器上，进一步提升访问速度。