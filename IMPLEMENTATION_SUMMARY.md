# SDWebImageWebPCoder 元数据选项实现总结

## 已完成的修改

### 1. 头文件定义 (SDWebImageWebPCoderDefine.h)

添加了新的选项常量声明：

```objective-c
/**
 String value
 Comma-separated list of metadata to copy from input to output (if any).
 Valid values: all, none, exif, icc, xmp. Default is none.
 */
FOUNDATION_EXPORT SDImageCoderOption _Nonnull const SDImageCoderEncodeWebPMetadata;
```

### 2. 实现文件常量定义 (SDWebImageWebPCoderDefine.m)

添加了常量的实际定义：

```objective-c
SDImageCoderOption _Nonnull const SDImageCoderEncodeWebPMetadata = @"webPMetadata";
```

### 3. 主要实现 (SDImageWebPCoder.m)

#### 3.1 添加了辅助函数

```objective-c
static NSString * GetStringValueForKey(NSDictionary * _Nonnull dictionary, NSString * _Nonnull key, NSString * _Nullable defaultValue);
```

#### 3.2 修改了单帧编码方法

在 `sd_encodedWebpDataWithImage:orientation:quality:maxPixelSize:maxFileSize:options:` 方法中添加了元数据处理：

```objective-c
// Handle metadata copying if requested
NSString *metadataOption = options[SDImageCoderEncodeWebPMetadata];
if (metadataOption && ![metadataOption isEqualToString:@"none"]) {
    webpData = [self sd_webpDataByAddingMetadata:webpData
                                    fromImageRef:imageRef
                                  metadataOption:metadataOption];
}
```

#### 3.3 修改了动画编码方法

在 `encodedDataWithFrames:loopCount:format:options:` 方法中添加了元数据处理：

```objective-c
// Handle metadata copying for animated WebP if requested
NSString *metadataOption = options[SDImageCoderEncodeWebPMetadata];
if (metadataOption && ![metadataOption isEqualToString:@"none"]) {
    [self sd_addMetadataToMux:mux fromImageRef:imageRef metadataOption:metadataOption];
}
```

#### 3.4 添加了元数据处理方法

1. **sd_webpDataByAddingMetadata:fromImageRef:metadataOption:** - 用于单帧 WebP 的元数据处理
2. **sd_addMetadataToMux:fromImageRef:metadataOption:** - 用于动画 WebP 的元数据处理
3. **sd_createExifDataFromDictionary:** - 创建 EXIF 数据的辅助方法

## 功能特性

### 支持的元数据类型

1. **EXIF** - 相机拍摄信息和图像参数
2. **ICC** - 颜色配置文件
3. **XMP** - 可扩展元数据平台（基础支持）

### 选项值

- `"none"` - 不复制任何元数据（默认）
- `"all"` - 复制所有支持的元数据
- `"exif"` - 只复制 EXIF 数据
- `"icc"` - 只复制 ICC 配置文件
- `"xmp"` - 只复制 XMP 数据
- `"exif,icc"` - 复制多种类型（逗号分隔）

### 实现细节

1. **元数据提取**: 使用 Core Graphics 的 `CGImageSource` API
2. **WebP 集成**: 使用 libwebp 的 `WebPMux` API
3. **选项解析**: 支持逗号分隔的多个值
4. **向后兼容**: 默认行为保持不变

## 技术架构

```
原始图像 (CGImageRef)
    ↓
提取元数据 (CGImageSource)
    ↓
编码 WebP 图像 (libwebp)
    ↓
添加元数据 (WebPMux)
    ↓
最终 WebP 文件
```

## 使用示例

```objective-c
// 基本用法
NSData *webpData = [coder encodedDataWithImage:image
                                        format:SDImageFormatWebP
                                       options:@{SDImageCoderEncodeWebPMetadata: @"exif,icc"}];

// 复制所有元数据
NSData *webpDataAll = [coder encodedDataWithImage:image
                                           format:SDImageFormatWebP
                                          options:@{SDImageCoderEncodeWebPMetadata: @"all"}];
```

## 测试和验证

创建了以下测试文件：

1. `MetadataTest.m` - 功能测试示例
2. `METADATA_FEATURE.md` - 详细使用文档

## 注意事项

1. **性能影响**: 复制元数据会略微增加编码时间和文件大小
2. **兼容性**: 保持向后兼容，现有代码无需修改
3. **依赖**: 需要 libwebp 库支持 WebPMux 功能
4. **限制**: XMP 支持目前是基础实现，需要进一步完善

## 下一步改进

1. 完善 XMP 数据的完整支持
2. 添加更多的 EXIF 标签处理
3. 优化元数据提取的性能
4. 添加单元测试覆盖
5. 支持更多元数据格式
