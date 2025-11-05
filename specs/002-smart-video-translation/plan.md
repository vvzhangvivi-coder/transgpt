# 实现计划：智能视频翻译

**功能分支**: `002-smart-video-translation` | **日期**: 2025-10-22 | **规格**: [spec.md](./spec.md)
**输入**: 来自 `/specs/002-smart-video-translation/spec.md` 的功能规格说明

## 概述

智能视频翻译是一个创新的实时翻译应用，通过摄像头实时识别画面中的文字并提供即时翻译和语音播报。核心技术方案采用 React Native 跨平台框架，结合云端 OCR/翻译 API 和本地实时处理能力，实现低延迟（<2秒）的实时翻译体验。关键创新点包括画面锁定功能（解决长时间举手机痛点）和防打断机制（确保翻译连续性）。

## 技术上下文

**语言/版本**: React Native 0.72+ (TypeScript 5.0+)  
**主要依赖**: 
- 摄像头：`react-native-vision-camera` 3.x (高性能实时捕获)
- OCR服务：Google Cloud Vision API (备选: AWS Textract)
- 翻译服务：Google Cloud Translation API (备选: DeepL API)
- TTS语音：`react-native-tts` + 系统原生引擎 (iOS: AVSpeechSynthesizer, Android: TextToSpeech)
- AI问答：OpenAI GPT-4 API (备选: Claude API)
- 状态管理：Zustand 4.x (轻量级、简单易用)
- 本地存储：AsyncStorage (配置持久化)
- 网络请求：Axios + React Query (请求缓存和重试)

**存储**: 
- 本地：AsyncStorage (用户设置、语言偏好、API密钥)
- 缓存：React Query (翻译结果缓存、减少重复请求)
- 不存储：翻译历史、用户画面（隐私保护）

**测试**: Jest + React Native Testing Library + Detox (E2E)  
**目标平台**: iOS 13+ / Android 8.0+ (主流设备覆盖率>95%)  
**项目类型**: 移动应用 (跨平台)  
**性能目标**: 
- 识别延迟：<2秒 (从画面稳定到显示翻译)
- 帧率：15fps (摄像头预览)
- 播报延迟：<1秒 (点击到开始播放)
- 内存占用：<150MB (正常使用)

**约束**: 
- 网络依赖：需要稳定网络连接（<500ms延迟）
- API配额：每日翻译量受限于云服务配额
- 电池消耗：30分钟使用<30%电量
- 发热控制：温度上升<5°C

**规模/范围**: 
- 预计用户：初期1000-10000用户
- 支持语言：4种（中/英/日/韩）
- 屏幕数量：5个主要屏幕
- 核心功能：5个用户故事

## 宪章检查

*必须在阶段0研究前通过。在阶段1设计后重新检查。*

- ✅ **简单性原则**: 项目结构采用单一移动应用架构，避免不必要的微服务拆分
- ✅ **依赖最小化**: 仅使用必要的核心库，优先系统原生能力（如TTS）
- ✅ **性能优先**: 采用帧采样、结果缓存、防抖等策略优化实时性能
- ✅ **隐私保护**: 不存储用户画面和翻译历史，符合隐私优先原则
- ⚠️ **测试覆盖**: 需确保核心功能有完整的单元和集成测试（目标80%覆盖率）

## 项目结构

### 文档（本功能）

```
specs/002-smart-video-translation/
├── plan.md              # 本文件（实现计划）
├── spec.md              # 功能规格说明（已创建）
├── research.md          # 技术研究（阶段0）
├── data-model.md        # 数据模型设计（阶段1）
├── quickstart.md        # 快速开始指南（阶段1）
├── contracts/           # 接口契约（阶段1）
│   ├── ocr-service.md
│   ├── translation-service.md
│   ├── tts-service.md
│   └── ai-service.md
└── tasks.md             # 任务清单（阶段2，由 /speckit.tasks 生成）
```

