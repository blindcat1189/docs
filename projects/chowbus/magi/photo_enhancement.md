# Java Libraries for Menu Image Preprocessing Pipeline

> 菜单图像预处理 Java 库选型指南

## Overview

本文档总结了用于菜单识别系统图像预处理的 Java 库，涵盖边缘裁剪、去噪锐化、超分辨率重建和智能矫正四个核心模块。

---

## 1. 菜单边缘裁剪 (Menu Edge Cropping)

**目标**: 检测菜单边界，去除桌面、人手等无关区域

| Library             | Description      | Use Case               |
| ------------------- | ---------------- | ---------------------- |
| **OpenCV (JavaCV)** | 传统轮廓检测     | 简单背景、高对比度场景 |
| **BoofCV**          | 纯 Java 轮廓检测 | 无需 native 依赖的场景 |
| **DJL + YOLO**      | 深度学习目标检测 | 复杂背景、语义分割     |

### Code Examples

```java
// OpenCV 轮廓检测
Mat gray = new Mat();
Imgproc.cvtColor(src, gray, Imgproc.COLOR_BGR2GRAY);
Imgproc.threshold(gray, binary, 0, 255, Imgproc.THRESH_BINARY + Imgproc.THRESH_OTSU);

List<MatOfPoint> contours = new ArrayList<>();
Imgproc.findContours(binary, contours, new Mat(),
    Imgproc.RETR_EXTERNAL, Imgproc.CHAIN_APPROX_SIMPLE);

// 找到最大轮廓并近似为多边形
MatOfPoint2f approxCurve = new MatOfPoint2f();
double epsilon = 0.02 * Imgproc.arcLength(contour2f, true);
Imgproc.approxPolyDP(contour2f, approxCurve, epsilon, true);
```

```java
// DJL YOLO 目标检测
Criteria<Image, DetectedObjects> criteria = Criteria.builder()
    .optApplication(Application.CV.OBJECT_DETECTION)
    .setTypes(Image.class, DetectedObjects.class)
    .optEngine("OnnxRuntime")
    .optProgress(new ProgressBar())
    .build();

try (ZooModel<Image, DetectedObjects> model = criteria.loadModel();
     Predictor<Image, DetectedObjects> predictor = model.newPredictor()) {
    DetectedObjects detection = predictor.predict(img);
}
```

---

## 2. 自适应去噪与锐化 (Denoising & Sharpening)

**目标**: 去除餐厅暗光噪点，增强文字边缘

| Library    | Method                   | Performance             |
| ---------- | ------------------------ | ----------------------- |
| **OpenCV** | `fastNlMeansDenoising()` | 效果好，速度较慢        |
| **OpenCV** | `GaussianBlur()`         | 快速，简单去噪          |
| **BoofCV** | `GBlurImageOps`          | 纯 Java，无 native 依赖 |

### Code Examples

```java
// Non-Local Means 去噪 (推荐用于暗光场景)
Photo.fastNlMeansDenoising(src, denoised, 10, 7, 21);
// 参数: h=10 (滤波强度), templateWindowSize=7, searchWindowSize=21

// Unsharp Mask 锐化 (比 Laplacian 更可控)
Mat blurred = new Mat();
Imgproc.GaussianBlur(src, blurred, new Size(0, 0), 3);
Core.addWeighted(src, 1.5, blurred, -0.5, 0, sharpened);
// alpha=1.5 (原图权重), beta=-0.5 (模糊图权重)
```

```java
// BoofCV 锐化
GrayU8 input = ConvertBufferedImage.convertFrom(bufferedImage, (GrayU8)null);
GrayU8 output = input.createSameShape();
EnhanceImageOps.sharpen4(input, output);
```

### 参数调优建议

| 参数             | 推荐值  | 说明           |
| ---------------- | ------- | -------------- |
| 去噪强度 (h)     | 8-12    | 过高会丢失细节 |
| 锐化系数 (alpha) | 1.3-1.8 | 过高会产生光晕 |
| 高斯核大小       | 3-5     | 文字场景用小核 |

