# 任务清单：智能视频翻译

**输入**: 来自 `/specs/002-smart-video-translation/` 的设计文档
**前置条件**: plan.md（必需）, spec.md（必需）, research.md, data-model.md, contracts/

**测试说明**: 本功能规格未明确要求 TDD，因此不生成测试任务

**组织方式**: 任务按用户故事分组，确保每个故事可以独立实现和测试

## 格式说明: `[ID] [P?] [Story?] 描述 + 文件路径`
- **[P]**: 可并行执行（不同文件、无依赖）
- **[Story]**: 任务所属用户故事（如 US1, US2, US3）
- Setup/Foundational 阶段不添加 Story 标签
- 描述中必须包含具体文件路径

## 路径约定
- React Native 单应用项目：`src/`, `android/`, `ios/` 在仓库根目录
- 所有路径基于 `/Users/wepie/Documents/Mywork/transgpt/` 根目录

---

## 阶段 1: Setup（项目初始化）

**目的**: 项目结构搭建和基础依赖安装

- [ ] T001 创建 React Native 项目基础结构，配置 TypeScript 和 ESLint 在根目录
- [ ] T002 安装核心依赖包（react-native 0.72+, zustand 4.x, @react-native-async-storage/async-storage）在 package.json
- [ ] T003 [P] 安装摄像头依赖 react-native-vision-camera 3.x 并配置原生权限在 android/app/src/main/AndroidManifest.xml 和 ios/transgpt/Info.plist
- [ ] T004 [P] 安装 TTS 依赖 react-native-tts 4.x 在 package.json
- [ ] T005 [P] 安装网络和缓存依赖（axios, @tanstack/react-query）在 package.json
- [ ] T006 [P] 配置 TypeScript (tsconfig.json) 和 ESLint (.eslintrc.js) 规则
- [ ] T007 配置 Android 最低 SDK 版本为 26 (Android 8.0) 在 android/build.gradle
- [ ] T008 配置 iOS 最低部署目标为 13.0 在 ios/Podfile

**检查点**: 项目结构完成，所有依赖安装成功，可以运行空白应用

---

## 阶段 2: Foundational（基础设施）

**目的**: 核心基础设施，必须在所有用户故事开始前完成

**⚠️ 关键**: 本阶段完成前，任何用户故事工作都不能开始

- [ ] T009 创建 TypeScript 类型定义文件 src/types/camera.ts（相机状态、帧数据类型）
- [ ] T010 [P] 创建 TypeScript 类型定义文件 src/types/ocr.ts（OCR 结果、检测文本类型）
- [ ] T011 [P] 创建 TypeScript 类型定义文件 src/types/translation.ts（翻译结果、语言配置类型）
- [ ] T012 [P] 创建 TypeScript 类型定义文件 src/types/common.ts（通用类型、应用模式枚举）
- [ ] T013 创建配置文件 src/config/api.ts（API 端点、密钥配置）
- [ ] T014 [P] 创建配置文件 src/config/languages.ts（支持的语言列表：中英日韩）
- [ ] T015 [P] 创建配置文件 src/config/constants.ts（帧率、采样率、超时等常量）
- [ ] T016 创建权限管理服务 src/services/permission/PermissionManager.ts（检查和请求摄像头/麦克风权限）
- [ ] T017 创建日志工具 src/utils/logger.ts（分级日志、生产环境配置）
- [ ] T018 [P] 创建错误处理工具 src/utils/errorHandler.ts（统一错误处理、错误分类）
- [ ] T019 [P] 创建防抖工具 src/utils/debounce.ts（防抖函数封装）
- [ ] T020 [P] 创建节流工具 src/utils/throttle.ts（节流函数封装）
- [ ] T021 创建全局应用状态 Store src/store/appStore.ts（应用模式、网络状态管理）
- [ ] T022 [P] 创建摄像头状态 Store src/store/cameraStore.ts（锁定状态、画面状态管理）
- [ ] T023 [P] 创建翻译状态 Store src/store/translationStore.ts（翻译结果、识别文本管理）
- [ ] T024 [P] 创建用户设置状态 Store src/store/settingsStore.ts（语言偏好、自动播报设置）
- [ ] T025 创建导航结构 src/App.tsx（主屏幕导航、权限引导流程）
- [ ] T026 创建错误边界组件 src/components/common/ErrorBoundary.tsx（全局错误捕获）
- [ ] T027 [P] 创建加载指示器组件 src/components/common/LoadingIndicator.tsx（通用加载动画）

