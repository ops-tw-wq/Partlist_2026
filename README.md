<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PTW TW 備品庫存即時查詢系統</title>
    <!-- Tailwind CSS 樣式 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- PapaParse 解析工具 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
</head>
<body class="bg-slate-100 min-h-screen p-4 sm:p-6">

    <div class="max-w-7xl mx-auto bg-white rounded-xl shadow-md p-5 sm:p-8">
        
        <!-- Header 標頭區 -->
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center border-b border-slate-200 pb-4 mb-6">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">📦 PTW TW 備品庫存即時查詢系統</h1>
                <p class="text-sm text-slate-500 mt-1">資料即時同步自 Google Sheet 第一分頁 (Stock in TW)</p>
            </div>
            <button onclick="loadData()" class="mt-3 sm:mt-0 bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg text-sm font-medium transition shadow-sm flex items-center gap-2">
                🔄 重新整理最新庫存
            </button>
        </div>

        <!-- 搜尋與庫房位置下拉選單 -->
        <div class="flex flex-col sm:flex-row gap-3 mb-6">
            <input type="text" id="search-input" oninput="filterData()" 
                   placeholder="🔍 輸入品名 (Parts Name)、料號 (P/N)、機型 (Model) 或 備註 (Remarks) 搜尋..." 
                   class="flex-1 p-3 border border-slate-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 shadow-sm">
            
            <select id="location-filter" onchange="filterData()" class="p-3 border border-slate-300 rounded-lg text-sm bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500">
                <option value="">所有位置 (All Locations)</option>
                <option value="S1">S1</option>
                <option value="S2">S2</option>
                <option value="S3">S3</option>
                <option value="S4">S4</option>
                <option value="S5">S5</option>
                <option value="S6">S6</option>
                <option value="DRY BOX">DRY BOX</option>
                <option value="Clean room">Clean room</option>
            </select>
        </div>

        <!-- 表格顯示區 -->
        <div class="overflow-x-auto rounded-lg border border-slate-200">
            <table class="w-full text-left border-collapse">
                <thead>
                    <tr class="bg-slate-100 text-slate-700 text-xs uppercase font-bold border-b border-slate-200">
                        <th class="py-3 px-4">位置 (Location)</th>
                        <th class="py-3 px-4">料號 (P/N)</th>
                        <th class="py-3 px-4">品名 (Parts Name)</th>
                        <th class="py-3 px-4 text-center">現有庫存 (QTY)</th>
                        <th class="py-3 px-4">適用機型 (Model)</th>
                        <th class="py-3 px-4">備註 (Remarks)</th>
                    </tr>
                </thead>
                <tbody id="table-body" class="divide-y divide-slate-100 text-sm">
                    <tr>
                        <td colspan="6" class="py-8 text-center text-slate-400">連線讀取最新庫存資料中...</td>
                    </tr>
                </tbody>
            </table>
        </div>

        <!-- 頁尾資訊 -->
        <div class="flex justify-between items-center mt-4 text-xs text-slate-400">
            <span id="data-count">載入中...</span>
            <span>※ 工程師專用查詢介面</span>
        </div>
    </div>

    <script>
        // 您的 Google 試算表 CSV 讀取連結
        const csvUrl = 'https://docs.google.com/spreadsheets/d/1hND5f2twITUn2386YzICVKhMEQ5WVpswrnvILJHDrTQ/gviz/tq?tqx=out:csv';

        let allInventory = [];

        function loadData() {
            const tbody = document.getElementById('table-body');
            tbody.innerHTML = '<tr><td colspan="6" class="py-8 text-center text-slate-400">正在讀取最新庫存資料...</td></tr>';

            Papa.parse(csvUrl, {
                download: true,
                header: true,
                complete: function(results) {
                    allInventory = results.data;
                    renderTable(allInventory);
                },
                error: function(err) {
                    tbody.innerHTML = '<tr><td colspan="6" class="py-8 text-center text-rose-500">無法讀取資料，請確定 Google Sheet 已開啟「知道連結的人即可檢視」。</td></tr>';
                }
            });
        }

        // 通用自動搜尋屬性值函式 (解決欄位大小寫或命名些微差異)
        function getRowValue(row, keys) {
            for (let k of keys) {
                for (let prop in row) {
                    if (prop.trim().toLowerCase() === k.toLowerCase()) {
                        return row[prop];
                    }
                }
            }
            return '';
        }

        function renderTable(data) {
            const tbody = document.getElementById('table-body');
            tbody.innerHTML = '';

            // 超級強大的欄位自動捕捉功能
            const validData = data.filter(row => {
                const name = getRowValue(row, ['Parts Name', 'PartsName', 'Description', '品名']);
                const location = getRowValue(row, ['Location', 'Dry Box', '位置']);
                const pn = getRowValue(row, ['P/N (If any)', 'P/N', '料號']);
                return (name && name.trim() !== '' && name.toLowerCase() !== 'parts name') || 
                       (pn && pn.trim() !== '' && pn.toLowerCase() !== 'p/n (if any)');
            });

            document.getElementById('data-count').textContent = `共搜尋到 ${validData.length} 筆備品`;

            if (validData.length === 0) {
                tbody.innerHTML = '<tr><td colspan="6" class="py-8 text-center text-slate-400">連線成功，但 Google Sheet 第一頁找不到庫存資料，請確認資料有從第 1 列開始。</td></tr>';
                return;
            }

            validData.forEach(row => {
                const location = getRowValue(row, ['Location', 'Dry Box', '位置']) || '-';
                const pn = getRowValue(row, ['P/N (If any)', 'P/N', '料號']) || '-';
                const name = getRowValue(row, ['Parts Name', 'PartsName', 'Description', '品名']) || '未命名項目';
                const qty = getRowValue(row, ['QTY', 'QTY.1', '數量']) || '0';
                const model = getRowValue(row, ['Model', '機型']) || '-';
                const remark = getRowValue(row, ['Remarks', 'Remark', '備註']) || '-';

                const isLow = parseInt(qty) <= 0 || isNaN(parseInt(qty));

                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition";
                tr.innerHTML = `
                    <td class="py-3 px-4"><span class="px-2.5 py-1 rounded-md text-xs font-semibold ${getLocationBadge(location)}">${location}</span></td>
                    <td class="py-3 px-4 font-mono text-xs text-slate-600">${pn}</td>
                    <td class="py-3 px-4 font-bold text-slate-800">${name}</td>
                    <td class="py-3 px-4 text-center font-bold text-base ${isLow ? 'text-rose-500' : 'text-emerald-600'}">
                        ${qty}
                    </td>
                    <td class="py-3 px-4 text-xs text-slate-500">${model}</td>
                    <td class="py-3 px-4 text-xs text-slate-500">${remark}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function getLocationBadge(loc) {
            loc = loc.toUpperCase();
            if(loc.includes('S1') || loc.includes('S2')) return 'bg-blue-50 text-blue-700 border border-blue-200';
            if(loc.includes('S3') || loc.includes('S4')) return 'bg-indigo-50 text-indigo-700 border border-indigo-200';
            if(loc.includes('DRY')) return 'bg-amber-50 text-amber-700 border border-amber-200';
            if(loc.includes('CLEAN')) return 'bg-emerald-50 text-emerald-700 border border-emerald-200';
            return 'bg-slate-100 text-slate-700';
        }

        function filterData() {
            const keyword = document.getElementById('search-input').value.toLowerCase();
            const locationFilter = document.getElementById('location-filter').value.toLowerCase();

            const filtered = allInventory.filter(row => {
                const loc = (getRowValue(row, ['Location', 'Dry Box', '位置']) || '').toLowerCase();
                const pn = (getRowValue(row, ['P/N (If any)', 'P/N', '料號']) || '').toLowerCase();
                const name = (getRowValue(row, ['Parts Name', 'PartsName', 'Description', '品名']) || '').toLowerCase();
                const model = (getRowValue(row, ['Model', '機型']) || '').toLowerCase();
                const remark = (getRowValue(row, ['Remarks', 'Remark', '備註']) || '').toLowerCase();

                const matchesKeyword = loc.includes(keyword) || pn.includes(keyword) || name.includes(keyword) || model.includes(keyword) || remark.includes(keyword);
                const matchesLocation = locationFilter === '' || loc.includes(locationFilter);

                return matchesKeyword && matchesLocation;
            });

            renderTable(filtered);
        }

        // 初次自動載入資料
        loadData();
    </script>
</body>
</html>
