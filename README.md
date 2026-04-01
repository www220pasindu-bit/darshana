<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shift Attendance Tracker</title>
<style>
body { font-family: Arial; padding: 20px; }
input, select { padding: 5px; margin: 5px; }
button { padding: 8px 15px; margin: 5px; }
table { border-collapse: collapse; margin-top: 20px; width: 100%; }
th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
.work { background-color: #c8f7c5; }
.off { background-color: #f7c5c5; }
.poya { background-color: #ffe4b5; }
.marked { background-color: #f9d56e; }
</style>
</head>
<body>

<h1>Shift Attendance Tracker</h1>

<h3>Employee Info</h3>
Name: <input type="text" id="empName" placeholder="Employee Name">
EPF: <input type="text" id="empEPF" placeholder="EPF Number">
<button onclick="saveEmployee()">Save Info</button>

<h3>Select Month</h3>
<select id="monthSelect" onchange="generateCalendar()">
<option value="0">January</option>
<option value="1">February</option>
<option value="2">March</option>
<option value="3">April</option>
<option value="4">May</option>
<option value="5">June</option>
<option value="6">July</option>
<option value="7">August</option>
<option value="8">September</option>
<option value="9">October</option>
<option value="10">November</option>
<option value="11">December</option>
</select>

<h3>Actions</h3>
<button onclick="checkIn()">Check-in</button>
<button onclick="checkOut()">Check-out</button>
<button onclick="resetDay()">Reset</button>
<button onclick="generateReport()">Monthly Report</button>

<table id="calendar">
<tr>
<th>Date</th>
<th>Shift</th>
<th>Check-in</th>
<th>Check-out</th>
<th>Late (min)</th>
<th>Poya</th>
<th>Marked</th>
</tr>
</table>

<script>
// Shift pattern & Poya dates (example)
const workPattern = ['Day','Day','Night','Night','Off','Off','Off'];
const poyaDates = ['2026-04-05','2026-04-19','2026-05-04'];
let shifts = [];
let employee = {name:'', epf:''};

// Save employee info
function saveEmployee(){
    const name = document.getElementById('empName').value.trim();
    const epf = document.getElementById('empEPF').value.trim();
    if(name && epf){
        employee = {name, epf};
        localStorage.setItem('employee', JSON.stringify(employee));
        alert('Employee info saved!');
    } else alert('Enter Name & EPF number!');
}

// Shift & Poya helpers
function getShift(index){ return workPattern[index % workPattern.length]; }
function isPoya(dateStr){ return poyaDates.includes(dateStr); }

// Generate calendar
function generateCalendar(){
    shifts = [];
    const calendar = document.getElementById('calendar');
    calendar.innerHTML = `<tr>
    <th>Date</th><th>Shift</th><th>Check-in</th><th>Check-out</th><th>Late (min)</th><th>Poya</th><th>Marked</th>
    </tr>`;

    const today = new Date();
    const month = parseInt(document.getElementById('monthSelect').value);
    const year = today.getFullYear();
    const monthDays = new Date(year, month+1, 0).getDate();
    const savedShifts = JSON.parse(localStorage.getItem('shifts')) || [];
    const savedEmp = JSON.parse(localStorage.getItem('employee'));
    if(savedEmp) employee = savedEmp;

    for(let i=1;i<=monthDays;i++){
        const shift = getShift(i-1);
        const dateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(i).padStart(2,'0')}`;
        const isPoyaDay = isPoya(dateStr);
        const saved = savedShifts.find(s=>s.date===dateStr);
        const record = saved || {date:dateStr, shift, checkIn:'', checkOut:'', late:0, marked:false, isPoya:isPoyaDay, employee:employee};
        shifts.push(record);

        const tr = document.createElement('tr');
        tr.id = 'day-'+i;
        tr.className = record.isPoya ? 'poya' : (record.shift==='Off' ? 'off' : 'work');
        if(record.marked) tr.className='marked';
        tr.innerHTML = `<td>${i}</td><td>${shift}</td><td id="in-${i}">${record.checkIn}</td><td id="out-${i}">${record.checkOut}</td><td id="late-${i}">${record.late}</td><td>${record.isPoya?'Yes':'No'}</td><td><button onclick="markDay(${i})">Mark</button></td>`;
        calendar.appendChild(tr);
    }
    localStorage.setItem('shifts', JSON.stringify(shifts));
}

// Check-in
function checkIn(){
    const today = new Date().getDate();
    const now = new Date();
    const shiftStart = shifts[today-1].shift==='Day'?8:20;
    const lateMins = now.getHours()>shiftStart ? (now.getHours()-shiftStart)*60+now.getMinutes() : 0;

    shifts[today-1].checkIn = now.toLocaleTimeString();
    shifts[today-1].late = lateMins;
    shifts[today-1].employee = employee;

    document.getElementById('in-'+today).innerText = shifts[today-1].checkIn;
    document.getElementById('late-'+today).innerText = lateMins;
    localStorage.setItem('shifts', JSON.stringify(shifts));
}

// Check-out
function checkOut(){
    const today = new Date().getDate();
    const now = new Date();
    shifts[today-1].checkOut = now.toLocaleTimeString();
    document.getElementById('out-'+today).innerText = shifts[today-1].checkOut;
    localStorage.setItem('shifts', JSON.stringify(shifts));
}

// Reset
function resetDay(){
    const today = new Date().getDate();
    shifts[today-1].checkIn='';
    shifts[today-1].checkOut='';
    shifts[today-1].late=0;
    shifts[today-1].marked=false;
    const tr = document.getElementById('day-'+today);
    tr.className = shifts[today-1].isPoya?'poya':(shifts[today-1].shift==='Off'?'off':'work');
    document.getElementById('in-'+today).innerText='';
    document.getElementById('out-'+today).innerText='';
    document.getElementById('late-'+today).innerText='';
    localStorage.setItem('shifts', JSON.stringify(shifts));
}

// Mark manual day
function markDay(day){
    shifts[day-1].marked = !shifts[day-1].marked;
    const tr = document.getElementById('day-'+day);
    tr.className = shifts[day-1].marked?'marked':(shifts[day-1].isPoya?'poya':(shifts[day-1].shift==='Off'?'off':'work'));
    localStorage.setItem('shifts', JSON.stringify(shifts));
}

// Generate monthly report
function generateReport(){
    let reportWindow = window.open("", "Monthly Report", "width=900,height=600");
    reportWindow.document.write("<h1>Monthly Attendance Report</h1>");
    reportWindow.document.write(`<h3>Employee: ${employee.name} | EPF: ${employee.epf}</h3>`);
    reportWindow.document.write("<table border='1'><tr><th>Date</th><th>Shift</th><th>Check-in</th><th>Check-out</th><th>Late (min)</th><th>Poya</th><th>Marked</th></tr>");
    shifts.forEach(s=>{
        reportWindow.document.write(`<tr>
        <td>${s.date.split('-')[2]}</td>
        <td>${s.shift}</td>
        <td>${s.checkIn}</td>
        <td>${s.checkOut}</td>
        <td>${s.late}</td>
        <td>${s.isPoya?'Yes':'No'}</td>
        <td>${s.marked?'Yes':'No'}</td>
        </tr>`);
    });
    reportWindow.document.write("</table>");
}

// Init calendar on load
generateCalendar();

</script>

</body>
</html>