**检查点**: 基础设施就绪，用户故事实现可以并行开始

---

## 阶段 3: User Story 1 - 实时视频翻译基础功能（优先级: P1）🎯 MVP

**目标**: 用户打开应用，将手机对准文字内容，系统在2秒内显示翻译字幕，移动手机时持续更新翻译

**独立测试**: 打开应用，对准中英文书籍/路牌，验证2秒内显示字幕，移动手机后翻译持续更新

### US1 实现任务

- [ ] T028 [P] [US1] 创建摄像头管理服务 src/services/camera/CameraManager.ts（初始化、启动、停止摄像头）
- [ ] T029 [P] [US1] 创建帧处理器服务 src/services/camera/FrameProcessor.ts（帧采样、画面稳定检测）
- [ ] T030 [US1] 创建摄像头控制 Hook src/hooks/useCamera.ts（封装摄像头生命周期、状态管理）
- [ ] T031 [P] [US1] 创建 OCR 服务抽象接口 src/services/ocr/OCRService.ts（定义 detectText 方法）
- [ ] T032 [P] [US1] 实现 Google Cloud Vision OCR 集成 src/services/ocr/GoogleVisionOCR.ts（REST API 调用、结果解析）
- [ ] T033 [US1] 创建 OCR 结果缓存服务 src/services/ocr/OCRCache.ts（基于内容哈希的缓存、LRU 策略）
- [ ] T034 [US1] 创建 OCR 处理 Hook src/hooks/useOCR.ts（触发识别、缓存查询、错误处理）
- [ ] T035 [P] [US1] 创建翻译服务抽象接口 src/services/translation/TranslationService.ts（定义 translate 方法）
- [ ] T036 [P] [US1] 实现 Google Cloud Translation 集成 src/services/translation/GoogleTranslation.ts（REST API 调用、多语言支持）
- [ ] T037 [US1] 创建翻译结果缓存服务 src/services/translation/TranslationCache.ts（翻译结果缓存、防重复请求）
- [ ] T038 [US1] 创建翻译处理 Hook src/hooks/useTranslation.ts（触发翻译、缓存管理、状态更新）
- [ ] T039 [P] [US1] 创建摄像头预览组件 src/components/camera/CameraView.tsx（使用 vision-camera 显示实时画面）
- [ ] T040 [P] [US1] 创建对焦指示器组件 src/components/camera/FocusIndicator.tsx（显示对焦状态、画面稳定提示）
- [ ] T041 [P] [US1] 创建图像质量警告组件 src/components/camera/QualityWarning.tsx（光线不足、模糊提示）
- [ ] T042 [US1] 创建图像质量评估工具 src/utils/imageQuality.ts（评估模糊度、光线充足度）
- [ ] T043 [US1] 创建图像质量检测 Hook src/hooks/useImageQuality.ts（实时质量评分、警告触发）
- [ ] T044 [P] [US1] 创建字幕叠加层组件 src/components/translation/SubtitleOverlay.tsx（半透明字幕显示）
- [ ] T045 [P] [US1] 创建翻译卡片组件 src/components/translation/TranslationCard.tsx（格式化显示翻译结果）
- [ ] T046 [P] [US1] 创建语言选择器组件 src/components/translation/LanguageSelector.tsx（源语言和目标语言切换）
- [ ] T047 [US1] 创建权限引导屏幕 src/screens/PermissionGuideScreen.tsx（引导用户授权摄像头权限）
- [ ] T048 [US1] 集成所有组件到主翻译屏幕 src/screens/CameraTranslationScreen.tsx（摄像头预览 + 实时翻译 + 字幕显示）
- [ ] T049 [US1] 实现帧采样策略（1秒采样1帧）和防抖逻辑（500ms）在 src/hooks/useCamera.ts
- [ ] T050 [US1] 实现画面稳定检测（连续帧差异<5%才触发识别）在 src/services/camera/FrameProcessor.ts
- [ ] T051 [US1] 处理网络错误和 API 失败重试（3次指数退避）在 src/hooks/useOCR.ts 和 src/hooks/useTranslation.ts
- [ ] T052 [US1] 添加识别和翻译的加载状态显示在 src/screens/CameraTranslationScreen.tsx

