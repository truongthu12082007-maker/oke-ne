<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ứng Dụng Chấm Điểm Phát Âm Tiếng Anh (en-US)</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 500px;
            text-align: center;
        }
        h2 {
            color: #2c3e50;
            margin-bottom: 20px;
        }
        .sentence-box {
            background-color: #eef2f3;
            padding: 15px;
            border-radius: 8px;
            font-size: 18px;
            font-weight: bold;
            color: #34495e;
            margin-bottom: 20px;
            border-left: 5px solid #3498db;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 16px;
            border-radius: 25px;
            cursor: pointer;
            transition: background 0.3s;
            margin: 5px;
        }
        button:hover {
            background-color: #2980b9;
        }
        button:disabled {
            background-color: #bdc3c7;
            cursor: not-allowed;
        }
        #btnStop {
            background-color: #e74c3c;
        }
        #btnStop:hover {
            background-color: #c0392b;
        }
        .status {
            margin-top: 15px;
            font-style: italic;
            color: #7f8c8d;
        }
        .result-box {
            margin-top: 25px;
            padding: 15px;
            border-radius: 8px;
            display: none;
        }
        .score {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .details {
            font-size: 16px;
            color: #555;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Phát Âm Chuẩn Anh - Mỹ</h2>
    <p style="color: #7f8c8d;">Nhấn "Bắt đầu nói" và đọc to câu dưới đây:</p>
    
    <div class="sentence-box" id="targetSentence">
        Beautiful weather we are having today.
    </div>

    <button id="btnStart">🎤 Bắt đầu nói</button>
    <button id="btnStop" disabled>🛑 Dừng</button>
    <button id="btnNext">Đổi câu khác ➡️</button>

    <div class="status" id="statusText">Sẵn sàng.</div>

    <div class="result-box" id="resultBox">
        <div class="score" id="scoreText">0%</div>
        <div class="details" id="detailsText"></div>
    </div>
</div>

<script>
    // Danh sách câu mẫu để luyện tập
    const sentences = [
        "Beautiful weather we are having today.",
        "An apple a day keeps the doctor away.",
        "How are you doing this morning?",
        "Practice makes perfect.",
        "Google Speech Recognition is very accurate."
    ];

    let currentIdx = 0;
    const targetSentenceElem = document.getElementById('targetSentence');
    const btnStart = document.getElementById('btnStart');
    const btnStop = document.getElementById('btnStop');
    const btnNext = document.getElementById('btnNext');
    const statusText = document.getElementById('statusText');
    const resultBox = document.getElementById('resultBox');
    const scoreText = document.getElementById('scoreText');
    const detailsText = document.getElementById('detailsText');

    // Khởi tạo Web Speech API (Google api tích hợp trong browser)
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
        alert("Trình duyệt của bạn không hỗ trợ Web Speech API. Vui lòng dùng Google Chrome.");
    }

    const recognition = new SpeechRecognition();
    recognition.continuous = false; // Dừng khi người dùng ngắt câu
    recognition.lang = 'en-US';     // Thiết lập chuẩn Anh - Mỹ
    recognition.interimResults = false; // Chỉ lấy kết quả cuối cùng
    recognition.maxAlternatives = 1;

    // Sự kiện khi bấm nút Đổi câu
    btnNext.addEventListener('click', () => {
        currentIdx = (currentIdx + 1) % sentences.length;
        targetSentenceElem.innerText = sentences[currentIdx];
        resultBox.style.display = 'none';
        statusText.innerText = "Sẵn sàng.";
    });

    // Sự kiện khi bấm nút Bắt đầu
    btnStart.addEventListener('click', () => {
        recognition.start();
        btnStart.disabled = true;
        btnStop.disabled = false;
        statusText.innerText = "Đang lắng nghe giọng nói của bạn (en-US)...";
        resultBox.style.display = 'none';
    });

    // Sự kiện khi bấm nút Dừng
    btnStop.addEventListener('click', () => {
        recognition.stop();
    });

    // Xử lý khi có kết quả trả về từ Google API
    recognition.onresult = (event) => {
        const textSpoken = event.results[0][0].transcript;
        const confidence = event.results[0][0].confidence;
        
        analyzePronunciation(textSpoken);
    };

    // Xử lý khi kết thúc nhận diện
    recognition.onend = () => {
        btnStart.disabled = false;
        btnStop.disabled = true;
    };

    // Xử lý lỗi
    recognition.onerror = (event) => {
        statusText.innerText = "Có lỗi xảy ra: " + event.error;
        btnStart.disabled = false;
        btnStop.disabled = true;
    };

    // Hàm thuật toán so sánh chuỗi để tính điểm % chính xác (Levenshtein Distance)
    function getLevenshteinDistance(a, b) {
        const matrix = [];
        for (let i = 0; i <= b.length; i++) { matrix[i] = [i]; }
        for (let j = 0; j <= a.length; j++) { matrix[0][j] = j; }
        for (let i = 1; i <= b.length; i++) {
            for (let j = 1; j <= a.length; j++) {
                if (b.charAt(i - 1) === a.charAt(j - 1)) {
                    matrix[i][j] = matrix[i - 1][j - 1];
                } else {
                    matrix[i][j] = Math.min(
                        matrix[i - 1][j - 1] + 1, // thay thế
                        matrix[i][j - 1] + 1,     // chèn
                        matrix[i - 1][j] + 1      // xóa
                    );
                }
            }
        }
        return matrix[b.length][a.length];
    }

    // Hàm tính điểm và hiển thị kết quả
    function analyzePronunciation(spoken) {
        statusText.innerText = "Đã nhận diện xong!";
        
        // Chuẩn hóa chuỗi (viết thường, xóa dấu câu cơ bản) để so sánh chính xác hơn
        const cleanText = (str) => str.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~()?]/g,"").trim();
        
        const target = cleanText(targetSentenceElem.innerText);
        const userSpoken = cleanText(spoken);

        const distance = getLevenshteinDistance(target, userSpoken);
        const maxLength = Math.max(target.length, userSpoken.length);
        
        // Tính phần trăm chính xác dựa trên khoảng cách ký tự
        let accuracyScore = Math.round(((maxLength - distance) / maxLength) * 100);
        if (accuracyScore < 0) accuracyScore = 0;

        // Hiển thị kết quả trực quan
        resultBox.style.display = 'block';
        scoreText.innerText = accuracyScore + "%";
        
        if (accuracyScore >= 80) {
            resultBox.style.backgroundColor = "#d4edda";
            scoreText.style.color = "#155724";
        } else if (accuracyScore >= 50) {
            resultBox.style.backgroundColor = "#fff3cd";
            scoreText.style.color = "#856404";
        } else {
            resultBox.style.backgroundColor = "#f8d7da";
            scoreText.style.color = "#721c24";
        }

        detailsText.innerHTML = `
            <p><strong>Câu gốc:</strong> "${targetSentenceElem.innerText}"</p>
            <p><strong>Bạn đã nói:</strong> "<span style="color: #2980b9;">${spoken}</span>"</p>
        `;
    }
</script>

</body>
</html>
