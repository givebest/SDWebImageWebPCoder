# SDWebImageWebPCoder 元数据选项功能

## 概述

SDWebImageWebPCoder 现在支持在编码 WebP 图像时复制原始图像的元数据。这个功能通过新的 `SDImageCoderEncodeWebPMetadata` 选项来控制。

## 新增选项

### SDImageCoderEncodeWebPMetadata

**类型**: `NSString *`  
**描述**: 要从输入复制到输出的元数据（如果有）的逗号分隔列表  
**默认值**: `"none"`

#### 有效值

- `"none"` - 不复制任何元数据（默认行为）
- `"all"` - 复制所有支持的元数据类型
- `"exif"` - 只复制 EXIF 数据
- `"icc"` - 只复制 ICC 颜色配置文件
- `"xmp"` - 只复制 XMP 数据（暂未完全实现）
- 组合值 - 使用逗号分隔多个类型，如 `"exif,icc"`

## 使用示例

### Objective-C

```objective-c
#import "SDImageWebPCoder.h"
#import "SDWebImageWebPCoderDefine.h"

// 获取编码器实例
SDImageWebPCoder *coder = [SDImageWebPCoder sharedCoder];

// 示例 1: 不复制任何元数据（默认行为）
NSData *webpData1 = [coder encodedDataWithImage:image
                                          format:SDImageFormatWebP
                                         options:nil];

// 示例 2: 复制所有支持的元数据
NSData *webpData2 = [coder encodedDataWithImage:image
                                          format:SDImageFormatWebP
                                         options:@{SDImageCoderEncodeWebPMetadata: @"all"}];

// 示例 3: 只复制 EXIF 数据
NSData *webpData3 = [coder encodedDataWithImage:image
                                          format:SDImageFormatWebP
                                         options:@{SDImageCoderEncodeWebPMetadata: @"exif"}];

// 示例 4: 复制 EXIF 和 ICC 配置文件
NSData *webpData4 = [coder encodedDataWithImage:image
                                          format:SDImageFormatWebP
                                         options:@{SDImageCoderEncodeWebPMetadata: @"exif,icc"}];

// 示例 5: 明确指定不复制元数据
NSData *webpData5 = [coder encodedDataWithImage:image
                                          format:SDImageFormatWebP
                                         options:@{SDImageCoderEncodeWebPMetadata: @"none"}];
```

### Swift

```swift
import SDWebImageWebPCoder

// 获取编码器实例
let coder = SDImageWebPCoder.shared

// 示例 1: 不复制任何元数据（默认行为）
let webpData1 = coder.encodedData(with: image, format: .webP, options: nil)

// 示例 2: 复制所有支持的元数据
let webpData2 = coder.encodedData(with: image,
                                  format: .webP,
                                  options: [SDImageCoderEncodeWebPMetadata: "all"])

// 示例 3: 只复制 EXIF 数据
let webpData3 = coder.encodedData(with: image,
                                  format: .webP,
                                  options: [SDImageCoderEncodeWebPMetadata: "exif"])

// 示例 4: 复制 EXIF 和 ICC 配置文件
let webpData4 = coder.encodedData(with: image,
                                  format: .webP,
                                  options: [SDImageCoderEncodeWebPMetadata: "exif,icc"])
```

## 技术实现

### 支持的元数据类型

1. **EXIF 数据** - 包含相机设置、拍摄参数等信息
2. **ICC 颜色配置文件** - 用于颜色管理的配置文件
3. **XMP 数据** - 可扩展元数据平台（暂未完全实现）

### 实现细节

- 使用 Core Graphics 的 `CGImageSource` API 从原始图像中提取元数据
- 使用 libwebp 的 `WebPMux` API 将元数据添加到 WebP 文件中
- 支持静态和动画 WebP 图像
- 元数据选项解析支持逗号分隔的多个值
- 默认行为保持不变（不复制元数据）以确保向后兼容性

### 性能考虑

- 复制元数据会略微增加编码时间和输出文件大小
- 如果不需要元数据，建议使用默认设置（`"none"`）以获得最佳性能
- ICC 配置文件通常是最大的元数据块

## 兼容性

- 向后兼容：现有代码无需修改即可继续工作
- 支持 iOS 9.0+, macOS 10.11+, tvOS 9.0+, watchOS 2.0+
- 需要 libwebp 库支持

## 注意事项

1. XMP 数据支持目前是基础实现，完整的 XMP 支持需要进一步开发
2. 并非所有原始图像格式都包含所有类型的元数据
3. 元数据的存在和质量取决于原始图像的来源
4. 复制元数据会增加输出文件的大小

## 测试

可以使用提供的 `MetadataTest.m` 文件来测试新功能：

```bash
# 编译并运行测试
clang -framework Foundation -framework UIKit MetadataTest.m -o MetadataTest
./MetadataTest
```
