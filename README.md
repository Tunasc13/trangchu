# luckywheeloftuna
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vòng Xoay May Mắn</title>
    <style>
        /* CSS cho Vòng Xoay */
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f0f0;
            margin: 0;
            overflow: hidden; /* Ngăn cuộn khi quay */
        }

        h1 {
            color: #333;
            margin-bottom: 30px;
        }

        .spinner-container {
            position: relative;
            width: 300px; /* Kích thước vòng quay */
            height: 300px;
            margin-bottom: 50px;
        }

        .spinner {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            border: 10px solid #ccc; /* Viền vòng quay */
            display: flex;
            justify-content: center;
            align-items: center;
            transition: transform 4s cubic-bezier(0.25, 0.1, 0.25, 1); /* Hiệu ứng quay */
            box-shadow: 0 0 20px rgba(0,0,0,0.2);
            position: relative;
            overflow: hidden; /* Che đi phần thừa của các miếng */
        }

        .spinner-slice {
            position: absolute;
            top: 0;
            left: 0;
            width: 50%;
            height: 50%;
            transform-origin: 100% 100%; /* Tâm xoay là góc dưới bên phải */
            clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%); /* Để tạo hình tam giác nếu cần, ở đây dùng background */
            display: flex;
            justify-content: center;
            align-items: flex-start; /* Để chữ ở gần tâm */
            padding-top: 20px;
            box-sizing: border-box;
            color: white;
            font-weight: bold;
            font-size: 1.2em;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
            line-height: 1; /* Để chữ không bị chiếm quá nhiều diện tích */
        }
        .spinner-slice span {
            /* Để chữ nằm ngang và không bị xoay theo miếng */
            transform: rotate(0deg); 
            position: absolute; /* Giữ vị trí tương đối */
            bottom: 60%; /* Đẩy chữ ra xa tâm một chút */
            left: 50%;
            transform: translateX(-50%) rotate(0deg); /* Giữ chữ ở giữa và không xoay */
            text-align: center;
            white-space: nowrap; /* Không cho xuống dòng */
        }


        .pointer {
            position: absolute;
            top: -25px; /* Đẩy mũi tên lên trên vòng quay */
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 15px solid transparent;
            border-right: 15px solid transparent;
            border-bottom: 25px solid #e74c3c; /* Màu mũi tên */
            z-index: 10; /* Đảm bảo mũi tên nằm trên vòng quay */
            filter: drop-shadow(0 2px 2px rgba(0,0,0,0.3));
        }

        button {
            padding: 12px 30px;
            font-size: 1.2em;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: background-color 0.3s ease;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        button:hover {
            background-color: #2980b9;
        }

        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }

        #result {
            margin-top: 30px;
            font-size: 1.8em;
            font-weight: bold;
            color: #27ae60;
            text-align: center;
            min-height: 40px; /* Giữ chỗ cho kết quả */
        }
    </style>
