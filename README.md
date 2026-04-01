<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Fingerprint Shift App</title>
  <style>
    body { font-family: Arial; padding: 20px; }
    button { padding: 10px 20px; margin: 5px; }
    table { border-collapse: collapse; margin-top: 20px; width: 100%; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    .work { background-color: #c8f7c5; }
    .off { background-color: #f7c5c5; }
  </style>
</head>
<body>
  <h1>Shift Attendance Tracker</h1>
  <button onclick="checkIn()">Check-in</button>
  <button onclick="checkOut()">Check-out</button>
  <button onclick="resetDay()">Reset</button>

  <table id="calendar">
    <tr>
      <th>Date</th>
      <th>Shift</th>
      <th>Check-in</th>
      <th>Check-out</th>
      <th>Late (min)</th>
    </tr>
  </table>

  <script>
    // Shift pattern: 4-on, 3-off (Day/Night rotation)
    const workPattern = ['Day','Day','Night','Night','Off','Off','Off'];
    const shifts = [];

    function getShift(dateIndex){
      return workPattern[dateIndex % workPattern.length];
    }

    function initCalendar(){
      const calendar = document.getElementById('calendar');
      const today = new Date();
      const monthDays = new Date(today.getFullYear(), today.getMonth()+1, 0).getDate();

      for(let i=1; i<=monthDays; i++){
        const shift = getShift(i-1);
        shifts.push({date: i, shift, checkIn: '', checkOut: '', late: 0});
        const tr = document.createElement('tr');
        tr.id = 'day-'+i;
        tr.className = shift==='Off' ? 'off' : 'work';
        tr.innerHTML = `<td>${i}</td><td>${shift}</td><td id="in-${i}"></td><td id="out-${i}"></td><td id="late-${i}"></td>`;
        calendar.appendChild(tr);
      }

      // Load saved data from localStorage
      const saved = JSON.parse(localStorage.getItem('shifts'));
      if(saved){
        saved.forEach(s => {
          if(s.checkIn) document.getElementById('in-'+s.date).innerText = s.checkIn;
          if(s.checkOut) document.getElementById('out-'+s.date).innerText = s.checkOut;
          if(s.late) document.getElementById('late-'+s.date).innerText = s.late;
        });
      }
    }

    function checkIn(){
      const today = new Date().getDate();
      const now = new Date();
      shifts[today-1].checkIn = now.toLocaleTimeString();

      // Late logic: Day shift 08:00, Night shift 20:00
      const shiftStart = shifts[today-1].shift === 'Day' ? 8 : 20;
      const lateHours = now.getHours() - shiftStart;
      const lateMinutes = lateHours > 0 ? lateHours*60 + now.getMinutes() : 0;
      shifts[today-1].late = lateMinutes;

      document.getElementById('in-'+today).innerText = shifts[today-1].checkIn;
      document.getElementById('late-'+today).innerText = shifts[today-1].late;
      localStorage.setItem('shifts', JSON.stringify(shifts));
    }

    function checkOut(){
      const today = new Date().getDate();
      const now = new Date();
      shifts[today-1].checkOut = now.toLocaleTimeString();
      document.getElementById('out-'+today).innerText = shifts[today-1].checkOut;
      localStorage.setItem('shifts', JSON.stringify(shifts));
    }

    function resetDay(){
      const today = new Date().getDate();
      shifts[today-1].checkIn = '';
      shifts[today-1].checkOut = '';
      shifts[today-1].late = 0;
      document.getElementById('in-'+today).innerText = '';
      document.getElementById('out-'+today).innerText = '';
      document.getElementById('late-'+today).innerText = '';
      localStorage.setItem('shifts', JSON.stringify(shifts));
    }

    initCalendar();
  </script>
</body>
</html>
