<!DOCTYPE html>
<html>
<head>
  <title>Shift Schedule</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #121212;
      color: #fff;
      text-align: center;
      margin: 20px;
    }
    h2 {margin-bottom: 10px;}
    table {border-collapse: collapse; width: 95%; margin: auto;}
    th, td {border: 1px solid #fff; padding: 5px; text-align: center;}
    th {background-color: #333;}
    .today {background-color: #ffeb3b; color: #000; font-weight: bold;}
    .dayShift {background-color: #4caf50; color: #000;}
    .nightShift {background-color: #2196f3; color: #fff;}
    .attended {background-color: #ff9800; color: #000;} /* highlight attended day */
    input, button {margin: 5px; padding: 5px;}
    .resetBtn {background-color: #f44336; color: #fff; border: none; padding: 8px 12px; cursor: pointer;}
  </style>
</head>
<body>

<h2>Shift Schedule (2 Day / 2 Night)</h2>

<p>Current Time: <span id="time"></span></p>

<label>Employee Name: <input type="text" id="empName" placeholder="Enter name"></label>
<label>EPF No: <input type="text" id="epfNo" placeholder="Enter EPF"></label><br>

<label>Start Date: <input type="date" id="start"></label>
<button onclick="generateSchedule()">Generate</button>
<button class="resetBtn" onclick="resetAttendance()">Reset Check In/Out</button>

<table id="scheduleTable">
  <thead>
    <tr>
      <th>Date</th>
      <th>Day</th>
      <th>Shift</th>
      <th>Employee</th>
      <th>EPF No</th>
      <th>Check In</th>
      <th>Check Out</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
// Live time display
function updateTime() {
  const now = new Date();
  document.getElementById("time").innerText = now.toLocaleTimeString();
}
setInterval(updateTime, 1000);
updateTime();

let attendanceData = []; // Stores check in/out data

function generateSchedule() {
  const startInput = document.getElementById("start").value;
  if (!startInput) return alert("Please select a start date");

  const empName = document.getElementById("empName").value || "-";
  const epfNo = document.getElementById("epfNo").value || "-";

  const startDate = new Date(startInput);
  const tbody = document.querySelector("#scheduleTable tbody");
  tbody.innerHTML = "";

  const shifts = ["Day", "Day", "Night", "Night"];
  let shiftIndex = 0;
  const today = new Date();
  const days = ["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"];

  attendanceData = []; // reset attendance

  for (let i = 0; i < 30; i++) {
    let current = new Date(startDate);
    current.setDate(startDate.getDate() + i);

    const shiftType = shifts[shiftIndex];
    const shiftClass = shiftType === "Day" ? "dayShift" : "nightShift";

    // Initialize attendance for this row
    attendanceData.push({checkIn: "", checkOut: ""});

    // Create row
    const tr = document.createElement("tr");
    tr.id = `row${i}`;
    if (current.toDateString() === today.toDateString()) {
      tr.classList.add("today");
    }

    tr.innerHTML = `
      <td>${current.toLocaleDateString('en-GB')}</td>
      <td>${days[current.getDay()]}</td>
      <td class="${shiftClass}" id="shift${i}">${shiftType}</td>
      <td>${empName}</td>
      <td>${epfNo}</td>
      <td id="checkIn${i}"><button onclick="checkIn(${i})">Check In</button></td>
      <td id="checkOut${i}"><button onclick="checkOut(${i})">Check Out</button></td>
    `;
    tbody.appendChild(tr);

    shiftIndex = (shiftIndex + 1) % shifts.length;
  }
}

function checkIn(index) {
  const now = new Date().toLocaleTimeString();
  attendanceData[index].checkIn = now;
  document.getElementById(`checkIn${index}`).innerHTML = now;

  // highlight row as attended
  document.getElementById(`row${index}`).classList.add("attended");
}

function checkOut(index) {
  if (!attendanceData[index].checkIn) {
    alert("Please Check In first!");
    return;
  }
  const now = new Date().toLocaleTimeString();
  attendanceData[index].checkOut = now;
  document.getElementById(`checkOut${index}`).innerHTML = now;

  // highlight row as attended (if not already)
  document.getElementById(`row${index}`).classList.add("attended");
}

function resetAttendance() {
  for (let i = 0; i < attendanceData.length; i++) {
    attendanceData[i].checkIn = "";
    attendanceData[i].checkOut = "";
    document.getElementById(`checkIn${i}`).innerHTML = `<button onclick="checkIn(${i})">Check In</button>`;
    document.getElementById(`checkOut${i}`).innerHTML = `<button onclick="checkOut(${i})">Check Out</button>`;
    document.getElementById(`row${i}`).classList.remove("attended");
  }
}
</script>

</body>
</html>
