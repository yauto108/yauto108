<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF 回転とダウンロード</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.10.377/pdf.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        #pdf-container {
            width: 100%;
            height: 500px;
            border: 1px solid #ccc;
            margin-bottom: 20px;
            position: relative;
        }
    </style>
</head>
<body>
    <h1>PDF 回転とダウンロード</h1>
    
    <!-- PDF ファイルアップロードフォーム -->
    <input type="file" id="pdf-upload" accept="application/pdf">
    
    <!-- PDF表示エリア -->
    <div id="pdf-container"></div>
    
    <!-- 回転ボタン -->
    <button onclick="rotatePdf()">回転</button>
    
    <!-- ダウンロードボタン -->
    <button onclick="downloadPdf()">回転したPDFをダウンロード</button>

    <script>
        let pdfDoc = null;  // PDFドキュメント
        let currentPage = 1; // 現在表示しているページ番号
        let rotateAngle = 0;  // 回転角度

        // PDFファイルを読み込む
        function loadPdf(file) {
            const reader = new FileReader();
            reader.onload = function(event) {
                const typedarray = new Uint8Array(event.target.result);

                // PDF.jsを使ってPDFを読み込む
                pdfjsLib.getDocument(typedarray).promise.then(function(doc) {
                    pdfDoc = doc;
                    renderPage(currentPage);  // 最初のページを描画
                });
            };
            reader.readAsArrayBuffer(file);
        }

        // PDFのページを描画する
        function renderPage(pageNum) {
            pdfDoc.getPage(pageNum).then(function(page) {
                const scale = 1.5;
                const viewport = page.getViewport({ scale: scale, rotation: rotateAngle });

                const canvas = document.createElement('canvas');
                const context = canvas.getContext('2d');
                canvas.width = viewport.width;
                canvas.height = viewport.height;

                // 現在のPDF表示エリアをクリア
                document.getElementById('pdf-container').innerHTML = '';  
                document.getElementById('pdf-container').appendChild(canvas);

                // PDFの内容をキャンバスに描画
                page.render({ canvasContext: context, viewport: viewport });
            });
        }

        // PDFを回転させる
        function rotatePdf() {
            rotateAngle = (rotateAngle + 90) % 360;  // 90度ずつ回転
            renderPage(currentPage);  // 回転したページを再描画
        }

        // 回転したPDFをダウンロードする
        function downloadPdf() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            pdfDoc.getPage(currentPage).then(function(page) {
                const scale = 1.5;
                const viewport = page.getViewport({ scale: scale, rotation: rotateAngle });

                // jsPDFを使って回転したPDFを作成
                doc.addImage(viewport.canvas, 'JPEG', 0, 0, viewport.width, viewport.height);
                doc.save('rotated.pdf');  // 回転したPDFをダウンロード
            });
        }

        // ファイルアップロード時にPDFを読み込む
        document.getElementById('pdf-upload').addEventListener('change', function(event) {
            const file = event.target.files[0];
            if (file && file.type === 'application/pdf') {
                loadPdf(file);
            } else {
                alert('PDFファイルを選択してください');
            }
        });
    </script>
</body>
</html>
