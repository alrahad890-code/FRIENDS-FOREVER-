<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Friends Forever</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body class="bg-slate-100 min-h-screen text-slate-800 flex flex-col items-center">

    <!-- মেইন কন্টেইনার -->
    <div id="app" class="w-full max-w-xl mx-auto p-4"></div>

    <script>
        // লোকাল স্টোরেজ থেকে ডেটা লোড বা ইনিশিয়ালাইজ করা
        let currentUser = localStorage.getItem('ff_user') || '';
        let posts = JSON.parse(localStorage.getItem('ff_posts')) || [];
        let tempMedia = { url: '', type: '' };

        function saveData() {
            localStorage.setItem('ff_posts', JSON.stringify(posts));
        }

        function render() {
            const app = document.getElementById('app');
            
            // যদি ইউজার লগইন করা না থাকে
            if (!currentUser) {
                app.innerHTML = `
                    <div class="flex items-center justify-center min-h-[80vh]">
                        <div class="bg-white p-8 rounded-2xl shadow-xl w-full text-center border border-slate-200">
                            <div class="w-16 h-16 bg-indigo-600 text-white rounded-full flex items-center justify-center mx-auto mb-4 text-2xl font-bold shadow-lg">
                                <i class="fa-solid fa-user-group"></i>
                            </div>
                            <h1 class="text-3xl font-extrabold text-indigo-600 mb-2">Friends Forever</h1>
                            <p class="text-slate-500 mb-6 text-sm">বন্ধুদের সাথে যুক্ত হতে আপনার নাম লিখুন</p>
                            
                            <form onsubmit="handleLogin(event)" class="space-y-4">
                                <input type="text" id="usernameInput" placeholder="আপনার নাম লিখুন..." required 
                                    class="w-full px-4 py-3 border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500 text-center text-lg font-medium">
                                <button type="submit" 
                                    class="w-full bg-indigo-600 text-white py-3 rounded-xl font-bold hover:bg-indigo-700 transition shadow-md">
                                    প্রবেশ করুন <i class="fa-solid fa-arrow-right ml-1"></i>
                                </button>
                            </form>
                        </div>
                    </div>
                `;
                return;
            }

            // মূল অ্যাপ ইন্টারফেস
            app.innerHTML = `
                <!-- হেডার -->
                <header class="bg-white border-b border-slate-200 sticky top-0 z-50 shadow-sm rounded-2xl mb-4 px-4 py-3 flex justify-between items-center">
                    <div class="flex items-center space-x-2">
                        <span class="w-10 h-10 bg-indigo-600 text-white rounded-xl flex items-center justify-center font-bold text-lg">FF</span>
                        <h1 class="text-xl font-black text-indigo-600 tracking-wide">Friends Forever</h1>
                    </div>
                    <div class="flex items-center space-x-3">
                        <span class="font-semibold text-slate-700 bg-slate-100 px-3 py-1.5 rounded-full text-sm">
                            <i class="fa-solid fa-circle-user text-indigo-600 mr-1"></i> ${currentUser}
                        </span>
                        <button onclick="logout()" class="text-slate-400 hover:text-red-500 p-2"><i class="fa-solid fa-right-from-bracket"></i></button>
                    </div>
                </header>

                <!-- ইনপুট বার (মেসেজ, লিঙ্ক, ছবি/ভিডিও) -->
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-200 mb-6 space-y-3">
                    <textarea id="postMsg" rows="2" placeholder="কী শেয়ার করতে চান? (মেসেজ বা ইমোজি লিখুন...)"
                        class="w-full p-3 border border-slate-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500 resize-none text-sm"></textarea>
                    
                    <div class="relative">
                        <i class="fa-solid fa-link absolute left-3 top-3 text-slate-400"></i>
                        <input type="url" id="postLink" placeholder="ভিডিও বা যেকোনো লিঙ্ক দিন (YouTube ইত্যাদি)"
                            class="w-full pl-9 pr-3 py-2 border border-slate-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                    </div>

                    <!-- মিডিয়া প্রিভিউ এলাকা -->
                    <div id="mediaPreviewArea" class="${tempMedia.url ? '' : 'hidden'} relative inline-block">
                        <div id="previewContainer"></div>
                        <button onclick="clearMedia()" class="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-6 h-6 flex items-center justify-center text-xs shadow">✕</button>
                    </div>

                    <!-- অ্যাকশন এবং আপলোড বাটন -->
                    <div class="flex justify-between items-center pt-2 border-t border-slate-100 flex-wrap gap-2">
                        <div class="flex items-center space-x-2 flex-wrap">
                            <label class="cursor-pointer bg-slate-100 hover:bg-slate-200 text-slate-600 px-3 py-2 rounded-xl text-xs font-medium flex items-center space-x-1">
                                <i class="fa-solid fa-image text-emerald-500"></i>
                                <span>ছবি/ভিডিও</span>
                                <input type="file" accept="image/*,video/*" class="hidden" onchange="handleFileUpload(event)">
                            </label>
                            <button onclick="addEmoji('😊')" class="bg-slate-100 hover:bg-slate-200 px-2 py-1.5 rounded-lg text-sm">😊</button>
                            <button onclick="addEmoji('🔥')" class="bg-slate-100 hover:bg-slate-200 px-2 py-1.5 rounded-lg text-sm">🔥</button>
                            <button onclick="addEmoji('❤️')" class="bg-slate-100 hover:bg-slate-200 px-2 py-1.5 rounded-lg text-sm">❤️</button>
                        </div>

                        <button onclick="createPost()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 py-2 rounded-xl font-bold text-sm shadow-sm transition">
                            পোস্ট করুন <i class="fa-solid fa-paper-plane ml-1"></i>
                        </button>
                    </div>
                </div>

                <!-- ফিড / পোস্ট লিস্ট -->
                <div id="feedList" class="space-y-4">
                    ${posts.length === 0 ? '<p class="text-center text-slate-400 py-10">এখনো কোনো পোস্ট নেই। প্রথম পোস্টটি আপনিই করুন!</p>' : ''}
                    ${posts.map((post, index) => `
                        <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-200 space-y-3">
                            <div class="flex justify-between items-center">
                                <div class="flex items-center space-x-2">
                                    <div class="w-8 h-8 bg-indigo-100 text-indigo-600 font-bold rounded-full flex items-center justify-center text-xs">
                                        ${post.user[0].toUpperCase()}
                                    </div>
                                    <div>
                                        <span class="font-bold text-slate-800 text-sm block">${post.user}</span>
                                        <span class="text-[10px] text-slate-400">${post.time}</span>
                                    </div>
                                </div>
                                ${post.user === currentUser ? `<button onclick="deletePost(${index})" class="text-slate-300 hover:text-red-500 text-xs"><i class="fa-solid fa-trash"></i></button>` : ''}
                            </div>

                            ${post.msg ? `<p class="text-slate-700 text-sm whitespace-pre-line">${escapeHtml(post.msg)}</p>` : ''}
                            
                            ${post.link ? renderLink(post.link) : ''}
                            
                            ${post.mediaUrl ? (post.mediaType === 'video' ? `<video src="${post.mediaUrl}" controls class="w-full rounded-xl max-h-80 object-cover"></video>` : `<img src="${post.mediaUrl}" class="w-full rounded-xl max-h-80 object-cover">`) : ''}

                            <!-- লাইক ও কমেন্ট সেকশন -->
                            <div class="pt-2 border-t border-slate-100 flex items-center justify-between text-xs text-slate-500">
                                <button onclick="toggleLike(${index})" class="flex items-center space-x-1 font-medium ${post.likes.includes(currentUser) ? 'text-rose-500' : 'hover:text-slate-700'}">
                                    <i class="fa-solid fa-heart"></i>
                                    <span>${post.likes.length} পসন্দ</span>
                                </button>
                                <span><i class="fa-solid fa-comment mr-1"></i>${post.comments.length} কমেন্ট</span>
                            </div>

                            <!-- কমেন্ট লিস্ট -->
                            <div class="bg-slate-50 p-3 rounded-xl space-y-2">
                                <div class="space-y-1">
                                    ${post.comments.map(c => `
                                        <div class="text-xs bg-white p-2 rounded-lg border border-slate-100">
                                            <span class="font-bold text-indigo-600">${c.user}:</span> ${escapeHtml(c.text)}
                                        </div>
                                    `).join('')}
                                </div>
                                <div class="flex space-x-2 pt-1">
                                    <input type="text" id="commentInput_${index}" placeholder="কমেন্ট লিখুন..." 
                                        class="w-full px-3 py-1.5 border border-slate-200 rounded-lg text-xs focus:outline-none focus:ring-1 focus:ring-indigo-500">
                                    <button onclick="addComment(${index})" class="bg-indigo-600 text-white px-3 py-1.5 rounded-lg text-xs font-bold hover:bg-indigo-700">পাঠান</button>
                                </div>
                            </div>
                        </div>
                    `).join('')}
                </div>
            `;
        }

        function handleLogin(e) {
            e.preventDefault();
            const val = document.getElementById('usernameInput').value.trim();
            if (val) {
                currentUser = val;
                localStorage.setItem('ff_user', currentUser);
                render();
            }
        }

        function logout() {
            currentUser = '';
            localStorage.removeItem('ff_user');
            render();
        }

        function addEmoji(emoji) {
            const input = document.getElementById('postMsg');
            input.value += emoji;
            input.focus();
        }

        function handleFileUpload(e) {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(uploadEvent) {
                tempMedia.url = uploadEvent.target.result;
                tempMedia.type = file.type.startsWith('video') ? 'video' : 'image';
                
                const container = document.getElementById('previewContainer');
                if (tempMedia.type === 'video') {
                    container.innerHTML = `<video src="${tempMedia.url}" class="h-20 rounded-lg border"></video>`;
                } else {
                    container.innerHTML = `<img src="${tempMedia.url}" class="h-20 rounded-lg border object-cover">`;
                }
                document.getElementById('mediaPreviewArea').classList.remove('hidden');
            };
            reader.readAsDataURL(file);
        }

        function clearMedia() {
            tempMedia = { url: '', type: '' };
            document.getElementById('mediaPreviewArea').classList.add('hidden');
            document.getElementById('previewContainer').innerHTML = '';
        }

        function createPost() {
            const msg = document.getElementById('postMsg').value.trim();
            const link = document.getElementById('postLink').value.trim();

            if (!msg && !link && !tempMedia.url) {
                alert('দয়া করে কিছু লিখে বা মিডিয়া যুক্ত করে পোস্ট করুন!');
                return;
            }

            const newPost = {
                user: currentUser,
                msg: msg,
                link: link,
                mediaUrl: tempMedia.url,
                mediaType: tempMedia.type,
                time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
                likes: [],
                comments: []
            };

            posts.unshift(newPost);
            saveData();
            tempMedia = { url: '', type: '' };
            render();
        }

        function deletePost(index) {
            if (confirm('আপনি কি পোস্টটি মুছে ফেলতে চান?')) {
                posts.splice(index, 1);
                saveData();
                render();
            }
        }

        function toggleLike(index) {
            const post = posts[index];
            const likeIndex = post.likes.indexOf(currentUser);
            if (likeIndex > -1) {
                post.likes.splice(likeIndex, 1);
            } else {
                post.likes.push(currentUser);
            }
            saveData();
            render();
        }

        function addComment(index) {
            const input = document.getElementById(`commentInput_${index}`);
            const text = input.value.trim();
            if (text) {
                posts[index].comments.push({ user: currentUser, text: text });
                saveData();
                render();
            }
        }

        function renderLink(url) {
            if (url.includes('youtube.com') || url.includes('youtu.be')) {
                let videoId = '';
                if (url.includes('youtu.be/')) {
                    videoId = url.split('youtu.be/')[1].split('?')[0];
                } else if (url.includes('watch?v=')) {
                    videoId = url.split('watch?v=')[1].split('&')[0];
                }
                if (videoId) {
                    return `<div class="aspect-video w-full"><iframe src="https://www.youtube.com/embed/${videoId}" class="w-full h-full rounded-xl" allowfullscreen></iframe></div>`;
                }
            }
            return `<a href="${url}" target="_blank" class="text-indigo-600 text-sm underline block break-all"><i class="fa-solid fa-link mr-1"></i>${url}</a>`;
        }

        function escapeHtml(text) {
            return text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }

        // ইনিশিয়াল রেন্ডার
        render();
    </script>
</body>
</html>
