# DroidBot

AI-Powered Android Compatibility Testing Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

## Overview

DroidBot is an AI-driven automated testing platform for Android application compatibility testing. It enables users to describe test requirements in natural language, while the AI Agent automatically plans tasks, generates executable scripts, and executes tests across multiple devices in parallel.

**Key Capabilities:**
- Natural language to test script generation
- Multi-device parallel execution (USB/TCPIP)
- AI-powered test result analysis
- Comprehensive HTML test reports
- Real-time device management

## Architecture

### Local Deployment Architecture
```
【客户端层】→ 【核心服务层】→ 【执行层】→ 【数据存储层】
（Web/桌面端）  （AI+调度核心）  （真机集群）  （本地存储）
```

### Key Modules
| Module | Description | Status |
|--------|-------------|--------|
| AI Programming Engine | LLM-powered test scenario planning & script generation | ✅ |
| Device Management | ADB device discovery, status monitoring, allocation | ✅ |
| Execution Engine | Multi-threaded parallel test execution | ✅ |
| Result Analysis | AI-powered screenshot/log analysis | ✅ |
| Report Generation | Standardized HTML report generation | ✅ |

## Features

### 🤖 AI Script Generation
- Describe test requirements in natural language
- AI automatically generates uiautomator2 test scripts
- Support for multiple test scenarios: device page, settings, events, details

### 📱 Multi-Device Testing
- **ADB USB Connection**: Stable, high-speed wired connection
- **ADB TCPIP Connection**: Wireless, flexible device placement
- Real-time device status monitoring
- Auto-reconnection on failure

### 📊 Intelligent Reports
- Auto-generated detailed HTML test reports
- AI-powered failure analysis and suggestions
- Screenshot capture and comparison
- Compatibility issue detection (UI glitches, crashes, ANR)

### 📦 APK Management
- Easy APK file upload and version management
- Test script storage and organization
- Log and screenshot archival

### ⚙️ Configuration
- LLM provider configuration (OpenAI, Anthropic, Local Models)
- Device connection settings
- Test execution parameters

## Quick Start

### Prerequisites
- Python 3.8+
- Android SDK (ADB tools)
- Android devices with USB debugging enabled

### Installation
```bash
# Clone repository
git clone https://github.com/joe-qai/DroidBot.git
cd DroidBot

# Install dependencies
pip install uiautomator2 fastapi

# Start the backend service
python -m uiautomator2 init
```

### Running the Demo
1. Open `index.html` in a browser to view the project introduction
2. Click "体验 Demo" to access the interactive demo dashboard
3. Navigate through the following modules:
   - **设备管理**: View connected Android devices
   - **新建测试**: Create new test tasks with natural language
   - **AI脚本**: View and manage generated scripts
   - **执行管理**: Monitor test execution progress
   - **测试报告**: View detailed test reports
   - **APK管理**: Upload and manage APK files
   - **系统设置**: Configure LLM and system settings

## Project Structure

```
DroidBot/
├── index.html              # Project homepage
├── demo.html               # Main demo dashboard
├── devices.html            # Device management page
├── new-test.html           # New test creation page
├── ai-script.html          # AI script generation page
├── execution.html          # Execution management page
├── reports.html            # Reports list page
├── report-detail.html      # Report detail page
├── apk-manager.html        # APK management page
├── settings.html           # System settings page
├── README.md               # Project documentation
└── Android兼容性测试Agent技术架构方案.md  # Technical architecture document
```

## Tech Stack

- **Frontend**: HTML5, CSS3, JavaScript (Modern UI design)
- **Backend**: Python (FastAPI, uiautomator2)
- **AI Integration**: LLM APIs (OpenAI, Anthropic, Local Models)
- **Device Communication**: ADB (Android Debug Bridge)
- **Storage**: Local file system + SQLite metadata

## Demo Pages

| Page | Description |
|------|-------------|
| `index.html` | Project overview and feature showcase |
| `demo.html` | Main dashboard with navigation |
| `devices.html` | Connected device list and status |
| `new-test.html` | Natural language test creation |
| `ai-script.html` | Generated script preview |
| `execution.html` | Test execution monitoring |
| `reports.html` | Test report gallery |
| `report-detail.html` | Detailed report view |
| `apk-manager.html` | APK upload and management |
| `settings.html` | LLM and system configuration |

## Reference Demos

- **DroidBot Web Demo**: [https://joe-qai.github.io/DroidBot/](https://joe-qai.github.io/DroidBot/)
- **Feishu Miaoda Demo**: [https://sd6syw7aku.aiforce.cloud/app/app_4k4a3yhc4ueev](https://sd6syw7aku.aiforce.cloud/app/app_4k4a3yhc4ueev)

## License

MIT License - See [LICENSE](LICENSE) for details

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## Contact

For questions or support, please open an issue in the repository.
