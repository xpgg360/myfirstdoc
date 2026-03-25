# 全新M芯片Mac OpenClaw部署全流程指南

> 适用设备：Apple Silicon M1/M2/M3/M4 Mac  
> 安装方式：源码安装  
> 预计耗时：60–90分钟

---

## 一、基础开发环境安装

### 1.1 安装 Xcode 命令行工具

**【做什么】**
- 打开终端，执行：`xcode-select --install`
- 弹窗点击「安装」，等待完成
- 验证安装成功：`git --version` 和 `gcc --version`

**【为什么】**
- Xcode命令行工具是Mac开发的基石，包含了`git`、`gcc`等必备工具
- 没有它，后续的编译、安装都会失败

**【注意事项】**
- 弹窗点「安装」即可，无需下载完整的Xcode应用

---

### 1.2 安装 Homebrew

**【做什么】**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
- 验证安装：`brew --version`

**【为什么】**
- Homebrew是macOS官方的包管理器，用来安装和管理各种开发工具
- 类似于Ubuntu的apt、CentOS的yum

**【注意事项】**
- **M芯片Mac必须配置环境变量：**
  ```bash
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)"
  ```
- 安装时可能遇到GitHub连接超时，解决方法：
  1. 多重试几次（我就是这么解决的）
  2. 开启VPN
  3. 切换国内源（可咨询豆包）

---

### 1.3 安装系统依赖库

**【做什么】**
```bash
brew install ffmpeg libvips python3
```

**【为什么】**
- `ffmpeg`：音视频处理，OpenClaw处理媒体文件需要
- `libvips`：高效的图像处理库，sharp模块依赖
- `python3`：OpenClaw部分功能需要Python环境

**【注意事项】**
- 这是必装项，无跳过选项
- 如果`python3`已自带Homebrew版本，会避免系统Python冲突

---

### 1.4 Git 国内镜像加速

**【做什么】**
```bash
git config --global url."https://mirror.ghproxy.com/https://github.com/".insteadOf "https://github.com/"
git config --global http.sslVerify false
```

**【为什么】**
- 国内访问GitHub经常超时或速度极慢
- 镜像加速可以将GitHub请求代理到国内CDN，大幅提升速度

**【注意事项】**
无

---

## 二、核心运行环境

### 2.1 安装 nvm + Node.js 24

**【做什么】**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 配置环境变量
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.zshrc
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.zshrc
source ~/.zshrc

echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zprofile
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.zprofile
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.zprofile
source ~/.zprofile

# 安装 Node.js 24
nvm install 24
nvm alias default 24
```

- 验证安装：`node --version`、`npm --version`、`nvm --version`

**【为什么】**
- **nvm**：Node Version Manager，管理多个Node版本
- **Node.js 24**：OpenClaw要求的最新LTS版本
- 使用nvm可以轻松切换版本，避免系统Node冲突
- 为啥要zshrc和zprofile来回导呢，是为了让脚本无论依赖哪个文件，都能找到命令

**【注意事项】**
- 如果GitHub连接超时，解决方法：
  1. 浏览器手动下载：https://github.com/nvm-sh/nvm/archive/refs/tags/v0.39.7.zip
  2. 解压到`~/.nvm`目录
	mkdir -p ~/.nvm
	unzip ~/Downloads/nvm-0.39.7.zip -d ~/.nvm
	mv ~/.nvm/nvm-0.39.7/* ~/.nvm/
	rm -rf ~/.nvm/nvm-0.39.7
- **必须配置zsh环境变量**，Mac默认终端是zsh


---

### 2.2 升级 pip3

**【做什么】**
```bash
python3 -m pip install --upgrade pip
```

**【为什么】**
- 全新系统中pip版本可能过旧，升级确保兼容性
- 避免后续安装Python包时出现版本冲突

**【注意事项】**
- 全新系统必做
- 使用`python3 -m pip`而非直接`pip`，避免macOS系统Python冲突

---

### 2.3 安装 pnpm

**【做什么】**
```bash
npm install -g pnpm
pnpm config set registry https://registry.npmmirror.com
```
- 验证安装：`pnpm --version`

**【为什么】**
- **pnpm**：比npm更高效的包管理器，节省磁盘空间
- 切换国内镜像加速安装

**【注意事项】**

---

## 三、OpenClaw 源码安装

### 3.1 克隆项目

**【做什么】**
```bash
git clone https://gitee.com/OpenClaw-CN/openclaw-cn.git
cd openclaw-cn
```

**【为什么】**
- 从Gitee（国内镜像）克隆，比GitHub更快更稳定
- Gitee是OpenClaw-CN的中文镜像站

**【注意事项】**
- 目录名默认为`openclaw-cn`

---

### 3.2 安装依赖

**【做什么】**
```bash
pnpm install sharp
# 如果sharp安装失败，执行：
SHARP_IGNORE_GLOBAL_LIBVIPS=1 pnpm install
```

**【为什么】**
- 安装OpenClaw运行所需的Node.js依赖
- sharp是图像处理核心模块，libvips的Node绑定

**【注意事项】**
无

---

### 3.3 构建项目

**【做什么】**
```bash
pnpm ui:build
pnpm build
```

**【为什么】**
- `ui:build`：构建前端界面
- `build`：编译后端服务
- 两者缺一不可，否则服务无法启动

**【注意事项】**
- 首次构建可能需要几分钟，耐心等待
- 如果报错，检查Node.js版本是否为大于22

---

### 3.4 初始化

**【做什么】**
```bash
pnpm openclaw onboard --install-daemon
```

**【做什么】**
当然是安装啦！

**【注意事项】**
安装时有两种让你选，一种是快速安装，一种是手动详细配置，选快速，别装大以巴狼
建议先注册好模型的api key，这一步绕不过去的，我以我自己的minimax为例：
1. 登录 MiniMax 开放平台：
国内版：https://platform.minimaxi.com
国际版：https://platform.minimax.io
2. 进入「控制台 / 余额中心」，查看你的 API Key 对应的余额 / 剩余额度：
新用户通常有少量免费额度，用完就会触发这个报错；
若显示「余额为 0」或「额度已耗尽」，就是计费问题。
3. 充值，否则龙虾安装好后，你问他啥他都和你说你没余额了。。。。
4. Api key要保存下来，存在某个文件里，防止后面不好找

最后选择打开方式时选择网页ui打开，但是我打开时遇到了报错：Control UI assetsnotfound. Build them with `pnpm ui:build` (auto-installs UI deps),orrun `pnpm ui:dev` during development.
原因：你可能安装了原版openclaw而不是汉化版，或者安装过程中断导致文件不完整。
我的解法：清除 npm 缓存 npm cache clean --force
重新安装汉化版 npm install -g @qingchencloud/openclaw-zh@latest

其他的一些问题可以查阅知乎，挺全的：https://zhuanlan.zhihu.com/p/2003990399765717256
---