### 源代码（仓库根目录）

```
transgpt/
├── src/
│   ├── screens/                    # 屏幕组件
│   │   ├── CameraTranslationScreen.tsx   # 主翻译屏幕
│   │   ├── SettingsScreen.tsx            # 设置页面
│   │   └── PermissionGuideScreen.tsx     # 权限引导页
│   │
│   ├── components/                 # UI组件
│   │   ├── camera/
│   │   │   ├── CameraView.tsx           # 摄像头视图
│   │   │   ├── FocusIndicator.tsx       # 对焦指示器
│   │   │   └── QualityWarning.tsx       # 图像质量警告
│   │   ├── translation/
│   │   │   ├── SubtitleOverlay.tsx      # 字幕叠加层
│   │   │   ├── TranslationCard.tsx      # 翻译卡片
│   │   │   └── LanguageSelector.tsx     # 语言选择器
│   │   ├── controls/
│   │   │   ├── LockButton.tsx           # 画面锁定按钮
│   │   │   ├── TTSControls.tsx          # 播报控制按钮
│   │   │   ├── ModeSwitch.tsx           # 模式切换按钮
│   │   │   └── VoiceInput.tsx           # 语音输入组件
│   │   └── common/
│   │       ├── LoadingIndicator.tsx     # 加载指示器
│   │       └── ErrorBoundary.tsx        # 错误边界
│   │
│   ├── services/                   # 业务服务
│   │   ├── ocr/
│   │   │   ├── OCRService.ts            # OCR服务抽象
│   │   │   ├── GoogleVisionOCR.ts       # Google Vision实现
│   │   │   └── OCRCache.ts              # OCR结果缓存
│   │   ├── translation/
│   │   │   ├── TranslationService.ts    # 翻译服务抽象
│   │   │   ├── GoogleTranslation.ts     # Google翻译实现
│   │   │   └── TranslationCache.ts      # 翻译缓存
│   │   ├── tts/
│   │   │   ├── TTSService.ts            # TTS服务抽象
│   │   │   └── NativeTTS.ts             # 系统原生TTS实现
│   │   ├── ai/
│   │   │   ├── AIService.ts             # AI问答服务
│   │   │   └── OpenAIProvider.ts        # OpenAI实现
│   │   ├── camera/
│   │   │   ├── CameraManager.ts         # 摄像头管理
│   │   │   └── FrameProcessor.ts        # 帧处理器
│   │   └── permission/
│   │       └── PermissionManager.ts     # 权限管理
│   │
│   ├── hooks/                      # 自定义Hooks
│   │   ├── useCamera.ts                 # 摄像头控制Hook
│   │   ├── useOCR.ts                    # OCR处理Hook
│   │   ├── useTranslation.ts            # 翻译Hook
│   │   ├── useTTS.ts                    # TTS控制Hook
│   │   ├── useFrameLock.ts              # 画面锁定Hook
│   │   ├── useAppMode.ts                # 模式切换Hook
│   │   └── useImageQuality.ts           # 图像质量检测Hook
│   │
│   ├── store/                      # 状态管理
│   │   ├── appStore.ts                  # 全局应用状态
│   │   ├── translationStore.ts          # 翻译状态
│   │   ├── settingsStore.ts             # 用户设置状态
│   │   └── cameraStore.ts               # 摄像头状态
│   │
│   ├── utils/                      # 工具函数
│   │   ├── imageQuality.ts              # 图像质量评估
│   │   ├── debounce.ts                  # 防抖工具
│   │   ├── throttle.ts                  # 节流工具
│   │   ├── logger.ts                    # 日志工具
│   │   └── errorHandler.ts              # 错误处理
│   │
│   ├── types/                      # TypeScript类型定义
│   │   ├── camera.ts
│   │   ├── translation.ts
│   │   ├── ocr.ts
│   │   └── common.ts
│   │
│   ├── config/                     # 配置文件
│   │   ├── api.ts                       # API配置
│   │   ├── languages.ts                 # 语言配置
│   │   └── constants.ts                 # 常量定义
│   │
│   └── App.tsx                     # 应用入口
│
├── tests/                          # 测试文件
│   ├── unit/                       # 单元测试
│   │   ├── services/
│   │   ├── hooks/
│   │   └── utils/
│   ├── integration/                # 集成测试
│   │   ├── translation-flow.test.ts
│   │   ├── lock-mode.test.ts
│   │   └── tts-integration.test.ts
│   └── e2e/                        # E2E测试
│       ├── main-flow.e2e.ts
│       └── permission.e2e.ts
│
├── android/                        # Android原生代码
├── ios/                            # iOS原生代码
├── package.json
├── tsconfig.json
└── README.md
```

