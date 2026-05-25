<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Đừng giận anh nữa nhé?</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-select: none;
        }

        body {
            background-color: #fce2ec;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            overflow: hidden;
        }

        /* Khung màn hình dọc chuẩn mobile */
        .phone-container {
            width: 100vw;
            height: 100vh;
            max-width: 450px;
            max-height: 900px;
            background-color: #fce2ec;
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 20px;
            overflow: hidden;
        }

        /* Dòng chữ nhỏ trên cùng */
        .sub-title {
            font-size: 11px;
            color: #555;
            text-align: center;
            position: absolute;
            top: 40px;
            width: 90%;
            font-weight: 500;
        }

        /* Tiêu đề chính */
        .main-title {
            font-size: 26px;
            font-weight: bold;
            color: #dc3545;
            text-align: center;
            margin-bottom: 30px;
            margin-top: -40px;
            line-height: 1.4;
            min-height: 72px;
        }

        /* Khu vực hiển thị nhãn dán */
        .sticker-container {
            width: 220px;
            height: 220px;
            margin-bottom: 40px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .sticker-container img {
            width: 100%;
            height: 100%;
            object-fit: contain;
            border-radius: 15px;
        }

        /* Khung chứa nút bấm */
        .btn-container {
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative;
            height: 180px;
        }

        .btn {
            padding: 12px 0;
            font-size: 18px;
            font-weight: bold;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            outline: none;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }

        #btn-yes {
            background-color: #dc3545;
            color: white;
            width: 160px;
            z-index: 10;
            position: relative;
        }

        #btn-no {
            background-color: #28a745;
            color: white;
            width: 100px;
            font-size: 14px;
            padding: 8px 0;
            position: absolute;
            top: 80px;
            transition: all 0.1s ease-out;
            z-index: 5;
            touch-action: none;
        }
    </style>
</head>
<body>

    <div class="phone-container">
        <div class="sub-title" id="sub-text">Em mà thoát ra hoặc ko chọn là coi như đồng ý nhé =))</div>
        <div class="main-title" id="main-text">Đừng giận anh<br>nữa nhé?</div>

        <div class="sticker-container">
            <img id="sticker-img" src="https://i.postimg.cc/8P0jmn9k/anh-meme-xin-loi-6-2.jpg" alt="sticker">
        </div>

        <div class="btn-container">
            <button class="btn" id="btn-yes">Vâng</button>
            <button class="btn" id="btn-no">Không</button>
        </div>
    </div>

    <script>
        const btnNo = document.getElementById('btn-no');
        const btnYes = document.getElementById('btn-yes');
        const mainText = document.getElementById('main-text');
        const subText = document.getElementById('sub-text');
        const stickerImg = document.getElementById('sticker-img');

        let countNo = 0;

        function xuLyNutNo(e) {
            if (e) {
                e.preventDefault();
                e.stopPropagation();
            }

            countNo++;

            if (countNo === 1) {
                mainText.innerHTML = "xin lỗi mà t bt sai rồi";
                stickerImg.src = "https://i.postimg.cc/pL2TJMXQ/Screenshot-20260525-163429.jpg";
            } else if (countNo === 2) {
                mainText.innerHTML = "không tha lỗi t sẽ buồn đó";
                stickerImg.src = "https://i.postimg.cc/kgtjsz17/Screenshot-20260525-163311.jpg";
            } else if (countNo === 3) {
                mainText.innerHTML = "xloi thật là mà phuong ơiiiiiiii";
                stickerImg.src = "https://i.postimg.cc/htJYwfGk/Screenshot-20260525-163409.jpg";
            } else if (countNo === 4) {
                mainText.innerHTML = "thật sự biết lỗi rồi mà";
                stickerImg.src = "https://i.postimg.cc/h41RbzJs/Screenshot-20260525-163337.jpg";
            }

            if (countNo <= 4) {
                const maxX = 140; 
                const maxY = 60;  
                const xNgauNhien = (Math.random() - 0.5) * maxX * 2;
                const yNgauNhien = (Math.random() - 0.5) * maxY * 2;
                btnNo.style.transform = `translate(${xNgauNhien}px, ${yNgauNhien}px)`;
            } else {
                btnNo.style.transition = "all 0.3s ease-in-out";
                btnNo.style.top = "0px"; 
                btnNo.style.transform = "translate(0px, 0px)"; 
                btnNo.style.width = "160px"; 
                btnNo.style.zIndex = "1"; 
            }
        }

        btnNo.addEventListener('mouseover', xuLyNutNo);
        btnNo.addEventListener('touchstart', xuLyNutNo, {passive: false});
        btnNo.addEventListener('mousedown', xuLyNutNo);

        btnYes.addEventListener('click', () => {
            btnYes.style.display = 'none';
            btnNo.style.display = 'none';
            subText.style.display = 'none';

            // Đảm bảo link ảnh cuối dùng HTTPS sạch để không bị chặn bảo mật
            stickerImg.src = "https://i.postimg.cc/CKT8gQFs/Sticker-muon-om-giup-tro-chuyen-them-am.webp";

            mainText.innerHTML = "iu thí =)))❤️";
            
            setTimeout(() => {
                mainText.innerHTML = "Cho mình xin 1 hẹn đi chơii bạn ê😁😁😁😁😁!";
            }, 2500);
        });
    </script>
</body>
</html>