**检查点**: User Story 1 完整可用，可独立测试实时翻译功能，满足 FR-001 至 FR-004

---

## 阶段 4: User Story 2 - 画面锁定功能（优先级: P1）🎯 MVP

**目标**: 用户对准长文本后点击"锁定"按钮，画面固定，用户可放下手机，系统对静态图像进行完整翻译

**独立测试**: 在实时翻译基础上，对准一页书籍，点击"锁定"，验证画面固定、翻译继续、可放下手机、可重新拍摄

### US2 实现任务

- [ ] T053 [P] [US2] 创建画面锁定按钮组件 src/components/controls/LockButton.tsx（锁定/解锁状态切换、视觉反馈）
- [ ] T054 [P] [US2] 创建重新拍摄按钮组件 src/components/controls/RetakeButton.tsx（锁定模式下重新捕获画面）
- [ ] T055 [US2] 实现画面锁定逻辑在 src/hooks/useFrameLock.ts（捕获当前帧、暂停相机预览、恢复预览）
- [ ] T056 [US2] 在摄像头状态 Store 添加锁定模式管理 src/store/cameraStore.ts（isLocked 状态、lockedFrame 数据）
- [ ] T057 [US2] 实现高质量图像捕获（720p JPEG 85%压缩）在 src/services/camera/CameraManager.ts
- [ ] T058 [US2] 实现锁定画面的完整 OCR 识别（不受帧采样限制）在 src/hooks/useOCR.ts
- [ ] T059 [US2] 在主翻译屏幕添加锁定按钮和状态显示 src/screens/CameraTranslationScreen.tsx
- [ ] T060 [US2] 实现锁定模式下的 UI 切换（显示静态图像、隐藏实时预览）在 src/screens/CameraTranslationScreen.tsx
- [ ] T061 [US2] 实现解锁功能（恢复实时相机流、清除锁定状态）在 src/hooks/useFrameLock.ts
- [ ] T062 [US2] 添加锁定/解锁的视觉提示（"画面已锁定"、"画面已解锁"Toast）在 src/screens/CameraTranslationScreen.tsx
- [ ] T063 [US2] 处理锁定图像质量不佳的情况（评分<0.3 时提示重新锁定）在 src/hooks/useImageQuality.ts
- [ ] T064 [US2] 实现长文本的进度显示（处理中显示进度条）在 src/components/translation/TranslationCard.tsx
- [ ] T065 [US2] 优化锁定模式下的内存管理（及时释放未使用的帧数据）在 src/store/cameraStore.ts

**检查点**: User Story 2 完整可用，画面锁定功能正常工作，满足 FR-005 和 FR-006

---

## 阶段 5: User Story 3 - 语音播报功能（优先级: P2）

**目标**: 用户在查看翻译时，点击"播报"按钮，系统用目标语言朗读翻译结果，支持暂停/继续和自动播报模式

**独立测试**: 在已有翻译结果的基础上，点击播报按钮验证发音，测试暂停/继续、音量调节、自动播报模式

### US3 实现任务

