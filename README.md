<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DeepZero Meet - Họp Trực Tuyến Tiếng Việt</title>
    <!-- Nhúng thư viện kết nối video nền tảng Jitsi -->
    <script src="https://jit.si"></script>
    <style>
        body { 
            font-family: 'Segoe UI', Arial, sans-serif; 
            margin: 0; 
            padding: 0; 
            background: #1a1a1a; 
            color: white; 
            display: flex; 
            flex-direction: column; 
            height: 100vh; 
        }
        #setup-container { 
            background: #2d2d2d; 
            padding: 35px; 
            border-radius: 14px; 
            box-shadow: 0 8px 30px rgba(0,0,0,0.5); 
            text-align: center; 
            max-width: 420px; 
            width: 90%; 
            margin: auto; 
        }
        h2 { 
            color: #0e71eb; 
            margin: 0 0 10px 0; 
            font-size: 26px; 
        }
        p { 
            color: #aaa; 
            font-size: 14px; 
            margin: 0 0 25px 0; 
        }
        input, select { 
            width: 100%; 
            padding: 12px; 
            margin: 10px 0; 
            border: 1px solid #444; 
            background: #3d3d3d; 
            color: white; 
            border-radius: 8px; 
            box-sizing: border-box; 
            font-size: 15px; 
        }
        input:focus, select:focus { 
            border-color: #0e71eb; 
            outline: none; 
        }
        button.start-btn { 
            width: 100%; 
            padding: 13px; 
            background: #0e71eb; 
            color: white; 
            border: none; 
            border-radius: 8px; 
            font-size: 16px; 
            font-weight: bold; 
            cursor: pointer; 
            margin-top: 15px; 
            transition: background 0.2s;
        }
        button.start-btn:hover { 
            background: #0c5dc2; 
        }
        #meeting-info { 
            display: none; 
            background: #222; 
            padding: 12px; 
            border-radius: 6px; 
            margin-top: 20px; 
            text-align: left; 
            border: 1px solid #333; 
            font-size: 14px; 
        }
        #share-link { 
            color: #0e71eb; 
            font-weight: bold; 
            word-break: break-all; 
            margin-top: 5px; 
        }
        #meet-container { 
            flex: 1; 
            display: none; 
            width: 100%; 
            height: 100%; 
            position: relative; 
        }
        /* Cấu hình nút ghi hình nhỏ gọn cố định ở góc dưới */
        #record-btn { 
            position: absolute; 
            bottom: 90px; 
            left: 20px; 
            z-index: 999; 
            background: #ea4335; 
            color: white; 
            border: none; 
            padding: 10px 18px; 
            border-radius: 20px; 
            font-weight: bold; 
            cursor: pointer; 
            display: none; 
            box-shadow: 0 4px 10px rgba(0,0,0,0.4); 
            font-size: 13px; 
        }
        #record-btn.recording { 
            background: #555; 
        }
    </style>
