DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Student Attendance Tracker</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
<header class="topbar">
  <h1>Student Attendance Tracker</h1>
  <p>Manage students and daily attendance</p>
</header>
<main class="container">
  <section class="summary-grid">
    <article class="card">Total Students <strong id="totalStudents">0</strong></article>
    <article class="card">Present Today <strong id="presentToday">0</strong></article>
    <article class="card">Absent Today <strong id="absentToday">0</strong></article>
    <article class="card">Overall Attendance <strong id="overallPercent">0%</strong></article>
  </section>
  <section class="panel">
    <h2>Add / Edit Student</h2>
    <form id="studentForm">
      <input id="studentId" placeholder="Student ID" required>
      <input id="studentName" placeholder="Student Name" required>
      <input id="department" placeholder="Department" required>
      <button type="submit">Save Student</button>
      <button type="button" id="clearForm" class="secondary">Clear</button>
    </form>
  </section>
  <section class="panel">
    <h2>Attendance Register</h2>
    <div class="toolbar">
      <label>Date <input type="date" id="attendanceDate" required></label>
      <input id="searchInput" placeholder="Search name, ID, department">
      <select id="statusFilter">
        <option>All</option><option>Present</option><option>Absent</option>
        <option>Not Marked</option>
      </select>
      <button type="button" id="exportBtn">Export CSV</button>
    </div>
    <div class="table-wrap"><table>
      <thead><tr><th>ID</th><th>Name</th><th>Department</th>
      <th>Mark Attendance</th><th>Attendance %</th><th>Actions</th></tr></thead>
      <tbody id="studentRows"></tbody>
    </table></div>
    <p id="message"></p>
  </section>
  <section class="panel">
    <h2>Student Attendance History</h2>
    <select id="historyStudent"><option value="">Choose student</option></select>
    <div id="historyOutput"></div>
  </section>
