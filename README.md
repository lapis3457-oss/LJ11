입금 상태를 선택하는 드롭다운 방식을 체크박스(체크 시 '입금 완료', 해제 시 '미수금') 형태로 변경한 전체 코드입니다.

기존 코드를 모두 지우고 아래 코드로 교체해 주세요.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>비정기 수입 관리 캘린더</title>
<style>
  * { box-sizing: border-box; }
  body { 
    margin: 0; padding: 0; font-family: 'Malgun Gothic', dotum, sans-serif; 
    background: #ffffff; height: 100vh; overflow: hidden; display: flex; flex-direction: column;
  }
  
  .app-container { display: flex; flex-direction: column; width: 100%; height: 100%; background: #ffffff; }
  
  .header { 
    display: flex; justify-content: space-between; align-items: center; 
    padding: 12px 15px; background: #ffffff; border-bottom: 2px solid #333; flex-shrink: 0; 
  }
  .header select, .header button { 
    padding: 8px 12px; font-size: 15px; border: 1px solid #ccc; 
    border-radius: 6px; background: #fff; cursor: pointer; font-weight: bold;
  }
  .header button:hover { background: #eee; }

  .calendar-wrap { 
    flex-grow: 1; display: flex; flex-direction: column; 
    padding: 10px; overflow-y: auto; background: #ffffff; 
  }
  
  .grid-header { 
    display: grid; grid-template-columns: repeat(7, 1fr); 
    border-top: 1px solid #ddd; border-left: 1px solid #ddd; flex-shrink: 0;
  }
  .day-name { 
    text-align: center; background: #f8f9fa; padding: 10px 0; font-size: 13px; font-weight: bold; 
    border-right: 1px solid #ddd; border-bottom: 1px solid #ddd;
  }
  .day-name.sun { color: #d32f2f; }
  .day-name.sat { color: #1976d2; }
  
  .grid-days { 
    display: grid; grid-template-columns: repeat(7, 1fr); 
    grid-auto-rows: minmax(80px, 1fr); 
    border-left: 1px solid #ddd; flex-grow: 1; 
  }
  
  .day { 
    background: #ffffff; padding: 6px; cursor: pointer; 
    display: flex; flex-direction: column; transition: 0.2s;
    border-right: 1px solid #ddd; border-bottom: 1px solid #ddd;
    overflow-y: auto; 
  }
  .day:hover { background: #f4f8ff; }
  .day.empty { background: #fafafa; cursor: default; }
  
  .date-row { display: flex; align-items: baseline; margin-bottom: 4px; }
  .date-num { font-size: 14px; font-weight: bold; color: #333; }
  .day.sun .date-num { color: #d32f2f; }
  .day.sat .date-num { color: #1976d2; }
  .holiday-name { font-size: 10px; color: #d32f2f; margin-left: 5px; }

  .data-badge { 
    margin-top: 2px; padding: 5px; font-size: 12px; border-radius: 4px; 
    color: #fff; text-align: center; line-height: 1.3; word-break: break-all; 
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  }
  .status-paid { background: #4caf50; }
  .status-unpaid { background: #ff9800; }
  
  .bottom-bar { 
    flex-shrink: 0; display: flex; justify-content: space-around; 
    background: #333; color: #fff; padding: 15px 0; font-size: 15px; font-weight: bold; 
  }
  
  .modal-overlay { 
    position: fixed; inset: 0; background: rgba(0,0,0,0.6); 
    display: none; align-items: center; justify-content: center; z-index: 99999; 
  }
  .modal-content { 
    background: #fff; padding: 25px; border-radius: 12px; width: 90%; max-width: 320px; 
    box-shadow: 0 4px 15px rgba(0,0,0,0.3); 
  }
  .modal-content h3 { margin: 0 0 15px 0; text-align: center; color: #333; }
  .input-group { margin-bottom: 12px; }
  .input-group label { display: block; font-size: 12px; color: #666; margin-bottom: 4px; }
  .input-group input[type="text"], .input-group input[type="number"] { 
    width: 100%; padding: 10px; font-size: 15px; border: 1px solid #ccc; border-radius: 6px; 
  }
  
  /* 체크박스 전용 스타일 */
  .checkbox-group { display: flex; align-items: center; gap: 8px; margin-bottom: 12px; margin-top: 5px; }
  .checkbox-group label { font-size: 14px; color: #333; margin: 0; cursor: pointer; font-weight: bold; }
  .checkbox-group input[type="checkbox"] { width: 20px; height: 20px; cursor: pointer; margin: 0; }

  .btn-group { display: flex; gap: 10px; margin-top: 15px; }
  .btn { flex: 1; padding: 12px; border: none; border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer; }
  .btn-close { background: #e0e0e0; color: #333; }
  .btn-save { background: #1976d2; color: #fff; }
  .btn-delete { background: #ffebee; color: #d32f2f; margin-top: 10px; width: 100%; border: 1px solid #ffcdd2; }
</style>
</head>
<body>

<div class="app-container">
  <div class="header">
    <button id="prevBtn">&lt; 이전</button>
    <select id="monthSelect"></select>
    <button id="nextBtn">다음 &gt;</button>
  </div>
  
  <div class="calendar-wrap">
    <div class="grid-header">
      <div class="day-name sun">일</div>
      <div class="day-name">월</div>
      <div class="day-name">화</div>
      <div class="day-name">수</div>
      <div class="day-name">목</div>
      <div class="day-name">금</div>
      <div class="day-name sat">토</div>
    </div>
    <div class="grid-days" id="calendarGrid"></div>
  </div>
  
  <div class="bottom-bar">
    <div>총 입금액: <span id="totalPaid">0</span>원</div>
    <div>총 미수금: <span id="totalUnpaid">0</span>원</div>
  </div>
</div>

<div class="modal-overlay" id="inputModal">
  <div class="modal-content">
    <h3 id="modalDateTitle"></h3>
    <div class="input-group">
      <label>작업 내용 메모</label>
      <input type="text" id="memoInput" placeholder="예: 디자인 외주">
    </div>
    <div class="input-group">
      <label>금액 (원)</label>
      <input type="number" id="amountInput" placeholder="숫자만 입력">
    </div>
    
    <!-- 드롭다운 대신 체크박스로 변경된 영역 -->
    <div class="checkbox-group">
      <input type="checkbox" id="statusCheckbox">
      <label for="statusCheckbox">입금 완료</label>
    </div>

    <div class="btn-group">
      <button class="btn btn-close" id="closeModal">닫기</button>
      <button class="btn btn-save" id="saveData">저장</button>
    </div>
    <button class="btn btn-delete" id="deleteData">이 날짜 내역 삭제</button>
  </div>
</div>

<script>
(function() {
  var holidays = {
    '2026-12-25': '성탄절', '2027-01-01': '신정',
    '2027-02-06': '설연휴', '2027-02-07': '설날', '2027-02-08': '설연휴',
    '2027-03-01': '삼일절', '2027-05-05': '어린이날', '2027-05-13': '부처님오신날',
    '2027-06-06': '현충일', '2027-08-15': '광복절',
    '2027-09-14': '추석연휴', '2027-09-15': '추석', '2027-09-16': '추석연휴',
    '2027-10-03': '개천절', '2027-10-09': '한글날', '2027-12-25': '성탄절'
  };

  var dataStore = {};
  var currentDate = new Date(2026, 10, 1);
  var selectedDateString = "";

  function safeLoadData() {
    try {
      if (typeof window.localStorage !== 'undefined') {
        var saved = window.localStorage.getItem('incomeDataV2');
        if (saved) return JSON.parse(saved);
      }
    } catch (e) {
      console.warn("Storage blocked");
    }
    return {};
  }

  function safeSaveData() {
    try {
      if (typeof window.localStorage !== 'undefined') {
        window.localStorage.setItem('incomeDataV2', JSON.stringify(dataStore));
      }
    } catch (e) {}
  }

  dataStore = safeLoadData();

  var monthSelect = document.getElementById('monthSelect');
  var calendarGrid = document.getElementById('calendarGrid'); 
  var modal = document.getElementById('inputModal');

  for (var y = 2026; y <= 2027; y++) {
    var startM = (y === 2026) ? 11 : 1;
    for (var m = startM; m <= 12; m++) {
      var option = document.createElement('option');
      option.value = y + '-' + m;
      option.text = y + '년 ' + m + '월';
      monthSelect.appendChild(option);
    }
  }

  monthSelect.addEventListener('change', function(e) {
    var parts = e.target.value.split('-');
    currentDate = new Date(parts[0], parts[1] - 1, 1);
    renderCalendar();
  });

  document.getElementById('prevBtn').addEventListener('click', function() {
    currentDate.setMonth(currentDate.getMonth() - 1);
    if(currentDate < new Date(2026, 10, 1)) currentDate = new Date(2026, 10, 1);
    monthSelect.value = currentDate.getFullYear() + '-' + (currentDate.getMonth() + 1);
    renderCalendar();
  });

  document.getElementById('nextBtn').addEventListener('click', function() {
    currentDate.setMonth(currentDate.getMonth() + 1);
    if(currentDate > new Date(2027, 11, 1)) currentDate = new Date(2027, 11, 1);
    monthSelect.value = currentDate.getFullYear() + '-' + (currentDate.getMonth() + 1);
    renderCalendar();
  });

  function renderCalendar() {
    calendarGrid.innerHTML = '';
    var year = currentDate.getFullYear();
    var month = currentDate.getMonth();
    var firstDay = new Date(year, month, 1).getDay();
    var daysInMonth = new Date(year, month + 1, 0).getDate();

    for (var i = 0; i < firstDay; i++) {
      var emptyDiv = document.createElement('div');
      emptyDiv.className = 'day empty';
      calendarGrid.appendChild(emptyDiv);
    }

    for (var k = 1; k <= daysInMonth; k++) {
      (function(dayNum) {
        var mStr = (month + 1 < 10) ? '0' + (month + 1) : (month + 1);
        var dStr = (dayNum < 10) ? '0' + dayNum : dayNum;
        var dateStr = year + '-' + mStr + '-' + dStr;
        var dayOfWeek = new Date(year, month, dayNum).getDay();
        
        var dayDiv = document.createElement('div');
        dayDiv.className = 'day ' + (dayOfWeek === 0 ? 'sun' : (dayOfWeek === 6 ? 'sat' : ''));
        
        var html = '<div class="date-row"><span class="date-num">' + dayNum + '</span>';
        if(holidays[dateStr]) {
          dayDiv.classList.add('holiday');
          html += '<span class="holiday-name">' + holidays[dateStr] + '</span>';
        }
        html += '</div>';
        
        if(dataStore[dateStr] && dataStore[dateStr].amount) {
          var d = dataStore[dateStr];
          var badgeClass = d.status === 'paid' ? 'status-paid' : 'status-unpaid';
          var amountStr = Number(d.amount).toLocaleString();
          html += '<div class="data-badge ' + badgeClass + '">' + d.memo + '<br>' + amountStr + '원</div>';
        }
        
        dayDiv.innerHTML = html;
        dayDiv.addEventListener('click', function() { openModal(dateStr); });
        calendarGrid.appendChild(dayDiv);
      })(k);
    }
    updateTotals();
  }

  function updateTotals() {
    var paid = 0; var unpaid = 0;
    for(var key in dataStore) {
      if(dataStore.hasOwnProperty(key)) {
        if(dataStore[key].status === 'paid') paid += Number(dataStore[key].amount || 0);
        else unpaid += Number(dataStore[key].amount || 0);
      }
    }
    document.getElementById('totalPaid').innerText = paid.toLocaleString();
    document.getElementById('totalUnpaid').innerText = unpaid.toLocaleString();
  }

  function openModal(dateStr) {
    selectedDateString = dateStr;
    document.getElementById('modalDateTitle').innerText = dateStr;
    
    var data = dataStore[dateStr] || { memo: '', amount: '', status: 'unpaid' };
    document.getElementById('memoInput').value = data.memo;
    document.getElementById('amountInput').value = data.amount;
    
    // 체크박스 상태 연동 (데이터가 'paid'이면 체크)
    document.getElementById('statusCheckbox').checked = (data.status === 'paid');
    
    document.getElementById('deleteData').style.display = dataStore[dateStr] ? 'block' : 'none';
    modal.style.display = 'flex';
  }

  document.getElementById('closeModal').addEventListener('click', function() { modal.style.display = 'none'; });
  
  document.getElementById('saveData').addEventListener('click', function() {
    var amount = document.getElementById('amountInput').value;
    if(!amount) { alert("금액을 입력해주세요."); return; }
    
    // 체크박스가 체크되어 있으면 'paid', 아니면 'unpaid'로 저장
    var isPaid = document.getElementById('statusCheckbox').checked;
    
    dataStore[selectedDateString] = {
      memo: document.getElementById('memoInput').value,
      amount: amount,
      status: isPaid ? 'paid' : 'unpaid'
    };
    
    safeSaveData();
    modal.style.display = 'none';
    renderCalendar();
  });

  document.getElementById('deleteData').addEventListener('click', function() {
    if(confirm('이 날짜의 내역을 삭제하시겠습니까?')) {
      delete dataStore[selectedDateString];
      safeSaveData();
      modal.style.display = 'none';
      renderCalendar();
    }
  });

  monthSelect.value = currentDate.getFullYear() + '-' + (currentDate.getMonth() + 1);
  renderCalendar();
})();
</script>
</body>
</html>

```
