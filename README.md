# Partlist_2026
Partlist_2026
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>工程師庫存與備品即時查詢</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- PapaParse -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
</head>
<body class="bg-slate-100 min-h-screen p-4 sm:p-6">

    <div class="max-w-6xl mx-auto bg-white rounded-xl shadow-md p-5 sm:p-8">
        
        <!-- Header -->
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center border-b border-slate-200 pb-4 mb-6">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">📦 公司庫存與備品即時查詢系統</h1>
                <p class="text-sm text-slate-500 mt-1">適用於 S1~S6、Dry Box、Clean room 即時庫存查閱</p>
            </div>
            <button onclick="loadData()" class="mt-3 sm:mt-0 bg-indigo-50 text-indigo-600 hover:bg-indigo-100 px-3 py-1.5 rounded-lg text-sm font-medium border border-indigo-200 transition flex items-center gap-1">
                🔄 重新整理資料
            </button>
        </div>

        <!-- Search Bar -->
        <div class="relative mb-6">
            <input type="text" id="search-input" oninput="filterData()" 
                   placeholder="輸入 Parts Name (品名)、P/N (料號)、Model (機型) 或 Location (位置, 如 S2, Dry Box) 搜尋..." 
                   class="w-full pl-4 pr-10 py-3 border border-slate-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 shadow-sm">
        </div>

        <!-- Table -->
        <div class="overflow-x-auto rounded-lg border border-slate-200">
            <table class="w-full text-left border-collapse">
                <thead>
                    <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold border-b border-slate-200">
                        <th class="py-3 px-4">存放位置 (Location)</th>
                        <th class="py-3 px-4">品名 (Parts Name)</th>
                        <th class="py-3 px-4">料號 (P/N)</th>
                        <th class="py-3 px-4">適用機型 (Model)</th>
                        <th class="py-3 px-4 text-center">庫存數量 (QTY)</th>
                    </tr>
                </thead>
                <tbody id="table-body" class="divide-y divide-slate-100 text-sm">
                    <tr>
                        <td colspan="5" class="py-8 text-center text-slate-400">正在讀取最新庫存資料中...</td>
                    </tr>
                </tbody>
            </table>
        </div>

        <p class="text-xs text-slate-400 mt-4 text-right">※ 資料與公司 Google 試算表同步</p>
    </div>

    <script>
        // 自動抓取您的 Google Sheet 試算表 CSV 格式
        const csvUrl = 'https://docs.google.com/spreadsheets/d/1Drz8qnb90xHrS2-DRh2ezCzpgzJ1yr82AXfb2usD4os/gviz/tq?tqx=out:csv';

        let allData = [];

        function loadData() {
            const tbody = document.getElementById('table-body');
            tbody.innerHTML = '<tr><td colspan="5" class="py-8 text-center text-slate-400">正在更新資料中...</td></tr>';

            Papa.parse(csvUrl, {
                download: true,
                header: true,
                complete: function(results) {
                    allData = results.data;
                    renderTable(allData);
                },
                error: function(err) {
                    tbody.innerHTML = '<tr><td colspan="5" class="py-8 text-center text-red-500">無法讀取資料，請確認 Google Sheet 共用權限已設為「知道連結的人」。</td></tr>';
                }
            });
        }

        function renderTable(data) {
            const tbody = document.getElementById('table-body');
            tbody.innerHTML = '';

            // 動態搜尋品名與料號欄位（相容多種欄位命名方式）
            const validData = data.filter(row => {
                return row['Parts Name'] || row['PartsName'] || row['Description'] || row['P/N (If any)'] || row['P/N'];
            });

            if (validData.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="py-8 text-center text-slate-400">目前沒有讀取到庫存資料，請檢查試算表第一個頁籤是否有庫存列表。</td></tr>';
                return;
            }

            validData.forEach(row => {
                const location = row['Location'] || row['Dry Box'] || '-';
                const partsName = row['Parts Name'] || row['PartsName'] || row['Description'] || '未命名項目';
                const pn = row['P/N (If any)'] || row['P/N'] || '-';
                const model = row['Model'] || '-';
                const qty = row['QTY'] || row['QTY.1'] || '0';

                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition";
                tr.innerHTML = `
                    <td class="py-3 px-4"><span class="px-2 py-0.5 rounded text-xs font-semibold bg-indigo-50 text-indigo-700 border border-indigo-200">${location}</span></td>
                    <td class="py-3 px-4 font-bold text-slate-800">${partsName}</td>
                    <td class="py-3 px-4 text-slate-600 font-mono text-xs">${pn}</td>
                    <td class="py-3 px-4 text-slate-500 text-xs">${model}</td>
                    <td class="py-3 px-4 text-center font-bold ${parseInt(qty) > 0 ? 'text-emerald-600' : 'text-rose-500'}">${qty}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function filterData() {
            const keyword = document.getElementById('search-input').value.toLowerCase();
            const filtered = allData.filter(row => {
                const location = (row['Location'] || row['Dry Box'] || '').toLowerCase();
                const partsName = (row['Parts Name'] || row['PartsName'] || row['Description'] || '').toLowerCase();
                const pn = (row['P/N (If any)'] || row['P/N'] || '').toLowerCase();
                const model = (row['Model'] || '').toLowerCase();
                
                return location.includes(keyword) || partsName.includes(keyword) || pn.includes(keyword) || model.includes(keyword);
            });
            renderTable(filtered);
        }

        loadData();
    </script>
</body>
</html>
