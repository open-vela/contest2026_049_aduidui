# contest2026_049_aduidui

# 电睛之臂——基于Openvela的单目视觉机械臂分拣系统

## 一、作品简介

在农业果蔬筛检领域，传统方式依赖工人依靠个人经验进行人工分拣，存在效率低、一致性差等问题。随着智慧农业技术的发展，服务型机器人依靠节能增效等优点也广泛被各行业运用在生产制造工作中。因此本研究针对机械臂的智能抓取任务，基于单目视觉的果蔬目标检测以及定位抓取算法，构建并实现一套满足需求的智能果蔬分拣系统。主要的研究内容和工作如下。

**第一**，基于  CSPDarknet 骨干网络的 YOLO目标检测算法，使用卷积神经网络提取图像特征，并通过 Head 层多尺度特征融合，让检测目标在进行服务分类和回归任务时，可以完成网络骨架的轻量化，同时增强梯度流，保持卷积的特征。USB免驱摄像头直连上位机，在上位机完成水果类别识别与缺陷判定，输出分类结果、置信度、目标坐标。通过 状态机通信将结构化结果（类别、置信度、坐标）发送R528。视觉推理与运动控制在物理和逻辑上双重隔离。

**第二**，本文使用 VS Code 开发环境来完成机械臂步长的自适应设计和开发工作。本文分析了实际作业和机械臂工作空间，完成了机械臂结构参数的优化设计。还使用 3D 建模技术来制作机械臂的三维模型，通过观察物理轨迹以保证运动学分析结果的准确，并且为系统搭建提供支持。

**第三**，本文完成了对智能果蔬分拣系统的研究。系统使用分体模块化模式，把感知层、交互层、执行层和通信层等模块集成在一起，开发工作包括张正友手眼标定、目标检测、基于闭环梯形加减速算法控制机械臂完成抓取、搬运、放置动作。、端对端状态机通信。整体系统可以实现实时采集环境数据，并且支持数据可视化与系统管理。

**第四**，基于 openvela AI Agent 的端侧自然语音交互控制功能。用户通过语音与 AI Agent 对话，Agent 将语义解析为机械臂控制指令，端侧通过 TFLite-Micro 运行关键词唤醒模型，检测到唤醒词后触发完整语音采集；云端通过openvela ai_agent 框架调用 MiMo-V2.5 大模型完成语义理解与意图推理。ai_agent 的 ReAct推理引擎将语义输出映射为端侧工具调用驱动执行层完成抓取、分拣、归位等操作。，通过 Function Calling 实现多轮工具调度闭环。具备低延迟、高可靠、隐私安全等优势。

## 二、选题方向

新硬件适配    润芯微 gemini-s1 开发板

结合本项目智能果蔬分拣系统的技术需求与系统架构，选用润芯微 gemini-s1 作为核心控制平台，主要基于以下考量：

1. **端侧 AI 调度能力，契合轻量化检测需求**  
    openvela 开源了嵌入式 AI Agent 引擎，采用主流语言编写，可读性高。较小空间即可运行完整的 ReAct 推理循环和工具调度。Agent 框架负责管理对话历史、调度 FunctionCalling、校验工具返回结果，实际语义理解交给云端 MiMo-V2-Flash 完成。它可以判断当前机械臂状态是否允许执行用户指令、在云端断连时自动降级为固定指令模板匹配。这套框架是本项目实现端云混合语音交互的软件基础，

2. **支持 AI Agent 框架部署，赋能人机交互**  
    openvela 社区提供了 TFLite-Micro 的适配指南和 CMSIS-NN加速支持。在本项目中，TFLite-Micro 承载一条核心推理任务：关键词唤醒模型，7×24运行在后台，检测到唤醒词后触发完整语音采集流程。该模型在 R528 的 Cortex-A7 上使用 NEON 指令集加速。云端 API断连时，语音交互降级为固定指令模板匹配，端侧关键词唤醒加命令词匹配可完全离线工作。端侧推理不依赖网络、不依赖上位机，在核心交互入口上避免了外部依赖带来的不确定性。端侧部署意味着用户无需联网即可完成语音指令的解析与执行具有显著优势。

3. **多接口支持，满足系统模块化集成需求**  
   本系统采用分体模块化架构，集成了感知层（相机）、执行层（机械臂）、通信层（状态机）等多个模块。开发板提供丰富的外设接口，8 通道 PWM 可以满足多路PWM控制输出和6 路 UART可以独立分配给上位机通信、串口屏、调试终端、外部传感器模块，互不干扰。在联调阶段，调试口和业务通信口物理隔离，打印日志不会挤占控制指令的带宽。 R528 支持最多 8 路 PDM 数字麦克风，可以组成麦克风阵列做远场拾音。阵列拾音比单麦克风实用得多。音频输出有 2 路 DAC 加 I2S 接口，TTS语音播报直接输出到外接扬声器，不需要额外音频编解码芯片。内置的 HiFi4 DSP 可用于音频特征提取，将 MFCC 等预处理从Cortex-A7 卸载到 DSP，减轻 CPU 负载。，为系统集成提供了硬件基础，避免了多板堆叠带来的复杂度与稳定性风险。系统运行状态、分拣统计、故障信息等数据通过 UART发送至串口屏实时显示。开发简洁、运行稳定，适合现场长期使用。

