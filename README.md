<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>小手机 · 虚拟OS</title>
    <!-- 简单reset & 基础风格 -->
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: system-ui, -apple-system, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
        }

        :root {
            /* 品牌色 */
            --brand: #07C160;
            /* 浅色模式 */
            --bg-primary: #f8fafc;
            --bg-secondary: #ffffff;
            --bg-card: rgba(255,255,255,0.8);
            --bg-blur: rgba(255,255,255,0.7);
            --text-primary: #1e293b;
            --text-secondary: #475569;
            --text-inverse: #ffffff;
            --border-light: rgba(0,0,0,0.05);
            --shadow: 0 20px 40px rgba(0,0,0,0.08), 0 8px 20px rgba(0,0,0,0.06);
            --lockscreen-bg: #e2e8f0;
            --digit-bg: #f1f5f9;
            --digit-text: #0f172a;
        }

        /* 深色模式 —— 跟随系统 */
        @media (prefers-color-scheme: dark) {
            :root {
                --bg-primary: #0f172a;
                --bg-secondary: #1e293b;
                --bg-card: rgba(30,41,59,0.8);
                --bg-blur: rgba(15,23,42,0.8);
                --text-primary: #f1f5f9;
                --text-secondary: #cbd5e1;
                --text-inverse: #ffffff;
                --border-light: rgba(255,255,255,0.08);
                --shadow: 0 20px 40px rgba(0,0,0,0.5);
                --lockscreen-bg: #0b1120;
                --digit-bg: #1e293b;
                --digit-text: #f1f5f9;
            }
        }

        body, html {
            width: 100%;
            height: 100%;
            background-color: var(--bg-primary);
            color: var(--text-primary);
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background-color 0.2s, color 0.2s;
        }

        /* 整个设备模拟卡片 */
        .phone-frame {
            width: 380px;
            height: 740px;
            background-color: var(--bg-secondary);
            border-radius: 44px;
            box-shadow: var(--shadow);
            overflow: hidden;
            position: relative;
            border: 1px solid var(--border-light);
            backdrop-filter: blur(2px);
            transition: background-color 0.2s;
        }

        /* 屏幕内容区域 */
        .screen {
            width: 100%;
            height: 100%;
            position: relative;
            background-color: var(--bg-primary);
            transition: background-color 0.2s;
            display: flex;
            flex-direction: column;
        }

        /* 页面切换视图容器 (简单用display控制, 无路由) */
        .view {
            width: 100%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 0;
            transition: opacity 0.3s ease, transform 0.3s ease;
            opacity: 0;
            pointer-events: none;
            display: flex;
            flex-direction: column;
        }

        .view.active {
            opacity: 1;
            pointer-events: all;
            transform: scale(1);
        }

        /* 加载界面 (view1) */
        .loading-view {
            background-color: var(--bg-secondary);
            justify-content: center;
            align-items: center;
            gap: 24px;
        }

        .brand-circle {
            width: 96px;
            height: 96px;
            background-color: var(--brand);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 30px rgba(7,193,96,0.5);
            animation: pulse 1.5s infinite ease-in-out;
        }

        .brand-circle span {
            font-size: 48px;
            font-weight: 600;
            color: white;
        }

        .loading-dots {
            display: flex;
            gap: 10px;
        }

        .loading-dots div {
            width: 12px;
            height: 12px;
            background-color: var(--brand);
            border-radius: 50%;
            animation: bounce 1.4s infinite;
        }

        .loading-dots div:nth-child(2) { animation-delay: 0.2s; }
        .loading-dots div:nth-child(3) { animation-delay: 0.4s; }

        @keyframes pulse {
            0% { transform: scale(1); opacity: 0.9; }
            50% { transform: scale(1.08); opacity: 1; }
            100% { transform: scale(1); opacity: 0.9; }
        }

        @keyframes bounce {
            0%, 80%, 100% { transform: scale(0.6); opacity: 0.4; }
            40% { transform: scale(1); opacity: 1; }
        }

        /* 锁屏界面 (view2) + 首次密码设置 */
        .lockscreen-view {
            background-color: var(--lockscreen-bg);
            background-image: radial-gradient(circle at 30% 30%, rgba(7,193,96,0.05) 0%, transparent 40%);
            justify-content: space-between;
            padding: 60px 24px 40px;
            align-items: center;
        }

        .lock-top {
            text-align: center;
            width: 100%;
        }

        .lock-time {
            font-size: 72px;
            font-weight: 300;
            letter-spacing: 2px;
            color: var(--text-primary);
            line-height: 1.1;
        }

        .lock-date {
            font-size: 18px;
            color: var(--text-secondary);
            margin-top: 8px;
        }

        .lock-icon {
            margin: 40px 0 20px;
            font-size: 44px;
            opacity: 0.8;
        }

        .lock-message {
            font-size: 16px;
            color: var(--text-secondary);
            margin-bottom: 30px;
        }

        .digit-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 16px;
            width: 100%;
            max-width: 280px;
            margin: 20px auto;
        }

        .digit-key {
            background-color: var(--digit-bg);
            border-radius: 50px;
            aspect-ratio: 1/1;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 28px;
            font-weight: 500;
            color: var(--digit-text);
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            cursor: pointer;
            transition: 0.1s;
            border: 1px solid var(--border-light);
            user-select: none;
        }

        .digit-key:active {
            background-color: var(--brand);
            color: white;
            transform: scale(0.94);
        }

        .dots-row {
            display: flex;
            gap: 16px;
            justify-content: center;
            margin: 24px 0 16px;
        }

        .dot {
            width: 14px;
            height: 14px;
            border-radius: 14px;
            background-color: var(--text-secondary);
            opacity: 0.3;
            transition: 0.1s;
        }

        .dot.filled {
            background-color: var(--brand);
            opacity: 1;
        }

        .lock-footer {
            font-size: 14px;
            color: var(--text-secondary);
            text-align: center;
        }

        /* 主屏幕 (view3) */
        .homescreen-view {
            background-color: var(--bg-primary);
            position: relative;
            padding: 60px 16px 20px;
            overflow-y: auto;
        }

        /* 状态栏模拟 */
        .status-bar {
            display: flex;
            justify-content: space-between;
            padding: 12px 20px;
            font-size: 15px;
            font-weight: 500;
            color: var(--text-primary);
            background-color: transparent;
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            z-index: 5;
        }

        /* 天气&时间小组件 */
        .weather-widget {
            background-color: var(--bg-card);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border-radius: 24px;
            padding: 16px 20px;
            margin: 16px 0 24px;
            border: 1px solid var(--border-light);
            box-shadow: 0 8px 20px rgba(0,0,0,0.02);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .weather-left {
            display: flex;
            flex-direction: column;
            gap: 4px;
        }

        .weather-city {
            font-size: 16px;
            font-weight: 500;
            color: var(--text-secondary);
        }

        .weather-temp {
            font-size: 40px;
            font-weight: 300;
            color: var(--text-primary);
            line-height: 1;
        }

        .weather-desc {
            font-size: 15px;
            color: var(--brand);
            font-weight: 500;
        }

        .weather-icon {
            font-size: 48px;
            background-color: rgba(7,193,96,0.1);
            width: 70px;
            height: 70px;
            border-radius: 35px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* 时间小组件 (其实是和天气一起, 但额外显示) */
        .time-sub {
            font-size: 17px;
            font-weight: 500;
            color: var(--text-primary);
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .time-sub span {
            background-color: var(--brand);
            color: white;
            padding: 4px 10px;
            border-radius: 40px;
            font-size: 13px;
        }

        /* App 图标网格 */
        .app-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px 8px;
            margin-top: 20px;
        }

        .app-icon {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 6px;
            cursor: default;
            transition: 0.1s;
        }

        .app-icon:active {
            opacity: 0.7;
            transform: scale(0.96);
        }

        .icon-bg {
            width: 60px;
            height: 60px;
            border-radius: 18px;
            background-color: var(--bg-card);
            backdrop-filter: blur(8px);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 32px;
            border: 1px solid var(--border-light);
            box-shadow: 0 5px 10px rgba(0,0,0,0.03);
            color: var(--brand);
        }

        .app-label {
            font-size: 11px;
            font-weight: 500;
            color: var(--text-secondary);
            max-width: 64px;
            text-align: center;
        }

        /* 品牌色点缀 */
        .brand-bg {
            background-color: var(--brand);
        }

        .dock {
            margin-top: 30px;
            padding: 12px 10px;
            background-color: var(--bg-blur);
            backdrop-filter: blur(20px);
            border-radius: 36px;
            display: flex;
            justify-content: space-around;
            border: 1px solid var(--border-light);
        }
        .dock .app-icon .icon-bg {
            width: 54px;
            height: 54px;
            border-radius: 16px;
        }

        /* 首次进入提示语 */
        .hint-text {
            color: var(--brand);
            font-size: 13px;
            text-align: center;
            margin: 16px 0 -8px;
            font-weight: 500;
        }
    </style>
</head>
<body>
    <div class="phone-frame">
        <div class="screen">

            <!-- 加载界面 view 1 (首次显示) -->
            <div id="view-loading" class="view loading-view active">
                <div class="brand-circle">
                    <span>小</span>
                </div>
                <div class="loading-dots">
                    <div></div>
                    <div></div>
                    <div></div>
                </div>
                <div style="color: var(--brand); font-weight: 500; margin-top: 20px;">启动中 ...</div>
            </div>

            <!-- 锁屏/首次密码设置 view 2 -->
            <div id="view-lockscreen" class="view lockscreen-view">
                <div class="lock-top">
                    <div class="lock-time" id="lock-time">19:30</div>
                    <div class="lock-date" id="lock-date">2025年3月5日 周三</div>
                    <div class="lock-icon">🔒</div>
                    <div class="lock-message" id="lock-message">输入6位密码</div>
                </div>

                <!-- 密码点 -->
                <div style="width: 100%;">
                    <div class="dots-row" id="pin-dots">
                        <span class="dot"></span>
                        <span class="dot"></span>
                        <span class="dot"></span>
                        <span class="dot"></span>
                        <span class="dot"></span>
                        <span class="dot"></span>
                    </div>

                    <!-- 数字键盘 -->
                    <div class="digit-grid" id="digit-keyboard">
                        <!-- 会用JS生成 -->
                    </div>

                    <div style="text-align: center; margin: 12px 0;" id="pin-action">
                        <span style="color: var(--brand); font-size: 15px; font-weight: 500;" id="clear-pin">清除</span>
                    </div>
                </div>

                <div class="lock-footer">
                    <span>首次使用请设置密码</span>
                </div>
            </div>

            <!-- 主屏幕 view 3 -->
            <div id="view-homescreen" class="view homescreen-view">
                <!-- 状态占位符 (内容随系统) -->
                <div class="status-bar">
                    <span id="home-time">19:30</span>
                    <span style="display: flex; gap: 4px;">📶 🔋 91%</span>
                </div>

                <!-- 天气+时间小组件 -->
                <div class="weather-widget">
                    <div class="weather-left">
                        <span class="weather-city">🏠 我的位置</span>
                        <span class="weather-temp">21°</span>
                        <div class="time-sub">
                            <span id="home-time-large">19:30</span>
                            <span>🌤️ 微风</span>
                        </div>
                        <span class="weather-desc">空气质量 · 优</span>
                    </div>
                    <div class="weather-icon">
                        🌤️
                    </div>
                </div>

                <!-- 常见app占位 -->
                <div class="app-grid">
                    <div class="app-icon">
                        <div class="icon-bg">💬</div>
                        <span class="app-label">聊天</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">👤</div>
                        <span class="app-label">通讯录</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">📞</div>
                        <span class="app-label">拨号</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">📷</div>
                        <span class="app-label">相机</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">🌐</div>
                        <span class="app-label">浏览器</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">⚙️</div>
                        <span class="app-label">设置</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">🎵</div>
                        <span class="app-label">音乐</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">📁</div>
                        <span class="app-label">文件</span>
                    </div>
                </div>

                <!-- 底部dock (常用) -->
                <div class="dock">
                    <div class="app-icon">
                        <div class="icon-bg">💬</div>
                        <span class="app-label">聊天</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">⚙️</div>
                        <span class="app-label">设置</span>
                    </div>
                    <div class="app-icon">
                        <div class="icon-bg">📞</div>
                        <span class="app-label">通讯录</span>
                    </div>
                </div>

                <div class="hint-text">小手机 · 全AI陪伴</div>
            </div>
        </div>
    </div>

    <script>
        (function() {
            // 状态管理
            let currentPin = '';
            const EXPECTED_PIN = '123456'; // 默认预设一个密码，但因为是首次进入，我们会引导用户设置，然后保存到localStorage

            // 视图元素
            const viewLoading = document.getElementById('view-loading');
            const viewLockscreen = document.getElementById('view-lockscreen');
            const viewHome = document.getElementById('view-homescreen');
            const lockMessage = document.getElementById('lock-message');
            const pinDots = document.querySelectorAll('#pin-dots .dot');
            const digitKeyboard = document.getElementById('digit-keyboard');
            const clearPinBtn = document.getElementById('clear-pin');

            // 时间元素
            const lockTimeEl = document.getElementById('lock-time');
            const lockDateEl = document.getElementById('lock-date');
            const homeTimeEl = document.getElementById('home-time');
            const homeTimeLarge = document.getElementById('home-time-large');

            // 判断是否首次 (localStorage 中无 'pin')
            const isFirstLaunch = !localStorage.getItem('user_pin');

            // 模拟首次加载界面停留后自动跳转到锁屏 (加载1.5s)
            setTimeout(() => {
                // 如果是首次，显示锁屏并提示设置密码；否则直接验证密码模式
                if (isFirstLaunch) {
                    lockMessage.innerText = '设置6位数字密码';
                    viewLoading.classList.remove('active');
                    viewLockscreen.classList.add('active');
                } else {
                    // 已经有密码，展示正常的锁屏验证
                    lockMessage.innerText = '输入密码';
                    viewLoading.classList.remove('active');
                    viewLockscreen.classList.add('active');
                }
                updateDateTime(); // 更新时间
            }, 1500);

            // 实时更新时间
            function updateDateTime() {
                const now = new Date();
                const hours = now.getHours().toString().padStart(2,'0');
                const minutes = now.getMinutes().toString().padStart(2,'0');
                const timeStr = `${hours}:${minutes}`;
                const year = now.getFullYear();
                const month = (now.getMonth()+1).toString().padStart(2,'0');
                const day = now.getDate().toString().padStart(2,'0');
                const weekdays = ['周日','周一','周二','周三','周四','周五','周六'];
                const weekday = weekdays[now.getDay()];
                const dateStr = `${year}年${month}月${day}日 ${weekday}`;

                if (lockTimeEl) lockTimeEl.innerText = timeStr;
                if (lockDateEl) lockDateEl.innerText = dateStr;
                if (homeTimeEl) homeTimeEl.innerText = timeStr;
                if (homeTimeLarge) homeTimeLarge.innerText = timeStr;
            }
            setInterval(updateDateTime, 1000);
            updateDateTime();

            // 生成数字键盘 1-9, 0, 删除占位
            function buildKeyboard() {
                let html = '';
                for (let i = 1; i <= 9; i++) {
                    html += `<div class="digit-key" data-digit="${i}">${i}</div>`;
                }
                // 第10个 空白占位用删除符号
                html += `<div class="digit-key" data-action="clear">⌫</div>`;
                html += `<div class="digit-key" data-digit="0">0</div>`;
                html += `<div class="digit-key" data-action="confirm" style="background-color: var(--brand); color: white;">↵</div>`;
                digitKeyboard.innerHTML = html;
            }
            buildKeyboard();

            // 更新小圆点
            function updateDots(length) {
                pinDots.forEach((dot, idx) => {
                    if (idx < length) dot.classList.add('filled');
                    else dot.classList.remove('filled');
                });
            }

            // 清除当前pin
            function resetPin() {
                currentPin = '';
                updateDots(0);
            }

            // 处理数字键盘点击
            digitKeyboard.addEventListener('click', (e) => {
                const target = e.target.closest('.digit-key');
                if (!target) return;

                const digit = target.dataset.digit;
                const action = target.dataset.action;

                if (digit !== undefined) {
                    // 数字
                    if (currentPin.length < 6) {
                        currentPin += digit;
                        updateDots(currentPin.length);
                    }
                } else if (action === 'clear') {
                    // 退格
                    currentPin = currentPin.slice(0, -1);
                    updateDots(currentPin.length);
                } else if (action === 'confirm') {
                    // 确认密码
                    handlePinSubmit();
                }
            });

            // 清除按钮
            clearPinBtn.addEventListener('click', () => {
                resetPin();
            });

            // 提交密码逻辑
            function handlePinSubmit() {
                if (currentPin.length !== 6) {
                    lockMessage.innerText = '请输入6位数字';
                    return;
                }

                if (isFirstLaunch) {
                    // 首次设置：保存密码到localStorage，然后进入主屏
                    localStorage.setItem('user_pin', currentPin);
                    // 跳转主屏幕
                    viewLockscreen.classList.remove('active');
                    viewHome.classList.add('active');
                    // 更新一下标记，以后就不是首次了，但为了演示，后续刷新页面就会用已存储的密码验证
                    location.reload(); // 简单刷新使isFirstLaunch变更 (实际可以更优雅，但为了演示流程)
                } else {
                    // 验证密码
                    const savedPin = localStorage.getItem('user_pin');
                    if (currentPin === savedPin) {
                        // 正确，进入主屏
                        viewLockscreen.classList.remove('active');
                        viewHome.classList.add('active');
                    } else {
                        lockMessage.innerText = '密码错误，重试';
                        resetPin();
                    }
                }
            }

            // 若已保存密码，且非首次，轻按锁屏键盘
            // 如果刷新后不是首次，锁屏正常

            // 对于首次之后再次加载，加载屏跳转锁屏，上面已处理

            // 额外的演示：如果已经登录过，但用户想回到锁屏？ 这里我们不做，因为默认加载完即锁屏。

            // 但为了让深色模式生效，完全跟随系统。 好

            // 增加锁屏界面的额外touch，防止键盘点击事件冒泡
        })();
    </script>

    <!-- 添加简单的说明，确保6位密码演示方便: 首次设置密码建议123456，但用户可自定义 -->
    <!-- 注意: 本示例为了演示首次设置流程，直接保存密码，刷新后会变为验证模式。请体验深色/浅色模式切换(跟随OS) -->
</body>
</html>
