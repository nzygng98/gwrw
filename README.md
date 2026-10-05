YES Lab 工位考核任务 - VINS-Fusion 复现报告

📌 一、 项目基本信息

· 项目名称：VINS-Fusion 开源项目复现与数据集验证
· 测试环境：
  · 宿主机：Windows 11 (16GB 内存)
  · 虚拟机：VMware Workstation + Ubuntu 20.04 LTS
  · 运行框架：ROS Noetic
  · 开发工具：VS Code + Remote-SSH + Cline (AI辅助编程)
· 复现目标：成功编译 VINS-Fusion 源码，输入 EuRoC 公开数据集，验证算法逻辑并输出轨迹数据。

🛠️ 二、 执行过程与操作步骤

1. 环境部署：配置 Ubuntu 虚拟机网络，安装 ROS Noetic 全套桌面版，配置 C++ 编译链（g++、CMake）。
2. 工程构建：在 ~/catkin_ws/src 下克隆 VINS-Fusion 源码，使用 catkin_make -j2 编译整个工作空间。
3. 数据准备：下载 EuRoC MAV Dataset 的 MH_01_easy.bag 数据集，并修正 vins_rviz.launch 文件中的绝对路径配置。
4. 算法运行：启动 roscore 与 vins_node，通过 rosbag play 播放数据集，提取算法输出的 vio.csv 轨迹文件。

🚧 三、 遇到的困难与解决方法（核心复盘）

1. 跨系统执行与 AI 沙箱拦截

· 问题：在 Windows 宿主机的 VS Code 中运行 AI 助手时，由于操作系统隔离，AI 无法直接访问 Ubuntu 虚拟机的文件系统和终端，触发 Windows 沙箱拦截报错。
· 解决方法：采用 Remote - SSH 插件，将 VS Code 远程连接到 Ubuntu 虚拟机内部。在虚拟机环境中重新安装 AI 插件，让 AI 直接运行在 Linux 环境下，彻底消除了跨系统障碍。

2. 编译依赖冲突（OpenCV 4 兼容性）

· 问题：VINS-Fusion 源码默认使用 OpenCV 3 编写，而 Ubuntu 20.04 默认的 ROS Noetic 自带 OpenCV 4。直接编译导致 camera_models、vins_estimator 等四个核心模块出现致命编译错误。
· 解决方法：排查终端编译日志，修改了四个模块的 CMakeLists.txt 文件。显式指定 C++ 11 标准，并在编译宏中追加 OpenCV 4 的头文件路径和兼容性定义，最终编译进度达到 100%，无错误通过。

3. 硬件资源瓶颈导致 3D 渲染崩溃

· 问题：在播放数据集时，RViz 图形化界面启动后，由于 Windows 宿主机仅有 16G 内存，虚拟机资源分配达到极限，RViz 进程因内存溢出被系统强制终止（Crash）。
· 解决方法：放弃图形界面（RViz）渲染，采用 Headless（无头）模式运行算法（rviz:=false）。通过降低数据播放速度（-r 0.1），成功跑通了完整的数据集，最终在终端提取到包含完整时间戳、坐标、四元数信息的 vio.csv 轨迹文件。

📊 四、 复现成果与验证

· 运行结果：成功输出轨迹数据，证明视觉与惯性融合算法（VIO）计算链路通畅。
· 数据验证：终端执行 head -n 20 ~/catkin_ws/src/VINS-Fusion/output/vio.csv 可查阅算法输出的位姿信息。

💡 五、 复盘总结

本次复现虽然使用了 AI 工具辅助排查依赖库冲突，但核心的系统级集成工作由本人独立完成，包括：跨系统 SSH 通道配置、虚拟机资源分配调优、软件源与编译链排错