- [ ] T066 [P] [US3] 创建 TTS 服务抽象接口 src/services/tts/TTSService.ts（定义 speak、pause、resume、stop 方法）
- [ ] T067 [P] [US3] 实现系统原生 TTS 集成 src/services/tts/NativeTTS.ts（iOS AVSpeechSynthesizer 和 Android TextToSpeech）
- [ ] T068 [US3] 创建 TTS 控制 Hook src/hooks/useTTS.ts（播报状态管理、音量控制、错误处理）
- [ ] T069 [P] [US3] 创建播报控制按钮组件 src/components/controls/TTSControls.tsx（播放/暂停/停止按钮）
- [ ] T070 [P] [US3] 创建音量控制组件 src/components/controls/VolumeControl.tsx（滑块调节音量）
- [ ] T071 [US3] 在翻译卡片组件添加播报按钮 src/components/translation/TranslationCard.tsx
- [ ] T072 [US3] 实现播报任务状态管理（播放中、暂停、完成）在 src/hooks/useTTS.ts
- [ ] T073 [US3] 实现翻译内容变化时自动触发播报（自动播报模式）在 src/screens/CameraTranslationScreen.tsx
- [ ] T074 [US3] 在用户设置 Store 添加自动播报开关 src/store/settingsStore.ts
- [ ] T075 [US3] 实现播报进度显示（当前播报内容高亮）在 src/components/translation/TranslationCard.tsx
- [ ] T076 [US3] 处理播报被其他音频打断的情况（电话、其他应用音频）在 src/hooks/useTTS.ts
- [ ] T077 [US3] 实现切换翻译内容时自动停止当前播报在 src/hooks/useTTS.ts
- [ ] T078 [US3] 添加播报错误处理（TTS 服务不可用时的提示）在 src/hooks/useTTS.ts
- [ ] T079 [US3] 在设置屏幕添加语音播报设置项 src/screens/SettingsScreen.tsx（自动播报开关、语速调节）

**检查点**: User Story 3 完整可用，语音播报功能正常工作，满足 FR-007、FR-008 和 FR-009

---

## 阶段 6: User Story 4 - 防打断功能（优先级: P2）

**目标**: 翻译模式下过滤环境噪音，防止翻译被打断，用户需长按按钮才能切换到问答模式

**独立测试**: 在翻译模式下测试环境噪音干扰，验证翻译不被打断，测试长按切换到问答模式，验证自动回退

### US4 实现任务

- [ ] T080 [US4] 在应用状态 Store 添加模式管理 src/store/appStore.ts（translationMode、qaMode 切换逻辑）
- [ ] T081 [US4] 创建应用模式控制 Hook src/hooks/useAppMode.ts（模式切换、自动回退定时器）
- [ ] T082 [P] [US4] 创建模式切换按钮组件 src/components/controls/ModeSwitch.tsx（长按1.5秒切换到问答模式）
- [ ] T083 [P] [US4] 创建模式指示器组件 src/components/common/ModeIndicator.tsx（显示当前模式：翻译/问答）
- [ ] T084 [US4] 实现翻译模式下禁用麦克风（不启动语音监听）在 src/hooks/useAppMode.ts
- [ ] T085 [US4] 实现问答模式自动回退逻辑（30秒无操作自动切回翻译模式）在 src/hooks/useAppMode.ts
- [ ] T086 [US4] 实现模式切换的视觉反馈（翻译模式绿色边框、问答模式蓝色边框+麦克风图标）在 src/screens/CameraTranslationScreen.tsx
- [ ] T087 [US4] 实现 TTS 播报时禁用语音输入（防止回声干扰）在 src/hooks/useAppMode.ts
- [ ] T088 [US4] 在设置屏幕添加防打断增强选项 src/screens/SettingsScreen.tsx（调节噪音过滤阈值）
- [ ] T089 [US4] 实现长文本翻译时自动提高噪音过滤阈值（内容>50字符）在 src/hooks/useAppMode.ts
- [ ] T090 [US4] 添加模式切换引导提示（首次使用显示如何切换模式）在 src/screens/CameraTranslationScreen.tsx
- [ ] T091 [US4] 在主翻译屏幕集成模式切换按钮和指示器 src/screens/CameraTranslationScreen.tsx