---

## 3. 超分辨率重建 (Super Resolution)

**目标**: 4倍超分提升模型识别准确度

| Library              | Model       | Scale | Notes               |
| -------------------- | ----------- | ----- | ------------------- |
| **DJL + TensorFlow** | ESRGAN      | 4x    | 效果最好，计算量大  |
| **DJL + ONNX**       | Real-ESRGAN | 4x    | ONNX 格式，推理更快 |
| **OpenCV DNN**       | Custom ONNX | 2x/4x | 需自行导出模型      |

### Code Example

```java
// DJL ESRGAN 超分辨率
public class SuperResolution {

    public static Image enhance(Image input) throws Exception {
        Criteria<Image, Image> criteria = Criteria.builder()
            .optApplication(Application.CV.IMAGE_ENHANCEMENT)
            .setTypes(Image.class, Image.class)
            .optModelUrls("https://tfhub.dev/captain-pool/esrgan-tf2/1")
            .optEngine("TensorFlow")
            .optProgress(new ProgressBar())
            .build();

        try (ZooModel<Image, Image> model = criteria.loadModel();
             Predictor<Image, Image> predictor = model.newPredictor()) {
            return predictor.predict(input);
        }
    }
}
```

### 性能优化建议

- **按需触发**: 检测到低分辨率 (< 720p) 时才启用
- **GPU 加速**: 配置 CUDA 可显著提升速度
- **批处理**: 多图同时处理提高吞吐量

---

## 4. 智能矫正与去眩光 (Perspective Transform & CLAHE)

**目标**: 透视校正恢复正视矩形，CLAHE 缓解反光

| Library    | Feature    | Method                                            |
| ---------- | ---------- | ------------------------------------------------- |
| **OpenCV** | 透视变换   | `getPerspectiveTransform()` + `warpPerspective()` |
| **OpenCV** | CLAHE      | `createCLAHE()`                                   |
| **OpenCV** | 角点检测   | `goodFeaturesToTrack()` / Harris                  |
| **BoofCV** | 直方图均衡 | `EnhanceImageOps.equalizeHistogram()`             |

### Code Examples

```java
// 透视变换 - 四角校正
public static Mat perspectiveCorrect(Mat src, Point[] corners) {
    // 源点 (检测到的四角)
    MatOfPoint2f srcPoints = new MatOfPoint2f(corners);

    // 目标点 (正视矩形)
    int width = 800, height = 600;
    MatOfPoint2f dstPoints = new MatOfPoint2f(
        new Point(0, 0),
        new Point(width - 1, 0),
        new Point(width - 1, height - 1),
        new Point(0, height - 1)
    );

    // 计算变换矩阵并应用
    Mat transform = Imgproc.getPerspectiveTransform(srcPoints, dstPoints);
    Mat dst = new Mat();
    Imgproc.warpPerspective(src, dst, transform, new Size(width, height));

    return dst;
}
```

```java
// CLAHE 自适应直方图均衡化
public static Mat applyCLAHE(Mat src) {
    Mat gray = new Mat();
    Imgproc.cvtColor(src, gray, Imgproc.COLOR_BGR2GRAY);

    // clipLimit=2.0 控制对比度放大限制
    // tileGridSize=8x8 局部区域大小
    CLAHE clahe = Imgproc.createCLAHE(2.0, new Size(8, 8));

    Mat enhanced = new Mat();
    clahe.apply(gray, enhanced);

    return enhanced;
}
```

### 四角检测方法

```java
// Harris 角点 + RANSAC
MatOfPoint corners = new MatOfPoint();
Imgproc.goodFeaturesToTrack(gray, corners, 4, 0.01, 10, new Mat(), 3, true, 0.04);

// 或使用 Canny + Hough 线检测
Imgproc.Canny(gray, edges, 50, 150);
Mat lines = new Mat();
Imgproc.HoughLinesP(edges, lines, 1, Math.PI/180, 50, 50, 10);
```