**结构决策**: 采用标准的 React Native 单应用架构，按功能模块清晰划分目录。services/ 层负责外部API调用和业务逻辑，hooks/ 层封装可复用的逻辑，components/ 层专注UI展示。这种分层保证了代码的可维护性和可测试性。

## 复杂度追踪

*仅在宪章检查有违规需要说明时填写*

| 违规项 | 需要的原因 | 拒绝更简单方案的理由 |
|--------|-----------|---------------------|
| N/A    | N/A       | N/A                 |

## 核心技术决策

### 1. 实时处理流程设计

**问题**: 如何在保证用户体验（<2秒延迟）的同时避免频繁调用昂贵的云端API？

**方案**: 
- **帧采样策略**: 摄像头以15fps运行，但仅每1秒采样1帧进行OCR识别（降低API调用频率）
- **画面稳定检测**: 计算连续帧之间的差异（使用简单的像素差异算法），仅在画面稳定（差异<5%）后触发识别
- **防抖机制**: 使用500ms防抖，避免用户快速移动手机时频繁触发识别
- **结果缓存**: 使用内容哈希（MD5）缓存已识别的文本和翻译结果，相同内容直接返回缓存
- **渐进式显示**: 对于长文本，先显示部分翻译结果，后台继续处理完整内容

**权衡**: 帧采样会略微增加延迟（最多1秒），但能将API调用量降低93%（15fps → 1fps），显著降低成本和服务器负载。

### 2. 画面锁定实现方式

**问题**: 如何实现"锁定画面"功能，让用户可以放下手机但仍能继续查看和翻译内容？

**方案**:
- **帧捕获**: 用户点击"锁定"按钮时，立即捕获当前帧的完整图像（高分辨率JPEG，压缩质量85%）
- **相机暂停**: 暂停相机预览流（但不关闭相机，避免重启开销），UI显示冻结的图像
- **独立识别**: 对锁定的高质量图像进行完整OCR识别（不受帧采样限制），允许更长的处理时间（5-10秒）
- **状态管理**: 使用 `cameraStore` 中的 `isLocked` 状态控制UI和相机行为，确保锁定/解锁的一致性
- **快速恢复**: 用户点击"解锁"时，立即恢复相机预览流，无需重新初始化

**权衡**: 锁定模式需要额外的内存存储图像（约2-5MB），但换取了用户体验的巨大提升（无需长时间举手机）。

### 3. 防打断机制设计

**问题**: 如何防止翻译过程被环境噪音或旁人说话打断？

**方案**:
- **模式隔离**: 应用默认处于"纯翻译模式"，此模式下完全不启动麦克风，从根本上杜绝语音干扰
- **显式切换**: 用户需要长按"问答"按钮（1.5秒）才能激活麦克风和语音输入，避免误触
- **自动回退**: 问答模式下，如果30秒内无用户输入或完成问答后，自动切换回翻译模式并关闭麦克风
- **视觉反馈**: UI明确显示当前模式（翻译模式：绿色边框，问答模式：蓝色边框+麦克风图标）
- **TTS优先级**: 当TTS播报时，即使在问答模式也暂时禁用语音输入，防止回声干扰

