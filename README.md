<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>File Upload System</title>
    <!-- Tailwind CSS via jsdelivr -->
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <!-- Font Awesome for icons -->
    <link href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.0.0/css/all.min.css" rel="stylesheet">
</head>
<body class="bg-gray-100 min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <!-- Header Section -->
        <header class="bg-white shadow-md rounded-lg p-6 mb-8">
            <h1 class="text-3xl font-bold text-center text-blue-600">File Upload System</h1>
            <p class="text-center text-gray-600 mt-2">Upload CSV or Excel files (Max 1MB)</p>
        </header>

        <!-- Upload Section (Visible to all users) -->
        <div id="upload-section" class="bg-white shadow-md rounded-lg p-6 mb-8">
            <h2 class="text-xl font-semibold mb-4 text-gray-800">Upload Your File</h2>
            
            <div class="upload-area border-2 border-dashed border-gray-300 rounded-lg p-8 text-center" id="drop-area">
                <i class="fas fa-cloud-upload-alt text-5xl text-blue-500 mb-4"></i>
                <p class="mb-4">Drag & Drop your file here or</p>
                <label class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded cursor-pointer transition duration-300">
                    Browse Files
                    <input type="file" id="file-input" class="hidden" accept=".csv,.xls,.xlsx">
                </label>
                <p class="mt-3 text-sm text-gray-500">Allowed file types: .csv, .xls, .xlsx (Max: 1MB)</p>
            </div>
            
            <div id="file-details" class="mt-4 hidden">
                <div class="flex items-center p-3 bg-gray-100 rounded-lg">
                    <i class="fas fa-file-alt text-blue-500 text-xl mr-3"></i>
                    <div class="flex-grow">
                        <p id="file-name" class="font-medium">filename.csv</p>
                        <p id="file-size" class="text-sm text-gray-500">Size: 0 KB</p>
                    </div>
                    <button id="remove-file" class="text-red-500 hover:text-red-700">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
            </div>
            
            <div class="mt-6">
                <button id="upload-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-6 rounded disabled:opacity-50 disabled:cursor-not-allowed transition duration-300 w-full" disabled>
                    Upload File
                </button>
            </div>
            
            <!-- Upload Status -->
            <div id="upload-status" class="mt-4 hidden">
                <div id="success-message" class="bg-green-100 border-l-4 border-green-500 text-green-700 p-4 hidden">
                    <div class="flex items-center">
                        <i class="fas fa-check-circle mr-3"></i>
                        <p>File uploaded successfully!</p>
                    </div>
                </div>
                <div id="error-message" class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 hidden">
                    <div class="flex items-center">
                        <i class="fas fa-exclamation-circle mr-3"></i>
                        <p id="error-text">Error uploading file.</p>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Admin Section Toggle -->
        <div class="text-center mb-8">
            <button id="admin-toggle-btn" class="text-blue-600 hover:text-blue-800 font-medium">
                Admin Access <i class="fas fa-lock ml-1"></i>
            </button>
        </div>
        
        <!-- Admin Login Section (Initially Hidden) -->
        <div id="admin-login" class="bg-white shadow-md rounded-lg p-6 mb-8 hidden">
            <h2 class="text-xl font-semibold mb-4 text-gray-800">Admin Login</h2>
            <div class="mb-4">
                <label for="admin-password" class="block text-gray-700 mb-2">Password</label>
                <input type="password" id="admin-password" class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
            </div>
            <button id="login-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-6 rounded transition duration-300 w-full">
                Login
            </button>
            <p id="login-error" class="text-red-500 mt-2 hidden">Incorrect password. Please try again.</p>
        </div>
        
        <!-- Admin Files Section (Initially Hidden) -->
        <div id="admin-files" class="bg-white shadow-md rounded-lg p-6 mb-8 hidden">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-semibold text-gray-800">Uploaded Files</h2>
                <button id="admin-logout" class="text-red-500 hover:text-red-700">
                    <i class="fas fa-sign-out-alt mr-1"></i> Logout
                </button>
            </div>
            
            <div id="files-list" class="space-y-3">
                <!-- Files will be listed here via JavaScript -->
                <div id="no-files-message" class="text-center text-gray-500 py-8">
                    No files have been uploaded yet.
                </div>
            </div>
        </div>
        
        <!-- Footer -->
        <footer class="text-center text-gray-600 mt-8">
            <p>&copy; 2023 File Upload System. All rights reserved.</p>
        </footer>
    </div>

    <script>
        // Configuration
        const ADMIN_PASSWORD = "admin123"; // Simple password for demo purposes
        const MAX_FILE_SIZE = 1 * 1024 * 1024; // 1MB in bytes
        const ALLOWED_FILE_TYPES = [
            'application/vnd.ms-excel', 
            'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
            'text/csv'
        ];
        const ALLOWED_FILE_EXTENSIONS = ['.csv', '.xls', '.xlsx'];

        // DOM Elements
        const fileInput = document.getElementById('file-input');
        const dropArea = document.getElementById('drop-area');
        const fileDetails = document.getElementById('file-details');
        const fileName = document.getElementById('file-name');
        const fileSize = document.getElementById('file-size');
        const removeFileBtn = document.getElementById('remove-file');
        const uploadBtn = document.getElementById('upload-btn');
        const uploadStatus = document.getElementById('upload-status');
        const successMessage = document.getElementById('success-message');
        const errorMessage = document.getElementById('error-message');
        const errorText = document.getElementById('error-text');
        const adminToggleBtn = document.getElementById('admin-toggle-btn');
        const adminLogin = document.getElementById('admin-login');
        const adminPassword = document.getElementById('admin-password');
        const loginBtn = document.getElementById('login-btn');
        const loginError = document.getElementById('login-error');
        const adminFiles = document.getElementById('admin-files');
        const filesList = document.getElementById('files-list');
        const noFilesMessage = document.getElementById('no-files-message');
        const adminLogout = document.getElementById('admin-logout');

        // Global Variables
        let selectedFile = null;

        // Utility Functions
        function formatBytes(bytes, decimals = 2) {
            if (bytes === 0) return '0 Bytes';
            const k = 1024;
            const dm = decimals < 0 ? 0 : decimals;
            const sizes = ['Bytes', 'KB', 'MB', 'GB'];
            const i = Math.floor(Math.log(bytes) / Math.log(k));
            return parseFloat((bytes / Math.pow(k, i)).toFixed(dm)) + ' ' + sizes[i];
        }

        function getFileExtension(filename) {
            return '.' + filename.split('.').pop().toLowerCase();
        }

        function isValidFileType(file) {
            const extension = getFileExtension(file.name);
            return ALLOWED_FILE_EXTENSIONS.includes(extension) || 
                   ALLOWED_FILE_TYPES.includes(file.type);
        }

        // File Handling Functions
        function handleFileSelect(file) {
            if (!file) return;
            
            // Validate file size
            if (file.size > MAX_FILE_SIZE) {
                showError(`File too large. Maximum size is ${formatBytes(MAX_FILE_SIZE)}.`);
                return;
            }
            
            // Validate file type
            if (!isValidFileType(file)) {
                showError('Invalid file type. Only CSV and Excel files are allowed.');
                return;
            }
            
            // Set the selected file and update UI
            selectedFile = file;
            fileName.textContent = file.name;
            fileSize.textContent = `Size: ${formatBytes(file.size)}`;
            fileDetails.classList.remove('hidden');
            uploadBtn.disabled = false;
            hideMessages();
        }

        function removeFile() {
            selectedFile = null;
            fileInput.value = '';
            fileDetails.classList.add('hidden');
            uploadBtn.disabled = true;
            hideMessages();
        }

        function uploadFile() {
            if (!selectedFile) return;
            
            try {
                // Get existing files from localStorage
                const files = JSON.parse(localStorage.getItem('uploadedFiles')) || [];
                
                // Read file contents
                const reader = new FileReader();
                reader.onload = function(e) {
                    const fileData = {
                        id: Date.now().toString(),
                        name: selectedFile.name,
                        size: selectedFile.size,
                        type: selectedFile.type,
                        content: e.target.result,
                        uploadDate: new Date().toISOString()
                    };
                    
                    // Add to files array and save to localStorage
                    files.push(fileData);
                    localStorage.setItem('uploadedFiles', JSON.stringify(files));
                    
                    // Show success message
                    showSuccess('File uploaded successfully!');
                    removeFile();
                    
                    // Update admin files list if visible
                    if (!adminFiles.classList.contains('hidden')) {
                        displayUploadedFiles();
                    }
                };
                
                reader.onerror = function() {
                    showError('Error reading file.');
                };
                
                reader.readAsDataURL(selectedFile);
            } catch (error) {
                showError('Error uploading file: ' + error.message);
            }
        }

        // UI Functions
        function showSuccess(message) {
            uploadStatus.classList.remove('hidden');
            successMessage.classList.remove('hidden');
            errorMessage.classList.add('hidden');
        }

        function showError(message) {
            uploadStatus.classList.remove('hidden');
            errorMessage.classList.remove('hidden');
            successMessage.classList.add('hidden');
            errorText.textContent = message;
        }

        function hideMessages() {
            uploadStatus.classList.add('hidden');
            successMessage.classList.add('hidden');
            errorMessage.classList.add('hidden');
        }

        // Admin Functions
        function toggleAdminLogin() {
            adminLogin.classList.toggle('hidden');
            adminFiles.classList.add('hidden');
            adminPassword.value = '';
            loginError.classList.add('hidden');
        }

        function loginAdmin() {
            if (adminPassword.value === ADMIN_PASSWORD) {
                adminLogin.classList.add('hidden');
                adminFiles.classList.remove('hidden');
                loginError.classList.add('hidden');
                displayUploadedFiles();
            } else {
                loginError.classList.remove('hidden');
            }
        }

        function logoutAdmin() {
            adminFiles.classList.add('hidden');
            adminLogin.classList.add('hidden');
        }

        function displayUploadedFiles() {
            const files = JSON.parse(localStorage.getItem('uploadedFiles')) || [];
            
            if (files.length === 0) {
                noFilesMessage.classList.remove('hidden');
                return;
            }
            
            noFilesMessage.classList.add('hidden');
            
            // Clear existing list
            const fileItems = filesList.querySelectorAll('.file-item');
            fileItems.forEach(item => {
                if (!item.id.includes('no-files-message')) {
                    item.remove();
                }
            });
            
            // Add files to list
            files.forEach(file => {
                const fileItem = document.createElement('div');
                fileItem.className = 'file-item flex items-center justify-between bg-gray-50 p-4 rounded-lg';
                fileItem.dataset.fileId = file.id;
                
                // Format date for display
                const uploadDate = new Date(file.uploadDate);
                const formattedDate = uploadDate.toLocaleDateString() + ' ' + uploadDate.toLocaleTimeString();
                
                fileItem.innerHTML = `
                    <div class="flex items-center">
                        <i class="fas fa-file-alt text-blue-500 text-xl mr-3"></i>
                        <div>
                            <p class="font-medium">${file.name}</p>
                            <p class="text-sm text-gray-500">Size: ${formatBytes(file.size)}</p>
                            <p class="text-xs text-gray-400">Uploaded: ${formattedDate}</p>
                        </div>
                    </div>
                    <div class="flex space-x-2">
                        <button class="download-btn bg-green-500 hover:bg-green-600 text-white px-3 py-1 rounded text-sm">
                            <i class="fas fa-download mr-1"></i> Download
                        </button>
                        <button class="delete-btn bg-red-500 hover:bg-red-600 text-white px-3 py-1 rounded text-sm">
                            <i class="fas fa-trash mr-1"></i> Delete
                        </button>
                    </div>
                `;
                
                filesList.appendChild(fileItem);
                
                // Add event listeners to buttons
                const downloadBtn = fileItem.querySelector('.download-btn');
                const deleteBtn = fileItem.querySelector('.delete-btn');
                
                downloadBtn.addEventListener('click', () => downloadFile(file));
                deleteBtn.addEventListener('click', () => deleteFile(file.id));
            });
        }

        function downloadFile(file) {
            const linkElement = document.createElement('a');
            linkElement.href = file.content;
            linkElement.download = file.name;
            document.body.appendChild(linkElement);
            linkElement.click();
            document.body.removeChild(linkElement);
        }

        function deleteFile(fileId) {
            const files = JSON.parse(localStorage.getItem('uploadedFiles')) || [];
            const updatedFiles = files.filter(file => file.id !== fileId);
            localStorage.setItem('uploadedFiles', JSON.stringify(updatedFiles));
            displayUploadedFiles();
        }

        // Event Listeners
        window.addEventListener('DOMContentLoaded', () => {
            // File input change
            fileInput.addEventListener('change', (e) => {
                if (e.target.files.length > 0) {
                    handleFileSelect(e.target.files[0]);
                }
            });
            
            // Remove file button
            removeFileBtn.addEventListener('click', removeFile);
            
            // Upload button
            uploadBtn.addEventListener('click', uploadFile);
            
            // Admin toggle button
            adminToggleBtn.addEventListener('click', toggleAdminLogin);
            
            // Login button
            loginBtn.addEventListener('click', loginAdmin);
            
            // Logout button
            adminLogout.addEventListener('click', logoutAdmin);
            
            // Enter key for password
            adminPassword.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    loginAdmin();
                }
            });
            
            // Drag and drop functionality
            ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
                dropArea.addEventListener(eventName, (e) => {
                    e.preventDefault();
                    e.stopPropagation();
                }, false);
            });
            
            dropArea.addEventListener('dragenter', () => {
                dropArea.classList.add('border-blue-500');
            });
            
            dropArea.addEventListener('dragleave', () => {
                dropArea.classList.remove('border-blue-500');
            });
            
            dropArea.addEventListener('drop', (e) => {
                dropArea.classList.remove('border-blue-500');
                if (e.dataTransfer.files.length > 0) {
                    handleFileSelect(e.dataTransfer.files[0]);
                }
            });
        });
    </script>
</body>
</html>
