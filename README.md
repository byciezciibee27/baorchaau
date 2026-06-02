<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chấm Điểm Phát Âm Tiếng Trung</title>
    <style>
        :root {
            --primary-color: #ff4d4f;
            --success-color: #52c41a;
            --dark-color: #262626;
            --light-bg: #f5f5f5;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--light-bg);
            color: var(--dark-color);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 500px;
            text-align: center;
        }

        h1 {
            color: var(--primary-color);
            margin-bottom: 30px;
            font-size: 24px;
        }

        .phrase-box {
            background: #fff0f6;
            border: 2px dashed var(--primary-color);
            padding: 20px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .chinese-text {
            font-size: 32px;
            font-weight: bold;
            margin: 0 0 10px 0;
            color: #000;
        }

        .pinyin-text {
            font-size: 18px;
            color: #8c8c8c;
            margin: 0;
        }

        .btn {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 18px;
            border-radius: 50px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 4px 10px rgba(255, 77, 79, 0.3);
            width: 100%;
        }

        .btn:hover {
            opacity: 0.9;
            transform: translateY(-2px);
        }

        .btn:disabled {
            background-color: #bfbfbf;
            box-shadow: none;
            cursor: not-allowed;
        }

        .btn.recording {
            background-color: #faad14;
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.03); }
            100% { transform: scale(1); }
        }

        .result-section {
            margin-top: 30px;
            padding-top: 20px;
            border-top: 1px solid #f0f0f0;
            display: none;
        }

        .score {
            font-size: 48px;
            font-weight: bold;
            margin: 10px 0;
        }

        .user-transcript {
            font-style: italic;
            color: #595959;
            margin-bottom: 10px;
        }

        .feedback {
            font-size: 16px;
            font-weight: 500;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Ứng Dụng Phát Âm Tiếng Trung</h1>
    
    <div class="phrase-box">
        <p class="chinese-text" id="targetChinese">你好</p>
        <p class="pinyin-text" id="targetPinyin">Nǐ hǎo</p>
    </div>

    <button class="btn" id="voiceBtn">Bắt đầu nói</button>

    <div class="result-section" id="resultSection">
        <h3>Kết quả của bạn:</h3>
        <p class="user-transcript">Bạn đã nói: "<span id="spokenText">...</span>"</p>
        <div class="score" id="scoreDisplay">0%</div>
        <p class="feedback" id="feedbackText"></p>
        
        <button class="btn" id="nextBtn" style="background-color: #52c41a; margin-top: 15px; box-shadow: 0 4px 10px rgba(82, 196, 26, 0.3);">Câu tiếp theo</button>
    </div>
</div>

<script>
    // Danh sách các câu tiếng Trung mẫu để luyện tập
    const samplePhrases = [
        { zh: "你好", py: "Nǐ hǎo" },
        { zh: "谢谢", py: "Xièxie" },
        { zh: "我爱你", py: "Wǒ ài nǐ" },
        { zh: "中国", py: "Zhōngguó" },
        { zh: "你叫 center 名字", py: "Nǐ jiào shénme míngzì? (Nút test)" }, // Đổi câu phức tạp hơn chút bên dưới
        { zh: "你叫什么名字", py: "Nǐ jiào shénme míngzì?" },
        { zh: "今天天气很好", py: "Jīntiān tiānqì hěn hǎo" }
    ];

    let currentIndex = 0;

    // Kiểm tra hỗ trợ trình duyệt
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
        alert("Trình duyệt của bạn không hỗ trợ Web Speech API. Hãy thử lại trên Google Chrome!");
    }

    const recognition = new SpeechRecognition();
    recognition.lang = 'zh-CN'; // Cài đặt nhận diện Tiếng Trung Quốc (Mandarin)
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    // DOM Elements
    const voiceBtn = document.getElementById('voiceBtn');
    const targetChinese = document.getElementById('targetChinese');
    const targetPinyin = document.getElementById('targetPinyin');
    const resultSection = document.getElementById('resultSection');
    const spokenText = document.getElementById('spokenText');
    const scoreDisplay = document.getElementById('scoreDisplay');
    const feedbackText = document.getElementById('feedbackText');
    const nextBtn = document.getElementById('nextBtn');

    // Khởi tạo câu đầu tiên
    function loadPhrase(index) {
        targetChinese.textContent = samplePhrases[index].zh;
        targetPinyin.textContent = samplePhrases[index].py;
        resultSection.style.display = 'none';
    }
    loadPhrase(currentIndex);

    // Sự kiện khi bấm nút nói
    voiceBtn.addEventListener('click', () => {
        recognition.start();
        voiceBtn.textContent = "Đang lắng nghe... Hãy nói đi!";
        voiceBtn.classList.add('recording');
        voiceBtn.disabled = true;
    });

    // Xử lý khi có kết quả trả về từ Google API
    recognition.onresult = function(event) {
        const resultText = event.results[0][0].transcript;
        // Xóa dấu câu cơ bản trong tiếng Trung nếu có để so sánh chính xác hơn
        const cleanSpoken = resultText.replace(/[。，？！、]/g, "").trim();
        const cleanTarget = samplePhrases[currentIndex].zh.replace(/[。，？！、]/g, "").trim();

        spokenText.textContent = cleanSpoken;

        // Tính điểm dựa trên thuật toán so sánh chuỗi (Dice's Coefficient đơn giản cho ký tự Trung)
        const score = calculateScore(cleanSpoken, cleanTarget);
        
        // Hiển thị kết quả
        scoreDisplay.textContent = `${score}%`;
        resultSection.style.display = 'block';

        if (score >= 80) {
            scoreDisplay.style.color = '#52c41a';
            feedbackText.textContent = "Tuyệt vời! Bạn phát âm rất chuẩn.";
        } else if (score >= 50) {
            scoreDisplay.style.color = '#faad14';
            feedbackText.textContent = "Khá tốt, nhưng cần chú ý bật hơi hoặc thanh điệu rõ hơn.";
        } else {
            scoreDisplay.style.color = '#ff4d4f';
            feedbackText.textContent = "Chưa chính xác lắm. Hãy nghe kỹ lại và thử lại nhé!";
        }
    };

    recognition.onspeechend = function() {
        resetButton();
    };

    recognition.onerror = function(event) {
        resetButton();
        alert("Có lỗi xảy ra hoặc không nghe thấy tiếng: " + event.error);
    };

    function resetButton() {
        voiceBtn.textContent = "Bắt đầu nói";
        voiceBtn.classList.remove('recording');
        voiceBtn.disabled = false;
    }

    // Hàm tính điểm khớp từ (Phù hợp với ký tự đơn của tiếng Trung)
    function calculateScore(str1, str2) {
        if (str1 === str2) return 100;
        if (!str1 || !str2) return 0;

        let matches = 0;
        const arr1 = Array.from(str1);
        const arr2 = Array.from(str2);

        arr1.forEach(char => {
            const index = arr2.indexOf(char);
            if (index !== -1) {
                matches++;
                arr2.splice(index, 1); // Tránh tính trùng ký tự
            }
        });

        // Điểm số dựa trên tỷ lệ phần trăm số ký tự khớp trên tổng số ký tự mẫu
        let finalScore = Math.round((matches / Array.from(str2).length) * 100);
        return finalScore > 100 ? 100 : finalScore;
    }

    // Nút chuyển câu
    nextBtn.addEventListener('click', () => {
        currentIndex = (currentIndex + 1) % samplePhrases.length;
        loadPhrase(currentIndex);
    });
</script>

</body>
</html>