**权衡**: 这种设计牺牲了"语音唤醒问答"的便利性，但换取了翻译功能的稳定性和可靠性，符合"默认翻译、辅助问答"的产品定位。

### 4. OCR服务选择

**问题**: 选择哪个OCR服务？考虑准确率、延迟、成本、多语言支持。

**方案**: **主选Google Cloud Vision API**，原因：
- **准确率**: 行业领先，对中英日韩的识别准确率>95%
- **延迟**: 全球CDN，平均响应<500ms
- **多语言**: 原生支持100+语言，无需切换模型
- **成本**: 前1000次/月免费，后续$1.5/1000次，在预算内
- **易用性**: RESTful API，集成简单

**备选**: AWS Textract（在Google Vision不可用时自动降级），Tesseract.js（离线备用，准确率较低）

**技术实现**: 使用适配器模式封装 `OCRService` 接口，方便切换实现：
```typescript
interface OCRService {
  detectText(imageData: string): Promise<OCRResult>;
  getSupportedLanguages(): string[];
}
```

### 5. 翻译服务选择

**问题**: 选择哪个翻译服务？考虑质量、成本、延迟。

**方案**: **主选Google Cloud Translation API**，原因：
- **翻译质量**: 神经机器翻译，质量接近专业翻译
- **成本**: $20/1M字符，假设平均50字/次翻译，约$1/1000次
- **延迟**: <300ms（文本翻译比OCR更快）
- **与OCR集成**: 同一云平台，便于配额管理和账单统计

**备选**: DeepL API（翻译质量更高但成本高2-3倍，作为付费增值选项）

### 6. 性能优化策略

**关键优化点**:

1. **图像预处理**:
   - 在设备端进行图像压缩和裁剪，减少上传数据量（从2MB压缩到200KB）
   - 对模糊图像进行简单的锐化处理（OpenCV.js），提高识别率

2. **并行处理**:
   - OCR和翻译分离：OCR完成后立即显示原文，翻译在后台并行进行
   - 长文本分段翻译：将大段文字切分为3-5个子段，并行翻译后合并

3. **智能缓存**:
   - React Query管理API响应缓存，5分钟内相同内容直接返回缓存
   - 使用LRU策略限制缓存大小（最多100条记录），防止内存溢出

4. **降级方案**:
   - 网络慢时显示"识别中..."进度条，超过5秒显示"网络较慢，请稍候"
   - API失败时重试3次（指数退避），失败后提示用户检查网络

5. **电池和发热控制**:
   - 摄像头使用中等分辨率（720p而非1080p），降低GPU负载
   - 后台时自动暂停相机和API调用
   - 长时间使用（>20分钟）弹出提示"建议休息片刻，避免设备过热"

### 7. 错误处理和降级

**错误分类和处理**:

| 错误类型 | 检测方式 | 用户提示 | 系统行为 |
|---------|---------|---------|---------|
| 权限拒绝 | 启动时检测 | 显示权限引导页 | 阻断核心功能，引导用户到设置 |
| 网络不可用 | API调用失败 | "网络不可用，请检查连接" | 禁用翻译功能，允许浏览已缓存内容 |
| API配额耗尽 | HTTP 429错误 | "今日翻译次数已达上限" | 停止新翻译，显示升级选项 |
| 图像质量差 | 质量评分<0.3 | "图像模糊，请调整角度" | 继续允许翻译但降低信心度 |
| OCR无结果 | 返回空文本 | "未检测到文字" | 提示调整角度或光线 |
| 翻译超时 | >5秒无响应 | "翻译超时，请重试" | 取消请求，允许用户手动重试 |

**降级策略**:
- Google Vision不可用 → 降级到AWS Textract → 进一步降级到本地Tesseract.js（提示准确率降低）
- Google Translate不可用 → 降级到DeepL（如果用户配置了API Key）
- TTS失败 → 禁用播报功能，显示"语音服务暂不可用"