---

## Maven Dependencies

```xml
<dependencies>
    <!-- JavaCV (OpenCV Java wrapper) -->
    <dependency>
        <groupId>org.bytedeco</groupId>
        <artifactId>javacv-platform</artifactId>
        <version>1.5.9</version>
    </dependency>

    <!-- DJL Core API -->
    <dependency>
        <groupId>ai.djl</groupId>
        <artifactId>api</artifactId>
        <version>0.29.0</version>
    </dependency>

    <!-- DJL Model Zoo -->
    <dependency>
        <groupId>ai.djl</groupId>
        <artifactId>model-zoo</artifactId>
        <version>0.29.0</version>
    </dependency>

    <!-- DJL TensorFlow Engine (for ESRGAN) -->
    <dependency>
        <groupId>ai.djl.tensorflow</groupId>
        <artifactId>tensorflow-engine</artifactId>
        <version>0.29.0</version>
        <scope>runtime</scope>
    </dependency>

    <!-- DJL ONNX Runtime Engine (for YOLO) -->
    <dependency>
        <groupId>ai.djl.onnxruntime</groupId>
        <artifactId>onnxruntime-engine</artifactId>
        <version>0.29.0</version>
        <scope>runtime</scope>
    </dependency>

    <!-- BoofCV (Pure Java alternative) -->
    <dependency>
        <groupId>org.boofcv</groupId>
        <artifactId>boofcv-core</artifactId>
        <version>1.1.4</version>
    </dependency>
</dependencies>
```

---

## Library Comparison Matrix

| Feature         | OpenCV (JavaCV) | DJL         | BoofCV     |
| --------------- | --------------- | ----------- | ---------- |
| **Native 依赖** | ✅ 需要         | ✅ 需要     | ❌ 纯 Java |
| **深度学习**    | DNN 模块        | ✅ 完整支持 | ❌ 不支持  |
| **文档质量**    | ⭐⭐⭐⭐⭐      | ⭐⭐⭐⭐    | ⭐⭐⭐     |
| **性能**        | ⭐⭐⭐⭐⭐      | ⭐⭐⭐⭐    | ⭐⭐⭐     |
| **易用性**      | ⭐⭐⭐          | ⭐⭐⭐⭐    | ⭐⭐⭐⭐   |
| **社区活跃度**  | ⭐⭐⭐⭐⭐      | ⭐⭐⭐⭐    | ⭐⭐⭐     |

---

## Recommended Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Menu Image Input                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  1. Menu Detection & Cropping                               │
│     └─ DJL + YOLO (complex) / OpenCV Contours (simple)     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Perspective Correction                                  │
│     └─ OpenCV getPerspectiveTransform + warpPerspective    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Denoising & Enhancement                                 │
│     └─ OpenCV fastNlMeansDenoising + CLAHE                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  4. Super Resolution (conditional)                          │
│     └─ DJL + ESRGAN (if resolution < threshold)            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  5. Sharpening                                              │
│     └─ OpenCV Unsharp Mask                                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 Enhanced Menu Image Output                  │
│                    (Ready for OCR/LLM)                      │
└─────────────────────────────────────────────────────────────┘
```

---

## References

- [OpenCV Java Documentation](https://docs.opencv.org/4.x/d9/d52/tutorial_java_dev_intro.html)
- [Deep Java Library (DJL)](https://djl.ai/)
- [DJL Super Resolution Example](http://djl.ai/examples/docs/super_resolution.html)
- [DJL Object Detection Example](http://djl.ai/examples/docs/object_detection.html)
- [BoofCV Documentation](https://boofcv.org/)
- [JavaCV GitHub](https://github.com/bytedeco/javacv)

---

_Last Updated: January 2026_