4. **实时操作系统支持，契合工业级控制需求**  
   机械臂状态机调度、通信协议解析、传感器数据采集等任务对算力要求不高。双核架构允许一核专注状态机和运动控制，另一核处理网络通信和 Agent 调度，互不抢占。openvela RTOS 基于 NuttX实时内核，提供确定性任务调度，机械臂控制指令的响应延迟可控制在微秒级，不会因后台任务产生抖动。相比 Linux这类非实时系统，openvela 在运动控制场景下天然更安全。具备确定性的任务调度与中断响应能力，能够保证机械臂控制指令的精确时序，满足分拣作业对运动控制精度的要求。
   
    综合以上，Gemini-S1（R528 加 openvela）与智能果蔬分拣系统的需求形成了一套精确匹配的对应关系：实时控制有 RTOS确定性调度，端侧推理有 TFLite-Micro 加 CMSIS-NN，云端交互有 ai_agent 框架加 MiMoAPI，视觉重计算卸载到上位机，数据显示用串口屏，开发板不支持的摄像头由上位机 USB口接管。这块开发板的每一项硬件能力都在系统中承担了明确的职责，每一项短板都通过架构设计在对应层级消化。
    
## 三、目录结构

```
  contest2026_049_aduidui/                         # 项目根目录
  │
  ├── app/hello_app/                               # 应用层（智能果蔬分拣系统核心代码）
  │   │
  │   ├── hello_app_main.c                         # 应用主程序入口，各模块初始化与启动
  │   ├── CMakeLists.txt                           # CMake 构建配置
  │   ├── Makefile                                 # Make 构建文件
  │   ├── Make.defs                                # Make 编译选项定义
  │   ├── Kconfig                                  # 内核配置项（功能模块开关与参数）
  │   ├── README.md                                # 应用说明文档
  │   │
  │   ├── core/                                    # 系统核心
  │   │   ├── task_manager.c                       # 任务创建与优先级分配（双核分工）
  │   │   ├── task_manager.h
  │   │   └── system_config.c                      # 全局配置集中管理
  │   │       system_config.h
  │   │
  │   ├── statemachine/                            # 状态机模块
  │   │   ├── system_fsm.c                         # 全局状态机
  │   │   ├── system_fsm.h
  │   │   ├── sort_fsm.c                           # 分拣流程状态
  │   │   │                                       
  │   │   └── sort_fsm.h
  │   │
  │   ├── vision/                                  # 视觉模块（结果接收与处理，不含图像采集）
  │   │   ├── result_parser.c                      # 上位机推理结果解析（类别 + 置信度 + 坐标 ）
  │   │   ├── result_parser.h
  │   │   ├── coordinate_transform.c               # 手眼标定坐标转换（图像坐标 → 机械臂坐标）
  │   │   ├── coordinate_transform.h
  │   │   ├── vision_manager.c                     # 视觉任务调度（上位机结果接收 / 超时处理 ）
  │   │   └── vision_manager.h
  │   │
  │   ├── arm/                                     # 机械臂模块
  │   │   ├── arm_controller.c                     # 机械臂总控（状态机 + 指令分发）
  │   │   ├── arm_controller.h
  │   │   ├── trapezoid_profile.c                  # 闭环梯形加减速算法
  │   │   ├── trapezoid_profile.h
  │   │   ├── kinematics.c                         # 运动学计算（ 逆解）
  │   │   ├── kinematics.h
  │   │   ├── gripper.c                            # 夹爪 
  │   │   └── gripper.h
  │   │
  │   ├── ai_agent/                                # AI Agent 模块
  │   │   ├── agent_manager.c                      # Agent 总调度
  │   │   ├── agent_manager.h
  │   │   ├── mimo_provider.c                      # MiMo API 客户端
  │   │   ├── mimo_provider.h
  │   │   ├── react_loop.c                         # ReAct 推理循环
  │   │   ├── react_loop.h
  │   │   ├── agent_tools.c                        # 端侧工具定义与注册
  │   │   ├── agent_tools.h                        # arm_pick / arm_place / arm_home 
  │   │   │                                        # set_speed / emergency_stop
  │   │   ├── asr_stub.c                           # 语音识别前端
  │   │   ├── asr_stub.h
  │   │   ├── tts_stub.c                           # 语音合成输出
  │   │   ├── tts_stub.h
  │   │   └── models/                              # 端侧量化模型
  │   │       └── keyword_wake.c                   # 关键词唤醒模型
  │   │
  │   ├── safety/                                  # 安全保护模块
  │   │   ├── safety_arbiter.c                     # 安全仲裁
  │   │   ├── safety_arbiter.h
  │   │   ├── fault_manager.c                      # 故障码管理与分级处理
  │   │   └── fault_manager.h
  │   │
  │   ├── communication/                           # 通信模块
  │   │   ├── protocol.c                           # 通信协议
  │   │   ├── protocol.h
  │   │   ├── camera_link.c                          # 上位机链路管理
  │   │   ├── camera_link.h
  │   │   ├── cloud_reporter.c                     # 云端数据上报
  │   │   └── cloud_reporter.h
  │   │
  │   ├── display/                                 # 串口屏显示模块
  │   │   ├── uart_display.c                       # 串口屏驱动（UART 指令发送，页面切换，数据）
  │   │   ├── uart_display.h
  │   │   ├── display_pages.c                      # 各页面数据组织
  │   │   └── display_pages.h
  │   │
  │   ├── modules/                                 # 可复用基础模块
  │   │   ├── ring_buffer.c                        # 环形缓冲区（用于传感器数据缓存）
  │   │   ├── ring_buffer.h
  │   │   ├── soft_timer.c                         # 软件定时器（用于超时判断）
  │   │   ├── soft_timer.h
  │   │   ├── json_parser.c                        # 轻量 JSON 解析（用于 MiMo API 响应）
  │   │   ├── json_parser.h
  │   │   ├── moving_average.c                     # 滑动平均滤波器（传感器数据去噪）
  │   │   └── moving_average.h
  │   │
  │   └── calibration/                             # 标定参数管理
  │       ├── calibration.c                        # 标定参数存储与版本管理
  │       └── calibration.h
  │
  ├── board/contest_board/                         # 板级支持包
  │   ├── src/
  │   │   ├── board_boot.c                         # 板级启动引导（外设、DMA、中断优先级初始化）
  │   │   ├── gpio_init.c                          # GPIO 初始化（急停按钮、LED、蜂鸣器）
  │   │   ├── uart_init.c                          # UART 初始化（上位机通信口、串口屏口、调试口）
  │   │   ├── pwm_init.c                           # PWM 初始化（机械臂关节、夹爪、传送带）
  │   │   ├── adc_init.c                           # ADC 初始化（温湿度传感器、编码器）
  │   │   ├── pdm_init.c                           # PDM 麦克风初始化（语音采集）
  │   │   ├── dac_i2s_init.c                       # DAC / I2S 初始化（TTS 语音输出）
  │   │   ├── eth_wifi_init.c                      # 网络初始化（ WiFi）
  │   │   └── CMakeLists.txt                       # BSP 源码构建配置
  │   ├── configs/nsh/defconfig                    # NSH 默认配置（内核特性与外设开关）
  │   ├── Kconfig                                  # 板级配置项
  │   ├── CMakeLists.txt                           # 板级构建配置
  │   └── README.md                                # 板级说明文档（引脚映射、外设清单、烧录方法）
  │
  ├── logs/                                        # AI Coding 日志
  │   ├── your-github-login/                       # 开发者日志（按 GitHub 用户名 / 日期组织）
  │   ├── README.md                                # 日志说明
  │   └── manifest.json                            # 日志清单
  │
  ├── docs/                                        # 项目文档
  │   ├── architecture.md                          # 系统架构说明（五层设计 + 数据流向）
  │   ├── state_machine.md                         # 状态机图与转移条件
  │   ├── protocol.md                              # 通信协议定义（帧结构与命令字表）
  │   ├── io_mapping.md                            # IO 引脚映射表
  │   ├── coordinate_system.md                     # 坐标系定义与手眼标定说明
  │   ├── calibration_guide.md                     # 标定操作流程
  │   ├── fault_codes.md                           # 故障码表
  │   └── known_issues.md                          # 已知问题与规避方式
  │
  ├── tools/                                       # 辅助工具（不部署到 R528）
  │   ├── upper_machine/                           # 上位机程序
  │   │   ├── yolo_inference.py                    # CSPDarknet YOLO 推理 + 结果发送
  │   │   ├── camera_capture.py                    # USB 免驱摄像头图像采集
  │   │   └── requirements.txt                     # Python 依赖
  │   ├── calibration_tool/                        # 手眼标定工具（图像采集 + 标定计算）
  │   ├── log_parser/                              # 日志解析脚本（离线分析和可视化）
  │   ├── serial_monitor/                          # 串口调试工具（数据收发 + 帧解析）
  │   └── display_design/                          # 串口屏界面设计工程文件
  │
  ├── .github/workflows/cla.yml                    # GitHub Actions CI 工作流（CLA 检查）
  ├── contest2026_049_aduidui.xml                  # 本仓库 repo 清单文件
  ├── openvela.xml                                 # openvela 平台 repo 清单文件
  ├── .gitignore.example                           # Git 忽略规则示例
  └── README.md                                    # 总项目说明文档
```

## 四、运行方式

> 拉取工程后，如何编译、烧录/部署、运行的完整步骤；最好能让评委照着一步步复现。

## 五、AI Coding 使用说明

> 说明本作品如何借助 AI 辅助开发：
> - 在需求拆解 / 方案设计 / 编码 / 调试 / 文档等环节如何与 AI 协作；
> - AI 对开发效率或质量带来的实际帮助。
>
> 完整对话日志见 logs/ 目录。

---

> **提示**：将会根据「作品本身 + 你的 README 说明 + logs/ 里的 AI Coding 日志」来理解和评估你的作品，README 写清楚很重要。