**检查点**: User Story 4 完整可用，防打断机制正常工作，满足 FR-010 和 FR-011

---

## 阶段 7: User Story 5 - 智能问答功能（优先级: P3）

**目标**: 用户在问答模式下可以针对画面内容提问，系统基于识别的文字和画面信息使用 AI 提供回答

**独立测试**: 切换到问答模式，输入问题（"这是什么菜？"、"这个词怎么发音？"），验证 AI 回答准确性和相关性

### US5 实现任务

- [ ] T092 [P] [US5] 创建 AI 问答服务抽象接口 src/services/ai/AIService.ts（定义 askQuestion 方法）
- [ ] T093 [P] [US5] 实现 OpenAI GPT-4 集成 src/services/ai/OpenAIProvider.ts（上下文管理、API 调用）
- [ ] T094 [US5] 创建问答会话管理在 src/store/appStore.ts（保存问题、回答、上下文历史）
- [ ] T095 [P] [US5] 创建语音输入组件 src/components/controls/VoiceInput.tsx（录音按钮、语音转文字）
- [ ] T096 [P] [US5] 创建问答界面组件 src/components/qa/QAInterface.tsx（显示问题和回答、输入框）
- [ ] T097 [P] [US5] 创建文本输入组件 src/components/qa/TextInput.tsx（手动输入问题）
- [ ] T098 [US5] 实现语音识别（使用系统原生语音识别 API）在 src/services/voice/SpeechRecognition.ts
- [ ] T099 [US5] 实现上下文管理（结合当前画面识别的文字内容）在 src/services/ai/AIService.ts
- [ ] T100 [US5] 实现问答结果显示和历史管理在 src/components/qa/QAInterface.tsx
- [ ] T101 [US5] 处理无法回答的问题（提示"无法从当前内容中找到相关信息"）在 src/services/ai/OpenAIProvider.ts
- [ ] T102 [US5] 实现多轮对话上下文保持（连续提问保持对话连贯性）在 src/store/appStore.ts
- [ ] T103 [US5] 添加"继续翻译"快速返回按钮在 src/components/qa/QAInterface.tsx
- [ ] T104 [US5] 在主翻译屏幕集成问答界面（问答模式时显示）src/screens/CameraTranslationScreen.tsx
- [ ] T105 [US5] 实现锁定画面模式下的区域定位问答（"第三段讲的是什么"）在 src/services/ai/AIService.ts
- [ ] T106 [US5] 处理 AI API 调用失败和超时（错误提示、重试机制）在 src/hooks/useAI.ts
- [ ] T107 [US5] 添加问答加载状态和动画在 src/components/qa/QAInterface.tsx

**检查点**: User Story 5 完整可用，智能问答功能正常工作，满足 FR-012 和 FR-013

---

## 阶段 8: Polish（优化和跨功能完善）

**目的**: 影响多个用户故事的优化和改进

