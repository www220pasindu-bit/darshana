<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shift Tracker</title>

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

<h2>Shift Tracker (12h)</h2>
<p>Time: <span id="time"></span></p>

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
<th>Status</th><th>Poya</th>
</tr>
</thead>
<tbody></tbody>
</table>

<script>

// clock
setInterval(()=>{
 document.getElementById("time").innerText = new Date().toLocaleTimeString();
},1000);

let data=[];

function generate(){
 const start=document.getElementById("start").value;
 if(!start) return alert("Select date");

 const sDate=new Date(start);
 const tbody=document.querySelector("#tbl tbody");
 tbody.innerHTML="";
 data=[];

 const shifts=['Day','Day','Night','Night','Off','Off','Off'];
 const days=['Sun','Mon','Tue','Wed','Thu','Fri','Sat'];

 for(let i=0;i<30;i++){
   let d=new Date(sDate);
   d.setDate(sDate.getDate()+i);

   const shift=shifts[i%7];
   const shiftClass=shift==='Day'?'dayShift':shift==='Night'?'nightShift':'offDay';

   data.push({in:'',out:'',late:0,off:shift==='Off',poya:false});

   const tr=document.createElement("tr");
   tr.id="r"+i; // ❗ no auto highlight

   tr.innerHTML=`
   <td>${d.toLocaleDateString('en-GB')}</td>
   <td>${days[d.getDay()]}</td>
   <td class="${shiftClass}" id="s${i}">${shift}</td>

   <td id="in${i}"><button onclick="cin(${i})">In</button></td>
   <td id="out${i}"><button onclick="cout(${i})">Out</button></td>
   <td id="late${i}">0</td>

   <td id="st${i}">${shift==='Off'?'Off':'Work'}</td>
   <td id="py${i}"><button onclick="poya(${i})">Poya</button></td>
   `;

   tbody.appendChild(tr);
 }
}

function cin(i){
 const now=new Date();
 data[i].in=now.toLocaleTimeString();
 document.getElementById("in"+i).innerText=data[i].in;

 if(data[i].off){
   data[i].off=false;
   document.getElementById("st"+i).innerText="Work";
 }

 lateCalc(i,now);

 // highlight AFTER check-in only
 document.getElementById("r"+i).classList.add("attended");
}

function cout(i){
 if(!data[i].in) return alert("Check In first");

 const now=new Date();
 data[i].out=now.toLocaleTimeString();
 document.getElementById("out"+i).innerText=data[i].out;

 document.getElementById("r"+i).classList.add("attended");
}

function lateCalc(i,now){
 if(data[i].off || data[i].poya){
   data[i].late=0;
   document.getElementById("late"+i).innerText=0;
   return;
 }

 const shift=document.getElementById("s"+i).innerText;
 const start=shift==='Day'?6:18;

 const late=Math.max(0,(now.getHours()-start)*60+now.getMinutes());
 data[i].late=late;
 document.getElementById("late"+i).innerText=late;
}

function poya(i){
 data[i].poya=true;
 document.getElementById("py"+i).innerText="Yes";
 document.getElementById("r"+i).classList.add("poya");
}

function report(){
 let w=window.open("","","width=800,height=600");
 w.document.write("<h2>Report</h2><table border=1>");
 w.document.write("<tr><th>Date</th><th>Shift</th><th>In</th><th>Out</th><th>Late</th><th>Status</th><th>Poya</th></tr>");

 for(let i=0;i<data.length;i++){
   const d=document.getElementById("r"+i).children[0].innerText;
   const s=document.getElementById("s"+i).innerText;

   w.document.write(`<tr>
   <td>${d}</td>
   <td>${s}</td>
   <td>${data[i].in}</td>
   <td>${data[i].out}</td>
   <td>${data[i].late}</td>
   <td>${data[i].off?'Off':'Work'}</td>
   <td>${data[i].poya?'Yes':'No'}</td>
   </tr>`);
 }

 w.document.write("</table>");
}

</script>

</body>
</html>
