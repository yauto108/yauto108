<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>オンラインファイル変換</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f9;
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #4CAF50;
            color: white;
            padding: 15px 0;
            text-align: center;
        }
        .container {
            width: 80%;
            margin: 20px auto;
            text-align: center;
        }
        .upload-section {
            background-color: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .upload-section input[type="file"] {
            padding: 10px;
            margin-bottom: 20px;
        }
        .upload-section select,
        .upload-section button {
            padding: 10px;
            margin-top: 10px;
        }
        .upload-section button {
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            font-size: 16px;
        }
        .upload-section button:hover {
            background-color: #45a049;
        }
        .result {
            margin-top: 20px;
            display: none;
        }
        .result a {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            text-decoration: none;
            font-size: 18px;
            border-radius: 5px;
        }
        .result a:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>

    <header>
        <h1>オンラインファイル変換サービス</h1>
    </header>

    <div class="container">
        <div class="upload-section">
            <h2>ファイルをアップロード</h2>
            <input type="file" id="fileInput" accept="*/*">
            <br>
            <h3>変換後の形式を選択</h3>
            <select id="outputFormat">
                <option value="pdf">PDF</option>
                <option value="docx">Word (.docx)</option>
                <option value="txt">テキスト (.txt)</option>
                <option value="jpg">JPEG (.jpg)</option>
                <option value="png">PNG (.png)</option>
                <!-- 他のフォーマットを追加できます -->
            </select>
            <br><br>
            <button onclick="convertFile()">変換を開始</button>
        </div>

        <div class="result" id="resultSection">
            <h3>変換が完了しました！</h3>
            <a id="downloadLink" href="#">ダウンロード</a>
        </div>
    </div>

    <script>
        function convertFile() {
            const fileInput = document.getElementById('fileInput');
            const outputFormat = document.getElementById('outputFormat').value;
            const resultSection = document.getElementById('resultSection');
            const downloadLink = document.getElementById('downloadLink');

            if (fileInput.files.length > 0) {
                // ファイルを取得
                const file = fileInput.files[0];

                // 仮の変換処理（実際の処理はサーバーサイドで行う）
                const fileName = file.name.split('.')[0] + "_converted"; // 仮の名前
                const downloadUrl = "/path/to/converted/" + fileName + "." + outputFormat; // 仮のURL

                // ダウンロードリンクを設定
                downloadLink.href = downloadUrl;
                downloadLink.textContent = `変換後の${outputFormat.toUpperCase()}ファイルをダウンロード`;

                // 結果セクションを表示
                resultSection.style.display = 'block';
            } else {
                alert("ファイルを選択してください。");
            }
        }
    </script>

</body>
</html>
