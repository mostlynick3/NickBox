<!DOCTYPE html>
<html>
<head>
    <title>NickBox - Simple Python Multiboxer</title>
    <style>
        body {
            font-family: sans-serif;
            margin: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }
        header {
            background-color: #f0f0f0;
            padding: 10px;
            text-align: center;
        }
        nav {
            background-color: #e0e0e0;
            padding: 10px;
            text-align: center;
        }
        nav a {
            margin: 0 15px;
            text-decoration: none;
            color: #333;
        }
        nav a:hover {
            text-decoration: underline;
        }
        .main-content {
            display: flex;
            flex: 1;
            overflow: auto;
        }
        .sidebar {
            width: 200px;
            background-color: #f8f8f8;
            padding: 20px;
            box-sizing: border-box;
        }
        .content {
            flex: 1;
            padding: 20px;
            box-sizing: border-box;
            overflow-y: auto;
        }
        .img-logo {
            max-width: 200px;
        }
        .demo-image {
            max-width: 100%;
            margin: 20px 0;
            border: 1px solid #ddd;
        }
        .video-container {
            margin-bottom: 20px;
        }
        .code-modal {
            display: none;
            position: fixed;
            z-index: 1;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.7);
        }
        .code-modal-content {
            background-color: #f8f8f8;
            margin: 5% auto;
            padding: 20px;
            width: 80%;
            max-height: 80vh;
            overflow-y: auto;
        }
        .close {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close:hover {
            color: black;
        }
        pre {
            white-space: pre-wrap;
            word-wrap: break-word;
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 5px;
            overflow-x: auto;
        }
        .faq-item {
            margin-bottom: 20px;
        }
        .faq-question {
            font-weight: bold;
            cursor: pointer;
            color: #2c3e50;
        }
        .faq-answer {
            margin-top: 10px;
            margin-left: 20px;
        }
        ul {
            padding-left: 20px;
        }
        ul li {
            margin-bottom: 10px;
        }
        .releases-iframe {
            width: 100%;
            height: 800px;
            border: none;
        }
    </style>
</head>
<body>
<header>
    <img src="https://github.com/user-attachments/assets/5ffecc35-27b9-45e4-8aca-dff1ba751f30" alt="NickBox Logo" class="img-logo">
</header>
<nav>
    <a href="#" onclick="loadHomePage()">Home</a>
    <a href="#" onclick="loadDemos()">Demos</a>
    <a href="#" onclick="loadDownloads()">Downloads</a>
    <a href="#" onclick="loadFAQ()">FAQ</a>
    <a href="#" onclick="loadLicense()">License</a>
</nav>
<div class="main-content">
    <div class="sidebar" id="sidebar">
        <!-- Will be populated dynamically -->
    </div>
    <div class="content" id="mainContent">
        <!-- Content will be loaded dynamically -->
        <div id="loadingIndicator">Loading content...</div>
    </div>
</div>

<!-- Code Modal for viewing source code -->
<div id="codeModal" class="code-modal">
    <div class="code-modal-content">
        <span class="close" onclick="closeCodeModal()">&times;</span>
        <h2 id="codeModalTitle">Source Code</h2>
        <pre id="codeModalContent">Loading code...</pre>
    </div>
</div>

<script>
    // Repository information
    const repoOwner = 'mostlynick3';
    const repoName = 'NickBox-simple-Python-multiboxer';
    const webBranch = 'web';  // Everything now references web branch
    
    // Function to fetch and parse content from web branch
    async function fetchContent(path) {
        try {
            const response = await fetch(`https://api.github.com/repos/${repoOwner}/${repoName}/contents/${path}?ref=${webBranch}`);
            if (!response.ok) throw new Error('Network response was not ok');
            
            const data = await response.json();
            // GitHub API returns content as base64 encoded
            const content = atob(data.content);
            return content;
        } catch (error) {
            console.error(`Error fetching ${path}:`, error);
            return `Failed to load ${path}. Error: ${error.message}`;
        }
    }
    
    // Enhanced markdown parser
    function parseMarkdown(markdown) {
        let html = markdown
            // Headers
            .replace(/^### (.*$)/gim, '<h3>$1</h3>')
            .replace(/^## (.*$)/gim, '<h2>$1</h2>')
            .replace(/^# (.*$)/gim, '<h1>$1</h1>')
            // Bold
            .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
            // Italic
            .replace(/\*(.*?)\*/g, '<em>$1</em>')
            // Bullet lists
            .replace(/^\* (.*$)/gim, '<li>$1</li>')
            // Links
            .replace(/\[(.*?)\]\((.*?)\)/g, '<a href="$2" target="_blank">$1</a>');
        
        // Wrap list items in ul tags
        html = html.replace(/<li>.*?<\/li>/gs, match => {
            return `<ul>${match}</ul>`;
        });
        
        // Convert line breaks to paragraphs (excluding lists and headers)
        const paragraphs = html.split(/\n\s*\n/);
        html = paragraphs.map(p => {
            if (!p.trim().startsWith('<')) {
                return `<p>${p.trim()}</p>`;
            }
            return p;
        }).join('\n');
        
        return html;
    }
    
    // Load homepage (README.md content from web branch)
    async function loadHomePage() {
        document.getElementById('mainContent').innerHTML = '<div id="loadingIndicator">Loading content...</div>';
        
        try {
            const readmeContent = await fetchContent('README.md');
            let parsedContent = parseMarkdown(readmeContent);
            
            // Update main content
            document.getElementById('mainContent').innerHTML = parsedContent;
            
            // Extract sections for sidebar
            updateSidebar(readmeContent);
        } catch (error) {
            document.getElementById('mainContent').innerHTML = `<p>Error loading content: ${error.message}</p>`;
        }
    }
    
    // Update sidebar based on README headers
    function updateSidebar(markdown) {
        const matches = markdown.match(/^##\s+(.*)$/gm);
        
        if (matches) {
            const sidebarContent = matches.map(match => {
                const header = match.replace(/^##\s+/, '').trim();
                return `<h2><a href="#" onclick="scrollToSection('${header}')">${header}</a></h2>`;
            }).join('');
            
            document.getElementById('sidebar').innerHTML = sidebarContent;
        } else {
            document.getElementById('sidebar').innerHTML = '<h2>Features</h2><h2>Limitations</h2><h2>Example Scenario</h2>';
        }
    }
    
    // Scroll to section
    function scrollToSection(sectionTitle) {
        const elements = document.querySelectorAll('h2');
        for (const element of elements) {
            if (element.textContent === sectionTitle) {
                element.scrollIntoView({ behavior: 'smooth' });
                return;
            }
        }
    }
    
    // Load downloads - using iframe directly to GitHub releases page
    function loadDownloads() {
        const releasesUrl = `https://github.com/${repoOwner}/${repoName}/releases`;
        document.getElementById('mainContent').innerHTML = `
            <h2>Downloads</h2>
            <iframe src="${releasesUrl}" class="releases-iframe" title="NickBox Releases"></iframe>
        `;
        document.getElementById('sidebar').innerHTML = '<h2>Downloads</h2>';
    }
    
    // Load license content directly in the main content area
    async function loadLicense() {
        document.getElementById('mainContent').innerHTML = '<div id="loadingIndicator">Loading license...</div>';
        
        try {
            const licenseContent = await fetchContent('LICENSE');
            document.getElementById('mainContent').innerHTML = `<h2>License</h2><pre>${licenseContent}</pre>`;
            document.getElementById('sidebar').innerHTML = '<h2>License</h2>';
        } catch (error) {
            document.getElementById('mainContent').innerHTML = `<h2>License</h2><p>Failed to load license. Error: ${error.message}</p>`;
        }
    }
    
    // Load FAQ content
    function loadFAQ() {
        const faqContent = `
            <h2>Frequently Asked Questions</h2>
            
            <div class="faq-item">
                <p>Open source development offers enhanced security through code transparency. When software is open source, anyone can inspect the code, making it more difficult for malicious features to be hidden. This collaborative approach often leads to better quality software as bugs are found and fixed by the community.</p>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">How can I see the source code?</div>
                <div class="faq-answer">
                    <p>By going to NickBox.py in the web branch, or clicking <a href="#" onclick="viewSourceCode('NickBox.py')">here</a> to view the main source code in a popup window.</p>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">How can I compile this myself?</div>
                <div class="faq-answer">
                    <p>You can use any pyinstaller implementation. This project is compiled to .exe using pyinstaller with UPX compression, an icon and the following tags:</p>
                    <pre>--clean --onefile --exclude-module=cv2 --exclude-module=numpy --exclude-module=PIL --exclude=libcrypto-3.dll --name="NickBox" --noconsole NickBox.py</pre>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">What does the 'Hide Taskbar' feature do?</div>
                <div class="faq-answer">
                    <p>The 'Hide Taskbar' feature, controlled by the hide_taskbar_var boolean variable, allows you to hide the Windows taskbar for all targeted windows. This gives you a cleaner view when multiboxing by removing the taskbar from all application windows.</p>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">How does the grid snapping work?</div>
                <div class="faq-answer">
                    <p>Grid snapping is enabled by default (grid_snap_enabled = True) with a default snap size of 20 pixels (grid_snap_size = 20). When enabled, windows will snap to an invisible grid when being repositioned, making it easier to align multiple windows precisely.</p>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">What is the purpose of the 'broadcast_all_keys' setting?</div>
                <div class="faq-answer">
                    <p>When broadcast_all_keys is set to True, all keyboard inputs will be sent to all target windows. When set to False, only specific keys (defined in your keys.csv file) will be broadcasted. This allows for more selective control when multiboxing.</p>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">How does auto-focusing work?</div>
                <div class="faq-answer">
                    <p>The auto-focusing feature (controlled by is_auto_focusing) automatically rotates focus between your target windows, allowing for round-robin input when needed. This can be paused temporarily (is_auto_focusing_paused) when you need to maintain focus on a specific window.</p>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">What happens during an Alt+Tab?</div>
                <div class="faq-answer">
                    <p>NickBox detects Alt+Tab keystrokes (alt_tab_pressed) and temporarily pauses broadcasting for a short duration (alt_tab_pause_time = 1 second) to allow you to switch between windows without causing unintended behaviors in your target applications.</p>
                </div>
            </div>
            
            <div class="faq-item">
                <div class="faq-question" onclick="toggleAnswer(this)">Can I save different window configurations?</div>
                <div class="faq-answer">
                    <p>Yes, NickBox supports profiles that save your window configurations and key mappings. The profiles are stored in the "settings" directory (profiles_dir) with the current profile name stored in current_profile. Window positions, sizes, and other settings are automatically saved to the selected profile.</p>
                </div>
            </div>
        `;
        
        document.getElementById('mainContent').innerHTML = faqContent;
        document.getElementById('sidebar').innerHTML = '<h2>FAQ</h2>';
    }
    
    function loadDemos() {
        document.getElementById('mainContent').innerHTML = '<div id="loadingIndicator">Loading demo videos...</div>';
        
        // Base path for demos directory
        const demosPath = 'media/demos/';
        
        let demosContent = '<h2>Demo Videos</h2>';
        
        // Videos section FIRST
        demosContent += '<div class="video-grid">';
        
        // Try to load common video formats for known demos
        const videoNames = ['demo-scaling', 'demo-startup'];
        const videoExtensions = ['.mp4', '.webm', '.ogg'];
        
        videoNames.forEach(videoName => {
            const formattedName = videoName
                .replace(/-|_/g, ' ')
                .replace(/\b\w/g, l => l.toUpperCase());
            
            demosContent += `
            <div class="video-item">
                <h3>${formattedName}</h3>
                <video controls width="100%">`;
                
            // Try different formats, browser will use the first one that works
            videoExtensions.forEach(ext => {
                demosContent += `<source src="${demosPath}${videoName}${ext}" type="video/${ext.substring(1)}">`;
            });
            
            demosContent += `
                    Your browser does not support these video formats.
                </video>
            </div>`;
        });
        
        demosContent += '</div>';
        
        // COMPLETELY SEPARATE section for images AFTER videos
        demosContent += `
        <div class="images-section">
            <h2>Screenshot Examples</h2>
            <div class="image-container">
                <h3>World of Warcraft Multiboxing Example</h3>
                <img src="https://github.com/user-attachments/assets/f1b6c891-7a65-4ecd-b383-0986be51acf2" alt="WoW Multiboxing" class="demo-image">
            </div>
        </div>`;
        
        document.getElementById('mainContent').innerHTML = demosContent;
        document.getElementById('sidebar').innerHTML = '<h2>Demos</h2>';
        
        // Alternative approach: scan for potential video files
        checkForAdditionalVideos();
    }
    
    // Function to check for additional videos beyond known ones
    function checkForAdditionalVideos() {
        const demosPath = 'media/demos/';
        const potentialVideos = [
            'demo-controls', 'demo-features', 'demo-overview', 
            'multibox-example', 'wow-demo', 'nickbox-intro'
        ];
        const videoGrid = document.querySelector('.video-grid');
        
        if (!videoGrid) return;
        
        potentialVideos.forEach(name => {
            // Try to load the video
            const video = document.createElement('video');
            video.controls = true;
            video.style.display = 'none'; // Hide until we know it exists
            
            const source = document.createElement('source');
            source.src = `${demosPath}${name}.mp4`;
            source.type = 'video/mp4';
            
            video.appendChild(source);
            
            // Add handlers to manage video discovery
            video.addEventListener('loadeddata', () => {
                // Video exists and loaded
                const formattedName = name
                    .replace(/-|_/g, ' ')
                    .replace(/\b\w/g, l => l.toUpperCase());
                    
                const videoItem = document.createElement('div');
                videoItem.className = 'video-item';
                videoItem.innerHTML = `
                    <h3>${formattedName}</h3>
                `;
                
                video.style.display = 'block';
                video.width = '100%';
                videoItem.appendChild(video);
                videoGrid.appendChild(videoItem);
            });
            
            video.addEventListener('error', () => {
                // Video doesn't exist or couldn't load, do nothing
                video.remove();
            });
            
            // Start loading to check if file exists
            document.body.appendChild(video);
        });
    }
    
    // View source code in modal
    async function viewSourceCode(filename) {
        document.getElementById('codeModalTitle').textContent = filename;
        document.getElementById('codeModalContent').textContent = 'Loading source code...';
        document.getElementById('codeModal').style.display = 'block';
        
        try {
            const sourceCode = await fetchContent(filename);
            document.getElementById('codeModalContent').textContent = sourceCode;
        } catch (error) {
            document.getElementById('codeModalContent').textContent = `Failed to load source code: ${error.message}`;
        }
    }
    
    // Close code modal
    function closeCodeModal() {
        document.getElementById('codeModal').style.display = 'none';
    }
    
    // Toggle FAQ answer visibility
    function toggleAnswer(element) {
        const answer = element.nextElementSibling;
        answer.style.display = answer.style.display === 'none' ? 'block' : 'none';
    }
    
    // Close modal when clicking outside of it
    window.onclick = function(event) {
        const modal = document.getElementById('codeModal');
        if (event.target === modal) {
            modal.style.display = 'none';
        }
    }
    
    // Initialize all FAQ answers to be visible
    document.addEventListener('DOMContentLoaded', function() {
        const answers = document.querySelectorAll('.faq-answer');
        answers.forEach(answer => {
            answer.style.display = 'block';
        });
        
        // Load home page
        loadHomePage();
    });
</script>
</body>
</html>
