<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>動態財報分析報告</title>
    <!-- 引入 Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入 Chart.js 以繪製圖表 -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- 引入 FontAwesome 圖示 -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&display=swap');
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #f4f6fa;
            color: #1a202c;
        }
        /* 自定義滾動條 */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        
        /* 載入動畫遮罩 */
        #loading-overlay {
            backdrop-filter: blur(5px);
            transition: opacity 0.3s ease;
        }
    </style>
</head>
<body class="antialiased min-h-screen py-8 px-4 sm:px-6 relative">

    <!-- 全螢幕載入遮罩 (預設隱藏) -->
    <div id="loading-overlay" class="fixed inset-0 bg-white/80 z-50 hidden flex-col items-center justify-center">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-indigo-600 mb-4"></div>
        <h2 class="text-xl font-bold text-indigo-900 tracking-wider">正在解析財報數據...</h2>
        <p class="text-gray-500 mt-2 text-sm" id="loading-text">獲取最新資料中</p>
    </div>

    <!-- 主容器 -->
    <div class="max-w-6xl mx-auto space-y-6">

        <!-- 頂部導覽列與美股搜尋 -->
        <div class="flex flex-col sm:flex-row justify-between items-center gap-4 mb-2 bg-white p-4 rounded-2xl shadow-sm border border-gray-100">
            <div class="font-black text-xl tracking-wider text-[#1a2347]">
                Stock<span class="text-indigo-500">Radar</span>
            </div>
            <div class="relative w-full sm:w-96 flex">
                <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                    <i class="fa-solid fa-magnifying-glass text-gray-400"></i>
                </div>
                <input type="text" id="searchInput" class="block w-full p-2.5 pl-10 text-sm text-gray-900 border border-gray-300 rounded-l-lg bg-gray-50 focus:ring-indigo-500 focus:border-indigo-500 outline-none uppercase" placeholder="輸入股票代號 (如: GEV, NVDA, TSLA)..." onkeypress="handleSearch(event)">
                <button onclick="executeSearch()" class="px-5 py-2.5 text-sm font-medium text-white bg-indigo-600 rounded-r-lg border border-indigo-600 hover:bg-indigo-700 focus:ring-4 focus:outline-none focus:ring-indigo-300 transition flex items-center">
                    解析
                </button>
            </div>
        </div>

        <!-- 頂部深色 Header 區域 -->
        <header class="bg-gradient-to-br from-[#1a2347] to-[#2b448b] rounded-[2rem] p-8 sm:p-10 text-white shadow-lg relative overflow-hidden transition-all duration-500" id="header-bg">
            <div class="relative z-10">
                <p class="text-indigo-200 text-sm font-bold tracking-widest mb-3 uppercase" id="report-tag">—— GEV Data Report ——</p>
                <h1 class="text-3xl sm:text-4xl lg:text-5xl font-extrabold mb-4 leading-tight">
                    <span id="stock-name">GE Vernova</span> <span class="font-light text-indigo-200 mx-2">|</span> 最新財報數據表、圖表與供應鏈地圖
                </h1>
                <p class="text-indigo-100 text-lg sm:text-xl mb-8 opacity-90" id="report-subtitle">
                    不是只看有沒有成長，而是看這份成長有沒有品質、有沒有延續性。
                </p>
                
                <!-- 總結高光區塊 -->
                <div class="bg-white/10 border border-white/20 rounded-2xl p-5 sm:p-6 backdrop-blur-sm max-w-4xl">
                    <p class="text-white text-base sm:text-lg leading-relaxed font-medium">
                        <span class="font-bold text-indigo-200 mr-1">一句話先看：</span><span id="highlight-text">這季真正的亮點是訂單大增、在手單夠厚、現金流強、全年財測上修，所以這不是只有單季好看，而是後面需求能見度也跟著變強。</span>
                    </p>
                </div>
            </div>
            <!-- 背景裝飾光暈 -->
            <div class="absolute -bottom-24 -right-24 w-96 h-96 bg-blue-500 rounded-full mix-blend-multiply filter blur-[100px] opacity-50"></div>
        </header>

        <!-- 數據卡片區域 (Grid) -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- 左側 6 個小卡 -->
            <div class="lg:col-span-2 grid grid-cols-2 md:grid-cols-3 gap-4">
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-gray-100 flex flex-col justify-between">
                    <p class="text-sm text-gray-500 mb-1">營收 Revenue</p>
                    <p class="text-2xl font-black text-gray-800 mb-1" id="card-rev">93 億美元</p>
                    <p class="text-xs text-gray-400" id="card-rev-yoy">YoY +16%</p>
                </div>
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-gray-100 flex flex-col justify-between">
                    <p class="text-sm text-gray-500 mb-1" id="card-orders-title">訂單 Orders</p>
                    <p class="text-2xl font-black text-gray-800 mb-1" id="card-orders">183 億美元</p>
                    <p class="text-xs text-gray-400" id="card-orders-yoy">有機成長 +71%</p>
                </div>
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-gray-100 flex flex-col justify-between">
                    <p class="text-sm text-gray-500 mb-1" id="card-margin-title">在手訂單</p>
                    <p class="text-2xl font-black text-gray-800 mb-1" id="card-margin">1630 億美元</p>
                    <p class="text-xs text-gray-400" id="card-margin-desc">未來能見度高</p>
                </div>
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-gray-100 flex flex-col justify-between">
                    <p class="text-sm text-gray-500 mb-1" id="card-net-title">Adj. EBITDA</p>
                    <p class="text-2xl font-black text-gray-800 mb-1" id="card-net">9 億美元</p>
                    <p class="text-xs text-gray-400" id="card-net-desc">Margin 9.6%</p>
                </div>
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-gray-100 flex flex-col justify-between">
                    <p class="text-sm text-gray-500 mb-1">自由現金流</p>
                    <p class="text-2xl font-black text-gray-800 mb-1" id="card-fcf">48 億美元</p>
                    <p class="text-xs text-gray-400">現金品質強</p>
                </div>
                <div class="bg-white rounded-2xl p-5 shadow-sm border border-gray-100 flex flex-col justify-between">
                    <p class="text-sm text-gray-500 mb-1">全年展望</p>
                    <p class="text-2xl font-black text-gray-800 mb-1 text-emerald-600" id="card-guidance">上修</p>
                    <p class="text-xs text-gray-400">正向 surprise</p>
                </div>
            </div>
            
            <!-- 右側快速判讀卡 -->
            <div class="bg-white rounded-2xl p-6 shadow-sm border border-gray-100 flex flex-col lg:col-span-1">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full w-max mb-4">快速判讀</span>
                <h3 class="text-lg font-bold text-gray-800 mb-3">這份財報在講什麼？</h3>
                <ul class="space-y-3 text-sm text-gray-600 list-disc pl-4 marker:text-gray-400" id="quick-read-list">
                    <li>營收有成長，但真正加分是接單更強。</li>
                    <li>Backlog 很厚，代表後面不是空的。</li>
                    <li>現金流漂亮，表示不是紙上富貴。</li>
                    <li>財測上修，管理層自己也偏樂觀。</li>
                </ul>
            </div>
        </div>

        <!-- 1. 核心數據表 -->
        <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
            <div class="mb-6">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Data Table</span>
                <h2 class="text-2xl font-bold text-gray-900 mt-4">1. 核心數據表</h2>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse min-w-[600px]">
                    <thead>
                        <tr class="bg-gray-50/50">
                            <th class="py-4 px-4 font-bold text-gray-800 rounded-l-xl w-1/4">項目</th>
                            <th class="py-4 px-4 font-bold text-gray-800 w-1/4">數值</th>
                            <th class="py-4 px-4 font-bold text-gray-800 rounded-r-xl w-1/2">重點解讀</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 text-sm sm:text-base divide-y divide-gray-100" id="core-data-table">
                        <!-- 資料由 JS 動態生成，預設為空，載入時會填入 -->
                    </tbody>
                </table>
            </div>
        </section>

        <!-- 2. 動態圖表視覺化 -->
        <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
            <div class="mb-8">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Charts</span>
                <h2 class="text-2xl font-bold text-gray-900 mt-4">2. 重點圖表視覺化</h2>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="bg-gray-50 p-5 rounded-xl border border-gray-100">
                    <h3 class="text-sm font-bold text-gray-700 mb-4 text-center" id="chart-title-1">營收與動能成長 (億美元)</h3>
                    <div class="relative h-[240px]">
                        <canvas id="growthChart"></canvas>
                    </div>
                </div>
                <div class="bg-gray-50 p-5 rounded-xl border border-gray-100">
                    <h3 class="text-sm font-bold text-gray-700 mb-4 text-center" id="chart-title-2">獲利與現金流表現 (億美元)</h3>
                    <div class="relative h-[240px]">
                        <canvas id="cashFlowChart"></canvas>
                    </div>
                </div>
            </div>
        </section>

        <!-- 3. 和市場預期比較 -->
        <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
            <div class="mb-6">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Beat / Miss</span>
                <h2 class="text-2xl font-bold text-gray-900 mt-4">3. 和市場預期比較</h2>
            </div>
            
            <div class="overflow-x-auto mb-6">
                <table class="w-full text-left border-collapse min-w-[600px]">
                    <thead>
                        <tr class="bg-gray-50/50">
                            <th class="py-4 px-4 font-bold text-gray-800 rounded-l-xl w-1/5">項目</th>
                            <th class="py-4 px-4 font-bold text-gray-800 w-1/5">公司表現</th>
                            <th class="py-4 px-4 font-bold text-gray-800 w-2/5">判讀</th>
                            <th class="py-4 px-4 font-bold text-gray-800 rounded-r-xl w-1/5 text-right">結果</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 text-sm sm:text-base divide-y divide-gray-100" id="beat-miss-table">
                        <!-- 資料由 JS 動態生成 -->
                    </tbody>
                </table>
            </div>

            <!-- 結論 -->
            <div class="bg-indigo-50/50 border-l-4 border-indigo-500 rounded-r-xl p-5">
                <p class="text-gray-800" id="beat-miss-conclusion"><span class="font-bold">結論：</span>這次不是單一數字好看，而是<span class="font-bold">營運、訂單、現金流、展望一起偏強</span>。</p>
            </div>
        </section>

        <!-- 4. 供應鏈地圖 -->
        <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
            <div class="mb-8">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Supply Chain Map</span>
                <h2 class="text-2xl font-bold text-gray-900 mt-4">4. 供應鏈地圖：目前的走向</h2>
            </div>
            
            <div class="relative pl-6 border-l border-indigo-100 space-y-6 mb-6" id="supply-chain-container">
                <!-- 節點由 JS 動態生成 -->
            </div>

            <!-- 結論 -->
            <div class="bg-indigo-50/50 border-l-4 border-indigo-500 rounded-r-xl p-5">
                <p class="text-gray-800" id="supply-chain-conclusion"><span class="font-bold">供應鏈一句話：</span>主軸已經從「單純 AI 晶片」擴散到「AI 用電基礎設施」，而 GEV 正站在這個擴散路徑的核心位置。</p>
            </div>
        </section>

        <!-- 5. 解析重點 -->
        <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
            <div class="mb-6">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Analysis</span>
                <h2 class="text-2xl font-bold text-gray-900 mt-4">5. 解析重點</h2>
            </div>
            <div class="space-y-8" id="analysis-container">
                <!-- 由 JS 動態生成 -->
            </div>
        </section>

        <!-- 6. 估值與 7. 投資觀點 -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <!-- 6. 值不值得現在追？ -->
            <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100 flex flex-col justify-between">
                <div>
                    <div class="mb-6">
                        <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Valuation View</span>
                        <h2 class="text-xl font-bold text-gray-900 mt-4" id="val-title">6. 值不值得現在追 GEV？</h2>
                    </div>
                    <ul class="space-y-4 text-gray-600 list-disc pl-4 marker:text-gray-400 leading-relaxed mb-6" id="val-list">
                        <!-- JS 填入 -->
                    </ul>
                </div>
                <!-- 結論 -->
                <div class="bg-indigo-50/50 border-l-4 border-indigo-500 rounded-r-xl p-5">
                    <p class="text-gray-800 font-bold" id="val-conclusion">一句話：公司夠好，但買點要挑。</p>
                </div>
            </section>

            <!-- 7. 投資觀點 -->
            <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
                <div class="mb-6">
                    <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Bull / Risk</span>
                    <h2 class="text-xl font-bold text-gray-900 mt-4">7. 投資觀點</h2>
                </div>
                
                <table class="w-full text-left text-sm sm:text-base border-collapse">
                    <thead>
                        <tr class="bg-gray-50/50 border-b border-gray-100">
                            <th class="py-3 px-2 font-bold text-gray-800 w-1/2">利多</th>
                            <th class="py-3 px-2 font-bold text-gray-800 w-1/2">風險</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 divide-y divide-gray-100" id="bull-risk-table">
                        <!-- JS 填入 -->
                    </tbody>
                </table>
            </section>
        </div>

        <!-- 8. 精簡版結論 -->
        <section class="bg-white rounded-[2rem] p-6 sm:p-8 shadow-sm border border-gray-100">
            <div class="mb-4">
                <span class="bg-indigo-50 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full">Quick Take</span>
                <h2 class="text-xl font-bold text-gray-900 mt-4">8. 精簡版結論</h2>
            </div>
            <ul class="space-y-3 text-gray-600 list-disc pl-4 marker:text-gray-400" id="final-take-list">
                <!-- JS 填入 -->
            </ul>
        </section>

        <!-- 頁尾 -->
        <footer class="text-center text-gray-400 text-sm mt-8 mb-4">
            <p>© 2026 StockRadar Dynamic Report. All rights reserved.</p>
        </footer>

    </div>

    <!-- ===== JavaScript 資料與邏輯 ===== -->
    <script>
        let growthChartInstance = null;
        let cashFlowChartInstance = null;

        // 完整版模擬資料庫
        const stockDataDB = {
            'GEV': {
                name: 'GE Vernova', ticker: 'GEV',
                subtitle: '不是只看有沒有成長，而是看這份成長有沒有品質、有沒有延續性。',
                highlight: '這季真正的亮點是訂單大增、在手單夠厚、現金流強、全年財測上修，所以這不是只有單季好看，而是後面需求能見度也跟著變強。',
                
                // 頂部卡片
                c1: '93 億美元', c1d: 'YoY +16%', c1t: '營收 Revenue',
                c2: '183 億美元', c2d: '有機成長 +71%', c2t: '訂單 Orders',
                c3: '1630 億美元', c3d: '未來能見度高', c3t: '在手訂單 Backlog',
                c4: '9 億美元', c4d: 'Margin 9.6%', c4t: 'Adj. EBITDA',
                c5: '48 億美元', c5d: '現金品質強', c5t: '自由現金流',
                c6: '上修',
                
                quickRead: ['營收有成長，但真正加分是接單更強。', 'Backlog 很厚，代表後面不是空的。', '現金流漂亮，表示不是紙上富貴。', '財測上修，管理層自己也偏樂觀。'],
                
                // Table 1: 核心數據
                coreTable: [
                    ['營收 Revenue', '93 億美元', '年增 16%，成長穩健，但不是本季最大亮點。'],
                    ['訂單 Orders', '183 億美元', '<span class="text-emerald-600 font-bold">非常強</span>，代表需求正在往後延伸，不是只有單季認列。'],
                    ['有機訂單成長', '+71%', '接單爆發，這是市場最容易加分的地方。'],
                    ['在手訂單 Backlog', '1630 億美元', '未來幾季到幾年營收能見度偏高。'],
                    ['Adjusted EBITDA', '9 億美元', '不只成長，獲利能力也有跟上。'],
                    ['自由現金流', '48 億美元', '<span class="text-emerald-600 font-bold">很強</span>，不是只有帳面數字漂亮。']
                ],

                // Charts
                chartGrowthLast: [80, 107], chartGrowthNow: [93, 183], chartGrowthLabels: ['營收 (Revenue)', '訂單 (Orders)'],
                chartCashData: [9, 48, 52, 102], chartCashLabels: ['Adj. EBITDA', '自由現金流', '營運現金流', '現金部位'],

                // Table 3: Beat/Miss
                beatMiss: [
                    ['營收', '93 億美元', '偏正面，至少不是失望值', '小幅 Beat', 'emerald'],
                    ['訂單', '183 億美元', '明顯強於市場原先期待', '明顯 Beat', 'emerald'],
                    ['自由現金流', '48 億美元', '現金流表現很強', '明顯 Beat', 'emerald'],
                    ['全年財測', '上修', '優於市場 base case', 'Beat', 'emerald']
                ],
                beatMissConclusion: '<span class="font-bold">結論：</span>這次不是單一數字好看，而是<span class="font-bold">營運、訂單、現金流、展望一起偏強</span>。',

                // 4. 供應鏈地圖
                supplyChain: [
                    { title: '上游：發電設備 / 電網重電零組件', badge: '偏強', badgeColor: 'emerald', desc: '需求來自電網升級與大型基建專案。變壓器、開關設備、電力控制元件都受惠，供給仍偏緊。' },
                    { title: '中游：輸配電系統 / Grid Solutions', badge: '最強主線', badgeColor: 'emerald', desc: '這裡是 GEV 最直接受惠的地方。AI data center 帶來高耗電需求，會把錢一路推向電網擴容。' },
                    { title: '下游：資料中心 / 公用事業資本支出', badge: '持續擴張', badgeColor: 'emerald', desc: 'Hyperscaler、雲端業者與公用事業的 CAPEX 還在往上。只要 AI 需求不掉，基建支出就有延續性。' },
                    { title: '延伸鏈：變壓器、散熱、備援電力、儲能', badge: '輪動受惠', badgeColor: 'orange', desc: '市場焦點不會只在 GEV，還會外溢到變壓器與儲能鏈。這條線通常會跟著 data center 輪動發酵。' }
                ],
                scConclusion: '<span class="font-bold">供應鏈一句話：</span>主軸已經從「單純 AI 晶片」擴散到「AI 用電基礎設施」，而 GEV 正站在這個擴散路徑的核心位置。',

                // 5. 解析重點
                analysis: [
                    { title: '重點一：訂單比營收更值得看', points: ['營收代表這季已經認列多少。', 'GEV 這次最漂亮的是 訂單與 backlog 同時很強，這會讓市場更願意給溢價。'] },
                    { title: '重點二：GEV 吃到的是結構性趨勢', points: ['電網升級不是一季題材，是長週期支出。', 'AI data center 用電需求，會外溢到變壓器、配電系統、電網設備。'] },
                    { title: '重點三：現金流強，代表品質不差', points: ['很多公司會有帳面獲利，但現金流未必漂亮。', 'GEV 這次自由現金流強，代表執行面和財務品質都算扎實。'] }
                ],

                // 6 & 7 & 8
                valList: ['<span class="font-bold text-gray-800">基本面值得偏多。</span>', '但不代表現在任何價格都值得追。', '如果財報後已經大漲，短線風險報酬比會變差。', '看拉回、看趨勢守不守、看後續接單動能。'],
                valConclusion: '一句話：公司夠好，但買點要挑。',
                bullRisk: [
                    ['訂單爆發，需求能見度高', '估值可能已經不便宜'],
                    ['在手單厚，後續營收支撐強', '市場預期墊高，下次若只是普通好也可能被賣'],
                    ['現金流強，財務體質佳', '大型專案認列節奏與供應鏈仍有波動']
                ],
                finalTake: ['這份財報是 <span class="font-bold text-gray-800">偏強財報</span>。', '最大亮點不是營收，而是 <span class="font-bold text-gray-800">訂單、backlog、現金流、財測上修</span>。', '如果你是看中期趨勢，GEV 邏輯還站得住。', '如果你想財報後追價，重點要看股價位置，不是只看公司好不好。']
            },
            
            'NVDA': {
                name: 'NVIDIA (輝達)', ticker: 'NVDA',
                subtitle: 'AI 晶片霸主：算力需求無極限，毛利率才是真正的護城河。',
                highlight: '資料中心營收再度打破華爾街天花板，Blackwell 晶片供不應求，毛利率維持在史詩級的 76% 以上，成長動能完全看不到盡頭。',
                
                c1: '260 億美元', c1d: 'YoY +262%', c1t: '營收 Revenue',
                c2: '226 億美元', c2d: 'YoY +427%', c2t: '資料中心營收',
                c3: '78.4%', c3d: '維持極高水準', c3t: '毛利率 Gross Margin',
                c4: '148 億美元', c4d: '年增 6 倍', c4t: '淨利 Net Income',
                c5: '115 億美元', c5d: '印鈔機級別', c5t: '自由現金流',
                c6: '大幅上修',
                
                quickRead: ['資料中心(Data Center)營收佔比超過 85%，絕對主力。', '毛利率 78%+，代表沒人能跟它打價格戰。', '下世代晶片 Blackwell 訂單已排到明年。', '公司表示「全球主權 AI 投資才剛開始」。'],
                
                coreTable: [
                    ['總營收', '260 億美元', '狂勝預期的 246 億，年增 262%。'],
                    ['資料中心營收', '226 億美元', '<span class="text-emerald-600 font-bold">絕對動能</span>，AI 晶片需求持續井噴。'],
                    ['遊戲營收', '26 億美元', '年增 18%，表現平穩。'],
                    ['毛利率 (Non-GAAP)', '78.4%', '較去年同期大幅擴張，定價權極高。'],
                    ['自由現金流', '115 億美元', '強大的印鈔能力，支撐後續研發與庫藏股。']
                ],

                chartGrowthLast: [71, 42], chartGrowthNow: [260, 226], chartGrowthLabels: ['總營收', '資料中心營收'],
                chartCashData: [152, 115, 120, 260], chartCashLabels: ['營業利益', '自由現金流', '營運現金流', '帳上現金'],

                beatMiss: [
                    ['總營收', '260 億', '遠超華爾街最樂觀預期', '大幅 Beat', 'emerald'],
                    ['資料中心', '226 億', 'H100 需求持續不墜', '大幅 Beat', 'emerald'],
                    ['下季財測', '280 億', '超乎市場 base case', 'Beat', 'emerald']
                ],
                beatMissConclusion: '<span class="font-bold">結論：</span>沒有任何死角的財報，<span class="font-bold">AI 需求放緩的謠言不攻自破</span>。',

                supplyChain: [
                    { title: '上游：台積電 / CoWoS 封裝', badge: '最緊缺', badgeColor: 'emerald', desc: 'NVDA 最大的瓶頸不是沒人買，而是產能。台積電先進封裝產能仍是最大關鍵。' },
                    { title: '中游：伺服器代工 / 散熱解決方案', badge: '規格升級', badgeColor: 'emerald', desc: 'GB200 帶動液冷散熱革命，台系伺服器代工廠(廣達、緯創等)與散熱廠迎來大單。' },
                    { title: '下游：CSP (雲端服務供應商)', badge: '軍備競賽', badgeColor: 'emerald', desc: '微軟、Google、Meta、Amazon 資本支出持續創高，深怕在 AI 競賽中落後。' }
                ],
                scConclusion: '<span class="font-bold">供應鏈一句話：</span>從 NVDA 往下看，整個 AI 硬體供應鏈在未來一年內依然是能見度最高的族群。',

                analysis: [
                    { title: '重點一：Blackwell 晶片將迎來「大豐收」', points: ['黃仁勳強調 Blackwell 架構的需求是「瘋狂的」。', '預計下半年開始帶來大量營收，且將與 Hopper 晶片並存。'] },
                    { title: '重點二：主權 AI 成為新藍海', points: ['各國政府開始建立自己的本國 AI 基礎設施。', '預計主權 AI 今年將為 NVDA 帶來數十億美元的額外收入。'] }
                ],

                valList: ['<span class="font-bold text-gray-800">估值並不貴。</span>用明年的預估獲利來看，本益比仍在合理區間。', '每次財報後的下跌，往往是長線買點。', '最大的風險是 CSP 業者的資本支出突然縮手，但目前毫無跡象。'],
                valConclusion: '一句話：AI 時代的絕對核心，持股必備。',
                bullRisk: [
                    ['AI 算力需求呈現指數級增長', '地緣政治與美國晶片出口禁令'],
                    ['軟體生態系(CUDA)護城河深厚', 'AMD 等競爭對手試圖搶佔低階市場'],
                    ['高毛利帶來的龐大自由現金流', '供應鏈產能(如 CoWoS)受限']
                ],
                finalTake: ['這份財報是 <span class="font-bold text-gray-800">史詩級財報</span>。', '完美粉碎了市場對於「AI 泡沫」的疑慮。', 'NVDA 不僅賣硬體，還在賣整個系統，這讓它的領先地位難以撼動。']
            }
        };

        // 產生隨機數據 (如果搜尋的股票不在資料庫中)
        function generateRandomData(ticker) {
            const rev = Math.floor(Math.random() * 500) + 50;
            const net = Math.floor(rev * (Math.random() * 0.3 + 0.05));
            return {
                name: ticker + ' Inc.', ticker: ticker,
                subtitle: `最新 ${ticker} 季報深度解析：關注核心營運數字與成長性。`,
                highlight: `根據最新系統解析，${ticker} 本季在營收表現與市場預期呈現拉鋸，重點在於營運現金流的穩定度以及管理層對下半年的財測指引。`,
                
                c1: rev + ' 億', c1d: 'YoY ' + (Math.random()>0.5?'+':'-') + Math.floor(Math.random()*20)+'%', c1t: '總營收',
                c2: Math.floor(rev*0.4) + ' 億', c2d: '核心業務', c2t: '主營業務收入',
                c3: (Math.random()*30+15).toFixed(1)+'%', c3d: '維持穩定', c3t: '毛利率',
                c4: net + ' 億', c4d: '獲利表現', c4t: '淨利潤',
                c5: Math.floor(net*0.8) + ' 億', c5d: '穩定產出', c5t: '自由現金流',
                c6: Math.random() > 0.5 ? '符合預期' : '保守看待',
                
                quickRead: ['營收表現符合華爾街多數分析師預期區間。', '毛利率維持在常態水準，未見明顯削價競爭。', '現金部位充足，具備抗風險能力。', '後續關注該產業的終端庫存去化狀況。'],
                
                coreTable: [
                    ['總營收', rev + ' 億', '表現中規中矩，無重大意外。'],
                    ['營業利益', Math.floor(net*1.2) + ' 億', '費用控管有成。'],
                    ['淨利潤', net + ' 億', '符合市場預期。'],
                    ['毛利率', (Math.random()*30+15).toFixed(1)+'%', '定價策略仍具彈性。']
                ],

                chartGrowthLast: [Math.floor(rev*0.9), Math.floor(net*0.9)], chartGrowthNow: [rev, net], chartGrowthLabels: ['總營收', '淨利潤'],
                chartCashData: [net, Math.floor(net*0.8), Math.floor(net*1.1), Math.floor(net*3)], chartCashLabels: ['營業利益', '自由現金流', '營運現金流', '帳上現金'],

                beatMiss: [
                    ['總營收', rev+' 億', '符合市場多數預估', 'In-line', 'blue'],
                    ['EPS', (Math.random()*5).toFixed(2), '略優於市場共識', '小幅 Beat', 'emerald']
                ],
                beatMissConclusion: '<span class="font-bold">結論：</span>整體財報表現平穩，未見明顯驚喜或驚嚇。',

                supplyChain: [
                    { title: '上游供應端', badge: '供給正常', badgeColor: 'blue', desc: '原物料與零組件供應順暢，無明顯斷鏈風險。' },
                    { title: '下游需求端', badge: '溫和復甦', badgeColor: 'orange', desc: '終端消費力道仍在觀察期，企業端採購較為保守。' }
                ],
                scConclusion: '<span class="font-bold">產業現況：</span>處於景氣週期的中段，需等待明確的催化劑出現。',

                analysis: [
                    { title: '重點一：營運效率提升', points: ['公司本季強調了費用削減計畫的成效。', '利潤率的維持主要來自內部流程優化。'] },
                    { title: '重點二：未來展望保守', points: ['管理層對下半年的總體經濟環境仍抱持謹慎態度。', '未給出過於積極的財測指引。'] }
                ],

                valList: ['目前估值處於歷史均值附近。', '缺乏短線暴發力，適合長線資金佈局。', '需留意總經數據對其產業板塊的影響。'],
                valConclusion: '一句話：防禦性高於攻擊性。',
                bullRisk: [
                    ['資產負債表健康', '產業成長性趨緩'],
                    ['穩定的現金股利配發', '面臨新興技術或對手的挑戰']
                ],
                finalTake: ['這是一份 <span class="font-bold text-gray-800">中規中矩</span> 的成績單。', '公司在逆風中展現了營運韌性。', '適合偏好穩健收息的投資人。']
            };
        }

        // --- 核心邏輯：執行搜尋並更新畫面 ---
        function executeSearch() {
            const input = document.getElementById('searchInput');
            let ticker = input.value.trim().toUpperCase();
            if (!ticker) { alert('請輸入股票代號！'); return; }

            const loader = document.getElementById('loading-overlay');
            loader.classList.remove('hidden'); loader.classList.add('flex');
            
            let loadingTexts = ["連線至財報資料庫...", "正在解析損益表與供應鏈...", "繪製圖表與重組報告..."];
            let step = 0;
            let textInterval = setInterval(() => {
                document.getElementById('loading-text').innerText = loadingTexts[step % loadingTexts.length];
                step++;
            }, 500);

            setTimeout(() => {
                clearInterval(textInterval);
                updateDashboard(ticker);
                loader.classList.add('hidden'); loader.classList.remove('flex');
                input.value = '';
            }, 1500);
        }

        function handleSearch(event) { if (event.key === 'Enter') executeSearch(); }

        // --- 畫面更新邏輯 ---
        function updateDashboard(ticker) {
            const data = stockDataDB[ticker] || generateRandomData(ticker);

            // Header 更新
            document.getElementById('report-tag').innerText = `—— ${ticker} Data Report ——`;
            document.getElementById('stock-name').innerText = data.name;
            document.getElementById('report-subtitle').innerText = data.subtitle;
            document.getElementById('highlight-text').innerText = data.highlight;
            
            const headerBg = document.getElementById('header-bg');
            if (ticker === 'NVDA') headerBg.className = "bg-gradient-to-br from-[#0a2515] to-[#1a472a] rounded-[2rem] p-8 sm:p-10 text-white shadow-lg relative overflow-hidden transition-all duration-500";
            else if (ticker === 'TSLA') headerBg.className = "bg-gradient-to-br from-[#2c1313] to-[#5a2121] rounded-[2rem] p-8 sm:p-10 text-white shadow-lg relative overflow-hidden transition-all duration-500";
            else headerBg.className = "bg-gradient-to-br from-[#1a2347] to-[#2b448b] rounded-[2rem] p-8 sm:p-10 text-white shadow-lg relative overflow-hidden transition-all duration-500";

            // 頂部 6 卡片
            document.getElementById('card-rev').innerText = data.c1; document.getElementById('card-rev-yoy').innerText = data.c1d;
            document.getElementById('card-orders').innerText = data.c2; document.getElementById('card-orders-yoy').innerText = data.c2d; document.getElementById('card-orders-title').innerText = data.c2t;
            document.getElementById('card-margin').innerText = data.c3; document.getElementById('card-margin-desc').innerText = data.c3d; document.getElementById('card-margin-title').innerText = data.c3t;
            document.getElementById('card-net').innerText = data.c4; document.getElementById('card-net-desc').innerText = data.c4d; document.getElementById('card-net-title').innerText = data.c4t;
            document.getElementById('card-fcf').innerText = data.c5;
            document.getElementById('card-guidance').innerText = data.c6;

            // 快速判讀
            document.getElementById('quick-read-list').innerHTML = data.quickRead.map(t => `<li>${t}</li>`).join('');

            // Table 1
            document.getElementById('core-data-table').innerHTML = data.coreTable.map(row => 
                `<tr><td class="py-4 px-4 font-medium">${row[0]}</td><td class="py-4 px-4 font-bold text-gray-900">${row[1]}</td><td class="py-4 px-4">${row[2]}</td></tr>`
            ).join('');

            // Table 3 Beat/Miss
            document.getElementById('beat-miss-table').innerHTML = data.beatMiss.map(row => 
                `<tr><td class="py-4 px-4">${row[0]}</td><td class="py-4 px-4">${row[1]}</td><td class="py-4 px-4">${row[2]}</td>
                <td class="py-4 px-4 text-right"><span class="bg-${row[4]}-50 text-${row[4]}-600 font-bold px-3 py-1 rounded-full text-xs">${row[3]}</span></td></tr>`
            ).join('');
            document.getElementById('beat-miss-conclusion').innerHTML = data.beatMissConclusion;

            // 4. 供應鏈
            document.getElementById('supply-chain-container').innerHTML = data.supplyChain.map(node => `
                <div class="relative bg-white border border-gray-100 rounded-2xl p-5 shadow-sm">
                    <div class="absolute w-4 h-4 bg-indigo-500 rounded-full -left-[2.1rem] top-6 ring-4 ring-white"></div>
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between mb-2">
                        <h3 class="text-lg font-bold text-gray-900">${node.title}</h3>
                        <span class="bg-${node.badgeColor}-50 text-${node.badgeColor}-600 font-bold px-3 py-1 rounded-full text-xs w-max mt-2 sm:mt-0">${node.badge}</span>
                    </div>
                    <p class="text-gray-500 text-sm">${node.desc}</p>
                </div>
            `).join('');
            document.getElementById('supply-chain-conclusion').innerHTML = data.scConclusion;

            // 5. 解析重點
            document.getElementById('analysis-container').innerHTML = data.analysis.map(item => `
                <div><h3 class="text-lg font-bold text-gray-800 mb-3">${item.title}</h3>
                <ul class="space-y-2 text-gray-600 list-disc pl-5 marker:text-gray-400">
                    ${item.points.map(p => `<li>${p}</li>`).join('')}
                </ul></div>
            `).join('');

            // 6, 7, 8
            document.getElementById('val-title').innerText = `6. 值不值得現在追 ${ticker}？`;
            document.getElementById('val-list').innerHTML = data.valList.map(t => `<li>${t}</li>`).join('');
            document.getElementById('val-conclusion').innerHTML = data.valConclusion;
            
            document.getElementById('bull-risk-table').innerHTML = data.bullRisk.map(row => 
                `<tr><td class="py-3 px-2 pr-4">${row[0]}</td><td class="py-3 px-2">${row[1]}</td></tr>`
            ).join('');
            
            document.getElementById('final-take-list').innerHTML = data.finalTake.map(t => `<li>${t}</li>`).join('');

            // 重繪圖表
            renderCharts(data);
        }

        // --- Chart.js 圖表繪製 ---
        function renderCharts(data) {
            Chart.defaults.font.family = "'Noto Sans TC', sans-serif";
            Chart.defaults.color = '#64748b';

            document.getElementById('chart-title-1').innerText = `${data.chartGrowthLabels.join(' 與 ')} (成長動能)`;

            const ctxGrowth = document.getElementById('growthChart').getContext('2d');
            if (growthChartInstance) growthChartInstance.destroy();
            growthChartInstance = new Chart(ctxGrowth, {
                type: 'bar',
                data: {
                    labels: data.chartGrowthLabels,
                    datasets: [
                        { label: '去年同期', data: data.chartGrowthLast, backgroundColor: '#cbd5e1', borderRadius: 4, barPercentage: 0.6 },
                        { label: '本季表現', data: data.chartGrowthNow, backgroundColor: '#4f46e5', borderRadius: 4, barPercentage: 0.6 }
                    ]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom' } } }
            });

            const ctxCashFlow = document.getElementById('cashFlowChart').getContext('2d');
            if (cashFlowChartInstance) cashFlowChartInstance.destroy();
            cashFlowChartInstance = new Chart(ctxCashFlow, {
                type: 'bar',
                data: {
                    labels: data.chartCashLabels,
                    datasets: [{
                        label: '金額', data: data.chartCashData,
                        backgroundColor: ['#38bdf8', '#10b981', '#34d399', '#818cf8'],
                        borderRadius: 4, barPercentage: 0.5
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } } }
            });
        }

        // 初始載入 GEV
        document.addEventListener('DOMContentLoaded', () => updateDashboard('GEV'));
    </script>
</body>
</html>
