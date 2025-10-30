<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Krishi Vaidya (Plant Diagnosis)</title>
    <!-- Load Tailwind CSS --><script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Inter Font --><link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0fdf4; /* Pale green background (Field-inspired) */
        }
        .krishi-card {
            background-color: #ffffff;
        }
        .header-color {
            background-color: #10b981; /* Emerald Green */
            color: #ecfdf5;
        }
        .submit-btn {
            background-color: #f59e0b; /* Mustard Yellow/Sunlight */
        }
        .submit-btn:hover {
            background-color: #d97706;
        }
        .loader {
            border-top-color: #10b981;
            border-left-color: #10b981;
            animation: spin 1s ease-in-out infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
        /* Custom file input style */
        .custom-file-input::-webkit-file-upload-button {
            visibility: hidden;
        }
        .custom-file-input::before {
            content: 'Select Photo';
            display: inline-block;
            background: #10b981;
            color: white;
            border: 1px solid #059669;
            border-radius: 4px;
            padding: 8px 12px;
            outline: none;
            white-space: nowrap;
            -webkit-user-select: none;
            cursor: pointer;
            font-weight: 600;
            text-align: center;
        }
        .custom-file-input:hover::before {
            background: #059669;
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4">

    <!-- Main Application Container --><div id="app" class="w-full max-w-3xl shadow-2xl rounded-xl overflow-hidden krishi-card">

        <!-- Header (Swadeshi Accent) --><header class="p-6 header-color rounded-t-xl">
            <h1 class="text-3xl font-extrabold tracking-tight">🌿 Krishi Vaidya (कृषि वैद्य)</h1>
            <p class="text-sm font-medium mt-1">AI-Powered Diagnosis for Your Crops. (Aatmanirbharta in Agriculture)</p>
        </header>

        <main class="p-6">

            <!-- Diagnosis Form --><div class="mb-8 p-4 border-2 border-green-500 rounded-lg shadow-md">
                <h2 class="text-xl font-bold mb-3 text-green-700">Submit Plant Symptoms</h2>
                <form id="diagnosis-form" class="space-y-4">
                    
                    <!-- Text Description Input --><textarea
                        id="symptom-prompt"
                        placeholder="Describe the plant's problem (e.g., 'My tomato leaves are turning yellow and have black spots on the underside.')"
                        rows="3"
                        maxlength="500"
                        class="w-full p-3 border border-gray-300 rounded-md focus:ring-green-500 focus:border-green-500 transition"
                        required
                    ></textarea>

                    <!-- Image Input --><div class="flex flex-col sm:flex-row items-start sm:items-center gap-4">
                        <div class="flex-shrink-0">
                            <input
                                type="file"
                                id="image-input"
                                accept="image/png, image/jpeg"
                                class="custom-file-input block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-md file:border-0 file:text-sm file:font-semibold file:bg-green-600 file:text-white hover:file:bg-green-700"
                            >
                            <p class="text-xs text-gray-500 mt-1 sm:mt-0">PNG or JPEG supported. (Use a detailed description if no image is provided)</p>
                        </div>
                        
                        <!-- Image Preview --><div id="image-preview-container" class="hidden">
                            <img id="image-preview" class="max-h-24 w-auto rounded-lg border-2 border-green-300 shadow-md" alt="Plant Photo Preview">
                        </div>
                    </div>
                    
                    <p class="text-sm text-gray-600 border-t pt-2">
                        For the best result, provide both a detailed **description** and a clear **photo** of the affected area.
                    </p>

                    <button
                        type="submit"
                        id="diagnose-btn"
                        class="w-full submit-btn text-white font-semibold py-3 rounded-lg shadow-md transition duration-200 disabled:opacity-50 flex items-center justify-center"
                    >
                        <span id="button-text">Get Grounded Diagnosis & Cure</span>
                        <div id="loader" class="loader w-5 h-5 border-4 border-gray-100 border-solid rounded-full hidden ml-3"></div>
                    </button>
                </form>
            </div>

            <!-- Result Area --><div id="result-area" class="min-h-[200px] border-t-2 border-green-300 pt-4 hidden">
                <h2 class="text-2xl font-bold mb-4 text-gray-700">🔬 Diagnosis Report</h2>
                
                <div id="diagnosis-content" class="bg-white p-4 rounded-lg shadow-inner">
                    <!-- Results will be injected here --></div>
            </div>

            <!-- Error/Message Box (Hidden by default) --><div id="message-box" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center p-4 z-50">
                <div class="bg-white p-6 rounded-lg shadow-xl max-w-sm w-full">
                    <h3 id="message-title" class="text-xl font-bold mb-3 text-red-600">Error</h3>
                    <p id="message-content" class="text-gray-700 mb-4">An unexpected error occurred.</p>
                    <button id="message-close" class="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-2 rounded-lg transition duration-200">
                        Close
                    </button>
                </div>
            </div>

        </main>
    </div>

    <!-- JavaScript for API Interaction --><script type="module">
        // --- UI ELEMENTS ---
        const symptomPrompt = document.getElementById('symptom-prompt');
        const imageInput = document.getElementById('image-input');
        const imagePreview = document.getElementById('image-preview');
        const imagePreviewContainer = document.getElementById('image-preview-container');
        const diagnosisForm = document.getElementById('diagnosis-form');
        const diagnoseBtn = document.getElementById('diagnose-btn');
        const resultArea = document.getElementById('result-area');
        const diagnosisContent = document.getElementById('diagnosis-content');
        const buttonText = document.getElementById('button-text');
        const loader = document.getElementById('loader');

        // Message Box Elements
        const messageBox = document.getElementById('message-box');
        const messageTitle = document.getElementById('message-title');
        const messageContent = document.getElementById('message-content');
        const messageCloseBtn = document.getElementById('message-close');

        // State for image data
        let base64Image = null;
        let mimeType = null;
        
        // --- DISEASE KNOWLEDGE BASE (NEW STRUCTURE) ---
        const PLANT_DISEASE_KNOWLEDGE = {
            'Fungal Diseases': [
                'Powdery Mildew', 'Rust', 'Leaf Spot (can also be bacterial)', 'Blight (can also be bacterial)',
                'Wilt (can also be bacterial)', 'Black Spot', 'Downy Mildew (fungus-like organism)', 'Clubroot (protist)'
            ],
            'Bacterial Diseases': [
                'Fire Blight', 'Leaf Spot (can also be fungal)', 'Blight (can also be fungal)', 'Wilt (can also be fungal)'
            ],
            'Viral Diseases': [
                'Tobacco Mosaic Virus (TMV)', 'Tomato Spotted Wilt Virus (TSWV)', 'Tomato Yellow Leaf Curl Virus (TYLCV)',
                'Cucumber Mosaic Virus (CMV)', 'Potato Virus Y (PVY)', 'Cauliflower Mosaic Virus (CaMV)',
                'African Cassava Mosaic Virus (ACMV)', 'Plum Pox Virus (PPV) (Sharka Disease)'
            ]
        };

        // --- API CONFIGURATION (NOW CALLS LOCAL PROXY) ---
        // The API Key is removed from the frontend for security.
        // We now call the serverless function endpoint: /api/diagnosis
        const PROXY_ENDPOINT = '/api/diagnosis';
        
        // --- UTILITY FUNCTIONS (Custom Modal) ---

        function showMessage(title, content, isError = false) {
            messageTitle.textContent = title;
            messageContent.textContent = content; 
            messageTitle.className = isError ? 'text-xl font-bold mb-3 text-red-600' : 'text-xl font-bold mb-3 text-green-700';
            messageBox.classList.remove('hidden');
            messageBox.classList.add('flex');
        }

        messageCloseBtn.addEventListener('click', () => {
            messageBox.classList.remove('flex');
            messageBox.classList.add('hidden');
        });

        // --- UI STATE FUNCTIONS ---

        function setLoading(isLoading) {
            diagnoseBtn.disabled = isLoading;
            if (isLoading) {
                buttonText.textContent = "Analyzing...";
                loader.classList.remove('hidden');
            } else {
                buttonText.textContent = "Get Grounded Diagnosis & Cure";
                loader.classList.add('hidden');
            }
        }

        function createContentBlock(title, content, colorClass) {
            let formattedContent = content;
            
            // Basic markdown parsing for boldness and emphasis
            formattedContent = formattedContent.replace(/(\*\*|__)(.*?)\1/g, '<strong>$2</strong>');
            formattedContent = formattedContent.replace(/(\*|_)(.*?)\1/g, '<em>$2</em>');
            // Bullet point conversion
            formattedContent = formattedContent.replace(/^[\*-]\s*/gm, '• ');
            
            return `
                <div class="mb-4 border-l-4 ${colorClass} pl-3">
                    <h4 class="font-semibold text-lg mb-1">${title}</h4>
                    <p class="text-gray-700 whitespace-pre-wrap">${formattedContent}</p>
                </div>
            `;
        }

        function renderSources(sources) {
            if (!sources || sources.length === 0) return '';
            
            const sourceItems = sources.map((source, index) => `
                <li class="mt-1 text-xs text-blue-700 hover:text-blue-900">
                    ${index + 1}. <a href="${source.uri}" target="_blank" rel="noopener noreferrer" class="underline">${source.title || 'Source Link'}</a>
                </li>
            `).join('');

            return `
                <div id="citation-block" class="mt-6 pt-4 border-t border-gray-200">
                    <h4 class="font-bold text-sm text-green-800">Citations (Grounding Sources)</h4>
                    <ul class="list-none p-0 mt-2">${sourceItems}</ul>
                </div>
            `;
        }
        
        // --- IMAGE HANDLING ---

        imageInput.addEventListener('change', function(event) {
            const file = event.target.files[0];
            if (!file) {
                base64Image = null;
                mimeType = null;
                imagePreviewContainer.classList.add('hidden');
                return;
            }

            mimeType = file.type;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                base64Image = e.target.result.split(',')[1];
                imagePreview.src = e.target.result;
                imagePreviewContainer.classList.remove('hidden');
            };
            reader.onerror = function() {
                showMessage("Image Error", "Could not read the file.", true);
                imageInput.value = '';
                base64Image = null;
                mimeType = null;
                imagePreviewContainer.classList.add('hidden');
            };
            reader.readAsDataURL(file);
        });

        // --- AI API LOGIC (Now calls the PROXY) ---

        async function fetchDiagnosis(userQuery) {
            
            // Construct payload to send to the serverless function
            const clientPayload = {
                query: userQuery,
                image: base64Image,
                mimeType: mimeType,
                diseaseKnowledge: PLANT_DISEASE_KNOWLEDGE // Send knowledge base to backend for system prompt
            };
            
            const maxRetries = 3;
            let delay = 1000;

            for (let i = 0; i < maxRetries; i++) {
                try {
                    // Call the PROXY endpoint instead of the direct Gemini API
                    const response = await fetch(PROXY_ENDPOINT, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(clientPayload)
                    });

                    if (!response.ok) {
                        // Throw specific error for network issues
                        const errorBody = await response.json();
                        throw new Error(`Proxy Error (${response.status}): ${errorBody.error || response.statusText}`);
                    }

                    const result = await response.json();
                    
                    if (result.text) {
                        return { text: result.text, sources: result.sources || [] };
                    } else {
                        throw new Error("Proxy response was empty or malformed.");
                    }
                } catch (error) {
                    console.error(`Attempt ${i + 1} failed:`, error);
                    if (i < maxRetries - 1) {
                        await new Promise(resolve => setTimeout(resolve, delay));
                        delay *= 2; // Exponential backoff
                    } else {
                        throw error; // Re-throw the error after final attempt
                    }
                }
            }
        }


        // --- EVENT HANDLERS ---

        diagnosisForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const query = symptomPrompt.value.trim();

            if (!base64Image && query.length < 10) {
                showMessage("Input Required", "Please provide a clear image OR a detailed description (at least 10 characters) of the plant's symptoms.", false);
                return;
            }

            setLoading(true);
            diagnosisContent.innerHTML = '';
            
            try {
                const { text, sources } = await fetchDiagnosis(query);
                
                let htmlOutput = '';
                
                // Process the markdown response into structured HTML blocks
                const sections = {
                    'Diagnosis': { content: '', color: 'border-green-600' },
                    'Swadeshi Cure': { content: '', color: 'border-yellow-600' },
                    'Prevention': { content: '', color: 'border-blue-600' }
                };
                
                let currentSection = null;
                const lines = text.split('\n');
                
                lines.forEach(line => {
                    if (line.startsWith('## Diagnosis')) {
                        currentSection = 'Diagnosis';
                    } else if (line.startsWith('## Swadeshi Cure')) {
                        currentSection = 'Swadeshi Cure';
                    } else if (line.startsWith('## Prevention')) {
                        currentSection = 'Prevention';
                    } else if (currentSection && line.trim() !== '') {
                        sections[currentSection].content += line + '\n';
                    }
                });

                for (const key in sections) {
                    if (sections[key].content.trim()) {
                        htmlOutput += createContentBlock(key, sections[key].content.trim(), sections[key].color);
                    }
                }
                
                htmlOutput += renderSources(sources);
                
                diagnosisContent.innerHTML = htmlOutput;

            } catch (error) {
                console.error("Diagnosis failed:", error);
                
                let errorMessage = error.message;

                // Specific checks for common proxy/deployment errors
                if (error.message.includes('404')) {
                    errorMessage = `Server Error (404 Not Found). 
                    
                    **TROUBLESHOOTING TIP:** This usually means the serverless function '/api/diagnosis' is not deployed correctly. Ensure you are using a hosting platform that supports serverless functions (like Netlify or Vercel).`;
                } else if (error.message.includes('403') || error.message.includes('Forbidden')) {
                    errorMessage = `API Key Error (403 Forbidden). 
                    
                    **TROUBLESHOOTING TIP:** The serverless function failed to authenticate with the Google AI API. Ensure you have set your API Key in the hosting platform's **Environment Variables** (Name: GEMINI_API_KEY).`;
                } else if (error.message.includes('Proxy Error')) {
                     // Catch custom error messages from the proxy
                     errorMessage = `Serverless Function Error: ${errorMessage}`;
                }
                
                showMessage(
                    "🚨 Diagnosis Failed", 
                    errorMessage, 
                    true
                );
                diagnosisContent.innerHTML = ''; 
                
            } finally {
                setLoading(false);
                resultArea.classList.remove('hidden');
            }
        });

    </script>
</body>
</html>
