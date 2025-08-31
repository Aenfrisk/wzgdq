<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文字阅读模拟器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#3B82F6',
                        secondary: '#6B7280',
                        accent: '#10B981',
                        dark: '#1F2937',
                        light: '#F9FAFB',
                        darkBg: '#111827',
                        darkCard: '#1F2937'
                    }
                }
            }
        }
    </script>
    
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .scroll-container {
                overflow: hidden;
                position: relative;
            }
            .text-scroll {
                position: absolute;
                top: 0;
                left: 0;
            }
            .resize-handle {
                position: absolute;
                right: 0;
                bottom: 0;
                width: 16px;
                height: 16px;
                cursor: nwse-resize;
            }
            .preserve-format {
                white-space: pre-wrap;
                word-wrap: break-word;
                overflow-wrap: break-word;
                word-break: normal;
            }
            .monospace {
                font-family: monospace;
            }
        }
    </style>
    
    <style>
        /* 基础样式 */
        body {
            background-color: #f3f4f6;
            color: #1F2937;
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        
        /* 深色模式样式 */
        body.dark-mode {
            background-color: #111827 !important;
            color: #F9FAFB !important;
        }
        
        body.dark-mode .bg-white {
            background-color: #1F2937 !important;
        }
        
        body.dark-mode .bg-light {
            background-color: #1F2937 !important;
        }
        
        body.dark-mode .border-gray-200 {
            border-color: #374151 !important;
        }
        
        body.dark-mode .text-gray-600 {
            color: #d1d5db !important;
        }
        
        .resize-handle {
            background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%236B7280' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M15 3h6v6'/%3E%3Cpath d='M9 21H3v-6'/%3E%3Cpath d='M21 3l-7 7'/%3E%3Cpath d='M3 21l7-7'/%3E%3C/svg%3E");
        }
        
        body.dark-mode .resize-handle {
            background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%239CA3AF' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M15 3h6v6'/%3E%3Cpath d='M9 21H3v-6'/%3E%3Cpath d='M21 3l-7 7'/%3E%3Cpath d='M3 21l7-7'/%3E%3C/svg%3E");
        }
        
        /* 滑块样式 */
        input[type="range"] {
            -webkit-appearance: none;
            height: 8px;
            border-radius: 4px;
            background: #e5e7eb;
            outline: none;
        }
        
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #3B82F6;
            cursor: pointer;
        }
        
        input[type="range"]::-moz-range-thumb {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #3B82F6;
            cursor: pointer;
            border: none;
        }
        
        body.dark-mode input[type="range"] {
            background: #374151;
        }
        
        /* 按钮反馈样式 */
        .btn-tooltip {
            position: relative;
        }
        
        .btn-tooltip:hover::after {
            content: attr(data-tooltip);
            position: absolute;
            bottom: 100%;
            left: 50%;
            transform: translateX(-50%);
            padding: 4px 8px;
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            border-radius: 4px;
            font-size: 12px;
            white-space: nowrap;
            z-index: 100;
        }
        
        body.dark-mode .btn-tooltip:hover::after {
            background-color: rgba(255, 255, 255, 0.8);
            color: black;
        }
        
        /* 禁用按钮样式 */
        button:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }
        
        /* 文本容器样式 */
        .text-container {
            width: 100%;
            box-sizing: border-box;
        }
    </style>