## 依赖库清单

### 核心依赖

```json
{
  "dependencies": {
    "react": "18.2.0",
    "react-native": "0.72.6",
    "react-native-vision-camera": "^3.6.0",
    "react-native-tts": "^4.1.0",
    "zustand": "^4.4.0",
    "@react-native-async-storage/async-storage": "^1.19.0",
    "axios": "^1.5.0",
    "@tanstack/react-query": "^5.0.0",
    "react-native-svg": "^13.14.0",
    "react-native-gesture-handler": "^2.13.0",
    "react-native-reanimated": "^3.5.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-native": "^0.72.0",
    "typescript": "^5.0.0",
    "jest": "^29.5.0",
    "@testing-library/react-native": "^12.3.0",
    "detox": "^20.13.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  }
}
```

### 云服务SDK

- **Google Cloud Vision**: 通过REST API直接调用（无需额外SDK，减少包体积）
- **Google Cloud Translation**: 通过REST API直接调用
- **OpenAI**: 官方SDK `openai` 4.x

### 原生模块

- **相机**: react-native-vision-camera（支持帧处理和高性能捕获）
- **TTS**: react-native-tts（封装iOS AVSpeechSynthesizer和Android TextToSpeech）
- **权限**: react-native-permissions（统一管理iOS和Android权限）

### 工具库

- **图像处理**: 使用原生Canvas API进行基础处理（压缩、裁剪），避免引入重型库
- **状态管理**: Zustand（比Redux更轻量，无需boilerplate代码）
- **缓存**: React Query（自动管理请求缓存、重试、状态）
- **日志**: 自定义轻量级logger（生产环境可对接Sentry）

## 开发阶段规划

### 阶段0: 技术研究与验证（1周）

**目标**: 验证核心技术可行性，确认API性能和成本

**任务**:
1. 搭建React Native基础项目（TypeScript + Zustand）
2. 集成react-native-vision-camera，验证帧捕获性能
3. 测试Google Cloud Vision API延迟和准确率（使用测试数据集）
4. 测试Google Translation API延迟和质量
5. 验证iOS/Android原生TTS能力和语音质量
6. 编写技术研究报告（research.md）

**交付物**:
- 可运行的Demo（摄像头预览 + 单次OCR调用）
- API性能测试报告（延迟、准确率、成本估算）
- research.md文档

### 阶段1: 核心架构设计（1周）

**目标**: 完成详细的架构设计和接口定义

**任务**:
1. 设计数据模型（相机状态、翻译结果、用户设置等）
2. 定义服务接口契约（OCR、翻译、TTS、AI问答）
3. 设计状态管理结构（Zustand stores）
4. 设计组件层次结构和复用策略
5. 编写快速开始指南（quickstart.md）
6. 编写接口契约文档（contracts/）

**交付物**:
- data-model.md（完整数据模型定义）
- quickstart.md（开发者上手指南）
- contracts/（4个服务接口文档）

### 阶段2: 任务分解与实现（5周）

**目标**: 按优先级实现所有用户故事

**子阶段**:
- **Week 1**: P1功能 - 实时视频翻译基础（FR-001至FR-004）
- **Week 2**: P1功能 - 画面锁定功能（FR-005至FR-006）
- **Week 3**: P2功能 - 语音播报功能（FR-007至FR-009）
- **Week 4**: P2功能 - 防打断机制（FR-010至FR-011）
- **Week 5**: P3功能 - 智能问答功能（FR-012至FR-013）

**交付物**:
- tasks.md（详细任务清单，由 /speckit.tasks 生成）
- 每周可演示的功能增量
- 单元测试和集成测试（覆盖率>80%）

### 阶段3: 测试与优化（2周）

**目标**: 完成端到端测试，优化性能和用户体验

