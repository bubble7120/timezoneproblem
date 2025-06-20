<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>Case 1E：查詢從 6/27 開始，錯誤邏輯漏掉最後一筆 ETA</title>
  <style>
    body { font-family: 'Segoe UI'; background: #eef4ff; padding: 2rem; }
    h2 { color: #0b5394; }
    label, input, button { font-size: 1rem; margin: 0.4rem; }
    button { padding: 0.4rem 0.8rem; cursor: pointer; }
    table { border-collapse: collapse; width: 100%; margin-top: 1rem; }
    th, td { border: 1px solid #aaa; padding: 0.5rem; }
    th { background: #dceeff; }
    tr.missed { background-color: #fff4b3; }
    .container { display: flex; gap: 2rem; flex-wrap: wrap; }
    .panel { flex: 1; background: #fff; padding: 1rem; border-radius: 6px; box-shadow: 0 0 4px rgba(0,0,0,0.1); min-width: 460px; }
    .query-info { font-size: 0.85rem; color: #555; margin-top: 0.5rem; }
  </style>
</head>
<body>
  <h2>Case 2：跟上一個例子一樣：雖然系統裡架設的是美西時間(PDT)，然而電腦是使用UTC(世界標準時間) 來運算時，可會漏塞最後一天，因為PDT比UTC慢7小時。 </h2>
   <h2> 這次假設ETA設定是美西時間的7/3號晚上23:59:59點。為方便起見，程式碼中會寫篩選到7/4號凌晨00點以前，這樣7/3號晚上23:59:59點前都會被包含到。
    例如：以今天上班為例，要搜尋的是美西時間6/27-7/3號，然而7/3號這一天，如果沒有在程式中，把搜尋時區做延長的話，那是搜不到完整的7/3的。</h2>
    <h2>問題出在：我們以為我們篩選的ETA應該要包含到美西時間7/4號凌晨12點以前，但因為系統很有可能是運用UTC來運算，
    因為使用UTC才會跟大部分瀏覽器運程式相容。所以系統很有可能會把美西時間7/4號凌晨12點以前，自動轉換成UTC時間7/4號凌晨12點來前篩選。</h2>
    <h2>然而系統輸入ETA的時候，放入的卻是美西時間(PDT)</h2>
    <h2>如果是以上這樣的話，美西時間因為比UTC慢了7小時，所以我們以為我們按下了7/4號凌晨12點以前的按鈕，系統應該按照這個規則來篩選，但轉統自動轉還成了UTD，結果就是：美西時間其實還在7/3號的下午17點鐘以前
    <h2>所以如果在輸入美西時間ETA的話，7/3號下午17點以後的ETA是搜尋不到的。但同一天7/3號17點以前是搜尋的到。這也是為什麼很有可能會出現誤會，覺得是人為疏失，明明同樣是顯示美西時間7/3號，有些卻能被篩選出來，有些卻沒有，其實是因為系統可能是轉換成UTC時間去篩選。</h2>
    <h2>請使用以下篩選鍵玩玩看~  左邊是上方提到的會有失誤的篩選機制，右邊是正確機制(在程式碼中延長了搜尋的時區)
    <h2>右邊的三筆黃色資料，在左邊錯誤的篩選機制下，有一筆晚於ETA晚於美西時間17點的不會被篩出來</h2></h2>

   <label>起始日（PDT）:</label>
   <input type="date" id="start" value="2025-06-27" />
   <label>結束日（PDT）:</label>
   <input type="date" id="end" value="2025-07-03" />
   <button onclick="runFilter()">查詢比較</button>

  <div class="container">
    <div class="panel">
      <h3>❌ 錯誤邏輯（end = UTC 7/4 00:00）</h3>
      <div class="query-info" id="infoWrong"></div>
      <table>
        <thead><tr><th>貨單</th><th>港口</th><th>CNT</th><th>ETA</th><th>UTC</th></tr></thead>
        <tbody id="wrongTable"></tbody>
      </table>
    </div>
    <div class="panel">
      <h3>✅ 修正邏輯（end = PDT 7/4 00:00 → UTC）</h3>
      <div class="query-info" id="infoRight"></div>
      <table>
        <thead><tr><th>貨單</th><th>港口</th><th>CNT</th><th>ETA</th><th>UTC</th></tr></thead>
        <tbody id="rightTable"></tbody>
      </table>
    </div>
  </div>

  <script>
    const shipments = [
      { code: 'LA001', port: 'Los Angeles', cnt: 'CNT0001', eta: "2025-06-27T12:00:00-07:00" },
      { code: 'LA002', port: 'Long Beach', cnt: 'CNT0002', eta: "2025-06-28T09:00:00-07:00" },
      { code: 'LA003', port: 'Oakland', cnt: 'CNT0003', eta: "2025-07-02T17:30:00-07:00" },
      { code: 'LA004', port: 'Portland', cnt: 'CNT0004', eta: "2025-07-03T14:00:00-07:00" },
      { code: 'LA005', port: 'Seattle', cnt: 'CNT0005', eta: "2025-07-03T16:30:00-07:00" },
      { code: 'LA006', port: 'Tacoma', cnt: 'CNT0006', eta: "2025-07-03T17:00:00-07:00" } // 被排除那筆
    ];

    function runFilter() {
      const startStr = document.getElementById("start").value;
      const endStr = document.getElementById("end").value;

      const start = new Date(`${startStr}T00:00:00-07:00`);
      const endWrong = new Date(`2025-07-04T00:00:00Z`); // ⚠️ 查到 PDT 7/3 17:00:00
      const endRight = new Date(`${endStr}T00:00:00-07:00`);
      endRight.setDate(endRight.getDate() + 1);

      document.getElementById("infoWrong").textContent = `查詢時間（錯誤）：${start.toISOString()} ~ ${endWrong.toISOString()}（→ PDT 7/3 17:00:00）`;
      document.getElementById("infoRight").textContent = `查詢時間（正確）：${start.toISOString()} ~ ${endRight.toISOString()}（完整 PDT 7/3）`;

      const wrongTable = document.getElementById("wrongTable");
      const rightTable = document.getElementById("rightTable");
      wrongTable.innerHTML = "";
      rightTable.innerHTML = "";

      shipments.forEach(s => {
        const eta = new Date(s.eta);
        const etaUTC = eta.toISOString();
        const missed = eta >= endWrong;

        const row = `<tr${missed ? ' class="missed"' : ''}>
          <td>${s.code}</td><td>${s.port}</td><td>${s.cnt}</td><td>${s.eta}</td><td>${etaUTC}</td>
        </tr>`;

        if (eta >= start && eta < endWrong) wrongTable.innerHTML += row;
        if (eta >= start && eta < endRight) rightTable.innerHTML += row;
      });
    }
  </script>
</body>
</html>