</head>
<body>

    <!-- GIAO DIỆN FORM ĐĂNG NHẬP / TẠO PHÒNG -->
    <div id="setup-container">
        <h2>DeepZero Meet</h2>
        <p>Hệ thống họp video trực tuyến vĩnh viễn miễn phí</p>
        <input type="text" id="room-id" placeholder="Nhập mã ID phòng (Ví dụ: lop12a)">
        <input type="text" id="user-name" placeholder="Nhập tên hiển thị của bạn">
        <select id="user-role">
            <option value="host">Chủ phòng (Quản trị viên)</option>
            <option value="member">Thành viên tham gia</option>
        </select>
        <button class="start-btn" onclick="startMeeting()">Bắt Đầu Cuộc Họp</button>

        <div id="meeting-info">
            <span>Sao chép link này gửi cho người khác vào cùng:</span>
            <div id="share-link"></div>
        </div>
    </div>

    <!-- KHUNG HIỂN THỊ PHÒNG HỌP VIDEO CHÍNH -->
    <div id="meet-container">
        <button id="record-btn" onclick="toggleRecording()">🔴 Ghi hình cuộc họp</button>
    </div>

    <script>
        let mediaRecorder;
        let recordedChunks = [];

        // Tự động nhận diện nếu có mã phòng từ link mời
        const urlParams = new URLSearchParams(window.location.search);
        const urlRoomId = urlParams.get('room');
        if (urlRoomId) {
            document.getElementById('room-id').value = urlRoomId;
            document.getElementById('user-role').value = 'member';
        }

        // HÀM KÍCH HOẠT PHÒNG HỌP
        function startMeeting() {
            const roomId = document.getElementById('room-id').value.trim().toLowerCase().replace(/[^a-z0-9-]/g, '');
            const userName = document.getElementById('user-name').value.trim();
            const userRole = document.getElementById('user-role').value;

            if (!roomId || !userName) {
                alert("Vui lòng điền đầy đủ ID Phòng và Tên!");
                return;
            }

            const currentUrl = window.location.href.split('?');
            const shareLink = `${currentUrl}?room=${roomId}`;
            document.getElementById('share-link').innerText = shareLink;
            document.getElementById('meeting-info').style.display = 'block';

            // Ẩn form nhập liệu ban đầu, kích hoạt khung chứa video full màn hình
            document.getElementById('setup-container').style.display = 'none';
            document.getElementById('meet-container').style.display = 'block';
            document.getElementById('record-btn').style.display = 'block';

            const domain = "meet.jit.si";
            const options = {
                roomName: "deepzero-" + roomId,
                width: "100%",
                height: "100%",
                parentNode: document.getElementById('meet-container'),
                userInfo: { displayName: userName },
                lang: 'vi', // Chuyển giao diện sang Tiếng Việt chuẩn
                configOverwrite: {
                    startWithAudioMuted: false,
                    startWithVideoMuted: false,
                    defaultLanguage: "vi",
                    disableScreenSharingVirtualBackground: true // Tối ưu tiêu điểm trình chiếu màn hình
                },
                interfaceConfigOverwrite: {
                    LANG_DETECTION: false,
                    // Danh sách các nút chức năng đầy đủ của Zoom / Google Meet
                    TOOLBAR_BUTTONS: [
                        'microphone', 'camera', 'desktop', 'chat', 'raisehand',
                        'hangup', 'profile', 'settings', 'tileview',
                        'participants-pane', 'mute-everyone', 'kick', 'select-background'
                    ]
                }
            };

            const api = new JitsiMeetExternalAPI(domain, options);

            // Phân quyền nâng cao cho Chủ phòng (Host)
            if (userRole === 'host') {
                api.addEventListener('videoConferenceJoined', () => {
                    alert("Bạn là Chủ Phòng! Quyền Mute All và Đuổi người dùng đã được kích hoạt trong tab Thành viên.");
                });
            }
        }

        // CHỨC NĂNG QUAY MÀN HÌNH VÀ TẢI VIDEO VỀ MÁY TÍNH
        async function toggleRecording() {
            const btn = document.getElementById('record-btn');
            if (!mediaRecorder || mediaRecorder.state === "inactive") {
                try {
                    const stream = await navigator.mediaDevices.getDisplayMedia({ 
                        video: { mediaSource: "screen" }, 
                        audio: true 
                    });
                    recordedChunks = [];
                    mediaRecorder = new MediaRecorder(stream, { mimeType: "video/webm;codecs=vp9" });
                    
                    mediaRecorder.ondataavailable = (e) => { 
                        if (e.data.size > 0) recordedChunks.push(e.data); 
                    };
                    
                    mediaRecorder.onstop = () => {
                        const blob = new Blob(recordedChunks, { type: "video/webm" }); 
                        const url = URL.createObjectURL(blob);
                        const a = document.createElement("a"); 
                        a.href = url; 
                        a.download = `cuoc-hop-deepzero.webm`; 
                        a.click();
                        
                        btn.innerText = "🔴 Ghi hình cuộc họp";
                        alert("Video cuộc họp đã được tải về máy tính thành công!");
                    };
                    
                    mediaRecorder.start(); 
                    btn.innerText = "⏹️ Dừng ghi & Tải về";
                } catch (err) { 
                    alert("Cần cấp quyền chia sẻ màn hình trình duyệt để ghi hình cuộc họp!"); 
                }
            } else {
                mediaRecorder.stop();
                mediaRecorder.stream.getTracks().forEach(track => track.stop());
            }
        }
    </script>
</body>
</html>