</head>
<body class="min-h-screen font-sans">
    <div class="container mx-auto px-4 py-8 max-w-7xl">
        <!-- 标题区域 -->
        <header class="mb-8 flex flex-col md:flex-row justify-between items-center">
            <div>
                <h1 class="text-[clamp(1.8rem,4vw,2.5rem)] font-bold mb-2">文字阅读模拟器</h1>
                <p class="text-gray-600 text-lg">支持特殊符号和加密文本，保持正确排版</p>
            </div>
            <!-- 深色模式切换按钮 -->
            <button id="theme-toggle" class="mt-4 md:mt-0 bg-primary hover:bg-primary/90 text-white py-2 px-4 rounded-lg transition flex items-center">
                <i class="fa fa-moon-o mr-2"></i>
                <span>切换深色模式</span>
            </button>
        </header>
        
        <!-- 主内容区域 -->
        <main class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- 控制面板 -->
            <div class="lg:col-span-1 bg-white rounded-xl shadow-md p-6 transition-colors duration-300">
                <h2 class="text-xl font-semibold mb-4 flex items-center">
                    <i class="fa fa-sliders mr-2 text-primary"></i>控制面板
                </h2>
                
                <!-- 滚动方向选择 -->
                <div class="mb-6">
                    <label class="block text-gray-600 font-medium mb-2">滚动方向</label>
                    <div class="flex space-x-4">
                        <label class="inline-flex items-center cursor-pointer">
                            <input type="radio" name="direction" value="vertical" class="form-radio h-5 w-5 text-primary" checked>
                            <span class="ml-2">竖向滚动</span>
                        </label>
                        <label class="inline-flex items-center cursor-pointer">
                            <input type="radio" name="direction" value="horizontal" class="form-radio h-5 w-5 text-primary">
                            <span class="ml-2">横向滚动</span>
                        </label>
                    </div>
                </div>
                
                <!-- 速度倍率调节（粗调） -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <label for="speed-multiplier" class="text-gray-600 font-medium">速度倍率（粗调）</label>
                        <span id="speed-multiplier-value" class="text-primary font-medium">1x</span>
                    </div>
                    <input 
                        type="range" 
                        id="speed-multiplier" 
                        min="1" 
                        max="5" 
                        step="0.5" 
                        value="1" 
                        class="w-full"
                    >
                </div>
                
                <!-- 速度调节（细调） -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <label for="speed" class="text-gray-600 font-medium">滚动速度（细调）</label>
                        <span id="speed-value" class="text-primary font-medium">50</span>
                    </div>
                    <input 
                        type="range" 
                        id="speed" 
                        min="10" 
                        max="200" 
                        value="50" 
                        class="w-full"
                    >
                </div>
                
                <!-- 行间距调节 -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <label for="line-height" class="text-gray-600 font-medium">行间距</label>
                        <span id="line-height-value" class="text-primary font-medium">1.5</span>
                    </div>
                    <input 
                        type="range" 
                        id="line-height" 
                        min="1" 
                        max="3" 
                        step="0.1" 
                        value="1.5" 
                        class="w-full"
                    >
                </div>
                
                <!-- 字距调节 -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <label for="letter-spacing" class="text-gray-600 font-medium">字间距</label>
                        <span id="letter-spacing-value" class="text-primary font-medium">0</span>
                    </div>
                    <input 
                        type="range" 
                        id="letter-spacing" 
                        min="0" 
                        max="10" 
                        value="0" 
                        class="w-full"
                    >
                </div>
                
                <!-- 字体大小调节 -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <label for="font-size" class="text-gray-600 font-medium">字体大小</label>
                        <span id="font-size-value" class="text-primary font-medium">16</span>
                    </div>
                    <input 
                        type="range" 
                        id="font-size" 
                        min="12" 
                        max="32" 
                        value="16" 
                        class="w-full"
                    >
                </div>
                
                <!-- 控制按钮 -->
                <div class="flex space-x-3">
                    <button id="start-btn" class="flex-1 bg-accent hover:bg-accent/90 text-white py-2 px-4 rounded-lg transition flex items-center justify-center">
                        <i class="fa fa-play mr-2"></i>开始
                    </button>
                    <button id="pause-btn" class="flex-1 bg-gray-500 hover:bg-gray-600 text-white py-2 px-4 rounded-lg transition flex items-center justify-center" disabled>
                        <i class="fa fa-pause mr-2"></i>暂停
                    </button>
                    <button id="reset-btn" class="flex-1 bg-primary hover:bg-primary/90 text-white py-2 px-4 rounded-lg transition flex items-center justify-center">
                        <i class="fa fa-refresh mr-2"></i>重置
                    </button>
                </div>
            </div>
            
            <!-- 阅读区域 -->
            <div class="lg:col-span-2">
                <div class="bg-white rounded-xl shadow-md p-6 mb-6 transition-colors duration-300">
                    <h2 class="text-xl font-semibold mb-4 flex items-center">
                        <i class="fa fa-book mr-2 text-primary"></i>阅读区域
                    </h2>
                    
                    <!-- 可调整大小的阅读面板 -->
                    <div id="reader-container" class="scroll-container border-2 border-gray-200 rounded-lg bg-light min-h-[300px] min-w-full relative overflow-hidden transition-colors duration-300">
                        <!-- 双容器实现无缝滚动 -->
                        <div id="text-content-1" class="text-scroll p-4 text-container preserve-format monospace"></div>
                        <div id="text-content-2" class="text-scroll p-4 text-container preserve-format monospace"></div>
                        <div id="resize-handle" class="resize-handle"></div>
                    </div>
                </div>
                
                <!-- 文本输入区域 -->
                <div class="bg-white rounded-xl shadow-md p-6 transition-colors duration-300">
                    <h2 class="text-xl font-semibold mb-4 flex items-center">
                        <i class="fa fa-paste mr-2 text-primary"></i>文本输入
                    </h2>
                    <textarea 
                        id="text-input" 
                        class="w-full h-32 p-4 border-2 border-gray-200 rounded-lg focus:outline-none focus:border-primary transition resize-none transition-colors duration-300 font-mono text-sm"
                        placeholder="请在此输入或粘贴要阅读的文本，包括特殊符号和加密内容..."
                    ></textarea>
                    <div class="mt-3 flex justify-end space-x-3">
                        <button id="clear-btn" class="bg-gray-500 hover:bg-gray-600 text-white py-2 px-6 rounded-lg transition flex items-center btn-tooltip" data-tooltip="清空文本">
                            <i class="fa fa-trash mr-2"></i>清空
                        </button>
                        <button id="copy-btn" class="bg-secondary hover:bg-secondary/90 text-white py-2 px-6 rounded-lg transition flex items-center btn-tooltip" data-tooltip="复制文本（文本不足时可用）">
                            <i class="fa fa-copy mr-2"></i>复制
                        </button>
                        <button id="load-text-btn" class="bg-primary hover:bg-primary/90 text-white py-2 px-6 rounded-lg transition flex items-center">
                            <i class="fa fa-download mr-2"></i>加载文本
                        </button>
                    </div>
                </div>
            </div>
        </main>
        
        <!-- 页脚 -->
        <footer class="mt-12 text-center text-gray-600 text-sm">
            <p>文字阅读模拟器 &copy; 2023</p>
            <p class="mt-2">
                作者: <a href="https://space.bilibili.com/395868952" target="_blank" class="text-primary hover:underline">Aenfrisk/荷兰烈马</a>
            </p>
        </footer>
    </div>

    <script>
        // 全局变量
        let scrollInterval;
        let isScrolling = false;
        let currentPosition = 0;
        let originalText = "▝█▞▓▝▐▚▟▜███▓▀▜▝▛▗▞▝▟▐▝▀▙██▓▗▓▓▓▝▘▝▓▓▞▐▄▓▓▞▀▖▓▓▞▀▖▓▓▞█▗▓▓▖▀▛▓▓▖▓▌▓▓▐▟▌▓▓▞█▗▓▓▟▖▐▓▓▞▀▖▓▓▞▝▌▓▓▖▗▄▓▓▖▓▌▓▓▄▗▌▓▓▄▖▚▓▓▄▄▓\n\n这是第二行示例文本，用于展示换行效果\n\n第三行文本，包含各种符号：!@#$%^&*()_+{}[]|\\:;\"'<>,.?/";
        let displayText = originalText;
        let textSize = { width: 0, height: 0 };
        let isDarkMode = false;
        let textRepeated = false;
        
        // DOM 元素
        const textContent1 = document.getElementById('text-content-1');
        const textContent2 = document.getElementById('text-content-2');
        const readerContainer = document.getElementById('reader-container');
        const resizeHandle = document.getElementById('resize-handle');
        const textInput = document.getElementById('text-input');
        const loadTextBtn = document.getElementById('load-text-btn');
        const clearBtn = document.getElementById('clear-btn');
        const copyBtn = document.getElementById('copy-btn');
        const startBtn = document.getElementById('start-btn');
        const pauseBtn = document.getElementById('pause-btn');
        const resetBtn = document.getElementById('reset-btn');
        const directionRadios = document.querySelectorAll('input[name="direction"]');
        const speedSlider = document.getElementById('speed');
        const speedValue = document.getElementById('speed-value');
        const speedMultiplier = document.getElementById('speed-multiplier');
        const speedMultiplierValue = document.getElementById('speed-multiplier-value');
        const lineHeightSlider = document.getElementById('line-height');
        const lineHeightValue = document.getElementById('line-height-value');
        const letterSpacingSlider = document.getElementById('letter-spacing');
        const letterSpacingValue = document.getElementById('letter-spacing-value');
        const fontSizeSlider = document.getElementById('font-size');
        const fontSizeValue = document.getElementById('font-size-value');
        const themeToggle = document.getElementById('theme-toggle');
        
        // 获取当前选中的方向
        function getCurrentDirection() {
            for (const radio of directionRadios) {
                if (radio.checked) {
                    return radio.value;
                }
            }
            return 'vertical';
        }
        
        // 安全地转义文本中的特殊字符，确保格式保留
        function escapeSpecialChars(text) {
            // 只转义必要的HTML字符，保留文本结构
            return text
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/\t/g, '    '); // 将制表符转换为四个空格
        }
        
        // 测量文本尺寸 - 特别优化了竖向滚动时的计算
        function measureText(text) {
            const tempDiv = document.createElement('div');
            tempDiv.style.visibility = 'hidden';
            tempDiv.style.position = 'absolute';
            tempDiv.style.whiteSpace = 'pre-wrap';
            tempDiv.style.wordWrap = 'break-word';
            tempDiv.style.overflowWrap = 'break-word';
            tempDiv.style.wordBreak = 'normal';
            tempDiv.style.fontSize = `${fontSizeSlider.value}px`;
            tempDiv.style.lineHeight = lineHeightSlider.value;
            tempDiv.style.letterSpacing = `${letterSpacingSlider.value}px`;
            tempDiv.style.width = getCurrentDirection() === 'horizontal' ? 'auto' : `${readerContainer.clientWidth}px`;
            tempDiv.classList.add('monospace');
            tempDiv.innerHTML = escapeSpecialChars(text);
            document.body.appendChild(tempDiv);
            
            // 强制重排以获取准确尺寸
            tempDiv.offsetHeight;
            
            const dimensions = {
                width: tempDiv.offsetWidth,
                height: tempDiv.offsetHeight
            };
            
            document.body.removeChild(tempDiv);
            return dimensions;
        }
        
        // 检查文本是否足够填充显示面板
        function isTextEnoughToFillPanel() {
            const containerSize = getCurrentDirection() === 'horizontal' 
                ? readerContainer.clientWidth 
                : readerContainer.clientHeight;
                
            const textDimension = getCurrentDirection() === 'horizontal'
                ? textSize.width
                : textSize.height;
                
            return textDimension >= containerSize;
        }
        
        // 更新复制按钮状态
        function updateCopyButtonState() {
            const textEnough = isTextEnoughToFillPanel();
            copyBtn.disabled = textEnough;
            
            if (textEnough) {
                copyBtn.setAttribute('data-tooltip', '文本已足够填充面板，无需复制');
            } else {
                copyBtn.setAttribute('data-tooltip', '复制文本（文本不足时可用）');
            }
        }
        
        // 调整文本以填充容器，同时保留格式 - 特别优化竖向滚动
        function adjustTextToFit() {
            const containerWidth = readerContainer.clientWidth;
            const containerHeight = readerContainer.clientHeight;
            const direction = getCurrentDirection();
            
            // 先测量原始文本尺寸
            textSize = measureText(originalText);
            
            // 检查文本是否足够填充面板
            const containerSize = direction === 'horizontal' ? containerWidth : containerHeight;
            const textDimension = direction === 'horizontal' ? textSize.width : textSize.height;
            
            // 重置重复标记
            textRepeated = false;
            
            let repeatCount = 1;
            // 只有当文本不足以填充面板时才重复文本
            if (textDimension < containerSize) {
                repeatCount = Math.ceil(containerSize / textDimension) + 1;
                repeatCount = Math.min(repeatCount, 10); // 限制最大重复次数
                textRepeated = true;
            }
            
            // 为重复文本添加分隔符，保持分段结构
            const separator = direction === 'horizontal' ? '    ' : '\n\n';
            const repeatedText = Array(repeatCount).fill(originalText).join(separator);
            
            // 使用innerHTML而非textContent以保留换行和空格
            textContent1.innerHTML = escapeSpecialChars(repeatedText);
            textContent2.innerHTML = escapeSpecialChars(repeatedText);
            
            [textContent1, textContent2].forEach(el => {
                el.style.fontSize = `${fontSizeSlider.value}px`;
                el.style.lineHeight = lineHeightSlider.value;
                el.style.letterSpacing = `${letterSpacingSlider.value}px`;
                el.style.width = direction === 'horizontal' ? 'auto' : '100%';
                
                // 横向滚动时不自动换行，保持一行显示
                if (direction === 'horizontal') {
                    el.style.whiteSpace = 'nowrap';
                    el.style.overflowWrap = 'normal';
                    el.style.wordWrap = 'normal';
                    el.style.wordBreak = 'normal';
                } else {
                    // 竖向滚动时自动换行，确保长文本正确显示
                    el.style.whiteSpace = 'pre-wrap';
                    el.style.overflowWrap = 'break-word';
                    el.style.wordWrap = 'break-word';
                    el.style.wordBreak = 'normal';
                }
            });
            
            resetPositions();
            updateCopyButtonState();
        }
        
        // 重置两个文本容器的位置
        function resetPositions() {
            currentPosition = 0;
            const direction = getCurrentDirection();
            
            textContent1.style.transform = 'translate(0, 0)';
            
            if (direction === 'horizontal') {
                textContent2.style.transform = `translateX(${textContent1.offsetWidth}px)`;
            } else {
                textContent2.style.transform = `translateY(${textContent1.offsetHeight}px)`;
            }
        }
        
        // 计算当前速度
        function getCurrentSpeed() {
            const baseSpeed = 200 - speedSlider.value;
            const multiplier = parseFloat(speedMultiplier.value);
            return baseSpeed * multiplier;
        }
        
        // 开始滚动
        function startScrolling() {
            if (isScrolling) return;
            
            isScrolling = true;
            startBtn.disabled = true;
            pauseBtn.disabled = false;
            
            if (scrollInterval) cancelAnimationFrame(scrollInterval);
            
            let lastTime = 0;
            function animate(timestamp) {
                if (!isScrolling) return;
                
                if (!lastTime) lastTime = timestamp;
                const deltaTime = timestamp - lastTime;
                lastTime = timestamp;
                
                const direction = getCurrentDirection();
                const speedFactor = getCurrentSpeed() / 500;
                const distance = deltaTime * speedFactor;
                
                currentPosition += distance;
                
                const container = readerContainer;
                const containerSize = direction === 'horizontal' ? container.clientWidth : container.clientHeight;
                const contentSize = direction === 'horizontal' ? textContent1.offsetWidth : textContent1.offsetHeight;
                
                // 当滚动到内容末尾时，循环回开头
                if (currentPosition >= contentSize) {
                    currentPosition -= contentSize;
                }
                
                // 应用滚动变换
                if (direction === 'horizontal') {
                    textContent1.style.transform = `translateX(-${currentPosition}px)`;
                    textContent2.style.transform = `translateX(${(contentSize - currentPosition)}px)`;
                } else {
                    textContent1.style.transform = `translateY(-${currentPosition}px)`;
                    textContent2.style.transform = `translateY(${(contentSize - currentPosition)}px)`;
                }
                
                scrollInterval = requestAnimationFrame(animate);
            }
            
            scrollInterval = requestAnimationFrame(animate);
        }
        
        // 暂停滚动
        function pauseScrolling() {
            if (!isScrolling) return;
            
            isScrolling = false;
            startBtn.disabled = false;
            pauseBtn.disabled = true;
            
            if (scrollInterval) {
                cancelAnimationFrame(scrollInterval);
                scrollInterval = null;
            }
        }
        
        // 重置滚动
        function resetScrolling() {
            pauseScrolling();
            resetPositions();
        }
        
        // 加载文本
        function loadText() {
            const text = textInput.value.trim() || originalText;
            originalText = text;
            adjustTextToFit();
            resetScrolling();
        }
        
        // 清空文本
        function clearText() {
            // 清空输入框
            textInput.value = '';
            
            // 重置阅读区域文本
            originalText = "▝█▞▓▝▐▚▟▜███▓▀▜▝▛▗▞▝▟▐▝▀▙██▓▗▓▓▓▝▘▝▓▓▞▐▄▓▓▞▀▖▓▓▞▀▖▓▓▞█▗▓▓▖▀▛▓▓▖▓▌▓▓▐▟▌▓▓▞█▗▓▓▟▖▐▓▓▞▀▖▓▓▞▝▌▓▓▖▗▄▓▓▖▓▌▓▓▄▗▌▓▓▄▖▚▓▓▄▄▓";
            adjustTextToFit();
            
            // 停止滚动
            resetScrolling();
            
            // 显示操作反馈
            showToast('文本已清空');
        }
        
        // 复制文本 - 仅在文本不足时生效
        function copyText() {
            // 检查文本是否足够填充面板
            if (isTextEnoughToFillPanel()) {
                showToast('文本已足够填充面板，无需复制');
                return;
            }
            
            const text = textInput.value.trim();
            
            if (!text) {
                showToast('没有可复制的内容');
                return;
            }
            
            // 使用Clipboard API复制文本
            navigator.clipboard.writeText(text).then(() => {
                showToast('文本已复制到剪贴板');
            }).catch(err => {
                console.error('复制失败:', err);
                showToast('复制失败，请手动复制');
            });
        }
        
        // 显示提示消息
        function showToast(message) {
            // 检查是否已有toast，如有则移除
            const existingToast = document.querySelector('.toast-notification');
            if (existingToast) {
                existingToast.remove();
            }
            
            // 创建新toast
            const toast = document.createElement('div');
            toast.className = 'toast-notification fixed bottom-4 right-4 bg-dark text-white px-4 py-2 rounded-lg shadow-lg z-50 transition-all duration-300 transform translate-y-0 opacity-100';
            toast.textContent = message;
            document.body.appendChild(toast);
            
            // 3秒后隐藏
            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-4');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }
        
        // 切换深色/浅色模式
        function toggleTheme() {
            isDarkMode = !isDarkMode;
            const body = document.body;
            
            if (isDarkMode) {
                body.classList.add('dark-mode');
                themeToggle.innerHTML = '<i class="fa fa-sun-o mr-2"></i><span>切换浅色模式</span>';
            } else {
                body.classList.remove('dark-mode');
                themeToggle.innerHTML = '<i class="fa fa-moon-o mr-2"></i><span>切换深色模式</span>';
            }
            
            // 强制重绘
            void body.offsetWidth;
            
            // 保存用户偏好
            localStorage.setItem('darkMode', isDarkMode);
        }
        
        // 初始化可调整大小功能
        function initResizable() {
            let isResizing = false;
            let startX, startY, startWidth, startHeight;
            
            resizeHandle.addEventListener('mousedown', (e) => {
                isResizing = true;
                startX = e.clientX;
                startY = e.clientY;
                startWidth = readerContainer.clientWidth;
                startHeight = readerContainer.clientHeight;
                
                document.addEventListener('mousemove', resize);
                document.addEventListener('mouseup', stopResize);
                e.preventDefault();
            });
            
            function resize(e) {
                if (!isResizing) return;
                
                const width = startWidth + (e.clientX - startX);
                const height = startHeight + (e.clientY - startY);
                
                if (width >= 300) {
                    readerContainer.style.width = `${width}px`;
                }
                if (height >= 200) {
                    readerContainer.style.height = `${height}px`;
                }
                
                adjustTextToFit();
            }
            
            function stopResize() {
                isResizing = false;
                document.removeEventListener('mousemove', resize);
                document.removeEventListener('mouseup', stopResize);
            }
        }
        
        // 事件监听
        loadTextBtn.addEventListener('click', loadText);
        clearBtn.addEventListener('click', clearText);
        copyBtn.addEventListener('click', copyText);
        startBtn.addEventListener('click', startScrolling);
        pauseBtn.addEventListener('click', pauseScrolling);
        resetBtn.addEventListener('click', resetScrolling);
        themeToggle.addEventListener('click', toggleTheme);
        
        directionRadios.forEach(radio => {
            radio.addEventListener('change', () => {
                adjustTextToFit();
                resetScrolling();
            });
        });
        
        speedSlider.addEventListener('input', () => {
            speedValue.textContent = speedSlider.value;
        });
        
        speedMultiplier.addEventListener('input', () => {
            speedMultiplierValue.textContent = `${parseFloat(speedMultiplier.value)}x`;
        });
        
        lineHeightSlider.addEventListener('input', () => {
            const value = parseFloat(lineHeightSlider.value).toFixed(1);
            lineHeightValue.textContent = value;
            adjustTextToFit();
        });
        
        letterSpacingSlider.addEventListener('input', () => {
            letterSpacingValue.textContent = letterSpacingSlider.value;
            adjustTextToFit();
        });
        
        fontSizeSlider.addEventListener('input', () => {
            fontSizeValue.textContent = fontSizeSlider.value;
            adjustTextToFit();
        });
        
        window.addEventListener('resize', () => {
            adjustTextToFit();
        });
        
        // 文本输入变化时更新状态
        textInput.addEventListener('input', () => {
            clearTimeout(textInput.updateTimer);
            textInput.updateTimer = setTimeout(() => {
                if (textInput.value.trim()) {
                    const tempOriginalText = originalText;
                    originalText = textInput.value.trim();
                    textSize = measureText(originalText);
                    updateCopyButtonState();
                    originalText = tempOriginalText;
                }
            }, 300);
        });
        
        // 初始化
        function init() {
            // 初始化输入框示例文本为用户提供的特殊字符
            textInput.value = originalText;
            
            // 应用保存的主题偏好
            const savedTheme = localStorage.getItem('darkMode');
            if (savedTheme === 'true') {
                isDarkMode = true;
                document.body.classList.add('dark-mode');
                themeToggle.innerHTML = '<i class="fa fa-sun-o mr-2"></i><span>切换浅色模式</span>';
            }
            
            adjustTextToFit();
            initResizable();
        }
        
        // 确保DOM完全加载后初始化
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>
