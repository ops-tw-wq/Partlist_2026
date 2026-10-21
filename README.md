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
        <div class="flex flex-col md:flex-row gap-3 mb-6">
            <!-- 關鍵字搜尋 -->
            <input type="text" id="search-input" oninput="filterData()" 
                   placeholder="🔍 輸入品名 (Parts Name)、料號 (P/N)、機型 (Model) 或 備註 (Remarks) 搜尋..." 
                   class="flex-1 p-3 border border-slate-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 shadow-sm">
            
            <!-- 位置篩選 -->
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

            <!-- 庫存數量顯示篩選 (預設僅顯示有庫存) -->
            <select id="stock-status-filter" onchange="filterData()" class="p-3 border border-emerald-300 bg-emerald-50 text-emerald-800 rounded-lg text-sm font-semibold focus:outline-none focus:ring-2 focus:ring-emerald-500">
                <option value="in_stock" selected>✅ 僅顯示有庫存 (QTY > 0)</option>
                <option value="all">📦 顯示全部 (含 QTY = 0)</option>
                <option value="out_of_stock">⚠️ 僅顯示已缺貨 (QTY = 0)</option>
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
        // 指定您的 Google 試算表 CSV 讀取連結
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
                    filterData(); // 讀取完畢後自動進行篩選渲染
                },
                error: function(err) {
                    tbody.innerHTML = '<tr><td colspan="6" class="py-8 text-center text-rose-500">無法讀取資料，請確定 Google Sheet 已開啟「知道連結的人即可檢視」。</td></tr>';
                }
            });
        }

        // 通用自動捕捉屬性值函式 (解決欄位命名大小寫或微小差異)
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

        function filterData() {
            const keyword = document.getElementById('search-input').value.toLowerCase();
            const locationFilter = document.getElementById('location-filter').value.toLowerCase();
            const stockStatusFilter = document.getElementById('stock-status-filter').value;

            const filtered = allInventory.filter(row => {
                const name = getRowValue(row, ['Parts Name', 'PartsName', 'Description', '品名']);
                const pn = getRowValue(row, ['P/N (If any)', 'P/N', '料號']);
                const loc = (getRowValue(row, ['Location', 'Dry Box', '位置']) || '').toLowerCase();
                const model = (getRowValue(row, ['Model', '機型']) || '').toLowerCase();
                const remark = (getRowValue(row, ['Remarks', 'Remark', '備註']) || '').toLowerCase();
                const rawQty = getRowValue(row, ['QTY', 'QTY.1', '數量']);
                const qty = parseInt(rawQty) || 0;

                // 排除無效標頭或完全空白列
                const isValidRow = (name && name.trim() !== '' && name.toLowerCase() !== 'parts name') || 
                                 (pn && pn.trim() !== '' && pn.toLowerCase() !== 'p/n (if any)');

                if (!isValidRow) return false;

                // 1. 庫存數量過濾
                let matchesStock = true;
                if (stockStatusFilter === 'in_stock') {
                    matchesStock = qty > 0; // 只看有庫存
                } else if (stockStatusFilter === 'out_of_stock') {
                    matchesStock = qty <= 0; // 只看缺貨
                }

                // 2. 關鍵字過濾
                const matchesKeyword = loc.includes(keyword) || 
                                       pn.toLowerCase().includes(keyword) || 
                                       name.toLowerCase().includes(keyword) || 
                                       model.includes(keyword) || 
                                       remark.includes(keyword);

                // 3. 位置過濾
                const matchesLocation = locationFilter === '' || loc.includes(locationFilter);

                return isValidRow && matchesStock && matchesKeyword && matchesLocation;
            });

            renderTable(filtered);
        }

        function renderTable(data) {
            const tbody = document.getElementById('table-body');
            tbody.innerHTML = '';

            document.getElementById('data-count').textContent = `共搜尋到 ${data.length} 筆備品`;

            if (data.length === 0) {
                tbody.innerHTML = '<tr><td colspan="6" class="py-8 text-center text-slate-400">目前無符合條件之備品 (可能皆已無庫存或無搜尋結果)</td></tr>';
                return;
            }

            data.forEach(row => {
                const location = getRowValue(row, ['Location', 'Dry Box', '位置']) || '-';
                const pn = getRowValue(row, ['P/N (If any)', 'P/N', '料號']) || '-';
                const name = getRowValue(row, ['Parts Name', 'PartsName', 'Description', '品名']) || '未命名項目';
                const rawQty = getRowValue(row, ['QTY', 'QTY.1', '數量']);
                const qty = parseInt(rawQty) || 0;
                const model = getRowValue(row, ['Model', '機型']) || '-';
                const remark = getRowValue(row, ['Remarks', 'Remark', '備註']) || '-';

                const isLow = qty <= 0;

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

        // 初次自動載入資料
        loadData();
    </script>
</body>
</html>
