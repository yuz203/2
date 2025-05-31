<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TS音乐播放器 | Taylor's Player</title>
    <style>
        /* 全局样式重置 */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', sans-serif;
        }

        body {
            min-height: 100vh;
            background: #f8e8f0; /* 泰勒标志性粉色背景 */
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        /* 播放器容器 */
        .player-container {
            background: white;
            max-width: 500px;
            width: 100%;
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(211, 122, 164, 0.2); /* 柔和阴影 */
        }

        /* 专辑封面 */
        .cover-img {
            width: 100%;
            height: 300px;
            border-radius: 15px;
            object-fit: cover; /* 图片填充方式 */
            margin-bottom: 20px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1); /* 封面阴影 */
        }

        /* 歌曲信息 */
        .song-info {
            text-align: center;
            margin-bottom: 20px;
        }
        .song-title {
            font-size: 1.5rem;
            color: #d37aa4; /* 主题色 */
            margin-bottom: 5px;
        }
        .artist {
            color: #999;
            font-size: 1rem;
        }

        /* 进度条 */
        .progress-container {
            background: #eee;
            height: 5px;
            border-radius: 3px;
            margin: 20px 0;
            cursor: pointer;
        }
        .progress {
            background: #d37aa4;
            height: 100%;
            width: 0%;
            border-radius: inherit;
            transition: width 0.1s linear;
        }

        /* 时间显示 */
        .time-container {
            display: flex;
            justify-content: space-between;
            color: #666;
            font-size: 0.9rem;
            margin-top: 5px;
        }

        /* 控制按钮 */
        .controls {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 30px;
            margin: 30px 0;
        }
        .control-btn {
            background: none;
            border: none;
            font-size: 1.5rem;
            color: #d37aa4;
            cursor: pointer;
            transition: transform 0.2s; /* 悬停动画 */
        }
        .control-btn:hover {
            transform: scale(1.1); /* 悬停放大 */
        }
        .play-pause-btn {
            font-size: 2.5rem; /* 播放/暂停按钮更大 */
        }

        /* 音量控制 */
        .volume-container {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
        }
        .volume-slider {
            width: 150px;
            height: 5px;
            background: #eee;
            border-radius: 3px;
            -webkit-appearance: none; /* 隐藏默认样式 */
            appearance: none;
        }
        .volume-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 15px;
            height: 15px;
            background: #d37aa4;
            border-radius: 50%;
            cursor: pointer;
        }

        /* 本地导入按钮 */
        .import-container {
            text-align: center;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="player-container">
        <!-- 专辑封面 -->
        <img src="https://picsum.photos/500/300?random=1" class="cover-img" id="coverImg">
        
        <!-- 歌曲信息 -->
        <div class="song-info">
            <h2 class="song-title" id="songTitle">Love Story</h2>
            <p class="artist" id="artist">Taylor Swift</p>
        </div>

        <!-- 进度条 -->
        <div class="progress-container" id="progressContainer">
            <div class="progress" id="progress"></div>
        </div>

        <!-- 时间显示 -->
        <div class="time-container">
            <span id="currentTime">0:00</span>
            <span id="duration">0:00</span>
        </div>

        <!-- 控制按钮（上一首/播放暂停/下一首） -->
        <div class="controls">
            <button class="control-btn" id="prevBtn">⏮</button>
            <button class="control-btn play-pause-btn" id="playPauseBtn">▶</button>
            <button class="control-btn" id="nextBtn">⏭</button>
        </div>

        <!-- 音量控制 -->
        <div class="volume-container">
            <i class="control-btn">🔊</i>
            <input type="range" min="0" max="1" step="0.1" value="1" class="volume-slider" id="volumeSlider">
        </div>

        <!-- 本地MP3导入按钮 -->
        <div class="import-container">
            <input type="file" id="localFileInput" accept="audio/mpeg" style="display: none;">
            <button class="control-btn" id="importBtn">📁 选择本地MP3</button>
        </div>
    </div>

    <script>
        // -------------------------- 配置区：替换为你的音乐资源 --------------------------
        // 网络歌曲列表（示例，替换为你的GitHub Raw URL或其他公网MP3链接）
        const songs = [
            {
                title: "Love Story",
                artist: "Taylor Swift",
                cover: "https://picsum.photos/500/300?random=1",  // 专辑封面图链接（可替换为你的图片）
                audio: "https://raw.githubusercontent.com/你的用户名/taylor-music/main/love-story.mp3"  // 替换为你的MP3 Raw URL
            },
            {
                title: "Shake It Off",
                artist: "Taylor Swift",
                cover: "https://picsum.photos/500/300?random=2",
                audio: "https://raw.githubusercontent.com/你的用户名/taylor-music/main/shake-it-off.mp3"
            },
            {
                title: "Blank Space",
                artist: "Taylor Swift",
                cover: "https://picsum.photos/500/300?random=3",
                audio: "https://raw.githubusercontent.com/你的用户名/taylor-music/main/blank-space.mp3"
            }
        ];
        // ---------------------------------------------------------------------------

        // 播放器核心变量
        let currentSongIndex = 0;  // 当前播放歌曲索引
        const audio = new Audio(); // HTML5 Audio对象
        const playPauseBtn = document.getElementById('playPauseBtn');
        const progress = document.getElementById('progress');
        const progressContainer = document.getElementById('progressContainer');
        const currentTime = document.getElementById('currentTime');
        const duration = document.getElementById('duration');
        const coverImg = document.getElementById('coverImg');
        const songTitle = document.getElementById('songTitle');
        const artist = document.getElementById('artist');
        const volumeSlider = document.getElementById('volumeSlider');

        // 初始化：加载第一首歌曲
        function loadSong(song) {
            audio.src = song.audio;
            coverImg.src = song.cover;
            songTitle.textContent = song.title;
            artist.textContent = song.artist;
        }
        loadSong(songs[currentSongIndex]);

        // 播放/暂停控制
        function togglePlayPause() {
            if (audio.paused) {
                audio.play();
                playPauseBtn.textContent = "⏸"; // 切换为暂停图标
            } else {
                audio.pause();
                playPauseBtn.textContent = "▶"; // 切换为播放图标
            }
        }

        // 进度条更新（随时间自动推进）
        function updateProgress(e) {
            const { duration, currentTime } = e.srcElement;
            const progressPercent = (currentTime / duration) * 100;
            progress.style.width = `${progressPercent}%`;

            // 格式化时间（0:00 格式）
            const formatTime = (time) => {
                const minutes = Math.floor(time / 60);
                const seconds = Math.floor(time % 60);
                return `${minutes}:${seconds.toString().padStart(2, '0')}`; // 补零（如 1:05）
            };

            currentTime.textContent = formatTime(currentTime);
            duration.textContent = formatTime(duration);
        }

        // 点击进度条跳转播放位置
        function setProgress(e) {
            const width = this.clientWidth; // 进度条总宽度
            const clickX = e.offsetX; // 用户点击的位置
            audio.currentTime = (clickX / width) * audio.duration; // 计算跳转时间
        }

        // 切换歌曲（上一首/下一首）
        function changeSong(direction) {
            currentSongIndex = (currentSongIndex + direction + songs.length) % songs.length; // 循环切换
            loadSong(songs[currentSongIndex]);
            audio.play(); // 切换后自动播放
            playPauseBtn.textContent = "⏸";
        }

        // 音量调节
        function setVolume() {
            audio.volume = this.value; // 范围 0-1
        }

        // 本地MP3导入逻辑
        const importBtn = document.getElementById('importBtn');
        const localFileInput = document.getElementById('localFileInput');

        importBtn.addEventListener('click', () => {
            localFileInput.click(); // 触发文件选择框
        });

        localFileInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (file && file.type === 'audio/mpeg') { // 仅允许MP3文件
                // 生成临时URL（浏览器创建的blob链接）
                const localAudioUrl = URL.createObjectURL(file);
                audio.src = localAudioUrl;
                audio.play();
                playPauseBtn.textContent = "⏸";

                // 更新歌曲信息（显示文件名）
                songTitle.textContent = file.name.replace('.mp3', ''); // 去除.mp3后缀
                artist.textContent = "Taylor Swift"; // 固定为泰勒
                coverImg.src = "https://picsum.photos/500/300?random=4"; // 本地音乐默认封面
            }
        });

        // 事件监听（绑定按钮和功能）
        playPauseBtn.addEventListener('click', togglePlayPause);
        audio.addEventListener('timeupdate', updateProgress); // 时间变化时更新进度条
        progressContainer.addEventListener('click', setProgress); // 点击进度条跳转
        audio.addEventListener('ended', () => changeSong(1)); // 歌曲结束自动切下一首
        document.getElementById('prevBtn').addEventListener('click', () => changeSong(-1)); // 上一首按钮
        document.getElementById('nextBtn').addEventListener('click', () => changeSong(1)); // 下一首按钮
        volumeSlider.addEventListener('input', setVolume); // 音量滑动条
    </script>
</body>
</html>
    
