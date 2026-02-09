<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>東京兩人之旅 2026</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
    <style>
        body { background-color: #FFF5F7; -webkit-tap-highlight-color: transparent; }
        ::-webkit-scrollbar { display: none; }
        .loader { border: 4px solid #f3f3f3; border-top: 4px solid #FF9CA8; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 50px auto; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="text-gray-700">
    <div id="app" class="max-w-md mx-auto min-h-screen pb-20 relative">
        <!-- Loading -->
        <div v-if="loading" class="fixed inset-0 bg-white z-50 flex flex-col items-center justify-center">
            <div class="loader"></div>
            <p class="text-gray-400 text-sm mt-4">連線中...</p>
        </div>

        <!-- Banner -->
        <div class="h-64 relative">
            <img :src="banner" class="w-full h-full object-cover">
            <div class="absolute inset-0 bg-black/30 flex items-end p-6">
                <h1 class="text-3xl font-bold text-white">TOKYO<br>TRIP</h1>
            </div>
        </div>

        <!-- Content -->
        <div class="bg-[#FFF5F7] -mt-6 rounded-t-3xl relative z-10 min-h-[500px] p-4">
            <!-- 行程 -->
            <div v-if="tab==='trip'">
                <div class="flex overflow-x-auto gap-2 mb-4">
                    <button v-for="(d,i) in dates" :key="i" @click="dayIdx=i" :class="['flex-col flex items-center min-w-[60px] p-2 rounded-xl border', dayIdx===i?'bg-[#FF9CA8] text-white':'bg-white']">
                        <span class="font-bold text-lg">{{d.w}}</span><span class="text-xs">{{d.d}}</span>
                    </button>
                </div>
                <div class="space-y-3">
                    <div v-if="currList.length===0" class="text-center py-10 text-gray-400 border-2 border-dashed rounded-xl">尚無行程</div>
                    <div v-for="(item,i) in currList" :key="item.id" class="bg-white p-4 rounded-xl shadow-sm relative">
                        <button @click="del('itinerary', i)" class="absolute top-2 right-2 text-gray-300"><i class="fa-solid fa-trash"></i></button>
                        <div class="flex justify-between font-bold mb-1">
                            <span class="text-xs bg-gray-100 px-2 py-1 rounded text-gray-500">{{item.cat}}</span>
                            <span class="text-[#FF9CA8] font-mono">{{item.time}}</span>
                        </div>
                        <div class="font-bold text-lg">{{item.name}}</div>
                    </div>
                </div>
                <button @click="addModal='itinerary'" class="fixed bottom-6 right-6 w-14 h-14 bg-[#FF9CA8] text-white rounded-full shadow-xl text-2xl flex items-center justify-center"><i class="fa-solid fa-plus"></i></button>
            </div>
            
            <!-- 購物 -->
            <div v-else-if="tab==='shop'" class="grid grid-cols-2 gap-4">
                <div v-for="(item,i) in shopList" :key="i" class="bg-white p-3 rounded-xl shadow-sm text-center relative">
                    <button @click="del('shopping', i)" class="absolute top-1 right-1 text-gray-300"><i class="fa-solid fa-xmark"></i></button>
                    <div class="font-bold">{{item.name}}</div>
                </div>
                <button @click="addModal='shopping'" class="fixed bottom-6 right-6 w-14 h-14 bg-[#FF9CA8] text-white rounded-full shadow-xl text-2xl flex items-center justify-center"><i class="fa-solid fa-plus"></i></button>
            </div>
        </div>

        <!-- Tabs -->
        <div class="fixed bottom-6 left-6 flex gap-2">
            <button @click="tab='trip'" :class="['w-12 h-12 rounded-full shadow-lg border-2 flex items-center justify-center', tab==='trip'?'bg-[#FF9CA8] text-white border-white':'bg-white text-[#FF9CA8]']"><i class="fa-regular fa-calendar-days"></i></button>
            <button @click="tab='shop'" :class="['w-12 h-12 rounded-full shadow-lg border-2 flex items-center justify-center', tab==='shop'?'bg-[#FF9CA8] text-white border-white':'bg-white text-[#FF9CA8]']"><i class="fa-solid fa-basket-shopping"></i></button>
        </div>

        <!-- Modal -->
        <div v-if="addModal" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center" @click.self="addModal=null">
            <div class="bg-white p-6 rounded-2xl w-[80%]">
                <h3 class="font-bold text-center mb-4">新增</h3>
                <div v-if="addModal==='itinerary'" class="space-y-3">
                    <input v-model="form.time" type="time" class="w-full bg-gray-50 p-2 rounded">
                    <input v-model="form.name" type="text" placeholder="名稱" class="w-full bg-gray-50 p-2 rounded">
                </div>
                <div v-if="addModal==='shopping'" class="space-y-3">
                    <input v-model="form.name" type="text" placeholder="商品名稱" class="w-full bg-gray-50 p-2 rounded">
                </div>
                <button @click="save" class="w-full bg-[#FF9CA8] text-white p-3 rounded-xl mt-4 font-bold">儲存</button>
            </div>
        </div>
    </div>

    <script>
        // ★★★ 請確認這裡填入正確的 Key ★★★
        const firebaseConfig = {
            apiKey: "AIzaSyDic5wPcadyif7tvclKRzpzZ8q_bJKgptI",
            authDomain: "tokyo-20502.firebaseapp.com",
            projectId: "tokyo-20502",
            storageBucket: "tokyo-20502.firebasestorage.app",
            messagingSenderId: "499120725183",
            appId: "1:499120725183:web:6d6caba2ac82c81e723a0a",
            measurementId: "G-29RJZM6KEG",
            // ★ 這行是修正後的網址
            databaseURL: "https://tokyo-20502-default-rtdb.asia-southeast1.firebasedatabase.app"
        };

        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        Vue.createApp({
            setup() {
                const loading = Vue.ref(true);
                const tab = Vue.ref('trip');
                const banner = Vue.ref('https://images.unsplash.com/photo-1540959733332-eab4deabeeaf?q=80&w=600&auto=format&fit=crop');
                const dayIdx = Vue.ref(0);
                const dates = [{d:'2/26',w:'Thu',k:'2026-02-26'},{d:'2/27',w:'Fri',k:'2026-02-27'},{d:'2/28',w:'Sat',k:'2026-02-28'},{d:'3/1',w:'Sun',k:'2026-03-01'},{d:'3/2',w:'Mon',k:'2026-03-02'},{d:'3/3',w:'Tue',k:'2026-03-03'}];
                
                const trips = Vue.reactive({});
                const shops = Vue.ref([]);
                const addModal = Vue.ref(null);
                const form = Vue.reactive({time:'10:00',name:'',cat:'景點'});

                Vue.onMounted(() => {
                    db.ref('itinerary').on('value', s => {
                        const v = s.val();
                        if(v) { Object.keys(trips).forEach(k=>delete trips[k]); Object.assign(trips, v); }
                        loading.value = false;
                    });
                    db.ref('shopping').on('value', s => { shops.value = s.val() || []; });
                });

                const sync = () => { db.ref('itinerary').set(trips); db.ref('shopping').set(shops.value); };
                
                const currList = Vue.computed(() => {
                    const k = dates[dayIdx.value].k;
                    return trips[k] || [];
                });

                const save = () => {
                    if(addModal.value === 'itinerary') {
                        const k = dates[dayIdx.value].k;
                        if(!trips[k]) trips[k] = [];
                        trips[k].push({id:Date.now(), ...form});
                        trips[k].sort((a,b)=>a.time.localeCompare(b.time));
                    } else {
                        shops.value.push({name:form.name});
                    }
                    sync();
                    addModal.value = null;
                };

                const del = (type, i) => {
                    if(!confirm('刪除?')) return;
                    if(type==='itinerary') trips[dates[dayIdx.value].k].splice(i,1);
                    else shops.value.splice(i,1);
                    sync();
                };

                return { loading, tab, banner, dates, dayIdx, currList, shopList:shops, addModal, form, save, del };
            }
        }).mount('#app');
    </script>
</body>
</html>