</head>
<body>
    <h1>Vòng Xoay May Mắn</h1>

    <div class="spinner-container">
        <div class="spinner" id="spinner">
            </div>
        <div class="pointer"></div>
    </div>

    <button id="spinButton">Quay Ngay!</button>

    <div id="result"></div>

    <script>
        // JavaScript cho Vòng Xoay
        const spinner = document.getElementById('spinner');
        const spinButton = document.getElementById('spinButton');
        const resultDiv = document.getElementById('result');

        // Các lựa chọn trên vòng xoay (màu sắc)
        const choices = [
            { name: "Đỏ", color: "#e74c3c" },
            { name: "Xanh Lá", color: "#27ae60" },
            { name: "Xanh Dương", color: "#3498db" },
            { name: "Vàng", color: "#f1c40f" },
            { name: "Tím", color: "#9b59b6" },
            { name: "Cam", color: "#e67e22" },
            { name: "Hồng", color: "#e91e63" },
            { name: "Đen", color: "#34495e" }
        ];

        const numChoices = choices.length;
        const sliceAngle = 360 / numChoices; // Góc của mỗi miếng

        // --- Tạo các miếng cho vòng xoay ---
        function createSpinnerSlices() {
            spinner.innerHTML = ''; // Xóa các miếng cũ (nếu có)
            choices.forEach((choice, index) => {
                const slice = document.createElement('div');
                slice.classList.add('spinner-slice');
                slice.style.backgroundColor = choice.color;
                // Xoay và dịch chuyển miếng để tạo thành hình tròn
                slice.style.transform = `rotate(${index * sliceAngle}deg) skewY(-${90 - sliceAngle}deg)`;
                slice.style.backgroundColor = choice.color; // Đặt màu nền

                // Tạo một div bên trong cho văn bản để nó không bị méo theo slice
                const textSpan = document.createElement('span');
                textSpan.textContent = choice.name;
                
                // Điều chỉnh vị trí của chữ để nó không bị xoay cùng với miếng
                // và nằm ở giữa miếng đó.
                const textRotation = (index * sliceAngle + sliceAngle / 2); // Góc chính giữa miếng
                const textTranslateX = -50; // Giữa theo trục X
                
                // Đặt vị trí ban đầu của miếng và sau đó xoay để tạo hiệu ứng quạt
                slice.style.clipPath = 'polygon(0% 0%, 100% 0%, 50% 100%)'; // Hình tam giác
                slice.style.backgroundColor = choice.color;
                slice.style.transform = `rotate(${index * sliceAngle}deg) skewY(-${90 - sliceAngle}deg)`; // Kỹ thuật tạo miếng tròn
                slice.style.transformOrigin = '0% 100%'; // Tâm xoay ở giữa dưới

                // Tạo miếng bằng cách sử dụng nền gradient để làm cho nó trông như một phần của hình tròn
                slice.style.background = `linear-gradient(to right top, transparent 50%, ${choice.color} 50%)`;
                slice.style.transform = `rotate(${index * sliceAngle}deg)`;
                slice.style.transformOrigin = '100% 100%'; // Vẫn giữ tâm xoay

                // Để tạo hình tròn chính xác hơn, mỗi miếng sẽ là một nửa hình tròn được xoay và đặt màu
                // Tùy chỉnh này phức tạp hơn với clip-path, chúng ta sẽ dùng cách đơn giản hơn là các div được xoay

                // Cách vẽ các miếng tròn bằng cách xoay các div hình tam giác
                // Đây là một kỹ thuật CSS phức tạp hơn để tạo các lát bánh pizza.
                // Một cách đơn giản hơn là dùng một hình ảnh nền hoặc SVG.
                // Đối với ví dụ này, chúng ta sẽ tạo các div và xoay chúng.
                const segment = document.createElement('div');
                segment.style.position = 'absolute';
                segment.style.width = '50%';
                segment.style.height = '50%';
                segment.style.backgroundColor = choice.color;
                segment.style.transformOrigin = '100% 100%';
                segment.style.transform = `rotate(${index * sliceAngle}deg) skewY(${90 - sliceAngle}deg)`;
                segment.style.overflow = 'hidden';

                const textDiv = document.createElement('div');
                textDiv.textContent = choice.name;
                textDiv.style.position = 'absolute';
                textDiv.style.top = '10px';
                textDiv.style.left = '50%';
                textDiv.style.transform = `translateX(-50%) rotate(-${index * sliceAngle + sliceAngle / 2}deg)`; // Đảm bảo chữ thẳng
                textDiv.style.color = 'white';
                textDiv.style.fontWeight = 'bold';
                textDiv.style.fontSize = '1.1em';
                textDiv.style.textAlign = 'center';
                textDiv.style.width = '100%'; // Để textDiv có đủ không gian
                textDiv.style.transformOrigin = 'center center'; // Tâm xoay của textDiv

                segment.appendChild(textDiv);
                spinner.appendChild(segment);
            });
             // Kỹ thuật này không hoạt động trực tiếp với nhiều slice mà không chồng lấn.
             // Để làm điều này chính xác, cần một SVG hoặc vẽ bằng Canvas.
             // Với HTML/CSS thuần, chúng ta sẽ làm một cách đơn giản hơn,
             // tạo các div và tô màu cho chúng như một hình tròn ảo.

            // Một cách tạo các miếng tròn bằng cách đặt background-image (CSS conial-gradient)
            // hoặc dùng JS để tạo từng phần.
            // Để đơn giản, tôi sẽ làm lại phần tạo slices bằng cách dùng CSS Conic-Gradient
            // và thêm overlay để hiển thị chữ.

            let conicGradientString = 'conic-gradient(';
            let currentAngle = 0;
            choices.forEach((choice, index) => {
                conicGradientString += `${choice.color} ${currentAngle}deg, ${choice.color} ${currentAngle + sliceAngle}deg`;
                if (index < numChoices - 1) {
                    conicGradientString += ', ';
                }
                currentAngle += sliceAngle;
            });
            conicGradientString += ')';
            spinner.style.background = conicGradientString;

            // Thêm các lớp overlay cho chữ
            choices.forEach((choice, index) => {
                const textOverlay = document.createElement('div');
                textOverlay.classList.add('slice-text-overlay');
                textOverlay.textContent = choice.name;
                
                // Tính toán vị trí và góc quay của chữ
                const midAngle = index * sliceAngle + sliceAngle / 2; // Góc giữa của miếng
                const radius = 100; // Khoảng cách từ tâm đến chữ
                
                textOverlay.style.position = 'absolute';
                textOverlay.style.top = '50%';
                textOverlay.style.left = '50%';
                textOverlay.style.transform = `translate(-50%, -50%) rotate(${midAngle}deg) translate(0, -${radius}px) rotate(-${midAngle}deg)`;
                textOverlay.style.color = 'white';
                textOverlay.style.fontWeight = 'bold';
                textOverlay.style.fontSize = '1.2em';
                textOverlay.style.textAlign = 'center';
                textOverlay.style.whiteSpace = 'nowrap';
                textOverlay.style.textShadow = '1px 1px 2px rgba(0,0,0,0.5)';
                textOverlay.style.pointerEvents = 'none'; // Không cản trở click
                spinner.appendChild(textOverlay);
            });
        }
        createSpinnerSlices(); // Gọi hàm tạo slices ban đầu

        // --- Logic quay ---
        spinButton.addEventListener('click', () => {
            spinButton.disabled = true; // Vô hiệu hóa nút khi đang quay
            resultDiv.textContent = ''; // Xóa kết quả cũ

            // Tính toán góc quay ngẫu nhiên
            // Quay ít nhất vài vòng để tạo hiệu ứng
            const minRevolutions = 5;
            const maxRevolutions = 8;
            const extraAngle = Math.random() * 360; // Góc quay thêm ngẫu nhiên
            
            // Tổng số độ quay
            const totalDegrees = (minRevolutions + Math.random() * (maxRevolutions - minRevolutions)) * 360 + extraAngle;

            spinner.style.transition = 'transform 4s cubic-bezier(0.25, 0.1, 0.25, 1)'; // Đảm bảo transition
            spinner.style.transform = `rotate(${totalDegrees}deg)`;

            // Xác định phần tử đã chọn sau khi quay xong
            setTimeout(() => {
                // Đảm bảo transition đã hoàn thành trước khi tính toán
                spinner.style.transition = 'none'; // Tắt transition để đặt lại góc
                
                // Lấy góc cuối cùng của vòng quay (modulo 360)
                // Lấy phần dư dương của góc quay để biết vị trí cuối cùng
                let finalAngle = totalDegrees % 360;
                finalAngle = (finalAngle + 360) % 360; // Đảm bảo là số dương từ 0-359

                // Do mũi tên ở phía trên (0 độ), chúng ta cần điều chỉnh góc
                // để tính từ phía mũi tên. Vòng xoay quay theo chiều kim đồng hồ.
                // Nếu mũi tên chỉ vào 0 độ, thì phần tử ở 0 độ sẽ là kết quả.
                // Tuy nhiên, các phần tử của chúng ta được sắp xếp từ 0 độ.
                // Điều chỉnh để mũi tên khớp với vị trí 0 độ.
                const pointerAngleOffset = 90; // Nếu mũi tên chỉ lên, 0 độ là bên phải.
                                                // Cần quay 90 độ để phần tử ở 0 độ nằm dưới mũi tên.

                // Chúng ta cần tính toán xem góc cuối cùng rơi vào miếng nào.
                // Các miếng được đánh số từ 0 đến N-1, bắt đầu từ 0 độ (hướng bên phải).
                // Mũi tên chỉ lên trên, tương đương với 270 độ (hoặc -90 độ)
                // Vì vậy, chúng ta cần "xoay" ngược lại 90 độ để 0 độ của vòng quay tương ứng với mũi tên.
                
                let winningAngle = (finalAngle + 270) % 360; // Điều chỉnh để 0 độ của miếng đầu tiên thẳng với mũi tên

                let selectedIndex = -1;
                for (let i = 0; i < numChoices; i++) {
                    const startAngle = i * sliceAngle;
                    const endAngle = startAngle + sliceAngle;
                    
                    if (winningAngle >= startAngle && winningAngle < endAngle) {
                        selectedIndex = i;
                        break;
                    }
                }
                
                // Nếu không tìm thấy, có thể do lỗi làm tròn, chọn miếng cuối cùng hoặc đầu tiên
                if (selectedIndex === -1) {
                    selectedIndex = numChoices - 1; 
                }

                const selectedChoice = choices[selectedIndex];
                resultDiv.textContent = `Bạn đã chọn: ${selectedChoice.name}!`;
                resultDiv.style.color = selectedChoice.color;

                spinButton.disabled = false; // Kích hoạt lại nút
            }, 4000); // Thời gian chờ bằng với thời gian transition
        });
    </script>
</body>
</html>
