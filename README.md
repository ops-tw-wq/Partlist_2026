# Partlist_2026
Partlist_2026
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>工程師庫存與備品即時查詢</title>
    <!-- 引入 Tailwind CSS 漂亮樣式 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 引入 PapaParse 自動解析表格工具 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
</head>
<body class="bg-slate-100 min-h-screen p-4 sm:p-6">

    <div class="max-w-6xl mx-auto bg-white rounded-xl shadow-md p-5 sm:p-8">
        
        <!-- 標頭 -->
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center border-b border-slate-200 pb-4 mb-6">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">📦 公司庫存與備品即時查詢</h1>
                <p class="text-sm text-slate-500 mt-1">工程師專用查詢頁面 (資料即時連動)</p>
            </div>
            <button onclick="loadData()" class="mt-3 sm:mt-0 bg-indigo-50 text-indigo-600 hover:bg-indigo-100 px-3 py-1.5 rounded-lg text-sm font-medium border border-indigo-200 transition">
                🔄 重新整理資料
            </button>
        </div>

        <!-- 搜尋欄 -->
        <div class="relative mb-6">
            <input type="text" id="search-input" oninput="filterData()" 
                   placeholder="請輸入：產品描述 Description、PO 號碼、客戶或料號搜尋..." 
                   class="w-full pl-4 pr-10 py-3 border border-slate-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 shadow-sm">
        </div>

        <!-- 庫存表格 -->
        <div class="overflow-x-auto rounded-lg border border-slate-200">
            <table class="w-full text-left border-collapse">
                <thead>
                    <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold border-b border-slate-200">
                        <th class="py-3 px-4">產品描述 / Description</th>
                        <th class="py-3 px-4">客戶 / PO#</th>
                        <th class="py-3 px-4 text-center">數量 / QTY</th>
                        <th class="py-3 px-4">狀態 / Status</th>
                        <th class="py-3 px-4">備註 / Remark</th>
                    </tr>
                </thead>
                <tbody id="table-body" class="divide-y divide-slate-100 text-sm">
                    <tr>
                        <td colspan="5" class="py-8 text-center text-slate-400">正在讀取最新庫存資料中...</td>
                    </tr>
                </tbody>
            </table>
        </div>

        <p class="text-xs text-slate-400 mt-4 text-right">※ 資料取自公司 Google 試算表</p>
    </div>

    <script>
        // ⚠️ 這裡已經將您的 Google Sheet 網址轉換為 CSV 讀取格式
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

            // 過濾掉空白列
            const validData = data.filter(row => row['Description'] || row['PO #'] || row['Customer']);

            if (validData.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="py-8 text-center text-slate-400">查無相關庫存資料</td></tr>';
                return;
            }

            validData.forEach(row => {
                const desc = row['Description'] || '未填寫描述';
                const customer = row['Customer'] || '';
                const po = row['PO #'] || '';
                const qty = row['QTY'] || '0';
                const status = row['Status'] || '-';
                const remark = row['Remark'] || '-';

                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition";
                tr.innerHTML = `
                    <td class="py-3 px-4 font-medium text-slate-800">${desc}</td>
                    <td class="py-3 px-4 text-slate-600">${customer} ${po ? '<span class="text-xs text-slate-400 font-mono">(' + po + ')</span>' : ''}</td>
                    <td class="py-3 px-4 text-center font-bold text-indigo-600">${qty}</td>
                    <td class="py-3 px-4"><span class="px-2 py-0.5 rounded text-xs font-semibold bg-slate-100 text-slate-600">${status}</span></td>
                    <td class="py-3 px-4 text-slate-500 text-xs">${remark}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function filterData() {
            const keyword = document.getElementById('search-input').value.toLowerCase();
            const filtered = allData.filter(row => {
                const desc = (row['Description'] || '').toLowerCase();
                const customer = (row['Customer'] || '').toLowerCase();
                const po = (row['PO #'] || '').toLowerCase();
                return desc.includes(keyword) || customer.includes(keyword) || po.includes(keyword);
            });
            renderTable(filtered);
        }

        // 頁面開啟時自動載入資料
        loadData();
    </script>
</body>
</html>