</main>
<script src="script.js"></script>
</body></html>
* { box-sizing: border-box; }
:root { --navy:#17365d; --blue:#2677d4; --pale:#eef4fb;         --line:#d1dbea; --green:#187647; --red:#b42318; } body { margin:0; font-family:Arial,sans-serif; background:var(--pale); color:#243247; } .topbar { background:var(--navy); color:white; padding:22px max(20px,calc((100% - 1200px)/2)); }
.topbar h1 { margin:0 0 6px; font-size:28px; }
.topbar p { margin:0; color:#dce8f7; }
.container { max-width:1240px; margin:22px auto; padding:0 18px; }
.summary-grid { display:grid; grid-template-columns:repeat(4,1fr); gap:14px; margin-bottom:18px; } .card,.panel { background:white; border:1px solid var(--line); border-radius:12px; padding:18px; }
.card { color:#66758a; }
.card strong { display:block; color:var(--navy); font-size:27px; margin-top:8px; } .panel { margin-bottom:18px; } h2 { margin:0 0 14px; color:var(--navy); font-size:20px; } form,.toolbar { display:flex; flex-wrap:wrap; align-items:center; gap:10px; } input,select,button { padding:10px 12px; border:1px solid #bdc9d8; border-radius:6px; font-size:14px; } button { background:var(--blue); color:white; border:0; cursor:pointer; } button.secondary { background:#64748b; } button.danger { background:var(--red); } .table-wrap { width:100%; overflow-x:auto; } table { width:100%; border-collapse:collapse; margin-top:14px; background:white; } th,td { padding:10px; border-bottom:1px solid var(--line); text-align:left; } th { background:#e3ecf8; color:var(--navy); } .low { background:#fff1d6; color:#9a3412; font-weight:bold; }
.present { color:var(--green); font-weight:bold; }
.absent { color:var(--red); font-weight:bold; }
.history-item { padding:8px; border-bottom:1px solid var(--line); }
#message { min-height:18px; color:var(--navy); }
@media(max-width:850px) { .summary-grid { grid-template-columns:repeat(2,1fr); } }
@media(max-width:520px) {
 .summary-grid { grid-template-columns:1fr 1fr; gap:8px; }  .card,.panel { padding:12px; }  form,.toolbar { flex-direction:column; align-items:stretch; }  input,select,button { width:100%; }
 const $ = id => document.getElementById(id); const KEY = "studentAttendanceTracker_v1", LIMIT = 75; let db = JSON.parse(localStorage.getItem(KEY) ||   '{"students":[],"records":[]}'); let editingId = null; $("attendanceDate").value = new Date().toISOString().slice(0,10);
function save() { localStorage.setItem(KEY, JSON.stringify(db)); } function esc(v) {   return String(v ?? "").replace(/[&<>"']/g, c =>     ({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c])); } function recordFor(id,date) {   return db.records.find(r => r.studentId === id && r.date === date); } function stats(id) {   const records = db.records.filter(r => r.studentId === id);   const present = records.filter(r => r.status === "Present").length;   return {total:records.length,present,     percent:records.length ? Math.round(present/records.length*100) : 0}; } function message(t) { $("message").textContent = t; }
$("studentForm").addEventListener("submit", e => {
  e.preventDefault();   const id=$("studentId").value.trim(), name=$("studentName").value.trim();   const department=$("department").value.trim();   if (!id || !name || !department) return message("Fill in all fields.");   if (db.students.some(s => s.id.toLowerCase()===id.toLowerCase() && s.id!==editingId))     return message("Student ID already exists.");   if (editingId) {     const s=db.students.find(s=>s.id===editingId);     if (s) Object.assign(s,{id,name,department});     db.records.forEach(r=>{if(r.studentId===editingId)r.studentId=id;});     editingId=null;   } else db.students.push({id,name,department});   save(); $("studentForm").reset(); render(); message("Student saved."); }); $("clearForm").addEventListener("click",()=>{   editingId=null; $("studentForm").reset(); message("Form cleared."); }); function editStudent(id) {   const s=db.students.find(s=>s.id===id); if(!s)return;   editingId=id; $("studentId").value=s.id;   $("studentName").value=s.name; $("department").value=s.department;   window.scrollTo({top:0,behavior:"smooth"}); } function deleteStudent(id) {   if(!confirm("Delete this student and their attendance records?"))return;   db.students=db.students.filter(s=>s.id!==id);   db.records=db.records.filter(r=>r.studentId!==id);   save(); render(); message("Student deleted."); } function mark(id,status) {   const date=$("attendanceDate").value;   if(!date)return message("Choose a date first.");   const old=recordFor(id,date);
  if(old) old.status=status; // update existing entry; do not duplicate   else db.records.push({studentId:id,date,status});   save(); render(); message("Attendance saved for "+date+"."); }
function render() {   const date=$("attendanceDate").value;
  const q=$("searchInput").value.toLowerCase().trim();   const filter=$("statusFilter").value;   const rows=$("studentRows"); rows.innerHTML="";   db.students.filter(s=>{     const text=(s.id+" "+s.name+" "+s.department).toLowerCase();     const r=recordFor(s.id,date), status=r?r.status:"Not Marked";     return text.includes(q)&&(filter==="All"||filter===status);   }).forEach(s=>{
    const r=recordFor(s.id,date), status=r?r.status:"Not Marked", st=stats(s.id);     const tr=document.createElement("tr");     tr.innerHTML=`<td>${esc(s.id)}</td><td>${esc(s.name)}</td>       <td>${esc(s.department)}</td><td>
      <button data-present>Present</button>
      <button data-absent class="secondary">Absent</button>
      <div class="${status==="Present"?"present":status==="Absent"?"absent":""}">${status}</div>       </td><td class="${st.total&&st.percent<LIMIT?"low":""}">${st.percent}% ${st.total&&st.percent<LIMIT?"LOW":""}</td>       <td><button data-edit>Edit</button><button data-delete class="danger">Delete</button></td>`;     tr.querySelector("[data-present]").onclick=()=>mark(s.id,"Present");     tr.querySelector("[data-absent]").onclick=()=>mark(s.id,"Absent");     tr.querySelector("[data-edit]").onclick=()=>editStudent(s.id);     tr.querySelector("[data-delete]").onclick=()=>deleteStudent(s.id);     rows.appendChild(tr);   });   const present=db.students.filter(s=>recordFor(s.id,date)?.status==="Present").length;   const absent=db.students.filter(s=>recordFor(s.id,date)?.status==="Absent").length;   const totalPresent=db.records.filter(r=>r.status==="Present").length;   $("totalStudents").textContent=db.students.length;   $("presentToday").textContent=present; $("absentToday").textContent=absent;   $("overallPercent").textContent=db.records.length?
    Math.round(totalPresent/db.records.length*100)+"%":"0%";
  const sel=$("historyStudent"), old=sel.value;   sel.innerHTML='<option value="">Choose student</option>'+db.students.map(s=>     `<option value="${esc(s.id)}">${esc(s.name)} (${esc(s.id)})</option>`).join("");   sel.value=old; history(); } function history() {   const id=$("historyStudent").value, box=$("historyOutput");   if(!id){box.innerHTML="<p>Select a student to view history.</p>";return;}   const s=db.students.find(s=>s.id===id), st=stats(id);   const list=db.records.filter(r=>r.studentId===id).sort((a,b)=>b.date.localeCompare(a.date));   box.innerHTML=`<p><b>${esc(s?.name||"")}</b> — ${st.present}/${st.total} present (${st.percent}%).</p>`+     (st.total&&st.percent<LIMIT?'<p class="low">Attendance is below 75%.</p>':"")+     list.map(r=>`<div class="history-item">${esc(r.date)} — ${esc(r.status)}</div>`).join(""); } function exportCSV() {   const lines=[["ID","Name","Department","Date","Status","Attendance %"]];   db.students.forEach(s=>{     const st=stats(s.id), recs=db.records.filter(r=>r.studentId===s.id);     if(!recs.length)lines.push([s.id,s.name,s.department,"","Not Marked",st.percent+"%"]);     recs.forEach(r=>lines.push([s.id,s.name,s.department,r.date,r.status,st.percent+"%"]));   });   const csv=lines.map(row=>row.map(v=>'"'+String(v).replace(/"/g,'""')+'"').join(",")).join("\n");   const url=URL.createObjectURL(new Blob([csv],{type:"text/csv"}));
  const a=document.createElement("a"); a.href=url; a.download="attendance-report.csv"; a.click();   URL.revokeObjectURL(url);
}
$("attendanceDate").addEventListener("change",render);
$("searchInput").addEventListener("input",render);
$("statusFilter").addEventListener("change",render);
$("historyStudent").addEventListener("change",history); $("exportBtn").addEventListener("click",exportCSV); render();
