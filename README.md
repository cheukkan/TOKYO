<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>東京兩人之旅 2026 (完整雲端版)</title>
    
    <!-- 1. 核心工具 -->
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Zen+Maru+Gothic:wght@400;500;700&family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- 2. Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>

    <!-- 3. 樣式設定 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'tokyo-pink': '#FF9CA8',
                        'tokyo-bg': '#FFF5F7',
                        'tokyo-dark': '#4A4A4A',
                        'tokyo-blue': '#89CFF0',
                    },
                    fontFamily: { sans: ['"Zen Maru Gothic"', 'sans-serif'] }
                }
            }
        }
    </script>
    <style>
        body { background-color: #FFF5F7; -webkit-tap-highlight-color: transparent; }
        ::-webkit-scrollbar { display: none; }
        .app-container { max-width: 480px; margin: 0 auto; min-height: 100vh; padding-bottom: 90px; position: relative; }
        .fade-enter-active, .fade-leave-active { transition: opacity 0.2s ease; }
        .fade-enter-from, .fade-leave-to { opacity: 0; }
        .loader { border: 4px solid #f3f3f3; border-top: 4px solid #FF9CA8; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 50px auto; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="text-tokyo-dark antialiased">

    <div id="app" class="app-container">
        
        <!-- Loading -->
        <div v-if="loading" class="fixed inset-0 bg-white z-50 flex flex-col items-center justify-center">
            <div class="loader"></div>
            <p class="text-gray-400 text-sm mt-4">正在同步雲端資料...</p>
        </div>

        <!-- 頁首 Banner -->
        <header class="relative w-full h-[280px] z-0">
            <img :src="bannerImage" class="w-full h-full object-cover">
            <div class="absolute inset-0 bg-gradient-to-t from-black/50 to-transparent"></div>
            <div class="absolute top-8 left-6 text-white drop-shadow-md">
                <h1 class="text-3xl font-bold tracking-wider">TOKYO<br>TRIP</h1>
                <p class="text-sm opacity-90 mt-1">2026.02.26 - 03.03</p>
            </div>
            <!-- Tabs -->
            <div class="absolute bottom-12 right-4 flex gap-3 z-30">
                <button v-for="tab in tabs" :key="tab.id" @click="currentTab = tab.id"
                    :class="['w-12 h-12 rounded-full flex items-center justify-center shadow-lg transition-all border-2 backdrop-blur-md',
                        currentTab === tab.id ? 'bg-tokyo-pink border-white text-white scale-110' : 'bg-white/80 border-white/50 text-tokyo-pink']">
                    <i :class="tab.icon" class="text-lg"></i>
                </button>
            </div>
        </header>

        <!-- 主內容區 -->
        <main class="relative -mt-8 rounded-t-[40px] bg-tokyo-bg min-h-[calc(100vh-250px)] z-10 pt-4 shadow-[0_-5px_20px_rgba(0,0,0,0.05)]">
            <div class="w-12 h-1.5 bg-tokyo-pink/30 rounded-full mx-auto mb-4"></div>

            <transition name="fade" mode="out-in">
                
                <!-- 1. 每日行程 -->
                <div v-if="currentTab === 'itinerary'" key="itinerary">
                    <!-- 日期選單 -->
                    <div class="flex overflow-x-auto gap-2 px-4 pb-2 sticky top-0 bg-tokyo-bg/95 backdrop-blur z-20">
                        <button v-for="(day, idx) in tripDates" :key="idx" @click="currentDayIndex = idx"
                            :class="['flex flex-col items-center min-w-[65px] py-2 rounded-2xl border transition',
                                currentDayIndex === idx ? 'bg-tokyo-pink border-tokyo-pink text-white shadow-md scale-105' : 'bg-white border-white text-gray-400']">
                            <span class="text-lg font-bold">{{ day.week }}</span>
                            <span class="text-xs">{{ day.dateDisplay }}</span>
                        </button>
                    </div>

                    <!-- 天氣 Widget -->
                    <div class="px-4 my-3">
                        <div class="bg-gradient-to-r from-blue-50 to-white rounded-2xl p-3 flex justify-between items-center shadow-sm border border-blue-50">
                            <div class="flex items-center gap-3">
                                <i :class="currentWeather.icon" class="text-2xl text-tokyo-blue"></i>
                                <div>
                                    <div class="font-bold">{{ currentWeather.temp }}°C</div>
                                    <div class="text-[10px] text-gray-400">體感 {{ currentWeather.feelsLike }}°C</div>
                                </div>
                            </div>
                            <div class="text-right text-xs text-gray-500">
                                <div class="font-bold">東京</div>
                                <div>{{ currentWeather.desc }}</div>
                            </div>
                        </div>
                    </div>

                    <!-- 行程列表 -->
                    <div class="px-4 pb-24 space-y-2">
                        <div v-if="currentDayItinerary.length === 0" class="text-center py-10 text-gray-400 border-2 border-dashed border-gray-200 rounded-3xl mt-4">
                            <i class="fa-regular fa-calendar-plus text-2xl mb-2"></i>
                            <p class="text-sm">點擊 + 新增行程</p>
                        </div>

                        <div v-for="(item, idx) in currentDayItinerary" :key="item.id" class="relative group">
                             <!-- 連接線與交通 -->
                             <div class="flex justify-center items-center py-2 gap-2 opacity-70 relative z-0">
                                <div v-if="idx === 0" class="flex items-center gap-1 text-[10px] text-gray-400 bg-white/50 px-2 py-0.5 rounded-full">
                                    <i class="fa-solid fa-bed"></i> <span>從飯店出發</span>
                                </div>
                                <div v-if="idx > 0" class="flex items-center gap-2 text-[10px] text-gray-500">
                                    <i :class="getTransportIcon(item.transportType)"></i> <span>{{ item.transportTime }} min</span>
                                </div>
                                <div class="h-[20px] w-0.5 bg-gray-300 absolute -top-2 left-1/2 -z-10"></div>
                            </div>

                            <!-- 卡片本體 -->
                            <div class="bg-white p-4 rounded-2xl shadow-sm border border-gray-50 relative overflow-hidden" 
                                 :class="{'border-l-4 border-l-blue-400 bg-blue-50/30': item.category === 'flight'}"
                                 @click="editItem(item, idx)">
                                
                                <button @click.stop="deleteItem('itinerary', idx)" class="absolute top-2 right-2 text-gray-300 hover:text-red-400 w-8 h-8 z-20"><i class="fa-solid fa-trash"></i></button>

                                <!-- 航班樣式 -->
                                <div v-if="item.category === 'flight'" class="flex flex-col gap-1">
                                    <div class="flex justify-between items-center text-blue-500 mb-1">
                                        <span class="text-[10px] font-bold border border-blue-200 px-2 py-0.5 rounded bg-white">FLIGHT</span>
                                        <i class="fa-solid fa-plane"></i>
                                    </div>
                                    <h3 class="font-bold text-lg text-slate-700">{{ item.name }}</h3>
                                    <p class="text-xs text-gray-500 font-mono">{{ item.time }} | {{ item.notes }}</p>
                                </div>

                                <!-- 一般行程 -->
                                <div v-else>
                                    <div class="flex justify-between items-start mb-1">
                                        <span class="text-[10px] px-2 py-1 rounded bg-gray-100 text-gray-500 font-bold">{{ getCategoryLabel(item.category) }}</span>
                                        <span class="font-mono font-bold text-tokyo-pink text-lg">{{ item.time }}</span>
                                    </div>
                                    <h3 class="font-bold text-lg mb-1">{{ item.name }}</h3>
                                    <div class="text-xs text-gray-400 flex flex-wrap gap-2 mb-1">
                                        <span v-if="item.openHours"><i class="fa-regular fa-clock mr-1"></i>{{ item.openHours }}</span>
                                        <span v-if="item.ticket"><i class="fa-solid fa-ticket mr-1 text-yellow-500"></i>{{ item.ticket }}</span>
                                    </div>
                                    <p v-if="item.notes" class="text-xs text-gray-600 bg-gray-50 p-2 rounded mt-1">{{ item.notes }}</p>
                                </div>

                                <button @click.stop="openMap(item.name)" class="absolute bottom-3 right-3 w-8 h-8 bg-tokyo-pink text-white rounded-full shadow-md z-20"><i class="fa-solid fa-location-arrow text-xs"></i></button>
                            </div>
                        </div>
                    </div>
                    <button @click="openAddModal('itinerary')" class="fixed bottom-6 right-6 w-14 h-14 bg-tokyo-pink text-white rounded-full shadow-xl flex items-center justify-center text-2xl z-40 hover:scale-105 transition"><i class="fa-solid fa-plus"></i></button>
                </div>

                <!-- 2. 資訊頁 (匯率、緊急) -->
                <div v-else-if="currentTab === 'info'" key="info" class="px-4 pb-20 space-y-5">
                    <!-- 匯率 -->
                    <div class="bg-white p-5 rounded-3xl shadow-sm">
                        <h3 class="font-bold text-gray-700 mb-3 flex items-center"><i class="fa-solid fa-coins mr-2 text-yellow-500"></i>匯率換算</h3>
                        <div class="flex items-center gap-3">
                            <div class="flex-1">
                                <label class="text-[10px] text-gray-400 font-bold ml-1">日幣 JPY</label>
                                <input type="number" v-model="calcJpy" @input="calculateHome" class="w-full bg-gray-50 rounded-xl p-3 font-mono font-bold outline-none focus:ring-2 ring-tokyo-pink/30 text-lg">
                            </div>
                            <i class="fa-solid fa-arrow-right-arrow-left text-gray-300 mt-4"></i>
                            <div class="flex-1">
                                <label class="text-[10px] text-gray-400 font-bold ml-1">港幣 HKD</label>
                                <input type="number" v-model="calcHome" @input="calculateJpy" class="w-full bg-gray-50 rounded-xl p-3 font-mono font-bold outline-none focus:ring-2 ring-tokyo-pink/30 text-lg">
                            </div>
                        </div>
                    </div>

                    <!-- 飯店 -->
                    <div class="bg-white rounded-3xl p-5 shadow-sm relative">
                        <div class="flex items-start gap-3">
                            <div class="w-10 h-10 rounded-full bg-purple-50 flex items-center justify-center text-purple-500 shrink-0"><i class="fa-solid fa-hotel"></i></div>
                            <div>
                                <h3 class="font-bold text-gray-700">六本木Act酒店</h3>
                                <p class="text-xs text-gray-400">2/26 - 3/3 (5 晚)</p>
                            </div>
                        </div>
                        <button @click="openMap('六本木Act酒店')" class="w-full bg-gray-50 py-2 rounded-xl text-gray-600 font-bold mt-3 text-sm"><i class="fa-solid fa-location-arrow mr-1 text-tokyo-pink"></i> 導航</button>
                    </div>

                    <!-- 緊急 -->
                    <div class="grid grid-cols-2 gap-3">
                        <div class="bg-white p-4 rounded-2xl shadow-sm text-center" @click="openMap('成田機場租車')">
                            <i class="fa-solid fa-car text-2xl text-green-500 mb-1"></i>
                            <div class="font-bold text-sm">租車導航</div>
                        </div>
                        <div class="bg-red-50 p-4 rounded-2xl shadow-sm text-center">
                            <i class="fa-solid fa-briefcase-medical text-2xl text-red-500 mb-1"></i>
                            <div class="font-bold text-sm">緊急 119/110</div>
                        </div>
                    </div>
                </div>

                <!-- 3. 購物清單 -->
                <div v-else-if="currentTab === 'shopping'" key="shopping" class="px-4 pb-24">
                    <div class="grid grid-cols-2 gap-4">
                        <div v-for="(item, idx) in shoppingList" :key="idx" class="bg-white rounded-2xl p-3 shadow-sm flex flex-col items-center relative group">
                            <button @click="deleteItem('shopping', idx)" class="absolute top-1 right-1 w-6 h-6 text-gray-300 hover:text-red-400 z-10"><i class="fa-solid fa-xmark"></i></button>
                            <div class="w-full aspect-square bg-gray-50 rounded-xl mb-2 flex items-center justify-center text-gray-200 overflow-hidden border border-gray-100">
                                <img v-if="item.image" :src="item.image" class="w-full h-full object-cover">
                                <i v-else class="fa-solid fa-image text-3xl"></i>
                            </div>
                            <div class="font-bold text-sm text-center mb-1 line-clamp-1">{{ item.name }}</div>
                            <label class="inline-flex items-center gap-2">
                                <input type="checkbox" v-model="item.checked" @change="syncData" class="w-5 h-5 accent-tokyo-pink">
                                <span class="text-xs text-gray-400">{{ item.checked ? '已買' : '未買' }}</span>
                            </label>
                        </div>
                    </div>
                    <button @click="openAddModal('shopping')" class="fixed bottom-6 right-6 w-14 h-14 bg-tokyo-pink text-white rounded-full shadow-xl flex items-center justify-center text-2xl z-40 hover:scale-105 transition"><i class="fa-solid fa-plus"></i></button>
                </div>

                <!-- 4. 花費 -->
                <div v-else-if="currentTab === 'expenses'" key="expenses" class="px-4 pb-24">
                    <div class="bg-gradient-to-br from-slate-700 to-slate-800 rounded-3xl p-6 text-white mb-5 shadow-lg relative overflow-hidden">
                        <div class="absolute -right-5 -top-5 w-32 h-32 bg-white/10 rounded-full blur-2xl"></div>
                        <div class="text-xs opacity-70 mb-1 font-bold tracking-wider">總花費 (JPY)</div>
                        <div class="text-4xl font-bold font-mono">¥ {{ totalExpenses.toLocaleString() }}</div>
                        <div class="mt-3 flex gap-2 text-[10px]">
                            <span class="px-2 py-1 bg-white/10 rounded-full">現金 ¥{{ totalCash.toLocaleString() }}</span>
                            <span class="px-2 py-1 bg-white/10 rounded-full">刷卡 ¥{{ totalCard.toLocaleString() }}</span>
                        </div>
                    </div>
                    <div class="space-y-3">
                        <div v-for="(exp, idx) in expensesList" :key="idx" class="bg-white p-4 rounded-2xl shadow-sm flex justify-between items-center border border-gray-50 relative">
                             <button @click="deleteItem('expenses', idx)" class="absolute top-1 right-1 w-5 h-5 text-gray-200 hover:text-red-400"><i class="fa-solid fa-xmark text-xs"></i></button>
                             <div class="flex items-center gap-3">
                                <div class="w-10 h-10 rounded-full bg-gray-50 flex items-center justify-center text-gray-400"><i :class="getExpenseIcon(exp.category)"></i></div>
                                <div>
                                    <div class="font-bold text-sm text-gray-700">{{ exp.name }}</div>
                                    <div class="text-[10px] text-gray-400">{{ exp.date }} · {{ exp.payment }}</div>
                                </div>
                             </div>
                             <div class="font-bold font-mono text-gray-700">¥{{ exp.amount }}</div>
                        </div>
                    </div>
                    <button @click="openAddModal('expenses')" class="fixed bottom-6 right-6 w-14 h-14 bg-tokyo-pink text-white rounded-full shadow-xl flex items-center justify-center text-2xl z-40 hover:scale-105 transition"><i class="fa-solid fa-plus"></i></button>
                </div>

            </transition>
        </main>

        <!-- 彈出視窗 (Modal) -->
        <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/40 backdrop-blur-sm" @click.self="showModal = false">
            <div class="bg-white w-[90%] max-w-[400px] rounded-3xl p-6 shadow-2xl max-h-[90vh] overflow-y-auto">
                <h2 class="text-xl font-bold mb-4 text-center text-gray-700">{{ isEditing ? '編輯' : '新增' }}{{ getModalTitle }}</h2>
                
                <!-- 行程表單 -->
                <div v-if="modalType === 'itinerary'" class="space-y-3">
                    <div class="grid grid-cols-2 gap-3">
                        <div>
                            <label class="text-xs text-gray-400 font-bold ml-1">時間</label>
                            <input type="time" v-model="form.time" class="w-full bg-gray-50 p-3 rounded-xl text-sm font-bold">
                        </div>
                        <div>
                            <label class="text-xs text-gray-400 font-bold ml-1">分類</label>
                            <select v-model="form.category" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                                <option value="sightseeing">📷 景點</option>
                                <option value="food">🍴 美食</option>
                                <option value="shopping">🛍️ 購物</option>
                                <option value="hotel">🏨 住宿</option>
                                <option value="transport">🚄 交通</option>
                                <option value="flight">✈️ 航班</option>
                            </select>
                        </div>
                    </div>
                    <input type="text" v-model="form.name" placeholder="名稱 (例如: 淺草寺)" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                    <div class="grid grid-cols-2 gap-3">
                        <input type="text" v-model="form.openHours" placeholder="營業時間" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                        <input type="text" v-model="form.ticket" placeholder="門票" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                    </div>
                    <div class="bg-blue-50/50 p-3 rounded-xl border border-blue-50 flex gap-2 items-center">
                        <label class="text-[10px] text-blue-400 font-bold shrink-0">交通(分)</label>
                        <select v-model="form.transportType" class="bg-white p-2 rounded-lg text-xs outline-none"><option value="train">地鐵</option><option value="walk">步行</option><option value="drive">車</option></select>
                        <input type="number" v-model="form.transportTime" class="flex-1 bg-white p-2 rounded-lg text-sm text-center outline-none">
                    </div>
                    <textarea v-model="form.notes" placeholder="備註..." rows="2" class="w-full bg-gray-50 p-3 rounded-xl text-sm"></textarea>
                </div>

                <!-- 購物表單 -->
                <div v-if="modalType === 'shopping'" class="space-y-3">
                    <input type="text" v-model="form.name" placeholder="商品名稱" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                    <input type="text" v-model="form.image" placeholder="圖片網址 (https://...)" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                    <p class="text-[10px] text-gray-400">*建議使用網路圖片連結</p>
                </div>

                <!-- 花費表單 -->
                <div v-if="modalType === 'expenses'" class="space-y-3">
                    <div class="grid grid-cols-2 gap-3">
                         <select v-model="form.category" class="w-full bg-gray-50 p-3 rounded-xl text-sm"><option value="food">飲食</option><option value="transport">交通</option><option value="shopping">購物</option><option value="stay">住宿</option><option value="other">其他</option></select>
                         <input type="number" v-model="form.amount" placeholder="金額 (JPY)" class="w-full bg-gray-50 p-3 rounded-xl text-sm font-bold">
                    </div>
                    <input type="text" v-model="form.name" placeholder="項目名稱" class="w-full bg-gray-50 p-3 rounded-xl text-sm">
                    <div class="flex gap-4 mt-2 justify-center">
                         <label class="flex items-center"><input type="radio" value="現金" v-model="form.payment" class="mr-2 accent-tokyo-pink">現金</label>
                         <label class="flex items-center"><input type="radio" value="信用卡" v-model="form.payment" class="mr-2 accent-tokyo-pink">信用卡</label>
                    </div>
                </div>

                <button @click="saveItem" class="w-full bg-tokyo-pink text-white py-3 rounded-xl font-bold mt-6 shadow-lg hover:opacity-90 transition">確認儲存</button>
            </div>
        </div>

    </div>

    <script>
        // ============================================
        // ★★★ Firebase 設定 (asia-southeast1) ★★★
        // ============================================
        const firebaseConfig = {
            apiKey: "AIzaSyDic5wPcadyif7tvclKRzpzZ8q_bJKgptI",
            authDomain: "tokyo-20502.firebaseapp.com",
            projectId: "tokyo-20502",
            storageBucket: "tokyo-20502.firebasestorage.app",
            messagingSenderId: "499120725183",
            appId: "1:499120725183:web:6d6caba2ac82c81e723a0a",
            measurementId: "G-29RJZM6KEG",
            databaseURL: "https://tokyo-20502-default-rtdb.asia-southeast1.firebasedatabase.app"
        };

        try { firebase.initializeApp(firebaseConfig); } catch(e){ console.error(e); }
        const db = firebase.database();

        const { createApp, ref, reactive, computed, onMounted } = Vue;

        createApp({
            setup() {
                const loading = ref(true);
                const bannerImage = ref('https://images.unsplash.com/photo-1540959733332-eab4deabeeaf?q=80&w=1000&auto=format&fit=crop');
                const currentTab = ref('itinerary');
                const tabs = [
                    { id: 'itinerary', icon: 'fa-regular fa-calendar-days' },
                    { id: 'info', icon: 'fa-solid fa-circle-info' },
                    { id: 'shopping', icon: 'fa-solid fa-basket-shopping' },
                    { id: 'expenses', icon: 'fa-solid fa-yen-sign' },
                ];
                const tripDates = [
                    { fullDate: '2026-02-26', dateDisplay: '2/26', week: 'Thu' },
                    { fullDate: '2026-02-27', dateDisplay: '2/27', week: 'Fri' },
                    { fullDate: '2026-02-28', dateDisplay: '2/28', week: 'Sat' },
                    { fullDate: '2026-03-01', dateDisplay: '3/1', week: 'Sun' },
                    { fullDate: '2026-03-02', dateDisplay: '3/2', week: 'Mon' },
                    { fullDate: '2026-03-03', dateDisplay: '3/3', week: 'Tue' },
                ];
                const currentDayIndex = ref(0);
                const weatherMocks = [{ temp: 8, feelsLike: 5, icon: 'fa-solid fa-sun text-orange-400', desc: '晴朗' },{ temp: 6, feelsLike: 3, icon: 'fa-solid fa-cloud text-slate-400', desc: '多雲' },{ temp: 5, feelsLike: 2, icon: 'fa-solid fa-cloud-rain text-blue-400', desc: '小雨' },{ temp: 9, feelsLike: 7, icon: 'fa-solid fa-cloud-sun text-yellow-500', desc: '晴時多雲' },{ temp: 4, feelsLike: 0, icon: 'fa-solid fa-snowflake text-blue-200', desc: '微雪' },{ temp: 10, feelsLike: 8, icon: 'fa-solid fa-sun text-orange-400', desc: '晴朗' }];
                const currentWeather = computed(() => weatherMocks[currentDayIndex.value] || weatherMocks[0]);

                // Data
                const itineraryData = reactive({});
                const shoppingList = ref([]);
                const expensesList = ref([]);

                // Sync
                onMounted(() => {
                    db.ref('itinerary').on('value', s => {
                        const v = s.val();
                        if(v) { Object.keys(itineraryData).forEach(k=>delete itineraryData[k]); Object.assign(itineraryData, v); }
                        loading.value = false;
                    });
                    db.ref('shopping').on('value', s => shoppingList.value = s.val() || []);
                    db.ref('expenses').on('value', s => expensesList.value = s.val() || []);
                });

                const syncData = () => {
                    db.ref('itinerary').set(itineraryData);
                    db.ref('shopping').set(shoppingList.value);
                    db.ref('expenses').set(expensesList.value);
                };

                const currentDayItinerary = computed(() => {
                    const key = tripDates[currentDayIndex.value].fullDate;
                    return itineraryData[key] || [];
                });

                // Expenses Logic
                const totalExpenses = computed(() => expensesList.value.reduce((sum, i) => sum + parseInt(i.amount||0), 0));
                const totalCash = computed(() => expensesList.value.filter(i=>i.payment==='現金').reduce((sum, i) => sum + parseInt(i.amount||0), 0));
                const totalCard = computed(() => expensesList.value.filter(i=>i.payment==='信用卡').reduce((sum, i) => sum + parseInt(i.amount||0), 0));

                // Modal
                const showModal = ref(false);
                const modalType = ref('');
                const isEditing = ref(false);
                const editingId = ref(null);
                const form = reactive({ category: 'sightseeing', time: '10:00', name: '', notes: '', image: '', checked: false, openHours:'', ticket:'', transportTime:15, transportType:'train', amount:0, payment:'現金', date:'' });

                const openAddModal = (type) => {
                    modalType.value = type;
                    isEditing.value = false;
                    showModal.value = true;
                    // Reset Form
                    Object.assign(form, { category: (type==='expenses'?'food':'sightseeing'), time: '10:00', name: '', notes: '', image: '', checked: false, openHours:'', ticket:'', transportTime:15, transportType:'train', amount:'', payment:'現金', date: tripDates[currentDayIndex.value].dateDisplay });
                };

                const editItem = (item, idx) => {
                    if (currentTab.value !== 'itinerary') return;
                    modalType.value = 'itinerary';
                    isEditing.value = true;
                    editingId.value = idx;
                    showModal.value = true;
                    Object.assign(form, item);
                };

                const saveItem = () => {
                    if (modalType.value === 'itinerary') {
                        const key = tripDates[currentDayIndex.value].fullDate;
                        if (!itineraryData[key]) itineraryData[key] = [];
                        const newItem = { id: Date.now(), ...form };
                        if (isEditing.value) itineraryData[key][editingId.value] = newItem;
                        else {
                            itineraryData[key].push(newItem);
                            itineraryData[key].sort((a, b) => a.time.localeCompare(b.time));
                        }
                    } else if (modalType.value === 'shopping') {
                        shoppingList.value.push({ ...form });
                    } else if (modalType.value === 'expenses') {
                        expensesList.value.unshift({ ...form, date: tripDates[currentDayIndex.value].dateDisplay });
                    }
                    syncData();
                    showModal.value = false;
                };

                const deleteItem = (type, idx) => {
                    if(!confirm('確定刪除？')) return;
                    if (type === 'itinerary') itineraryData[tripDates[currentDayIndex.value].fullDate].splice(idx, 1);
                    else if (type === 'shopping') shoppingList.value.splice(idx, 1);
                    else if (type === 'expenses') expensesList.value.splice(idx, 1);
                    syncData();
                };

                // Currency
                const exchangeRate = 0.052; 
                const calcJpy = ref(1000);
                const calcHome = ref(52);
                const calculateHome = () => { calcHome.value = (calcJpy.value * exchangeRate).toFixed(1); };
                const calculateJpy = () => { calcJpy.value = Math.round(calcHome.value / exchangeRate); };

                // Helpers
                const getCategoryLabel = (cat) => ({ sightseeing: '景點', food: '美食', shopping: '購物', transport: '交通', flight: '航班' }[cat] || cat);
                const getTransportIcon = (type) => ({ walk: 'fa-solid fa-person-walking', drive: 'fa-solid fa-car', train: 'fa-solid fa-train-subway' }[type] || 'fa-solid fa-arrow-right');
                const getExpenseIcon = (cat) => ({ food: 'fa-utensils', transport: 'fa-train', shopping: 'fa-bag-shopping', stay: 'fa-bed', other: 'fa-circle-question' }[cat] || 'fa-circle');
                const openMap = (loc) => window.open(`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(loc + ' 東京')}`, '_blank');
                const getModalTitle = computed(() => ({ itinerary: '行程', shopping: '購物', expenses: '花費' }[modalType.value]));

                return {
                    loading, bannerImage, tabs, currentTab, tripDates, currentDayIndex, currentDayItinerary, shoppingList, expensesList,
                    currentWeather, calcJpy, calcHome, calculateHome, calculateJpy,
                    showModal, modalType, isEditing, form, getModalTitle, totalExpenses, totalCash, totalCard,
                    openAddModal, editItem, saveItem, deleteItem, getCategoryLabel, getTransportIcon, getExpenseIcon, openMap, syncData
                };
            }
        }).mount('#app');
    </script>
</body>
</html>
