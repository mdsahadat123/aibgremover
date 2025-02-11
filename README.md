<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pro Background Remover | AI-Powered Editing</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        :root {
            --primary: #6366f1;
            --secondary: #4f46e5;
        }

        body {
            background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
            min-height: 100vh;
        }

        .upload-container {
            background: white;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s ease;
        }

        .upload-container:hover {
            transform: translateY(-5px);
        }

        .drop-zone {
            border: 2px dashed #cbd5e1;
            border-radius: 0.75rem;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .drop-zone.dragover {
            border-color: var(--primary);
            background: #eef2ff;
        }

        .preview-image {
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            max-height: 500px;
            object-fit: contain;
        }

        .loading-spinner {
            width: 3rem;
            height: 3rem;
            border-width: 0.25em;
        }

        .download-btn {
            background: var(--primary);
            color: white;
            padding: 0.75rem 2rem;
            border-radius: 0.5rem;
            transition: all 0.3s ease;
            text-decoration: none;
            display: inline-block;
        }

        .download-btn:hover {
            background: var(--secondary);
            transform: translateY(-2px);
        }
    </style>
</head>
<body>
    <div class="container py-5">
        <div class="text-center mb-5">
            <h1 class="display-4 fw-bold mb-3">AI Background Remover</h1>
            <p class="lead text-muted">Upload your image and get instant background removal powered by AI</p>
        </div>

        <div class="upload-container p-5 mx-auto" style="max-width: 800px">
            <div class="drop-zone p-5 text-center" id="dropZone">
                <input type="file" id="fileInput" accept="image/*" hidden>
                <div class="mb-3">
                    <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" fill="currentColor" class="bi bi-cloud-upload" viewBox="0 0 16 16">
                        <path d="M4.406 1.342A5.53 5.53 0 0 1 8 0c2.69 0 4.923 2 5.166 4.579C14.758 4.804 16 6.137 16 7.773 16 9.569 14.502 11 12.687 11H10a.5.5 0 0 1 0-1h2.688C13.979 10 15 8.988 15 7.773c0-1.216-1.02-2.228-2.313-2.228h-.5v-.5C12.188 2.825 10.328 1 8 1a4.53 4.53 0 0 0-2.941 1.1c-.757.652-1.153 1.438-1.153 2.055v.448l-.445.049C2.064 4.805 1 5.952 1 7.318 1 8.785 2.23 10 3.781 10H6a.5.5 0 0 1 0 1H3.781C1.708 11 0 9.366 0 7.318c0-1.763 1.266-3.223 2.942-3.593.143-.863.698-1.723 1.464-2.383z"/>
                        <path d="M7.646 4.146a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 5.707V14.5a.5.5 0 0 1-1 0V5.707L5.354 7.854a.5.5 0 1 1-.708-.708l3-3z"/>
                    </svg>
                </div>
                <h4 class="mb-2">Drag & Drop Image</h4>
                <p class="text-muted mb-0">or click to browse files</p>
            </div>

            <div class="text-center mt-4 d-none" id="loading">
                <div class="spinner-border text-primary loading-spinner" role="status"></div>
                <p class="mt-3 text-muted">Removing background...</p>
            </div>

            <div class="result-area mt-4 d-none" id="resultArea">
                <img src="" class="preview-image w-100 mb-4" id="outputImage">
                <a class="download-btn border-0" id="downloadBtn">Download Image</a>
            </div>
        </div>
    </div>

    <script>
        const API_KEY = 'Cj4oisfqzpqbB1nGAQs6UaTk'; // Replace with your actual API key
        const dropZone = document.getElementById('dropZone');
        const fileInput = document.getElementById('fileInput');
        const loading = document.getElementById('loading');
        const resultArea = document.getElementById('resultArea');
        const outputImage = document.getElementById('outputImage');
        const downloadBtn = document.getElementById('downloadBtn');

        // Handle file selection
        dropZone.addEventListener('click', () => fileInput.click());

        // Drag & drop handlers
        dropZone.addEventListener('dragover', (e) => {
            e.preventDefault();
            dropZone.classList.add('dragover');
        });

        dropZone.addEventListener('dragleave', () => {
            dropZone.classList.remove('dragover');
        });

        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.classList.remove('dragover');
            handleFile(e.dataTransfer.files[0]);
        });

        // File input change handler
        fileInput.addEventListener('change', (e) => {
            handleFile(e.target.files[0]);
        });

        async function handleFile(file) {
            if (!file) return;

            loading.classList.remove('d-none');
            resultArea.classList.add('d-none');

            try {
                const formData = new FormData();
                formData.append('image_file', file);
                formData.append('size', 'auto');

                const response = await fetch('https://api.remove.bg/v1.0/removebg', {
                    method: 'POST',
                    headers: { 'X-Api-Key': API_KEY },
                    body: formData
                });

                if (!response.ok) throw new Error('Background removal failed. Please try again.');

                const blob = await response.blob();
                const url = URL.createObjectURL(blob);

                outputImage.src = url;
                downloadBtn.href = url;
                downloadBtn.download = `bg-removed-${Date.now()}.png`;

                loading.classList.add('d-none');
                resultArea.classList.remove('d-none');
            } catch (error) {
                alert(error.message);
                loading.classList.add('d-none');
            }
        }
    </script>
</body>
</html>