- [ ] T108 [P] 实现应用级别的性能优化（React.memo、useMemo、useCallback）在所有组件中
- [ ] T109 [P] 优化图像预处理（设备端压缩和裁剪）在 src/utils/imageProcessing.ts
- [ ] T110 [P] 实现 API 请求的并行处理（OCR 和翻译分离）在 src/hooks/useOCR.ts 和 src/hooks/useTranslation.ts
- [ ] T111 完善错误处理（所有 API 调用的统一错误边界）在 src/utils/errorHandler.ts
- [ ] T112 [P] 添加网络状态监听和离线提示在 src/store/appStore.ts
- [ ] T113 [P] 实现 API 配额监控和用户提示（接近配额时警告）在 src/services/ocr/GoogleVisionOCR.ts
- [ ] T114 优化电池消耗（后台时自动暂停相机和 API 调用）在 src/hooks/useCamera.ts
- [ ] T115 [P] 添加长时间使用提示（>20分钟提示休息）在 src/screens/CameraTranslationScreen.tsx
- [ ] T116 [P] 实现设置页面完整功能 src/screens/SettingsScreen.tsx（语言偏好、API 配置、隐私设置）
- [ ] T117 完善引导流程（首次使用教程、功能介绍）在 src/components/onboarding/OnboardingTutorial.tsx
- [ ] T118 [P] 添加性能监控埋点（识别延迟、翻译延迟、内存占用）在 src/utils/analytics.ts
- [ ] T119 [P] 优化 UI 动画和过渡效果（使用 react-native-reanimated）在关键交互点
- [ ] T120 [P] 完善国际化支持（UI 文本多语言）在 src/i18n/translations.ts
- [ ] T121 实现隐私保护措施确认（不存储画面和历史、明确隐私政策）在 src/screens/PrivacyScreen.tsx
- [ ] T122 [P] 编写用户文档 README.md（功能说明、快速开始、常见问题）
- [ ] T123 运行 quickstart.md 验证（确保新开发者可以快速上手）

**检查点**: 所有功能完善，性能达标，准备发布

---

## 依赖关系和执行顺序

### 阶段依赖

- **Setup（阶段1）**: 无依赖 - 可立即开始
- **Foundational（阶段2）**: 依赖 Setup 完成 - **阻塞所有用户故事**
- **User Stories（阶段3-7）**: 所有用户故事都依赖 Foundational 阶段完成
  - 用户故事之间可以并行开发（如果有足够人力）
  - 或按优先级顺序执行（P1 → P2 → P3）
- **Polish（阶段8）**: 依赖所有期望的用户故事完成

### 用户故事依赖

- **User Story 1 (P1)**: Foundational 完成后可开始 - 无其他故事依赖
- **User Story 2 (P1)**: Foundational 完成后可开始 - 轻度依赖 US1（需要相机和翻译基础）
- **User Story 3 (P2)**: Foundational 完成后可开始 - 需要 US1 的翻译结果
- **User Story 4 (P2)**: Foundational 完成后可开始 - 需要 US1 的基础功能
- **User Story 5 (P3)**: Foundational 完成后可开始 - 需要 US1 的识别结果和 US4 的模式切换

### 用户故事内部依赖

- 服务接口 → 服务实现 → Hook 封装 → UI 组件 → 屏幕集成
- 核心实现 → 错误处理 → 优化改进

### 并行机会

- Setup 阶段所有标记 [P] 的任务可以并行
- Foundational 阶段所有标记 [P] 的任务可以并行
- Foundational 完成后，所有用户故事可以由不同团队成员并行开发
- 每个用户故事内部，标记 [P] 的任务可以并行
- Polish 阶段所有标记 [P] 的任务可以并行

---

## 并行执行示例: User Story 1

```bash
# 并行启动 US1 的所有模型/服务
Task: "创建摄像头管理服务 src/services/camera/CameraManager.ts"
Task: "创建帧处理器服务 src/services/camera/FrameProcessor.ts"

# 并行启动 OCR 相关
Task: "创建 OCR 服务抽象接口 src/services/ocr/OCRService.ts"
Task: "实现 Google Cloud Vision OCR 集成 src/services/ocr/GoogleVisionOCR.ts"

# 并行启动翻译相关
Task: "创建翻译服务抽象接口 src/services/translation/TranslationService.ts"
Task: "实现 Google Cloud Translation 集成 src/services/translation/GoogleTranslation.ts"

# 并行启动所有 UI 组件
Task: "创建摄像头预览组件 src/components/camera/CameraView.tsx"
Task: "创建对焦指示器组件 src/components/camera/FocusIndicator.tsx"
Task: "创建图像质量警告组件 src/components/camera/QualityWarning.tsx"
Task: "创建字幕叠加层组件 src/components/translation/SubtitleOverlay.tsx"
Task: "创建翻译卡片组件 src/components/translation/TranslationCard.tsx"
Task: "创建语言选择器组件 src/components/translation/LanguageSelector.tsx"
```

