<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shift Tracker PRO</title>

<style>
body {font-family: Arial; background:#121212; color:#fff; text-align:center;}
table {border-collapse: collapse; width:95%; margin:auto;}
th, td {border:1px solid #fff; padding:6px;}
th {background:#333;}

.dayShift {background:#4caf50; color:#000;}
.nightShift {background:#2196f3;}
.offDay {background:#f44336;}

.attended {background:#ffeb3b; color:#000;}
.poya {background:#ffa726; color:#000;}

button {padding:5px; margin:3px;}
</style>
</head>

<body>

<h2>Shift Tracker PRO</h2>
<p id="time"></p>

Name: <input id="empName">
EPF: <input id="empEPF"><br>

Start Date: <input type="date" id="start">
<button onclick="generate()">Generate</button>
<button onclick="report()">Report</button>

<table id="tbl">
<thead>
<tr>
<th>Date</th><th>Day</th><th>Shift</th>
<th>In</th><th>Out</th><th>Late</th>
<th>Status</th><th>Poya</th><th>Reset</th>
</tr>
</thead>
<tbody></tbody>
</table>

<script>

// clock
setInterval(()=>{
 document.getElementById("time").innerText=new Date().toLocaleTimeString();
},1000);

// load saved data
let data = JSON.parse(localStorage.getItem("shiftData")) || {};

function getMonthKey(date){
  return date.getFullYear()+"-"+(date.getMonth()+1);
}

function generate(){
 const start=document.getElementById("start").value;
 if(!start) return alert("Select date");

 const sDate=new Date(start);
 const monthKey=getMonthKey(sDate);

 if(!data[monthKey]) data[monthKey]=[];

 const tbody=document.querySelector("#tbl tbody");
 tbody.innerHTML="";

 const shifts=['Day','Day','Night','Night','Off','Off','Off'];
 const days=['Sun','Mon','Tue','Wed','Thu','Fri','Sat'];

 for(let i=0;i<30;i++){
   let d=new Date(sDate);
   d.setDate(sDate.getDate()+i);

   if(!data[monthKey][i]){
     data[monthKey][i]={in:'',out:'',late:0,off:shifts[i%7]==='Off',poya:false};
   }

   const shift=shifts[i%7];
   const shiftClass=shift==='Day'?'dayShift':shift==='Night'?'nightShift':'offDay';

   const tr=document.createElement("tr");
   tr.id="r"+i;

   tr.innerHTML=`
   <td>${d.toLocaleDateString('en-GB')}</td>
   <td>${days[d.getDay()]}</td>
   <td class="${shiftClass}" id="s${i}">${shift}</td>

   <td id="in${i}">${data[monthKey][i].in || `<button onclick="cin(${i})">In</button>`}</td>
   <td id="out${i}">${data[monthKey][i].out || `<button onclick="cout(${i})">Out</button>`}</td>
   <td id="late${i}">${data[monthKey][i].late}</td>

   <td id="st${i}">${data[monthKey][i].off?'Off':'Work'}</td>
   <td id="py${i}">${data[monthKey][i].poya?'Yes':`<button onclick="poya(${i})">Poya</button>`}</td>
   <td><button onclick="resetDay(${i})">Reset</button></td>
   `;

   if(data[monthKey][i].in) tr.classList.add("attended");
   if(data[monthKey][i].poya) tr.classList.add("poya");

   tbody.appendChild(tr);
 }

 localStorage.setItem("shiftData",JSON.stringify(data));
}

// check in
function cin(i){
 const now=new Date();
 const monthKey=Object.keys(data).slice(-1)[0];

 data[monthKey][i].in=now.toLocaleTimeString();

 if(data[monthKey][i].off){
   data[monthKey][i].off=false;
 }

 lateCalc(i,now);
 saveReload();
}

// check out
function cout(i){
 const now=new Date();
 const monthKey=Object.keys(data).slice(-1)[0];

 if(!data[monthKey][i].in) return alert("Check In first");

 data[monthKey][i].out=now.toLocaleTimeString();
 saveReload();
}

// late calc
function lateCalc(i,now){
 const monthKey=Object.keys(data).slice(-1)[0];

 if(data[monthKey][i].off || data[monthKey][i].poya){
   data[monthKey][i].late=0;
   return;
 }

 const shift=document.getElementById("s"+i).innerText;
 const start=shift==='Day'?6:18;

 data[monthKey][i].late=Math.max(0,(now.getHours()-start)*60+now.getMinutes());
}

// mark poya
function poya(i){
 const monthKey=Object.keys(data).slice(-1)[0];
 data[monthKey][i].poya=true;
 saveReload();
}

// reset single day
function resetDay(i){
 const monthKey=Object.keys(data).slice(-1)[0];

 data[monthKey][i]={in:'',out:'',late:0,off:true,poya:false};
 saveReload();
}

// save + reload UI
function saveReload(){
 localStorage.setItem("shiftData",JSON.stringify(data));
 generate();
}

// report
function report(){
 const monthKey=Object.keys(data).slice(-1)[0];
 let w=window.open("","","width=800,height=600");

 w.document.write("<h2>Monthly Report</h2><table border=1>");
 w.document.write("<tr><th>Date</th><th>Shift</th><th>In</th><th>Out</th><th>Late</th><th>Status</th><th>Poya</th></tr>");

 for(let i=0;i<data[monthKey].length;i++){
   const d=document.getElementById("r"+i).children[0].innerText;
   const s=document.getElementById("s"+i).innerText;

   w.document.write(`<tr>
   <td>${d}</td>
   <td>${s}</td>
   <td>${data[monthKey][i].in}</td>
   <td>${data[monthKey][i].out}</td>
   <td>${data[monthKey][i].late}</td>
   <td>${data[monthKey][i].off?'Off':'Work'}</td>
   <td>${data[monthKey][i].poya?'Yes':'No'}</td>
   </tr>`);
 }

 w.document.write("</table>");
}

</script>

</body>
</html>
