<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Creator - Smart Assignment Detector</title>
    <!-- นำเข้าฟอนต์ Kanit เพื่อความสวยงามแบบสากล -->
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Kanit', sans-serif;
        }

        body {
            background: linear-gradient(135deg, #0f172a 0%, #1e1b4b 100%);
            color: #f8fafc;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 24px;
            padding: 2.5rem;
            width: 100%;
            max-width: 550px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
            text-align: center;
        }

        h1 {
            font-size: 1.8rem;
            font-weight: 600;
            margin-bottom: 0.5rem;
            background: linear-gradient(to right, #38bdf8, #818cf8);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        p.subtitle {
            color: #94a3b8;
            font-size: 0.95rem;
            margin-bottom: 2rem;
        }

        .btn-start {
            background: linear-gradient(135deg, #4f46e5 0%, #3730a3 100%);
            color: white;
            border: none;
            padding: 14px 32px;
            font-size: 1.1rem;
            font-weight: 500;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 12px rgba(79, 70, 229, 0.3);
            width: 100%;
        }

        .btn-start:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(79, 70, 229, 0.5);
            background: linear-gradient(135deg, #6366f1 0%, #4f46e5 100%);
        }

        .btn-start:disabled {
            background: #475569;
            color: #94a3b8;
            cursor: not-allowed;
            transform: none;
            box-shadow: none;
        }

        #webcam-wrapper {
            position: relative;
            width: 280px;
            height: 280px;
            margin: 0 auto 2rem auto;
            display: none; /* ซ่อนไว้ก่อนกดเริ่มทำงาน */
        }

        #webcam-container {
            width: 100%;
            height: 100%;
            border-radius: 50%; /* ปรับกล้องเป็นวงกลมสไตล์มินิมอล */
            overflow: hidden;
            border: 4px solid #4f46e5;
            box-shadow: 0 0 25px rgba(79, 70, 229, 0.4);
        }

        #webcam-container canvas {
            width: 100% !important;
            height: 100% !important;
            object-fit: cover;
        }

        .status-badge {
            position: absolute;
            bottom: -10px;
            left: 50%;
            transform: translateX(-50%);
            background: #10b981;
            color: white;
            padding: 4px 16px;
            border-radius: 9999px;
            font-size: 0.8rem;
            font-weight: 500;
            letter-spacing: 0.5px;
            box-shadow: 0 4px 10px rgba(16, 185, 129, 0.4);
            display: flex;
            align-items: center;
            gap: 6px;
        }

        .status-badge::before {
            content: '';
            width: 8px;
            height: 8px;
            background: white;
            border-radius: 50%;
            animation: blink 1.5s infinite;
        }

        @keyframes blink {
            0% { opacity: 0.2; }
            50% { opacity: 1; }
            100% { opacity: 0.2; }
        }

        #label-container {
            margin-top: 1.5rem;
            text-align: left;
        }

        .predict-row {
            background: rgba(15, 23, 42, 0.4);
            padding: 12px 16px;
            border-radius: 12px;
            margin-bottom: 10px;
            border: 1px solid rgba(255, 255, 255, 0.05);
        }

        .predict-info {
            display: flex;
            justify-content: space-between;
            font-size: 0.95rem;
            margin-bottom: 6px;
            font-weight: 400;
        }

        .class-name { color: #e2e8f0; }
        .class-percent { color: #38bdf8; font-weight: 500; }

        .progress-bar-bg {
            background: #334155;
            height: 8px;
            border-radius: 9999px;
            overflow: hidden;
        }

        .progress-bar-fill {
            background: linear-gradient(90deg, #3b82f6, #06b6d4);
            height: 100%;
            width: 0%;
            transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
            border-radius: 9999px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>AI Student Assignment Detector</h1>
    <p class="subtitle">ระบบปัญญาประดิษฐ์ตรวจจับและคัดแยกภาระงานนักเรียน</p>
    
    <div id="webcam-wrapper">
        <div id="webcam-container"></div>
        <div class="status-badge" id="scan-status">สแกนทุกๆ 3 วินาที</div>
    </div>

    <button type="button" class="btn-start" id="btn-start" onclick="init()">เริ่มใช้งานระบบ AI</button>
    
    <div id="label-container"></div>
</div>

<!-- โหลดไลบรารีที่จำเป็นจากเครือข่ายภายนอก -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

<script type="text/javascript">
    // ลิงก์โมเดล Teachable Machine ของกลุ่มคุณ[cite: 1]
    const URL = "https://teachablemachine.withgoogle.com/models/MASXoYhja/";[cite: 1]

    let model, webcam, labelContainer, maxPredictions;
    let predictionInterval; // ตัวแปรสำหรับควบคุมลูปเวลา 3 วินาที

    // ฟังก์ชันเริ่มต้นระบบและเปิดกล้อง[cite: 1]
    async function init() {
        const startBtn = document.getElementById("btn-start");
        startBtn.disabled = true;
        startBtn.innerText = "กำลังเชื่อมต่อโมเดลระบบ...";

        const modelURL = URL + "model.json";[cite: 1]
        const metadataURL = URL + "metadata.json";[cite: 1]

        // โหลด AI Model และโครงสร้างข้อมูล[cite: 1]
        model = await tmImage.load(modelURL, metadataURL);[cite: 1]
        maxPredictions = model.getTotalClasses();[cite: 1]

        // เซ็ตอัปการทำงานของกล้องเว็บแคม[cite: 1]
        const flip = true;[cite: 1]
        webcam = new tmImage.Webcam(240, 240, flip);[cite: 1]
        await webcam.setup();[cite: 1]
        await webcam.play();[cite: 1]
        
        // รันลูปการแสดงผลของภาพสดบนกล้องแบบต่อเนื่องลื่นไหล[cite: 1]
        window.requestAnimationFrame(loop);[cite: 1]

        // จัดการเปิดการแสดงผล UI ส่วนตัวกล้อง
        document.getElementById("btn-start").style.display = "none";
        document.getElementById("webcam-wrapper").style.display = "block";
        
        // ผูกองค์ประกอบกล้องเข้ากับ HTML DOM[cite: 1]
        document.getElementById("webcam-container").appendChild(webcam.canvas);[cite: 1]
        labelContainer = document.getElementById("label-container");[cite: 1]
        labelContainer.innerHTML = ""; // เคลียร์พื้นที่แสดงผลเดิม

        // สร้างโครงสร้างแถบ Progress Bar สำหรับแต่ละคลาสข้อมูลแบบไดนามิก
        for (let i = 0; i < maxPredictions; i++) {[cite: 1]
            const row = document.createElement("div");
            row.className = "predict-row";
            row.innerHTML = `
                <div class="predict-info">
                    <span class="class-name" id="class-name-${i}">กำลังโหลด...</span>
                    <span class="class-percent" id="class-percent-${i}">0%</span>
                </div>
                <div class="progress-bar-bg">
                    <div class="progress-bar-fill" id="bar-fill-${i}"></div>
                </div>
            `;
            labelContainer.appendChild(row);
        }

        // สั่งทำงานทำนายผลครั้งแรกทันทีที่เปิดระบบ
        await predict();

        // 🎯 ไฮไลท์หลัก: สั่งการให้ AI สแกนวิเคราะห์ภาพทุกๆ 3 วินาที (3000 มิลลิวินาที)
        predictionInterval = setInterval(async () => {
            // ทำเอฟเฟกต์กระพริบสเตตัสสั้นๆ ตอนสแกนเพื่อให้รู้ว่าทำงานทุก 3 วิ
            const badge = document.getElementById("scan-status");
            badge.style.background = "#3b82f6";
            badge.innerText = "กำลังวิเคราะห์...";
            
            await predict();
            
            setTimeout(() => {
                badge.style.background = "#10b981";
                badge.innerText = "สแกนทุกๆ 3 วินาที";
            }, 500);
        }, 3000);
    }

    // ลูปนี้ใช้ควบคุมเพื่อให้ภาพจากกล้องขยับอย่างเป็นธรรมชาติ[cite: 1]
    async function loop() {
        webcam.update(); // ดึงเฟรมภาพปัจจุบันจากกล้อง[cite: 1]
        window.requestAnimationFrame(loop);[cite: 1]
    }

    // ฟังก์ชันส่งภาพไปให้ AI ทำนายผล[cite: 1]
    async function predict() {
        const prediction = await model.predict(webcam.canvas);[cite: 1]
        for (let i = 0; i < maxPredictions; i++) {[cite: 1]
            const name = prediction[i].className;
            const probability = prediction[i].probability;
            const percentage = (probability * 100).toFixed(0); // แปลงเป็นร้อยละจำนวนเต็ม

            // นำผลลัพธ์ไปเรนเดอร์ลงในแถบ UI ที่สร้างไว้
            document.getElementById(`class-name-${i}`).innerText = name;
            document.getElementById(`class-percent-${i}`).innerText = percentage + "%";
            document.getElementById(`bar-fill-${i}`).style.width = percentage + "%";
        }
    }
</script>

</body>
</html>