**任务**:
1. 编写E2E测试（Detox）
2. 性能优化（内存、电池、发热）
3. UI/UX打磨（动画、过渡、加载状态）
4. 错误处理完善和边界情况测试
5. 真实场景测试（不同光线、角度、内容类型）
6. Beta测试和用户反馈收集

**交付物**:
- 完整的测试套件（单元、集成、E2E）
- 性能测试报告（帧率、延迟、内存、电量）
- Beta版本APK/IPA

### 阶段4: 发布准备（1周）

**目标**: 准备生产发布

**任务**:
1. 配置生产环境API密钥和配额
2. 添加错误监控（Sentry集成）
3. 准备应用商店素材（截图、描述、视频）
4. 编写用户文档和帮助页面
5. 最终安全和隐私审查

**交付物**:
- 生产版本（iOS App Store + Google Play）
- 用户手册和常见问题
- 运维监控面板

## 风险和缓解措施

| 风险 | 影响 | 概率 | 缓解措施 |
|-----|------|------|---------|
| API成本超预算 | 高 | 中 | 1. 严格的缓存策略 2. 用户配额限制 3. 备选的更便宜API |
| 实时性能不达标 | 高 | 中 | 1. 提前进行性能测试 2. 帧采样优化 3. 降级到更低分辨率 |
| OCR准确率低 | 中 | 低 | 1. 图像质量预检 2. 用户引导（调整角度/光线） 3. 备选OCR服务 |
| 跨平台兼容性问题 | 中 | 中 | 1. 持续的iOS/Android设备测试 2. 使用成熟的跨平台库 |
| 权限被拒绝 | 中 | 高 | 1. 友好的权限引导页面 2. 明确说明权限用途 |
| 网络不稳定 | 中 | 高 | 1. 请求重试机制 2. 离线降级方案（Tesseract.js） |
| 电池消耗过高 | 低 | 中 | 1. 优化相机分辨率 2. 后台暂停处理 3. 用户提醒 |
| 第三方API服务中断 | 高 | 低 | 1. 多个备选服务 2. 优雅降级 3. 明确告知用户 |

## 成功标准验证计划

基于spec.md中的成功标准，验证方式：

- **SC-001（延迟<3秒）**: 自动化性能测试，记录100次翻译的平均延迟
- **SC-002（准确率>95%/90%）**: 使用标准测试数据集（包含中英日韩各100个样本），人工评估准确率
- **SC-003（30秒不中断）**: E2E测试，播放背景噪音，验证翻译持续性
- **SC-004（锁定响应<0.5秒）**: 性能测试，测量点击到UI更新的时间
- **SC-005（播报延迟<1秒）**: 自动化测试，测量TTS启动时间
- **SC-006（90%用户理解）**: Beta测试用户调研，首次使用成功率统计
- **SC-007（30分钟<5°C/<30%电量）**: 真机长时间测试，使用温度计和电量监控工具
- **SC-008（问答相关性>4.0/5.0）**: Beta测试用户评分，收集至少100个问答样本
- **SC-009（300ms网络延迟）**: 使用网络模拟工具（Charles Proxy）测试不同延迟下的表现
- **SC-010（满意度>4.5/5.0）**: Beta测试问卷调查（至少50个有效样本）

## 下一步行动

1. **立即**: 运行 `/speckit.plan` 命令生成 research.md（开始阶段0）
2. **第1周末**: 完成技术验证，确认API性能满足要求
3. **第2周末**: 完成架构设计，生成 data-model.md 和 contracts/
4. **第3周**: 运行 `/speckit.tasks` 命令生成详细任务清单（tasks.md）
5. **第3-7周**: 按优先级迭代开发用户故事
6. **第8-9周**: 测试和优化
7. **第10周**: 发布准备和上线

---

**文档版本**: 1.0  
**最后更新**: 2025-10-22  
**维护者**: 开发团队