---

## 实现策略

### MVP 优先（仅 User Story 1 + User Story 2）

1. 完成阶段1: Setup
2. 完成阶段2: Foundational（**关键** - 阻塞所有故事）
3. 完成阶段3: User Story 1（实时视频翻译）
4. **停止并验证**: 独立测试 User Story 1
5. 完成阶段4: User Story 2（画面锁定）
6. **停止并验证**: 独立测试 User Story 2
7. 部署/演示 MVP

### 增量交付

1. Setup + Foundational → 基础就绪
2. 添加 User Story 1 → 独立测试 → 部署/演示（MVP 核心功能！）
3. 添加 User Story 2 → 独立测试 → 部署/演示（MVP 完整版！）
4. 添加 User Story 3 → 独立测试 → 部署/演示（增强版）
5. 添加 User Story 4 → 独立测试 → 部署/演示（稳定版）
6. 添加 User Story 5 → 独立测试 → 部署/演示（完整版）
7. Polish → 优化 → 最终发布

### 并行团队策略

有多个开发者时：

1. 团队一起完成 Setup + Foundational
2. Foundational 完成后：
   - **开发者 A**: User Story 1（实时翻译）
   - **开发者 B**: User Story 2（画面锁定）
   - **开发者 C**: User Story 3（语音播报）
3. 各故事独立完成并集成

---

## 任务统计

### 总体统计

- **总任务数**: 123 个任务
- **Setup 阶段**: 8 个任务（5 个可并行）
- **Foundational 阶段**: 19 个任务（13 个可并行）
- **User Story 1**: 25 个任务（14 个可并行）
- **User Story 2**: 13 个任务（4 个可并行）
- **User Story 3**: 14 个任务（8 个可并行）
- **User Story 4**: 12 个任务（3 个可并行）
- **User Story 5**: 16 个任务（6 个可并行）
- **Polish 阶段**: 16 个任务（11 个可并行）

### MVP 范围（建议）

**MVP = User Story 1 + User Story 2**

- **MVP 任务数**: 65 个任务（Setup 8 + Foundational 19 + US1 25 + US2 13）
- **MVP 预计工期**: 3-4 周（包含 Setup 和 Foundational）
- **MVP 核心价值**: 实时视频翻译 + 画面锁定（产品核心创新点）

### 并行机会总结

- **Setup**: 62.5% 任务可并行（5/8）
- **Foundational**: 68.4% 任务可并行（13/19）
- **US1**: 56% 任务可并行（14/25）
- **US2**: 30.8% 任务可并行（4/13）
- **US3**: 57.1% 任务可并行（8/14）
- **US4**: 25% 任务可并行（3/12）
- **US5**: 37.5% 任务可并行（6/16）
- **Polish**: 68.75% 任务可并行（11/16）

### 优先级分布

- **P1（必需）**: User Story 1 + User Story 2 = 38 个任务（不含基础设施）
- **P2（重要）**: User Story 3 + User Story 4 = 26 个任务
- **P3（增强）**: User Story 5 = 16 个任务

---

## 注意事项

- **[P] 标记**: 表示任务操作不同文件、无依赖关系，可并行执行
- **[Story] 标签**: 映射任务到具体用户故事，便于追踪和独立测试
- 每个用户故事应该可以独立完成和测试
- 每个任务或逻辑组完成后应提交代码
- 在任何检查点停下来验证故事的独立功能
- 避免：模糊任务、文件冲突、破坏故事独立性的跨故事依赖

---

**文档版本**: 1.0  
**生成日期**: 2025-10-22  
**预计总工期**: 8-10 周（完整版）| 3-4 周（MVP）  
**建议策略**: MVP 优先（US1 + US2），验证核心价值后再迭代添加 P2/P3 功能