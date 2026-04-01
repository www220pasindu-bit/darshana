<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shift Tracker 12h</title>
<style>
body {font-family: Arial; background:#121212; color:#fff; margin:20px; text-align:center;}
h2 {margin-bottom:10px;}
table {border-collapse: collapse; width:95%; margin:auto;}
th, td {border:1px solid #fff; padding:5px; text-align:center;}
th {background:#333;}
.dayShift {background:#4caf50; color:#000;}
.nightShift {background:#2196f3; color:#fff;}
.offDay {background:#f44336; color:#fff;}
.poya {background:#ffa726; color:#000;}
.attended {background:#ffeb3b; color:#000;}
input, button, select {padding:5px; margin:5px;}
.resetBtn {background:#f44336; color:#fff; border:none; padding:8px 12px; cursor:pointer;}
</style>
</head>
<body>

<h2>Shift Tracker (12h Day/Night)</h2>
<p>Current Time: <span id="time"></span></p>

<label>Name: <input type="text" id="empName" placeholder="Employee Name"></label>
<label>EPF: <input type="text" id="empEPF" placeholder="EPF No"></label><br>

<label>Start Date: <input type="date" id="start"></label>
<button onclick="generateSchedule()">Generate Schedule</button>
<button class="resetBtn" onclick="resetAll()">Reset All</button>
<button onclick="generateReport()">Monthly Report</button>

<table id="scheduleTable">
<thead>
<tr>
<th>Date</th><th>Day</th><th>Shift</th><th>Employee</th><th>EPF</th>
<th>Check In</th><th>Check Out</th><th>Late(min)</th><th>Off/Work</th><th>Poya</th></tr>
</thead>
<tbody></tbody>
</table>

<script>
// Live clock
function updateTime(){
  document.getElementById("time").innerText = new Date().toLocaleTimeString();
}
setInterval(updateTime,1000); updateTime();

// Poya / National holidays example
const poyaDates = ['2026-04-05','2026-04-19','2026-05-04']; 
const nationalHolidays = ['2026-04-14','2026-05-01']; 

let attendanceData = [];

function generateSchedule(){
  const startInput = document.getElementById('start').value;
  if(!startInput) return alert('Select Start Date!');
  const empName = document.getElementById('empName').value || '-';
  const empEPF = document.getElementById('empEPF').value || '-';
  
  const startDate = new Date(startInput);
  const tbody = document.querySelector('#scheduleTable tbody');
  tbody.innerHTML='';
  attendanceData=[];

  const shifts = ['Day','Day','Night','Night','Off','Off','Off'];
  const days=['Sun','Mon','Tue','Wed','Thu','Fri','Sat'];

  for(let i=0;i<30;i++){
    let d = new Date(startDate);
    d.setDate(startDate.getDate()+i);
    const dateStr = d.toISOString().split('T')[0];
    const shiftType = shifts[i % shifts.length];
    const shiftClass = shiftType==='Day'?'dayShift':(shiftType==='Night'?'nightShift':'offDay');
    const isPoya = poyaDates.includes(dateStr);
    const isHoliday = nationalHolidays.includes(dateStr);

    attendanceData.push({checkIn:'',checkOut:'',late:0,off:shiftType==='Off',poya:isPoya||isHoliday});

    const tr = document.createElement('tr'); tr.id='row'+i;
    tr.className = shiftClass;
    if(isPoya||isHoliday) tr.classList.add('poya');

    tr.innerHTML = `<td>${d.toLocaleDateString('en-GB')}</td>
      <td>${days[d.getDay()]}</td>
      <td class="${shiftClass}" id="shift${i}">${shiftType}</td>
      <td>${empName}</td><td>${empEPF}</td>
      <td id="in${i}"><button onclick="checkIn(${i})">Check In</button></td>
      <td id="out${i}"><button onclick="checkOut(${i})">Check Out</button></td>
      <td id="late${i}">0</td>
      <td id="off${i}">${shiftType==='Off'?'Off':'Work'}</td>
      <td>${isPoya||isHoliday?'Yes':'No'}</td>`;
    tbody.appendChild(tr);
  }
}

function checkIn(i){
  const now = new Date();
  attendanceData[i].checkIn = now.toLocaleTimeString();
  document.getElementById('in'+i).innerText = attendanceData[i].checkIn;

  if(attendanceData[i].off){ 
    attendanceData[i].off=false;
    document.getElementById('off'+i).innerText='Work';
  }

  calculateLate(i, now);
  document.getElementById('row'+i).classList.add('attended');
}

function checkOut(i){
  if(!attendanceData[i].checkIn){alert('Check In first!'); return;}
  const now = new Date();
  attendanceData[i].checkOut = now.toLocaleTimeString();
  document.getElementById('out'+i).innerText = attendanceData[i].checkOut;
  document.getElementById('row'+i).classList.add('attended');
}

function calculateLate(i, now){
  if(attendanceData[i].off || attendanceData[i].poya){ 
    attendanceData[i].late=0; document.getElementById('late'+i).innerText=0; return; 
  }
  const shift = document.getElementById('shift'+i).innerText;
  let shiftStart = shift==='Day'?6:18;
  const late = Math.max(0,(now.getHours()-shiftStart)*60 + now.getMinutes());
  attendanceData[i].late=late;
  document.getElementById('late'+i).innerText=late;
}

function resetAll(){
  for(let i=0;i<attendanceData.length;i++){
    attendanceData[i].checkIn=''; attendanceData[i].checkOut=''; attendanceData[i].late=0;
    document.getElementById('in'+i).innerHTML='<button onclick="checkIn('+i+')">Check In</button>';
    document.getElementById('out'+i).innerHTML='<button onclick="checkOut('+i+')">Check Out</button>';
    document.getElementById('late'+i).innerText='0';
    document.getElementById('off'+i).innerText=attendanceData[i].off?'Off':'Work';
    document.getElementById('row'+i).classList.remove('attended');
  }
}

function generateReport(){
  let w=window.open('','Monthly Report','width=900,height=600');
  w.document.write('<h2>Monthly Attendance Report</h2>');
  w.document.write('<table border="1"><tr><th>Date</th><th>Shift</th><th>Check In</th><th>Check Out</th><th>Late</th><th>Off/Work</th><th>Poya</th></tr>');
  for(let i=0;i<attendanceData.length;i++){
    const d=document.getElementById('row'+i).children[0].innerText;
    const shift=document.getElementById('shift'+i).innerText;
    const inTime=attendanceData[i].checkIn;
    const outTime=attendanceData[i].checkOut;
    const late=attendanceData[i].late;
    const off=attendanceData[i].off?'Off':'Work';
    const poya=attendanceData[i].poya?'Yes':'No';
    w.document.write(`<tr><td>${d}</td><td>${shift}</td><td>${inTime}</td><td>${outTime}</td><td>${late}</td><td>${off}</td><td>${poya}</td></tr>`);
  }
  w.document.write('</table>');
}
</script>

</body>
</html>
