<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF 回転・PNG変換・ZIPダウンロード</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.10.377/pdf.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/0.4.1/html2canvas.min.js"></script>
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
            overflow: hidden;
        }
        canvas {
            width: 100% !important;
            height: auto !important;
        }
    </style>
</head>
<body>
    <h1>PDF 回転・PNG変換・ZIPダウンロード</h1>
    
    <!-- PDF アップロードフォーム -->
    <input type="file" id="pdf-upload" accept="application/pdf">
    
    <!-- PDF表示エリア -->
    <div id="pdf-container"></div>
    
    <!-- 回転ボタン -->
    <button onclick="rotatePdf()">回転</button>
    
    <!-- PNG変換とZIPダウンロードボタン -->
    <button onclick="convertPdfToPng()">PNGに変換してダウンロード</button>

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

        // PDFをPNGに変換し、ZIPに圧縮してダウンロードする
        function convertPdfToPng() {
            const zip = new JSZip();
            let promises = [];
            
            // すべてのページをPNGに変換してZIPに追加
            for (let i = 1; i <= pdfDoc.numPages; i++) {
                promises.push(pdfDoc.getPage(i).then(function(page) {
                    const scale = 1.5;
                    const viewport = page.getViewport({ scale: scale, rotation: rotateAngle });

                    const canvas = document.createElement('canvas');
                    const context = canvas.getContext('2d');
                    canvas.width = viewport.width;
                    canvas.height = viewport.height;

                    return page.render({ canvasContext: context, viewport: viewport }).promise.then(function() {
                        // PNG画像に変換
                        const imgData = canvas.toDataURL('image/png');

                        // ZIPに画像を追加
                        zip.file('page_' + i + '.png', imgData.split(',')[1], { base64: true });
                    });
                }));
            }

            // すべてのページが処理されたらZIPをダウンロード
            Promise.all(promises).then(function() {
                zip.generateAsync({ type: 'blob' }).then(function(content) {
                    // ダウンロードリンクを作成してダウンロードをトリガー
                    const link = document.createElement('a');
                    link.href = URL.createObjectURL(content);
                    link.download = 'pdf_pages.zip';
                    link.click();
                });
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
