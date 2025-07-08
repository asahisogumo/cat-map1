<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NekoMap - 保護猫と新しい家族をつなぐプラットフォーム</title>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700;800&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css" />

    <style>
        :root {
            --font-sans: 'Inter', 'Noto Sans JP', sans-serif;
            --primary-color: #4A90E2;
            --secondary-color: #1A202C;
            --accent-color: #F5A623;
            --danger-color: #E53E3E;
            --success-color: #38A169;
            --background-color: #F7FAFC;
            --surface-color: #FFFFFF;
            --text-color: #2D3748;
            --light-gray: #E2E8F0;
            --shadow-md: 0 10px 15px rgba(0, 0, 0, 0.07);
            --border-radius: 12px;
        }
        @keyframes fadeIn{from{opacity:0;transform:scale(.95)}to{opacity:1;transform:scale(1)}}
        
        *, *::before, *::after { box-sizing: border-box; }
        html { scroll-behavior: smooth; }
        body { font-family: var(--font-sans); margin: 0; background-color: var(--background-color); color: var(--text-color); line-height: 1.7; font-size: 16px; }
        .container { max-width: 1280px; margin: 0 auto; padding: 0 24px; }
        h1, h2, h3, h4 { font-weight: 800; color: var(--secondary-color); }
        
        .header { background-color: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px); padding: 16px 0; position: fixed; width: 100%; top: 0; z-index: 1000; border-bottom: 1px solid var(--light-gray); }
        .header .container { display: flex; justify-content: space-between; align-items: center; }
        .logo { font-size: 28px; font-weight: 800; color: var(--primary-color); text-decoration: none; }
        .header-buttons { display: flex; flex-wrap: wrap; gap: 10px; }
        .button { padding: 10px 20px; border-radius: 8px; text-decoration: none; font-weight: 700; transition: all 0.2s ease-out; border: none; cursor: pointer; font-size: 14px; }
        .button-primary { background: linear-gradient(45deg, var(--primary-color), #3A79C4); color: white; }
        .button-primary:hover:not(:disabled) { transform: translateY(-2px); box-shadow: var(--shadow-md); }
        .button-danger { background-color: var(--danger-color); color: white; }
        .button-danger:hover { background-color: #C53030; }
        .button-accent { background-color: var(--accent-color); color: white; }
        .button-accent:hover { background-color: #d8901e; }
        .button-success { background-color: var(--success-color); color: white; }
        .button-success:hover { background-color: #2F855A; }
        .button-secondary { background-color: var(--surface-color); color: var(--text-color); border: 2px solid var(--light-gray); }
        .button:disabled { background-color: #B0C4DE; color: #718096; cursor: not-allowed; }
        
        .hero { height: 100vh; min-height: 600px; position: relative; display: flex; align-items: center; justify-content: center; text-align: center; color: white; }
        .hero-image-slide { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-size: cover; background-position: center; z-index: -2; opacity: 0; transition: opacity 1.5s ease-in-out; }
        .hero-image-slide.active { opacity: 1; }
        .hero::after { content: ''; position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(0, 0, 0, 0.6); z-index: -1; }
        .hero-content h1 { font-size: clamp(2.5rem, 8vw, 4rem); color: white; }

        .section { padding: 80px 0; }
        .section-title { text-align: center; font-size: 2.5rem; }
        .section-subtitle { text-align: center; font-size: 1.1rem; max-width: 700px; margin: -10px auto 50px; color: #555; }
        
        .filter-buttons-container { display: flex; justify-content: center; flex-wrap: wrap; gap: 10px; margin-bottom: 50px; }
        .filter-btn { background-color: var(--surface-color); color: var(--text-color); border: 1px solid var(--light-gray); font-weight: 500; }
        .filter-btn.active { background-color: var(--primary-color); color: white; border-color: var(--primary-color); }

        .cat-grid { column-count: 3; column-gap: 30px; }
        @media (max-width: 992px) { .cat-grid { column-count: 2; } }
        @media (max-width: 768px) { .cat-grid { column-count: 1; } }
        .cat-card { background: var(--surface-color); border-radius: var(--border-radius); box-shadow: var(--shadow-md); overflow: hidden; margin-bottom: 30px; break-inside: avoid; animation: fadeIn 0.5s; }
        .cat-card-image-wrapper { position: relative; overflow: hidden; }
        .cat-card-image { width: 100%; height: auto; object-fit: cover; display: block; }
        .cat-card-status-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255,255,255,0.8); backdrop-filter: blur(4px); color: var(--secondary-color); display: flex; align-items: center; justify-content: center; text-align: center; font-size: 1.2rem; font-weight: bold; }
        .cat-card-content { padding: 25px; }
        .cat-card-header h3 { margin: 0; font-size: 1.8rem; }
        .cat-stats { display: flex; gap: 20px; margin: 15px 0; padding: 15px 0; border-top: 1px solid var(--light-gray); border-bottom: 1px solid var(--light-gray); }
        .cat-stat { display: flex; flex-direction: column; align-items: center; font-size: 0.9rem; color: #555; text-align: center; flex: 1; }
        .cat-stat i { font-size: 1.5rem; color: var(--primary-color); margin-bottom: 5px; }
        .cat-stat span { font-weight: bold; color: var(--text-color); font-size: 1rem; }
        .cat-tags { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 15px; }
        .cat-tag { background: var(--light-gray); color: var(--text-color); font-size: 0.8rem; padding: 4px 10px; border-radius: 99px; }
        .cat-card-body { font-size: 1rem; color: #4A5568; margin: 15px 0; }
        .cat-card-footer { margin-top: 20px; display: flex; justify-content: space-between; align-items: center; }
        .cat-fee p { margin: 0; font-size: 1.5rem; font-weight: 800; color: var(--primary-color); }
        .cat-fee-details { font-size: 0.85rem; color: #718096; }
        
        #graduates { background-color: #EDF2F7; }
        .adopted-cat-card { text-align: center; }
        .adopted-cat-card .cat-card-image { filter: grayscale(20%); }
        .adopted-cat-card-content { padding: 20px; }
        .adopted-cat-card-content h3 { font-size: 1.5rem; margin: 0 0 10px; }
        .adopted-cat-card-content p { color: var(--success-color); font-weight: bold; }
        .sold-out-banner { position: absolute; top: 25px; left: -50px; width: 200px; text-align: center; padding: 10px 0; background-color: var(--danger-color); color: white; font-weight: 800; font-size: 1.4rem; font-family: 'Noto Sans JP', sans-serif; transform: rotate(-45deg); box-shadow: 0 4px 6px rgba(0,0,0,0.2); letter-spacing: 2px; }

        .modal { display: none; position: fixed; z-index: 2000; left: 0; top: 0; width: 100%; height: 100%; overflow: auto; background-color: rgba(0,0,0,.6); backdrop-filter: blur(5px); align-items: center; justify-content: center; }
        .modal.active { display: flex; }
        .modal-content { background-color: #fff; border-radius: var(--border-radius); box-shadow: 0 20px 50px rgba(0,0,0,.2); max-width: 95%; width: 800px; max-height: 90vh; overflow: hidden; display: flex; flex-direction: column; }
        .modal-body { overflow-y: auto; padding: 20px; }
        .close-button { position: absolute; top: 15px; right: 25px; color: #aaa; font-size: 35px; font-weight: 700; cursor: pointer; line-height: 1; z-index: 10; }
        .form-container, .draw-tool-container { padding: 20px; }
        .form-container h2, .draw-tool-container h2 { text-align: center; margin-top: 0; font-size: 1.8rem; }
        .form-step { display: none; }
        .form-step.active { display: block; }
        .form-group { margin-bottom: 20px; }
        .form-group label { display: block; font-weight: 700; margin-bottom: 8px; }
        .form-group input, .form-group select, .form-group textarea { width: 100%; padding: 12px; border: 1px solid var(--light-gray); border-radius: 8px; box-sizing: border-box; font-family: var(--font-sans); font-size: 1rem; }
        .form-buttons { display: flex; justify-content: space-between; margin-top: 30px; }
        .form-note { font-size: .85rem; color: #777; margin-top: 5px; font-weight: 400; }
        .adoption-intro { text-align: center; margin-bottom: 30px; }
        .adoption-intro img { max-width: 150px; border-radius: 50%; border: 4px solid var(--primary-color); margin-bottom: 15px; }
        .adoption-intro h3 { font-size: 1.8rem; margin: 0; }
        .form-group.radio-group label { display: inline-block; margin-right: 15px; }
        .success-message { text-align: center; padding: 40px; }
        .success-message i { font-size: 4rem; color: var(--success-color); margin-bottom: 20px; }
        .success-message h2 { color: var(--success-color); }
        
        .drop-area { border: 2px dashed var(--light-gray); border-radius: var(--border-radius); padding: 40px; text-align: center; color: #718096; cursor: pointer; transition: border-color 0.3s, background-color 0.3s; position: relative; }
        .drop-area.hover { border-color: var(--primary-color); background-color: #EBF8FF; }
        .drop-area p { margin: 0; pointer-events: none; }
        #image-preview { max-width: 100%; max-height: 200px; margin-top: 15px; border-radius: 8px; }
        .hidden { display: none; }

        #map { height: 45vh; width: 100%; border-radius: var(--border-radius); border: 1px solid var(--light-gray); margin-top: 10px; }
        .cost-display { text-align: center; font-size: 1.5rem; font-weight: bold; color: var(--secondary-color); margin-top: 30px; background-color: var(--background-color); padding: 20px; border-radius: 8px;}
        
        .draw-tool-canvas { background: white; cursor: crosshair; border: 2px solid var(--light-gray); border-radius: 8px; width: 100%; height: auto; }
        .draw-tool-tools { display: flex; flex-wrap: wrap; justify-content: center; align-items: center; gap: 15px; margin-top: 15px; padding: 15px; background: var(--background-color); border-radius: 8px; }
        
        .footer { text-align: center; padding: 40px 0; background-color: #EDF2F7; border-top: 1px solid var(--light-gray); }
        .footer-sns { display: flex; justify-content: center; gap: 30px; margin-bottom: 25px; }
        .sns-icon { font-size: 2.8rem; text-decoration: none; transition: transform 0.3s, filter 0.3s, opacity 0.3s; }
        .sns-icon:hover { transform: scale(1.15) rotate(5deg); }
        .sns-icon.disabled { filter: grayscale(100%); opacity: 0.4; cursor: not-allowed; }
        #sns-link-x .fa-x-twitter { color: #14171A; }
        #sns-link-tiktok .fa-tiktok { color: #010101; text-shadow: -2px -2px 0px #FF0050, 2px 2px 0px #00F2EA; }
        #sns-link-instagram .fa-instagram { background: radial-gradient(circle at 30% 107%, #fdf497 0%, #fdf497 5%, #fd5949 45%,#d6249f 60%,#285AEB 90%); -webkit-background-clip: text; background-clip: text; color: transparent; -moz-background-clip: text; -moz-text-fill-color: transparent; }

        #admin-trigger { cursor: pointer; color: var(--primary-color); text-decoration: underline; font-weight: 700; }
        .admin-panel { display: none; padding: 50px 20px; background-color: var(--secondary-color); color: #fff; }
        .admin-panel h2, .admin-panel h3 { color: #fff; border-color: var(--primary-color); }
        .admin-section { margin-bottom: 40px; }
        .admin-item-list { list-style: none; padding: 0; display: grid; gap: 20px; }
        .admin-item-card { background: rgba(255, 255, 255, .1); padding: 20px; border-radius: 8px; }
        .admin-item-controls { display: flex; gap: 10px; margin-top: 15px; }
        .admin-item-controls button { font-size: .9em; padding: 8px 12px; cursor: pointer; border: none; border-radius: 4px; color: #fff; font-weight: bold; }
        .approve-btn { background-color: var(--accent-color); }
        .reject-btn, .delete-btn { background-color: #c14953; }
        .purge-btn { background-color: #8B0000; }
        
        .fade-in-up { opacity: 0; transform: translateY(30px); transition: opacity 0.6s ease-out, transform 0.6s ease-out; }
        .fade-in-up.is-visible { opacity: 1; transform: translateY(0); }
        
        @media (max-width: 768px) {
            .header .container { flex-direction: column; gap: 15px; }
            .header-buttons { justify-content: center; }
            .section { padding: 60px 15px; }
            .section-title { font-size: 2rem; }
            .cat-grid { column-count: 1; }
            .modal-content { width: 95%; max-height: 85vh; }
            .modal-body { padding: 10px; }
            .draw-tool-tools, .form-buttons { flex-direction: column; gap: 10px; align-items: stretch; }
            .form-buttons .button { width: 100%; }
        }
        @media (max-width: 480px) {
            .cat-stats { flex-direction: column; align-items: flex-start; gap: 15px; }
        }
    </style>
</head>
<body>
    <header class="header">
        <div class="container">
            <a href="#" class="logo">NekoMap</a>
            <div class="header-buttons">
                <button class="button button-primary" id="register-cat-btn">猫の掲載依頼</button>
                <button class="button button-danger" id="rescue-request-btn">🚨 緊急救助依頼</button>
                <button class="button button-accent" id="draw-tool-btn">🎨 イラスト作成</button>
            </div>
        </div>
    </header>

    <main>
        <section class="hero">
            <div class="hero-image-slide active" style="background-image: url('https://images.pexels.com/photos/45170/kittens-cat-cat-puppy-rush-45170.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1');"></div>
            <div class="hero-image-slide" style="background-image: url('https://images.pexels.com/photos/416160/pexels-photo-416160.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1');"></div>
            <div class="hero-content"><h1>未来へつなぐ、<br>小さな命のバトン。</h1></div>
        </section>
        
        <section id="adoption" class="section">
            <div class="container">
                <h2 class="section-title">新しい家族を待つ猫たち</h2>
                <div id="filter-buttons-container" class="filter-buttons-container"></div>
                <div class="cat-grid" id="cat-grid-container"></div>
            </div>
        </section>

        <section id="graduates" class="section">
            <div class="container">
                <h2 class="section-title">幸せになった猫たち</h2>
                <p class="section-subtitle">皆様の温かいご支援のおかげで、新しい家族を見つけた子たちです。</p>
                <div class="cat-grid" id="adopted-cats-grid"></div>
            </div>
        </section>
    </main>

    <footer class="footer">
        <div class="footer-sns">
            <a href="#" id="sns-link-instagram" target="_blank" class="sns-icon" aria-label="Instagram"><i class="fa-brands fa-instagram"></i></a>
            <a href="#" id="sns-link-x" target="_blank" class="sns-icon" aria-label="X"><i class="fa-brands fa-x-twitter"></i></a>
            <a href="#" id="sns-link-tiktok" target="_blank" class="sns-icon" aria-label="TikTok"><i class="fa-brands fa-tiktok"></i></a>
        </div>
        <p>&copy; 2025 NekoMap. All Rights Reserved. <span id="admin-trigger">管理者用</span></p>
    </footer>

    <div id="register-modal" class="modal">
        <div class="modal-content">
            <div class="modal-body">
                <span class="close-button">&times;</span>
                <div id="register-form-wrapper" class="form-container">
                    <form id="register-cat-form">
                        <h2>里親募集 登録依頼</h2>
                        <div class="form-group"><label for="cat-name">猫の名前</label><input type="text" id="cat-name" required></div>
                        <div class="form-group">
                            <label>猫の写真</label>
                            <div id="image-drop-area" class="drop-area">
                                <p>ここに写真をドラッグ＆ドロップ<br>またはクリックしてファイルを選択</p>
                                <input type="file" id="cat-photo-input" class="hidden" accept="image/*">
                                <img id="image-preview" class="hidden" src="" alt="プレビュー">
                            </div>
                        </div>
                        <div class="form-group"><label for="cat-fee">初期費用（円）</label><input type="number" id="cat-fee" required></div>
                        <div class="form-group"><label for="cat-tags">特徴タグ（コンマ区切り）</label><input type="text" id="cat-tags" required></div>
                        <div class="form-group"><label for="cat-desc">猫の紹介文</label><textarea id="cat-desc" rows="3" required></textarea></div>
                        <div class="form-group"><label for="cat-gender">性別</label><input type="text" id="cat-gender" placeholder="例: オス"></div>
                        <div class="form-group"><label for="cat-age">年齢</label><input type="text" id="cat-age" placeholder="例: 1歳"></div>
                        <div class="form-group"><label for="cat-breed">種類</label><input type="text" id="cat-breed" placeholder="例: MIX"></div>
                        <div class="form-buttons"><button type="submit" class="button button-primary" style="width: 100%;">掲載を依頼する</button></div>
                    </form>
                </div>
                <div id="register-success-view" class="success-message" style="display: none;">
                    <i class="fa-solid fa-check-circle"></i><h2>ご依頼ありがとうございます</h2><p>管理者による確認後、サイトに掲載されます。</p>
                </div>
            </div>
        </div>
    </div>
    
    <div id="rescue-modal" class="modal">
        <div class="modal-content">
            <div class="modal-body">
                <span class="close-button">&times;</span>
                <div class="form-container">
                    <form id="rescue-form">
                        <div class="form-step active" data-step="1">
                            <h2>緊急救助依頼 (1/3)</h2>
                            <div class="form-group"><label for="rescue-name">お名前</label><input type="text" id="rescue-name" required></div>
                            <div class="form-group"><label for="rescue-phone">電話番号</label><input type="tel" id="rescue-phone" required></div>
                            <div class="form-group"><label for="rescue-cat-status">猫の状態</label><select id="rescue-cat-status" required><option value="normal">保護可能（¥10,000）</option><option value="injured">ケガ（¥15,000）</option><option value="urgent">緊急（¥30,000）</option></select></div>
                            <div class="form-buttons"><button type="button" class="button button-primary next-btn" style="width:100%">場所の指定へ</button></div>
                        </div>
                        <div class="form-step" data-step="2">
                             <h2>救助場所の指定 (2/3)</h2>
                             <div class="form-group"><label for="location-search">住所または目印で検索</label><input type="text" id="location-search"></div>
                             <button type="button" class="button button-secondary geocode-btn" style="width:100%; margin-bottom:15px;">地図を検索</button>
                             <div id="map"></div>
                            <div class="form-buttons"><button type="button" class="button button-secondary prev-btn">戻る</button><button type="button" class="button button-primary next-btn">最終確認へ</button></div>
                        </div>
                        <div class="form-step" data-step="3">
                            <h2>ご依頼内容の確認 (3/3)</h2>
                            <div id="cost-display" class="cost-display"></div>
                            <div class="form-group" style="margin-top:20px;"><label><input type="checkbox" id="rescue-consent" required> 上記内容と費用を確認し、依頼します。</label></div>
                            <div class="form-buttons"><button type="button" class="button button-secondary prev-btn">戻る</button><button type="submit" class="button button-danger">依頼を確定する</button></div>
                        </div>
                    </form>
                    <div id="rescue-success-view" class="success-message" style="display: none;"><i class="fa-solid fa-truck-medical"></i><h2>ご依頼承りました</h2><p>担当者より折り返しご連絡いたします。</p></div>
                </div>
            </div>
        </div>
    </div>

    <div id="draw-modal" class="modal">
        <div class="modal-content">
            <span class="close-button">&times;</span>
            <div class="modal-body">
                <div class="draw-tool-container">
                    <h2>猫のイラスト作成ツール</h2>
                    <canvas class="draw-tool-canvas" id="canvas"></canvas>
                    <div class="draw-tool-tools">
                        <label for="color">色:</label><input type="color" id="color" value="#000000">
                        <label for="brushSize">太さ:</label><input type="range" id="brushSize" min="1" max="20" value="5">
                        <button id="eraser" class="button button-secondary">消しゴム</button>
                        <button id="clear" class="button button-secondary">全消去</button>
                        <button id="save" class="button button-primary">保存</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="adoption-modal" class="modal">
        <div class="modal-content">
            <div class="modal-body">
                <span class="close-button">&times;</span>
                <div id="adoption-form-container" class="form-container">
                    <form id="adoption-form">
                        <div class="adoption-intro"><img id="adoption-cat-img" src="" alt="猫の写真">
                            <h3 id="adoption-cat-name"></h3><p>への里親申し込み</p>
                        </div>
                        <div class="form-step active" data-step="1">
                            <h4>ステップ1: ご連絡先の入力</h4>
                            <div class="form-group"><label for="applicant-name">氏名</label><input type="text" id="applicant-name" required></div>
                            <div class="form-group"><label for="applicant-email">メールアドレス</label><input type="email" id="applicant-email" required></div>
                            <div class="form-group"><label for="applicant-phone">電話番号</label><input type="tel" id="applicant-phone" required></div>
                            <div class="form-group"><label for="applicant-contact-time">連絡可能な時間帯</label><input type="text" id="applicant-contact-time" placeholder="例: 平日 18:00〜21:00"></div>
                            <div class="form-group"><label for="applicant-line-id">LINE ID (任意)</label><input type="text" id="applicant-line-id"></div>
                            <div class="form-buttons"><button type="button" class="button button-primary next-btn" style="width:100%">アンケートへ進む</button></div>
                        </div>
                        <div class="form-step" data-step="2">
                             <h4>ステップ2: アンケート</h4>
                             <div class="form-group"><label>お住まいは？</label><select id="housing-type" required><option>持ち家</option><option>賃貸</option></select></div>
                             <div class="form-group"><label>ペット飼育は許可されていますか？</label><div class="radio-group"><input type="radio" name="pet-allowed" id="pet-yes" value="はい" checked><label for="pet-yes">はい</label><input type="radio" name="pet-allowed" id="pet-no" value="いいえ"><label for="pet-no">いいえ</label></div></div>
                             <div class="form-buttons"><button type="button" class="button button-secondary prev-btn">戻る</button><button type="button" class="button button-primary next-btn">最終確認へ</button></div>
                        </div>
                        <div class="form-step" data-step="3">
                            <h4>ステップ3: 最終確認</h4>
                            <div class="cost-display" id="adoption-fee-display"></div>
                            <div class="form-group" style="margin-top:20px;"><label><input type="checkbox" id="adoption-consent" required> 規約を読み、同意します。</label></div>
                            <div class="form-buttons"><button type="button" class="button button-secondary prev-btn">戻る</button><button type="submit" class="button button-success">申し込みを確定する</button></div>
                        </div>
                    </form>
                </div>
                <div id="adoption-success-view" class="success-message" style="display: none;"><i class="fa-solid fa-paper-plane"></i><h2>お申し込み完了</h2><p>担当者よりご連絡いたします。</p></div>
            </div>
        </div>
    </div>

    <div class="admin-panel" id="admin-panel">
        <div class="container">
            <h2 class="section-title" style="color:white;">管理者パネル</h2>
            <div class="admin-section"><h3>承認待ち</h3><ul id="pending-cats-list" class="admin-item-list"></ul></div>
            <div class="admin-section"><h3>掲載中</h3><ul id="adoption-cats-list" class="admin-item-list"></ul></div>
            <div class="admin-section"><h3>卒業した猫（履歴）</h3><ul id="graduated-cats-list" class="admin-item-list"></ul></div>
            <div class="admin-section">
                <h3>サイト設定</h3>
                <div class="form-group"><label for="setting-instagram">Instagram URL</label><input type="url" id="setting-instagram" placeholder="https://instagram.com/your-account"></div>
                <div class="form-group"><label for="setting-x">X (Twitter) URL</label><input type="url" id="setting-x" placeholder="https://x.com/your-account"></div>
                <div class="form-group"><label for="setting-tiktok">TikTok URL</label><input type="url" id="setting-tiktok" placeholder="https://tiktok.com/@your-account"></div>
                <button id="save-settings-btn" class="button button-primary">設定を保存</button>
            </div>
        </div>
    </div>
    
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
    document.addEventListener('DOMContentLoaded', function() {
        // --- データ管理 ---
        let adoptionCats = JSON.parse(localStorage.getItem('adoptionCats')) || [];
        let pendingCats = JSON.parse(localStorage.getItem('pendingCats')) || [];
        let adoptionApplications = JSON.parse(localStorage.getItem('adoptionApplications')) || [];
        let adoptedCats = JSON.parse(localStorage.getItem('adoptedCats')) || [];
        let siteSettings = JSON.parse(localStorage.getItem('siteSettings')) || { instagram: '', x: '', tiktok: '' };
        
        function initializeSampleData() {
             if (localStorage.getItem('sampleDataInitialized')) return;
            adoptionCats = [
                { id: 1, name: 'レオ', fee: 20000, img: 'https://i.imgur.com/k5CAN6F.jpeg', tags: ['人懐っこい', '遊び好き'], desc: '元気いっぱいな男の子。', gender: 'オス', age: '1歳', breed: 'MIX', status: 'available' },
                { id: 2, name: 'ルナ', fee: 25000, img: 'https://i.imgur.com/sOFg20X.jpeg', tags: ['おとなしい', '甘えん坊'], desc: '静かで落ち着いた女の子。', gender: 'メス', age: '2歳', breed: 'スコティッシュ', status: 'available' },
                { id: 3, name: 'ソラ', fee: 18000, img: 'https://i.imgur.com/qPKa9d4.jpeg', tags: ['好奇心旺盛'], desc: '新しいものに興味津々。', gender: 'オス', age: '8ヶ月', breed: 'キジトラ', status: 'available' },
            ];
             adoptedCats = [
                { id: 101, name: 'ココ', img: 'https://i.imgur.com/8mBCTtA.jpeg' }
            ];
            saveData();
            localStorage.setItem('sampleDataInitialized', 'true');
        }
        const saveData = () => {
            localStorage.setItem('adoptionCats', JSON.stringify(adoptionCats));
            localStorage.setItem('pendingCats', JSON.stringify(pendingCats));
            localStorage.setItem('adoptionApplications', JSON.stringify(adoptionApplications));
            localStorage.setItem('adoptedCats', JSON.stringify(adoptedCats));
            localStorage.setItem('siteSettings', JSON.stringify(siteSettings));
        };

        // --- 表示関連 ---
        const renderAdoptionCats = (catsToRender) => {
            const container = document.getElementById('cat-grid-container');
            if (!catsToRender || catsToRender.length === 0) { container.innerHTML = "<p style='text-align:center;'>現在、里親を募集している猫はいません。</p>"; return; }
            container.innerHTML = catsToRender.map(cat => {
                const isPending = cat.status === 'pending';
                return `
                <div class="cat-card">
                    <div class="cat-card-image-wrapper"><img src="${cat.img}" alt="${cat.name}" class="cat-card-image">
                        ${isPending ? `<div class="cat-card-status-overlay">新しい家族を<br>待っています</div>` : ''}
                    </div>
                    <div class="cat-card-content">
                        <div class="cat-card-header"><h3>${cat.name}</h3></div>
                        <div class="cat-stats">
                            <div class="cat-stat"><i class="fa-solid fa-venus-mars"></i><span>${cat.gender||'不明'}</span></div>
                            <div class="cat-stat"><i class="fa-solid fa-cake-candles"></i><span>${cat.age||'不明'}</span></div>
                            <div class="cat-stat"><i class="fa-solid fa-paw"></i><span>${cat.breed||'MIX'}</span></div>
                        </div>
                        <p class="cat-card-body">${cat.desc}</p>
                        <div class="cat-tags">${(cat.tags||[]).map(tag => `<span class="cat-tag">${tag}</span>`).join('')}</div>
                        <div class="cat-card-footer">
                            <div class="cat-fee"><p class="cat-fee-details">初期費用</p><p>¥${cat.fee.toLocaleString()}</p></div>
                            <button class="button button-primary adopt-btn" data-cat-id="${cat.id}" ${isPending?'disabled':''}>${isPending?'選考中':'里親になる'}</button>
                        </div>
                    </div>
                </div>`;
            }).join('');
        };
        const renderFilterButtons = () => {
            const container = document.getElementById('filter-buttons-container');
            const allTags = new Set(adoptionCats.flatMap(cat => cat.tags || []));
            container.innerHTML = '<button class="button filter-btn active" data-tag="all">すべて</button>' + [...allTags].map(tag => `<button class="button filter-btn" data-tag="${tag}">${tag}</button>`).join('');
            container.addEventListener('click', e => {
                if (e.target.matches('.filter-btn')) {
                    const selectedTag = e.target.dataset.tag;
                    container.querySelector('.active').classList.remove('active'); e.target.classList.add('active');
                    const filteredCats = (selectedTag === 'all') ? adoptionCats : adoptionCats.filter(cat => (cat.tags || []).includes(selectedTag));
                    renderAdoptionCats(filteredCats);
                }
            });
        };
        const renderAdoptedCats = () => {
            const container = document.getElementById('adopted-cats-grid');
            if (adoptedCats.length === 0) { container.innerHTML = "<p style='text-align:center;'>まだ新しい家族が見つかった猫はいません。</p>"; return; }
            container.innerHTML = adoptedCats.map(cat => `
                <div class="cat-card adopted-cat-card">
                    <div class="cat-card-image-wrapper">
                        <img src="${cat.img}" alt="${cat.name}" class="cat-card-image">
                        <div class="sold-out-banner">家族決定</div>
                    </div>
                    <div class="adopted-cat-card-content">
                        <h3>${cat.name}</h3><p><i class="fa-solid fa-house-chimney-heart"></i> 新しい家族が見つかりました！</p>
                    </div>
                </div>`).join('');
        };
        const updateSnsLinks = () => {
            const links = { instagram: document.getElementById('sns-link-instagram'), x: document.getElementById('sns-link-x'), tiktok: document.getElementById('sns-link-tiktok') };
            for (const key in links) {
                const linkElement = links[key];
                const url = siteSettings[key] ? siteSettings[key].trim() : '';
                if (url) {
                    linkElement.href = url;
                    linkElement.classList.remove('disabled');
                    linkElement.setAttribute('aria-label', `${key}のプロフィールへ`);
                    linkElement.onclick = null;
                } else {
                    linkElement.href = '#';
                    linkElement.classList.add('disabled');
                    linkElement.setAttribute('aria-label', `${key} (未設定)`);
                    linkElement.onclick = (e) => e.preventDefault();
                }
            }
        };
        
        const animatedElements = document.querySelectorAll('.fade-in-up');
        const scrollObserver = new IntersectionObserver((entries) => {
            entries.forEach(entry => { if (entry.isIntersecting) { entry.target.classList.add('is-visible'); scrollObserver.unobserve(entry.target); } });
        }, { threshold: 0.1 });
        animatedElements.forEach(el => scrollObserver.observe(el));
        
        const imageSlides = document.querySelectorAll('.hero-image-slide');
        if (imageSlides.length > 1) { let currentImageIndex = 0; setInterval(() => { imageSlides[currentImageIndex].classList.remove('active'); currentImageIndex = (currentImageIndex + 1) % imageSlides.length; imageSlides[currentImageIndex].classList.add('active'); }, 7000); }

        // --- モーダル制御 ---
        const allModals = document.querySelectorAll('.modal');
        const openModal = (modal) => modal.classList.add('active');
        const closeModal = (modal) => modal.classList.remove('active');
        allModals.forEach(modal => modal.querySelector('.close-button')?.addEventListener('click', () => closeModal(modal)));
        document.getElementById('register-cat-btn').addEventListener('click', () => openModal(document.getElementById('register-modal')));
        document.getElementById('rescue-request-btn').addEventListener('click', () => openModal(document.getElementById('rescue-modal')));
        document.getElementById('draw-tool-btn').addEventListener('click', () => { openModal(document.getElementById('draw-modal')); setTimeout(setupCanvas, 50); });

        // --- 里親申し込みプロセス ---
        const adoptionModal = document.getElementById('adoption-modal');
        document.getElementById('cat-grid-container').addEventListener('click', e => { if (e.target.matches('.adopt-btn')) { const cat = adoptionCats.find(c => c.id === parseInt(e.target.dataset.catId)); if (cat) startAdoptionProcess(cat); } });
        let adoptionCurrentStep = 1; let applyingCatId = null;
        function startAdoptionProcess(cat) { applyingCatId = cat.id; adoptionModal.querySelector('#adoption-cat-img').src = cat.img; adoptionModal.querySelector('#adoption-cat-name').textContent = cat.name; adoptionModal.querySelector('#adoption-fee-display').innerHTML = `<strong>初期費用:</strong> ¥${cat.fee.toLocaleString()}`; openModal(adoptionModal); }
        const updateAdoptionStep = () => { adoptionModal.querySelectorAll('.form-step').forEach(s => s.classList.remove('active')); adoptionModal.querySelector(`.form-step[data-step="${adoptionCurrentStep}"]`).classList.add('active'); };
        adoptionModal.addEventListener('click', e => { if (e.target.matches('.next-btn')) { if (adoptionCurrentStep === 1 && (!adoptionModal.querySelector('#applicant-name').value || !adoptionModal.querySelector('#applicant-email').value || !adoptionModal.querySelector('#applicant-phone').value)) { alert('氏名、メールアドレス、電話番号は必須です。'); return; } adoptionCurrentStep++; updateAdoptionStep(); } if (e.target.matches('.prev-btn')) { adoptionCurrentStep--; updateAdoptionStep(); } });
        document.getElementById('adoption-form').addEventListener('submit', e => { e.preventDefault(); if(!adoptionModal.querySelector('#adoption-consent').checked) { alert('規約への同意が必要です。'); return; } adoptionApplications.push({ catId: applyingCatId, applicantName: adoptionModal.querySelector('#applicant-name').value, email: adoptionModal.querySelector('#applicant-email').value, phone: adoptionModal.querySelector('#applicant-phone').value, contactTime: adoptionModal.querySelector('#applicant-contact-time').value, lineId: adoptionModal.querySelector('#applicant-line-id').value, housing: adoptionModal.querySelector('#housing-type').value, petAllowed: adoptionModal.querySelector('input[name="pet-allowed"]:checked').value, submittedAt: new Date().toISOString() }); const catIndex = adoptionCats.findIndex(c => c.id === applyingCatId); if (catIndex !== -1) adoptionCats[catIndex].status = 'pending'; saveData(); adoptionModal.querySelector('#adoption-form-container').style.display = 'none'; adoptionModal.querySelector('#adoption-success-view').style.display = 'block'; setTimeout(() => { closeModal(adoptionModal); setTimeout(() => { document.getElementById('adoption-form').reset(); adoptionCurrentStep = 1; updateAdoptionStep(); adoptionModal.querySelector('#adoption-form-container').style.display = 'block'; adoptionModal.querySelector('#adoption-success-view').style.display = 'none'; renderEverything(); }, 500); }, 3000); });
        
        // --- 猫の掲載依頼フォーム（画像アップロード付き） ---
        const registerModal = document.getElementById('register-modal');
        const dropArea = registerModal.querySelector('#image-drop-area');
        const fileInput = registerModal.querySelector('#cat-photo-input');
        const previewImage = registerModal.querySelector('#image-preview');
        const dropAreaText = dropArea.querySelector('p');
        let imageBase64Data = '';
        dropArea.addEventListener('click', () => fileInput.click());
        fileInput.addEventListener('change', (e) => { if (e.target.files[0]) handleFile(e.target.files[0]); });
        ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => dropArea.addEventListener(eventName, (e) => { e.preventDefault(); e.stopPropagation(); }, false));
        ['dragenter', 'dragover'].forEach(eventName => dropArea.addEventListener(eventName, () => dropArea.classList.add('hover'), false));
        ['dragleave', 'drop'].forEach(eventName => dropArea.addEventListener(eventName, () => dropArea.classList.remove('hover'), false));
        dropArea.addEventListener('drop', (e) => { if (e.dataTransfer.files[0]) handleFile(e.dataTransfer.files[0]); });
        function handleFile(file) { if (!file.type.startsWith('image/')) { alert('画像ファイルを選択してください。'); return; } const reader = new FileReader(); reader.onload = (e) => { previewImage.src = e.target.result; previewImage.classList.remove('hidden'); dropAreaText.classList.add('hidden'); imageBase64Data = e.target.result; }; reader.readAsDataURL(file); }
        function resetImagePreview() { fileInput.value = ''; imageBase64Data = ''; previewImage.src = ''; previewImage.classList.add('hidden'); dropAreaText.classList.remove('hidden'); }
        registerModal.querySelector('#register-cat-form').addEventListener('submit', e => { e.preventDefault(); if (!imageBase64Data) { alert('猫の写真をアップロードしてください。'); return; } pendingCats.push({ id: Date.now(), name: registerModal.querySelector('#cat-name').value, img: imageBase64Data, fee: parseInt(registerModal.querySelector('#cat-fee').value), tags: registerModal.querySelector('#cat-tags').value.split(',').map(t => t.trim()).filter(t => t), desc: registerModal.querySelector('#cat-desc').value, gender: registerModal.querySelector('#cat-gender').value, age: registerModal.querySelector('#cat-age').value, breed: registerModal.querySelector('#cat-breed').value, status: 'available' }); saveData(); registerModal.querySelector('#register-form-wrapper').style.display = 'none'; registerModal.querySelector('#register-success-view').style.display = 'block'; setTimeout(() => { closeModal(registerModal); setTimeout(() => { registerModal.querySelector('#register-cat-form').reset(); resetImagePreview(); registerModal.querySelector('#register-form-wrapper').style.display = 'block'; registerModal.querySelector('#register-success-view').style.display = 'none'; }, 500); }, 3000); });
        
        // --- 緊急救助依頼モーダル ---
        const rescueModal = document.getElementById('rescue-modal');
        let rescueCurrentStep = 1; let map, rescueMarker, mapInitialized = false;
        const updateRescueStep = () => { rescueModal.querySelectorAll('.form-step').forEach(s => s.classList.remove('active')); rescueModal.querySelector(`.form-step[data-step="${rescueCurrentStep}"]`).classList.add('active'); if (rescueCurrentStep === 2 && !mapInitialized) { initializeMap(); mapInitialized = true; } if (rescueCurrentStep === 3) calculateCost(); };
        rescueModal.addEventListener('click', e => { if (e.target.matches('.next-btn')) { if (rescueCurrentStep === 1 && (!rescueModal.querySelector('#rescue-name').value || !rescueModal.querySelector('#rescue-phone').value)) { alert('お名前と電話番号は必須です。'); return; } rescueCurrentStep++; updateRescueStep(); } if (e.target.matches('.prev-btn')) { rescueCurrentStep--; updateRescueStep(); } });
        function initializeMap() { map = L.map('map').setView([39.72, 140.1], 10); L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '&copy; OpenStreetMap' }).addTo(map); map.on('click', (e) => { if (rescueMarker) map.removeLayer(rescueMarker); rescueMarker = L.marker(e.latlng).addTo(map); }); setTimeout(() => map.invalidateSize(), 100); }
        rescueModal.querySelector('.geocode-btn').addEventListener('click', () => { fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(rescueModal.querySelector('#location-search').value)}`).then(res => res.json()).then(data => { if (data.length > 0) { const { lat, lon } = data[0]; map.setView([lat, lon], 15); if (rescueMarker) map.removeLayer(rescueMarker); rescueMarker = L.marker([lat, lon]).addTo(map); } else { alert('場所が見つかりません。'); } }); });
        const calculateCost = () => { const status = rescueModal.querySelector('#rescue-cat-status').value; let cost = (status === "injured") ? 15000 : (status === "urgent") ? 30000 : 10000; rescueModal.querySelector('#cost-display').innerHTML = `<strong>ご請求金額：</strong> ¥${cost.toLocaleString()}`; };
        rescueModal.querySelector('#rescue-form').addEventListener('submit', e => { e.preventDefault(); if(!rescueModal.querySelector('#rescue-consent').checked){ alert('同意チェックが必要です。'); return; } alert("Stripe連携処理に進みます（シミュレーション）。"); rescueModal.querySelector('#rescue-form').style.display = 'none'; rescueModal.querySelector('#rescue-success-view').style.display = 'block'; setTimeout(() => { closeModal(rescueModal); setTimeout(() => { rescueModal.querySelector('#rescue-form').reset(); rescueCurrentStep = 1; updateRescueStep(); rescueModal.querySelector('#rescue-form').style.display = 'block'; rescueModal.querySelector('#rescue-success-view').style.display = 'none'; }, 500); }, 3000); });

        // --- イラスト作成ツール ---
        const drawModal = document.getElementById('draw-modal');
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        let isDrawing = false, isErasing = false;
        const setupCanvas = () => { const dpr = window.devicePixelRatio || 1; const rect = canvas.getBoundingClientRect(); canvas.width = rect.width * dpr; canvas.height = rect.height * dpr; ctx.scale(dpr, dpr); ctx.fillStyle = '#ffffff'; ctx.fillRect(0, 0, canvas.width, canvas.height); };
        const getMousePos = (e) => { const rect = canvas.getBoundingClientRect(); return { x: e.clientX - rect.left, y: e.clientY - rect.top }; };
        const draw = (e) => { if (!isDrawing) return; e.preventDefault(); const pos = getMousePos(e.touches ? e.touches[0] : e); ctx.lineWidth = drawModal.querySelector('#brushSize').value; ctx.lineCap = 'round'; ctx.strokeStyle = isErasing ? '#ffffff' : drawModal.querySelector('#color').value; ctx.lineTo(pos.x, pos.y); ctx.stroke(); ctx.beginPath(); ctx.moveTo(pos.x, pos.y); };
        const startDrawing = (e) => { isDrawing = true; draw(e); }; const stopDrawing = () => { isDrawing = false; ctx.beginPath(); };
        canvas.addEventListener('mousedown', startDrawing); canvas.addEventListener('mouseup', stopDrawing); canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('touchstart', startDrawing, {passive: false}); canvas.addEventListener('touchend', stopDrawing); canvas.addEventListener('touchmove', draw, {passive: false});
        drawModal.querySelector('#eraser').addEventListener('click', () => { isErasing = !isErasing; const btn = drawModal.querySelector('#eraser'); btn.textContent = isErasing ? 'ペン' : '消しゴム'; btn.classList.toggle('button-danger', isErasing); });
        drawModal.querySelector('#clear').addEventListener('click', () => { ctx.fillStyle = '#ffffff'; ctx.fillRect(0, 0, canvas.width, canvas.height); });
        drawModal.querySelector('#save').addEventListener('click', () => { const link = document.createElement('a'); link.download = 'nekodraw.png'; link.href = canvas.toDataURL('image/png'); link.click(); });

        // --- 管理者パネル ---
        const adminPanel = document.getElementById('admin-panel');
        document.getElementById('admin-trigger').addEventListener('click', () => { if (sessionStorage.getItem('nekopass') !== 'ok') { if (prompt('パスワード:') !== 'nekoneko') return; sessionStorage.setItem('nekopass', 'ok'); } adminPanel.style.display = 'block'; renderAdminLists(); document.getElementById('setting-instagram').value = siteSettings.instagram; document.getElementById('setting-x').value = siteSettings.x; document.getElementById('setting-tiktok').value = siteSettings.tiktok; window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' }); });
        const renderAdminLists = () => {
            const pendingList = adminPanel.querySelector('#pending-cats-list');
            const adoptionList = adminPanel.querySelector('#adoption-cats-list');
            const graduatedList = adminPanel.querySelector('#graduated-cats-list');
            pendingList.innerHTML = pendingCats.map((cat, i) => `<li class="admin-item-card"><p>${cat.name}</p><div class="admin-item-controls"><button class="approve-btn" data-index="${i}">承認</button><button class="reject-btn" data-index="${i}">却下</button></div></li>`).join('') || '<li>承認待ちなし</li>';
            adoptionList.innerHTML = adoptionCats.map((cat) => `<li class="admin-item-card"><span>${cat.name}</span><div class="admin-item-controls"><button class="delete-btn" data-id="${cat.id}">掲載終了(保護決定)</button></div></li>`).join('') || '<li>掲載中なし</li>';
            graduatedList.innerHTML = adoptedCats.map((cat) => `<li class="admin-item-card"><span>${cat.name}</span><div class="admin-item-controls"><button class="purge-btn" data-id="${cat.id}">完全に削除</button></div></li>`).join('') || '<li>卒業した猫はいません。</li>';
        };
        adminPanel.addEventListener('click', e => {
            let changed = false;
            if (e.target.matches('.approve-btn')) { const [approvedCat] = pendingCats.splice(e.target.dataset.index, 1); adoptionCats.push(approvedCat); changed = true; } 
            else if (e.target.matches('.reject-btn')) { pendingCats.splice(e.target.dataset.index, 1); changed = true; } 
            else if (e.target.matches('.delete-btn')) { const catId = parseInt(e.target.dataset.id); const catIndex = adoptionCats.findIndex(c => c.id === catId); if (catIndex > -1) { const [graduatedCat] = adoptionCats.splice(catIndex, 1); adoptedCats.push(graduatedCat); changed = true; } }
            else if (e.target.matches('.purge-btn')) { if(confirm('この履歴を完全に削除しますか？')){ const catIdToPurge = parseInt(e.target.dataset.id); adoptedCats = adoptedCats.filter(c => c.id !== catIdToPurge); changed = true;} }
            if (changed) { saveData(); renderEverything(); }
        });
        document.getElementById('save-settings-btn').addEventListener('click', () => { siteSettings.instagram = document.getElementById('setting-instagram').value.trim(); siteSettings.x = document.getElementById('setting-x').value.trim(); siteSettings.tiktok = document.getElementById('setting-tiktok').value.trim(); saveData(); updateSnsLinks(); alert('サイト設定を保存しました。'); });
        
        // --- 汎用レンダリング ---
        const renderEverything = () => { renderAdoptionCats(adoptionCats); renderFilterButtons(); renderAdminLists(); renderAdoptedCats(); updateSnsLinks(); };

        // --- 初期表示 ---
        initializeSampleData();
        renderEverything();
    });
    </script>
</body>
</html>
