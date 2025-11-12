# learning-stm32

学习嵌入式的一些笔记，由 Obsidian 创建。这个仓库包含我对 STM32 的学习笔记、配置说明、外设使用示例和一些 HAL/寄存器层的调试记录。

目录（示例）：
- `ADC.md`：ADC 使用与配置
- `GPIO.md`：GPIO 模式与示例
- `I2C.md`：I2C 通讯笔记
- `MPU6050.md`、`使用MPU6050解算欧拉角.md`：IMU 数据处理

如果你想复现示例或构建代码，请参考各笔记中的工具链/IDE 说明（例如 Keil / STM32CubeIDE）。

如何使用
1. 克隆仓库：
	```bash
	git clone https://github.com/Buns321/learning-stm32.git
	```
2. 在本地使用你偏好的 IDE 打开对应示例或笔记中的工程文件。
3. 如果需要把代码烧录到开发板，请参考笔记中所列的工具链和连接说明。

许可证

本仓库默认采用 MIT 许可证（详见 `LICENSE` 文件）。

贡献

欢迎 issue/PR。提交前请先阅读 `README.md` 中的使用说明并在 issue 里描述你的改动。
