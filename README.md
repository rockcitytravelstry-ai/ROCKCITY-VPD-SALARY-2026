<link rel="manifest" href="manifest.json">
 import { useState, useMemo, useEffect } from "react";

// --- CONSTANTS ------------------------------------------------
const MONTHS = ["January","February","March","April","May","June","July","August","September","October","November","December"];
const DEPTS = ["Travels","Pharmaceutical"];
const DESIGNATIONS = ["Driver","Manager","Accountant","Pharmacist","Delivery","Supervisor","Staff","Other"];
const ATTENDANCE_STATUS = ["Present","Absent","Half Day","Leave"];
const ROLES = ["Admin","Manager","Employee"];

const INITIAL_EMPLOYEES = [
  { id:1, name:"Mr.Rathinakumar", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543210", empId:"RC001", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:2, name:"Mr.Mohamed Rasheed", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543211", empId:"RC002", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:3, name:"Mr.Arumugam P Suresh", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543212", empId:"RC003", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:4, name:"Mr.Saravanan T", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543213", empId:"RC004", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:5, name:"Mr.Azad Kalam Ali", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543214", empId:"RC005", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:6, name:"Mr.Dinesh Krishnan", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543215", empId:"RC006", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:7, name:"Mr.Ravichandran", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543216", empId:"RC007", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:8, name:"Mr.Katharsha", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543217", empId:"RC008", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:9, name:"Mr.Aarif", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543218", empId:"RC009", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:10, name:"Mr.Vetri Selvan", basicSalary:3500, workingDays:29, dept:"Travels", designation:"Driver", mobile:"9876543219", empId:"RC010", joined:"2023-01-01", photo:"", role:"Employee" },
  { id:11, name:"Mr.Alrasik", basicSalary:12500, workingDays:29, dept:"Travels", designation:"Manager", mobile:"9876543220", empId:"RC011", joined:"2022-06-01", photo:"", role:"Manager" },
  { id:12, name:"Mr.Bhuvaneswari", basicSalary:15000, workingDays:26, dept:"Travels", designation:"Supervisor", mobile:"9876543221", empId:"RC012", joined:"2022-06-01", photo:"", role:"Employee" },
  { id:13, name:"Mrs.Kavitha", basicSalary:7500, workingDays:26, dept:"Travels", designation:"Staff", mobile:"9876543222", empId:"RC013", joined:"2022-06-01", photo:"", role:"Employee" },
  { id:101, name:"Mr.R.Manivannan", basicSalary:15000, workingDays:26, dept:"Pharmaceutical", designation:"Manager", mobile:"9876500001", empId:"PH001", joined:"2022-01-01", photo:"", role:"Manager" },
  { id:102, name:"Ms.Nivedha", basicSalary:15000, workingDays:26, dept:"Pharmaceutical", designation:"Pharmacist", mobile:"9876500002", empId:"PH002", joined:"2022-01-01", photo:"", role:"Employee" },
  { id:103, name:"Mr.Pushparaj", basicSalary:15000, workingDays:26, dept:"Pharmaceutical", designation:"Delivery", mobile:"9876500003", empId:"PH003", joined:"2022-01-01", photo:"", role:"Employee" },
  { id:104, name:"Mr.Saravanan", basicSalary:12000, workingDays:26, dept:"Pharmaceutical", designation:"Delivery", mobile:"9876500004", empId:"PH004", joined:"2022-01-01", photo:"", role:"Employee" },
  { id:105, name:"Mrs.Kavitha", basicSalary:7500, workingDays:26, dept:"Pharmaceutical", designation:"Staff", mobile:"9876500005", empId:"PH005", joined:"2022-01-01", photo:"", role:"Employee" },
  { id:106, name:"Mrs.Umadevi", basicSalary:2500, workingDays:26, dept:"Pharmaceutical", designation:"Staff", mobile:"9876500006", empId:"PH006", joined:"2022-01-01", photo:"", role:"Employee" },
];

function calcSalary(basic, workingDays, entry) {
  const w = Number(workingDays)||1, p = Number(entry.presentDays)||0;
  const col = Number(entry.collection)||0, db = Number(entry.db)||0;
  const pf = Number(entry.pf)||0, gf = Number(entry.gf)||0, adv = Number(entry.salaryAdvance)||0;
  const salaryForMonth = parseFloat(((basic/w)*p).toFixed(2));
  const grossSalary = parseFloat((salaryForMonth+col+db).toFixed(2));
  const totalDeduction = pf+gf+adv;
  const netSalary = parseFloat((grossSalary-totalDeduction).toFixed(2));
  return { salaryForMonth, grossSalary, totalDeduction, netSalary, roundOff:Math.round(netSalary) };
}

const emptyEntry = () => ({ presentDays:"", collection:"", db:"", pf:"465", gf:"0", salaryAdvance:"0", workingDaysOverride:"" });

// WhatsApp helpers
function buildWASlip(emp, entry, calc, wd, month, year) {
  const mn = MONTHS[month];
  return `+-------------------+
¦  ?? ROCKCITY SALARY SLIP  ¦
+-------------------+

?? ${emp.name}
?? ID: ${emp.empId}  |  ?? ${emp.mobile}
?? ${emp.dept} — ${emp.designation}
?? Month: ${mn} ${year}

????????????????????????
?? Basic Salary     : ?${emp.basicSalary}
?? Working Days     : ${wd}
? Present Days     : ${entry.presentDays||0}
?? Salary           : ?${calc.salaryForMonth}
?? Collection       : ?${entry.collection||0}
?? DB               : ?${entry.db||0}
????????????????????????
?? Gross Salary    : ?${calc.grossSalary}
????????????????????????
?? PF               : ?${entry.pf||0}
?? GF               : ?${entry.gf||0}
?? Advance          : ?${entry.salaryAdvance||0}
?? Total Deduction  : ?${calc.totalDeduction}
????????????????????????
? NET SALARY      : ?${calc.roundOff}
????????????????????????
?? Rockcity Call Taxi & Travels
This is your official salary slip`;
}
function openWA(mobile, text) {
  let num = (mobile||"").replace(/\D/g,"");
  if (num.length === 10) num = "91" + num;
  window.open(https://wa.me/${num}?text=${encodeURIComponent(text)}, "_blank");
}

// --- THEME -----------------------------------------------------
const LIGHT = {
  bg:"#f0f4f8", card:"#ffffff", navy:"#1e3a5f", blue:"#2d6a9f",
  text:"#1a202c", subtext:"#718096", border:"#e2e8f0",
  green:"#276749", red:"#c53030", yellow:"#fefce8", teal:"#e6fffa",
  inputBg:"#ffffff", headerBg:"linear-gradient(135deg,#1e3a5f,#2d6a9f)",
  tabActive:"#1e3a5f", tabBg:"#ffffff", rowAlt:"#f7fafc",
};
const DARK = {
  bg:"#0f172a", card:"#1e293b", navy:"#38bdf8", blue:"#0ea5e9",
  text:"#f1f5f9", subtext:"#94a3b8", border:"#334155",
  green:"#4ade80", red:"#f87171", yellow:"#1e293b", teal:"#0f3460",
  inputBg:"#0f172a", headerBg:"linear-gradient(135deg,#0f172a,#1e3a5f)",
  tabActive:"#38bdf8", tabBg:"#1e293b", rowAlt:"#172033",
};

// --- TRANSLATIONS -----------------------------------------------
const TR = {
  en: {
    appName:"Rockcity Employee Salary Manager", dashboard:"Dashboard", employees:"Employees",
    salary:"Salary Sheet", attendance:"Attendance", advances:"Advances", reports:"Reports",
    totalEmployees:"Total Employees", monthlySalary:"Monthly Salary", pendingSalary:"Pending",
    totalAdvance:"Total Advance", addEmployee:"Add Employee", editEmployee:"Edit Employee",
    salarySheet:"Salary Sheet", department:"Department", designation:"Designation",
    basicSalary:"Basic Salary", workingDays:"Working Days", presentDays:"Present Days",
    collection:"Collection", grossSalary:"Gross Salary", netSalary:"Net Salary",
    roundOff:"Round OFF", totalDeduction:"Total Deduction", share:"Share",
    print:"Print", save:"Save", cancel:"Cancel", delete:"Delete", edit:"Edit",
    name:"Name", mobile:"Mobile", empId:"Emp ID", joined:"Joined", role:"Role",
    all:"All", travels:"Travels", pharma:"Pharmaceutical", darkMode:"Dark Mode",
    language:"Language", advance:"Advance", advanceHistory:"Advance History",
    giveAdvance:"Give Advance", amount:"Amount", reason:"Reason", date:"Date",
    markAttendance:"Mark Attendance", present:"Present", absent:"Absent",
    halfDay:"Half Day", leave:"Leave", monthlyReport:"Monthly Report",
    salarySlip:"Salary Slip", close:"Close", copied:"Copied!", noData:"No data",
    whatsapp:"WhatsApp Send", sendOne:"Send via WhatsApp", sendAll:"Send All via WhatsApp",
    bulkSend:"Bulk WhatsApp Send", selectAll:"Select All", clear:"Clear", preview:"Preview",
    startSending:"Start Sending", noMobile:"Mobile number missing",
  },
  ta: {
    appName:"?????????? ????? ????????", dashboard:"??????????", employees:"?????????",
    salary:"????? ????", attendance:"?????", advances:"????????", reports:"??????????",
    totalEmployees:"????? ?????????", monthlySalary:"??? ???????", pendingSalary:"??????",
    totalAdvance:"????? ????????", addEmployee:"?????? ????", editEmployee:"????????",
    salarySheet:"????? ????", department:"????", designation:"????",
    basicSalary:"???????? ???????", workingDays:"??? ???????", presentDays:"???? ???????",
    collection:"?????????", grossSalary:"????? ???????", netSalary:"???? ???????",
    roundOff:"??????? ????", totalDeduction:"????? ?????", share:"?????",
    print:"???????", save:"????", cancel:"?????", delete:"??????", edit:"????????",
    name:"?????", mobile:"??????", empId:"???", joined:"??????? ????", role:"????????",
    all:"?????????", travels:"?????????", pharma:"???????", darkMode:"?????? ????",
    language:"????", advance:"????????", advanceHistory:"??????",
    giveAdvance:"???????? ????", amount:"????", reason:"??????", date:"????",
    markAttendance:"????? ?????", present:"???????", absent:"????????",
    halfDay:"??? ????", leave:"????????", monthlyReport:"??? ???????",
    salarySlip:"????? ??????", close:"????", copied:"?????????????????!", noData:"???? ?????",
    whatsapp:"????????? ???????", sendOne:"????????? ???????", sendAll:"???????????? ???????",
    bulkSend:"???????????? ?????????", selectAll:"??????????? ??????", clear:"???", preview:"???????????",
    startSending:"??????? ???????", noMobile:"?????? ?????",
  }
};

// --- MAIN APP --------------------------------------------------
export default function App() {
  const [tab, setTab] = useState("dashboard");
  const [dark, setDark] = useState(false);
  const [lang, setLang] = useState("en");
  const [month, setMonth] = useState(4);
  const [year, setYear] = useState(2026);
  const [deptFilter, setDeptFilter] = useState("All");
  const [employees, setEmployees] = useState(INITIAL_EMPLOYEES);
  const [salaryData, setSalaryData] = useState({});
  const [advanceData, setAdvanceData] = useState({});
  const [attendanceData, setAttendanceData] = useState({});
  const [empModal, setEmpModal] = useState(false);
  const [editEmp, setEditEmp] = useState(null);
  const [empForm, setEmpForm] = useState({});
  const [shareModal, setShareModal] = useState(null);
  const [advModal, setAdvModal] = useState(false);
  const [advForm, setAdvForm] = useState({ empId:"", amount:"", reason:"", date:new Date().toISOString().slice(0,10) });
  const [toast, setToast] = useState("");
  const [slipModal, setSlipModal] = useState(null);
  const [bulkModal, setBulkModal] = useState(false);
  const [bulkProgress, setBulkProgress] = useState(null);
  const [bulkSelected, setBulkSelected] = useState([]);
  const [loaded, setLoaded] = useState(false);

  // Load persisted data on mount (works for years/months indefinitely)
  useEffect(() => {
    (async () => {
      try {
        const keys = ["employees","salaryData","advanceData","attendanceData"];
        const setters = { employees:setEmployees, salaryData:setSalaryData, advanceData:setAdvanceData, attendanceData:setAttendanceData };
        for (const k of keys) {
          try {
            const r = await window.storage.get(k, false);
            if (r && r.value) setters[k](JSON.parse(r.value));
          } catch (e) { /* key not found, ignore */ }
        }
      } catch (e) { /* storage unavailable */ }
      setLoaded(true);
    })();
  }, []);

  // Save persisted data whenever it changes (skip initial load)
  useEffect(() => { if(loaded) window.storage?.set("employees", JSON.stringify(employees), false).catch(()=>{}); }, [employees, loaded]);
  useEffect(() => { if(loaded) window.storage?.set("salaryData", JSON.stringify(salaryData), false).catch(()=>{}); }, [salaryData, loaded]);
  useEffect(() => { if(loaded) window.storage?.set("advanceData", JSON.stringify(advanceData), false).catch(()=>{}); }, [advanceData, loaded]);
  useEffect(() => { if(loaded) window.storage?.set("attendanceData", JSON.stringify(attendanceData), false).catch(()=>{}); }, [attendanceData, loaded]);

  const T = TR[lang];
  const theme = dark ? DARK : LIGHT;

  const showToast = (msg) => { setToast(msg); setTimeout(()=>setToast(""),2500); };

  // Keys
  const sKey = (id) => ${id}-${month}-${year};
  const aKey = (id, day) => ${id}-${year}-${month}-${day};

  const getEntry = (id) => salaryData[sKey(id)] || emptyEntry();
  const setField = (id, field, val) => setSalaryData(p=>({...p,[sKey(id)]:{...(p[sKey(id)]||emptyEntry()),[field]:val}}));

  const filteredEmps = useMemo(()=>
    deptFilter==="All" ? employees : employees.filter(e=>e.dept===deptFilter),
    [employees, deptFilter]
  );

  const rows = useMemo(()=>filteredEmps.map(emp=>{
    const entry = getEntry(emp.id);
    const wd = entry.workingDaysOverride || emp.workingDays;
    const calc = calcSalary(emp.basicSalary, wd, entry);
    return { emp, entry, calc, wd };
  }), [filteredEmps, salaryData, month, year]);

  const grand = rows.reduce((a,{calc})=>({
    roundOff:a.roundOff+calc.roundOff, net:a.net+calc.netSalary, ded:a.ded+calc.totalDeduction
  }),{roundOff:0,net:0,ded:0});

  // Dashboard stats
  const totalAdv = Object.values(advanceData).reduce((a,b)=>a+(b.reduce((x,y)=>x+Number(y.amount),0)),0);
  const todayStr = new Date().toISOString().slice(0,10);
  const todayPresent = employees.filter(e=>{
    const k = aKey(e.id, new Date().getDate());
    return attendanceData[k]==="Present";
  }).length;

  // Employee CRUD
  const openAdd = () => { setEditEmp(null); setEmpForm({ name:"",basicSalary:"",workingDays:"",dept:"Travels",designation:"Staff",mobile:"",empId:"",joined:todayStr,role:"Employee",photo:"" }); setEmpModal(true); };
  const openEdit = (emp) => { setEditEmp(emp); setEmpForm({...emp}); setEmpModal(true); };
  const saveEmp = () => {
    if (!empForm.name?.trim()) return;
    if (editEmp) {
      setEmployees(p=>p.map(e=>e.id===editEmp.id?{...e,...empForm,basicSalary:Number(empForm.basicSalary),workingDays:Number(empForm.workingDays)}:e));
      showToast("? "+T.save+"d!");
    } else {
      const newId = Math.max(0,...employees.map(e=>e.id))+1;
      setEmployees(p=>[...p,{...empForm,id:newId,basicSalary:Number(empForm.basicSalary),workingDays:Number(empForm.workingDays)}]);
      showToast("? Added!");
    }
    setEmpModal(false);
  };
  const delEmp = (id) => { if(!window.confirm("Delete?"))return; setEmployees(p=>p.filter(e=>e.id!==id)); showToast("??? Deleted!"); };

  // Advance
  const giveAdvance = () => {
    if(!advForm.empId||!advForm.amount)return;
    const key = adv-${advForm.empId};
    const rec = { amount:advForm.amount, reason:advForm.reason, date:advForm.date, id:Date.now() };
    setAdvanceData(p=>({...p,[key]:[...(p[key]||[]),rec]}));
    // Also update salary entry advance
    const emp = employees.find(e=>e.id===Number(advForm.empId));
    if(emp){ const cur=getEntry(emp.id); const prev=Number(cur.salaryAdvance)||0; setField(emp.id,"salaryAdvance",String(prev+Number(advForm.amount))); }
    showToast("? Advance added!");
    setAdvModal(false);
    setAdvForm({empId:"",amount:"",reason:"",date:todayStr});
  };

  // Attendance
  const markAttendance = (empId, day, status) => {
    const k = aKey(empId, day);
    setAttendanceData(p=>({...p,[k]:status}));
  };
  const getAttendance = (empId, day) => attendanceData[aKey(empId,day)] || "";

  // Monthly attendance count
  const getPresentCount = (empId) => {
    let count=0;
    for(let d=1;d<=31;d++){
      const s = attendanceData[aKey(empId,d)];
      if(s==="Present") count+=1;
      else if(s==="Half Day") count+=0.5;
    }
    return count;
  };

  // Salary slip text
  const slipText = (emp, entry, calc, wd) => {
    const mn = MONTHS[month];
    return `????????????????????????
?? ROCKCITY SALARY SLIP
????????????????????????
?? ${emp.name}
?? ${emp.empId} | ?? ${emp.mobile}
?? ${emp.dept} — ${emp.designation}
?? ${mn} ${year}
????????????????????????
?? Basic Salary    : ?${emp.basicSalary}
?? Working Days    : ${wd}
? Present Days    : ${entry.presentDays||0}
?? Salary          : ?${calc.salaryForMonth}
?? Collection      : ?${entry.collection||0}
?? DB              : ?${entry.db||0}
????????????????????????
?? Gross Salary    : ?${calc.grossSalary}
????????????????????????
?? PF              : ?${entry.pf||0}
?? GF              : ?${entry.gf||0}
?? Advance         : ?${entry.salaryAdvance||0}
?? Total Deduction : ?${calc.totalDeduction}
????????????????????????
? NET SALARY      : ?${calc.roundOff}
????????????????????????
Rockcity Call Taxi & Travels`;
  };

  const doShare = async (emp, entry, calc, wd) => {
    const text = slipText(emp,entry,calc,wd);
    if(navigator.share){ try{await navigator.share({title:Salary - ${emp.name},text});}catch(e){} }
    else { navigator.clipboard.writeText(text); showToast("?? "+T.copied); }
    setShareModal(null);
  };

  // --- WHATSAPP SEND --------------------------------------------
  const sendOneWA = (emp, entry, calc, wd) => {
    if (!emp.mobile || emp.mobile.replace(/\D/g,"").length < 10) { showToast("?? "+T.noMobile+": "+emp.name); return; }
    openWA(emp.mobile, buildWASlip(emp, entry, calc, wd, month, year));
    showToast("? "+emp.name+" — WhatsApp opening...");
  };

  const toggleBulkSelect = (id) => setBulkSelected(p => p.includes(id) ? p.filter(x=>x!==id) : [...p, id]);
  const selectAllBulk = () => setBulkSelected(rows.map(r=>r.emp.id));
  const clearBulkSelect = () => setBulkSelected([]);

  const startBulkSend = async () => {
    const selRows = rows.filter(r => bulkSelected.includes(r.emp.id));
    if (!selRows.length) { showToast("?? Select employees first"); return; }
    setBulkModal(false);
    for (let i=0;i<selRows.length;i++) {
      const { emp, entry, calc, wd } = selRows[i];
      setBulkProgress({ current:i+1, total:selRows.length, name:emp.name });
      if (emp.mobile && emp.mobile.replace(/\D/g,"").length>=10) {
        openWA(emp.mobile, buildWASlip(emp,entry,calc,wd,month,year));
        await new Promise(res=>setTimeout(res,2500));
      }
    }
    setBulkProgress(null);
    showToast(? Sent to ${selRows.length} employees!);
  };

  // --- STYLES -------------------------------------------------
  const t = theme;
  const S = {
    app:{ fontFamily:"'Segoe UI',sans-serif", background:t.bg, minHeight:"100vh", color:t.text, maxWidth:1100, margin:"0 auto", transition:"all 0.2s" },
    header:{ background:t.headerBg, color:"#fff", padding:"12px 16px", display:"flex", justifyContent:"space-between", alignItems:"center", position:"sticky", top:0, zIndex:100 },
    nav:{ display:"flex", background:t.tabBg, borderBottom:2px solid ${t.border}, overflowX:"auto", padding:"0 8px" },
    navBtn:(a)=>({ padding:"10px 12px", border:"none", background:"transparent", cursor:"pointer", fontWeight:a?700:500, color:a?t.tabActive:t.subtext, borderBottom:a?2px solid ${t.tabActive}:"2px solid transparent", marginBottom:"-2px", fontSize:12, whiteSpace:"nowrap", transition:"color 0.15s" }),
    body:{ padding:12 },
    card:{ background:t.card, borderRadius:10, boxShadow:dark?"0 2px 8px rgba(0,0,0,0.4)":"0 1px 4px rgba(0,0,0,0.08)", overflow:"hidden" },
    row:{ display:"flex", gap:10, marginBottom:12, flexWrap:"wrap", alignItems:"center" },
    sel:{ padding:"7px 10px", border:1px solid ${t.border}, borderRadius:6, fontSize:12, background:t.inputBg, color:t.text },
    btn:(bg,fg="#fff")=>({ padding:"7px 14px", background:bg, color:fg, border:"none", borderRadius:6, cursor:"pointer", fontSize:12, fontWeight:600 }),
    btnSm:(bg)=>({ padding:"4px 9px", background:bg, color:"#fff", border:"none", borderRadius:5, cursor:"pointer", fontSize:11, fontWeight:600 }),
    btnOut:{ padding:"7px 14px", background:"transparent", color:t.navy, border:1.5px solid ${t.navy}, borderRadius:6, cursor:"pointer", fontSize:12, fontWeight:600 },
    th:{ background:t.navy, color:"#fff", padding:"8px 5px", textAlign:"center", whiteSpace:"nowrap", fontSize:11, fontWeight:600 },
    td:(i)=>({ padding:"5px 4px", textAlign:"center", borderBottom:1px solid ${t.border}, background:i%2===0?t.card:t.rowAlt, color:t.text, whiteSpace:"nowrap", fontSize:11 }),
    inp:{ width:56, padding:"3px 4px", border:1px solid ${t.border}, borderRadius:4, fontSize:11, textAlign:"center", background:t.inputBg, color:t.text },
    inpWd:{ width:44, padding:"3px 4px", border:"1.5px solid #f6ad55", borderRadius:4, fontSize:11, textAlign:"center", background:"#fffbeb", color:"#744210", fontWeight:700 },
    modal:{ position:"fixed", inset:0, background:"rgba(0,0,0,0.6)", display:"flex", alignItems:"center", justifyContent:"center", zIndex:1000, padding:12 },
    mBox:{ background:t.card, borderRadius:12, padding:20, width:"100%", maxWidth:360, boxShadow:"0 8px 32px rgba(0,0,0,0.3)", maxHeight:"90vh", overflowY:"auto" },
    mInp:{ width:"100%", padding:"8px 10px", border:1px solid ${t.border}, borderRadius:6, fontSize:13, boxSizing:"border-box", marginBottom:10, background:t.inputBg, color:t.text },
    lbl:{ display:"block", fontSize:11, fontWeight:600, color:t.subtext, marginBottom:3 },
    statCard:(col)=>({ background:t.card, borderRadius:10, padding:"16px 14px", borderTop:3px solid ${col}, boxShadow:dark?"0 2px 8px rgba(0,0,0,0.3)":"0 1px 4px rgba(0,0,0,0.07)" }),
    badge:(col)=>({ background:col, color:"#fff", padding:"2px 8px", borderRadius:10, fontSize:10, fontWeight:700 }),
    slipBox:{ background:"#0f172a", color:"#e2e8f0", borderRadius:8, padding:14, fontFamily:"monospace", fontSize:11, whiteSpace:"pre", overflowX:"auto", marginBottom:12, lineHeight:1.7 },
    toast:{ position:"fixed", bottom:20, left:"50%", transform:"translateX(-50%)", background:t.blue, color:"#fff", padding:"10px 22px", borderRadius:20, fontWeight:600, fontSize:13, zIndex:9999, boxShadow:"0 4px 16px rgba(0,0,0,0.3)", whiteSpace:"nowrap" },
    progressBar:{ position:"fixed", bottom:0, left:0, right:0, background:"#128c7e", color:"#fff", padding:"14px 20px", display:"flex", alignItems:"center", gap:14, zIndex:9998, boxShadow:"0 -4px 16px rgba(0,0,0,0.3)" },
  };
  const WA = "#25d366", WAD = "#128c7e";

  const deptColor = (d) => d==="Travels"?"#2d6a9f":"#276749";
  const roleColor = (r) => r==="Admin"?"#9f1239":r==="Manager"?"#c05621":"#4a5568";
  const attColor = (s) => s==="Present"?"#276749":s==="Absent"?"#c53030":s==="Half Day"?"#c05621":"#718096";

  const today = new Date().getDate();
  const daysInMonth = new Date(year, month+1, 0).getDate();

  // --- RENDER -------------------------------------------------
  return (
    <div style={S.app}>
      <style>{@media print{.no-print{display:none!important}body{background:#fff}}input::-webkit-inner-spin-button{-webkit-appearance:none}::-webkit-scrollbar{width:4px;height:4px}::-webkit-scrollbar-thumb{background:#888;border-radius:4px}}</style>

      {/* Header */}
      <div style={S.header} className="no-print">
        <div>
          <div style={{ fontWeight:700, fontSize:15 }}>?? {T.appName}</div>
          <div style={{ fontSize:10, opacity:0.75, marginTop:1 }}>Rockcity Call Taxi & Travels | Pharma</div>
        </div>
        <div style={{ display:"flex", gap:8, alignItems:"center" }}>
          <button style={{ ...S.btn(dark?"#1e293b":"rgba(255,255,255,0.15)"), padding:"5px 10px", fontSize:11, border:"1px solid rgba(255,255,255,0.3)" }}
            onClick={()=>setDark(!dark)}>{dark?"??":"??"}</button>
          <button style={{ ...S.btn(dark?"#1e293b":"rgba(255,255,255,0.15)"), padding:"5px 10px", fontSize:11, border:"1px solid rgba(255,255,255,0.3)" }}
            onClick={()=>setLang(l=>l==="en"?"ta":"en")}>{lang==="en"?"?????":"EN"}</button>
        </div>
      </div>

      {/* Nav Tabs */}
      <div style={S.nav} className="no-print">
        {[
  ["dashboard","??"],
  ["employees","??"],
  ["salary","??"],
  ["attendance","??"],
  ["advances","??"],
  ["reports","??"],
  ["whatsapp","??"]
].map(([k,icon])=>(
  <button
    key={k}
    style={S.navBtn(tab===k)}
    onClick={()=>setTab(k)}
  >
    {icon} {T[k]}
  </button>
))}=>(<button key={k} style={S.navBtn(tab===k)} onClick={()=>setTab(k)}>{icon} {T[k]}</button>
        ))}
      </div>

      <div style={S.body}>

        {/* -- DASHBOARD -- */}
        {tab==="dashboard" && <>
          <div style={{ display:"grid", gridTemplateColumns:"repeat(auto-fit,minmax(130px,1fr))", gap:12, marginBottom:16 }}>
            {[
              [T.totalEmployees, employees.length, t.navy, "??"],
              ["Today Present", todayPresent, t.green, "?"],
              [T.monthlySalary, "?"+grand.roundOff.toLocaleString(), t.blue, "??"],
              [T.totalAdvance, "?"+totalAdv.toLocaleString(), t.red, "??"],
              ["Travels", employees.filter(e=>e.dept==="Travels").length, "#c05621","??"],
              ["Pharma", employees.filter(e=>e.dept==="Pharmaceutical").length, t.green,"??"],
            ].map(([lbl,val,col,icon])=>(
              <div key={lbl} style={S.statCard(col)}>
                <div style={{ fontSize:18, marginBottom:4 }}>{icon}</div>
                <div style={{ fontSize:11, color:t.subtext, fontWeight:600 }}>{lbl}</div>
                <div style={{ fontSize:20, fontWeight:800, color:col, marginTop:2 }}>{val}</div>
              </div>
            ))}
          </div>

          {/* Month selector for dashboard */}
          <div style={S.row}>
            <select style={S.sel} value={month} onChange={e=>setMonth(Number(e.target.value))}>
              {MONTHS.map((m,i)=><option key={i} value={i}>{m}</option>)}
            </select>
            <select style={S.sel} value={year} onChange={e=>setYear(Number(e.target.value))}>
              {[2024,2025,2026,2027,2028,2029,2030,2031,2032].map(y=><option key={y} value={y}>{y}</option>)}
            </select>
          </div>

          {/* Dept summary */}
          {["Travels","Pharmaceutical"].map(dept=>{
            const dEmps = employees.filter(e=>e.dept===dept);
            const dRows = dEmps.map(emp=>{ const entry=getEntry(emp.id); const wd=entry.workingDaysOverride||emp.workingDays; return calcSalary(emp.basicSalary,wd,entry); });
            const total = dRows.reduce((a,b)=>a+b.roundOff,0);
            return (
              <div key={dept} style={{ ...S.card, marginBottom:12, padding:"14px 16px" }}>
                <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center" }}>
                  <div>
                    <div style={{ fontWeight:700, fontSize:14, color:deptColor(dept) }}>{dept==="Travels"?"??":"??"} {dept}</div>
                    <div style={{ fontSize:12, color:t.subtext, marginTop:2 }}>{dEmps.length} employees — {MONTHS[month]} {year}</div>
                  </div>
                  <div style={{ textAlign:"right" }}>
                    <div style={{ fontWeight:800, fontSize:18, color:deptColor(dept) }}>?{total.toLocaleString()}</div>
                    <div style={{ fontSize:10, color:t.subtext }}>Total Pay</div>
                  </div>
                </div>
              </div>
            );
          })}
        </>}

        {/* -- EMPLOYEES -- */}
        {tab==="employees" && <>
          <div style={S.row}>
            <select style={S.sel} value={deptFilter} onChange={e=>setDeptFilter(e.target.value)}>
              {["All","Travels","Pharmaceutical"].map(d=><option key={d}>{d}</option>)}
            </select>
            <button style={S.btn(t.navy)} onClick={openAdd}>+ {T.addEmployee}</button>
          </div>
          <div style={S.card}>
            {filteredEmps.map((emp,i)=>(
              <div key={emp.id} style={{ display:"flex", gap:12, alignItems:"center", padding:"12px 14px", borderBottom:1px solid ${t.border}, background:i%2===0?t.card:t.rowAlt }}>
                <div style={{ width:40, height:40, borderRadius:"50%", background:deptColor(emp.dept), display:"flex", alignItems:"center", justifyContent:"center", color:"#fff", fontWeight:700, fontSize:14, flexShrink:0 }}>
                  {emp.name.charAt(0)}
                </div>
                <div style={{ flex:1, minWidth:0 }}>
                  <div style={{ fontWeight:700, fontSize:13, color:t.text }}>{emp.name}</div>
                  <div style={{ fontSize:11, color:t.subtext, marginTop:2 }}>
                    {emp.empId} · {emp.mobile}
                  </div>
                  <div style={{ display:"flex", gap:5, marginTop:4, flexWrap:"wrap" }}>
                    <span style={S.badge(deptColor(emp.dept))}>{emp.dept}</span>
                    <span style={S.badge(roleColor(emp.role))}>{emp.role}</span>
                    <span style={S.badge("#4a5568")}>{emp.designation}</span>
                  </div>
                </div>
                <div style={{ textAlign:"right", flexShrink:0 }}>
                  <div style={{ fontWeight:700, color:t.navy, fontSize:13 }}>?{emp.basicSalary}</div>
                  <div style={{ fontSize:10, color:t.subtext }}>{emp.workingDays} days</div>
                  <div style={{ display:"flex", gap:5, marginTop:6 }}>
                    <button style={S.btnSm(t.blue)} onClick={()=>openEdit(emp)}>??</button>
                    <button style={S.btnSm(t.red)} onClick={()=>delEmp(emp.id)}>???</button>
                  </div>
                </div>
              </div>
            ))}
          </div>
        </>}

        {/* -- SALARY SHEET -- */}
        {tab==="salary" && <>
          <div style={S.row} className="no-print">
            <select style={S.sel} value={deptFilter} onChange={e=>setDeptFilter(e.target.value)}>
              {["All","Travels","Pharmaceutical"].map(d=><option key={d}>{d}</option>)}
            </select>
            <select style={S.sel} value={month} onChange={e=>setMonth(Number(e.target.value))}>
              {MONTHS.map((m,i)=><option key={i} value={i}>{m}</option>)}
            </select>
            <select style={S.sel} value={year} onChange={e=>setYear(Number(e.target.value))}>
              {[2024,2025,2026,2027,2028,2029,2030,2031,2032].map(y=><option key={y} value={y}>{y}</option>)}
            </select>
            <button style={S.btn(t.navy)} onClick={()=>window.print()}>??? {T.print}</button>
            <button style={S.btn(WAD)} onClick={()=>{ setBulkSelected(rows.map(r=>r.emp.id)); setBulkModal(true); }}>?? {T.sendAll}</button>
          </div>

          <div style={S.card} className="print-area">
            <div style={{ padding:"10px 14px", background:t.navy, color:"#fff", textAlign:"center", fontWeight:700, fontSize:13 }}>
              SALARY SHEET — {MONTHS[month].toUpperCase()} {year} {deptFilter!=="All"?— ${deptFilter}:""}
            </div>
            <div style={{ overflowX:"auto" }}>
              <table style={{ width:"100%", borderCollapse:"collapse", fontSize:11 }}>
                <thead>
                  <tr>{["#","Name","Basic","W.Days","P.Days","Salary","Collection","DB","Gross","PF","GF","Advance","Ded.","Net","Round OFF","Slip","WA"].map(h=>(
                    <th key={h} style={S.th}>{h}</th>
                  ))}</tr>
                </thead>
                <tbody>
                  {rows.map(({emp,entry,calc,wd},i)=>(
                    <tr key={emp.id}>
                      <td style={S.td(i)}>{i+1}</td>
                      <td style={{...S.td(i),textAlign:"left",minWidth:120,paddingLeft:7}}>
                        <div style={{fontWeight:600}}>{emp.name}</div>
                        <div style={{fontSize:9,color:t.subtext}}>{emp.dept}</div>
                      </td>
                      <td style={S.td(i)}>?{emp.basicSalary}</td>
                      <td style={S.td(i)}>
                        <input style={S.inpWd} type="number" value={entry.workingDaysOverride||emp.workingDays}
                          onChange={e=>setField(emp.id,"workingDaysOverride",e.target.value)} title="Edit working days"/>
                      </td>
                      <td style={S.td(i)}><input style={S.inp} type="number" value={entry.presentDays} onChange={e=>setField(emp.id,"presentDays",e.target.value)}/></td>
                      <td style={{...S.td(i),background:dark?t.yellow:"#fefce8",fontWeight:600}}>{calc.salaryForMonth||0}</td>
                      <td style={S.td(i)}><input style={S.inp} type="number" value={entry.collection} onChange={e=>setField(emp.id,"collection",e.target.value)}/></td>
                      <td style={S.td(i)}><input style={S.inp} type="number" value={entry.db} onChange={e=>setField(emp.id,"db",e.target.value)}/></td>
                      <td style={{...S.td(i),fontWeight:600}}>{calc.grossSalary||0}</td>
                      <td style={S.td(i)}><input style={S.inp} type="number" value={entry.pf} onChange={e=>setField(emp.id,"pf",e.target.value)}/></td>
                      <td style={S.td(i)}><input style={S.inp} type="number" value={entry.gf} onChange={e=>setField(emp.id,"gf",e.target.value)}/></td>
                      <td style={S.td(i)}><input style={S.inp} type="number" value={entry.salaryAdvance} onChange={e=>setField(emp.id,"salaryAdvance",e.target.value)}/></td>
                      <td style={{...S.td(i),color:t.red,fontWeight:600}}>{calc.totalDeduction}</td>
                      <td style={{...S.td(i),fontWeight:600}}>{calc.netSalary}</td>
                      <td style={{...S.td(i),background:dark?t.teal:"#e6fffa",fontWeight:800,color:t.navy,fontSize:12}}>{calc.roundOff||0}</td>
                      <td style={S.td(i)} className="no-print">
                        <button style={S.btnSm("#25d366")} onClick={()=>setShareModal({emp,entry,calc,wd})}>??</button>
                      </td>
                      <td style={S.td(i)} className="no-print">
                        <button style={S.btnSm(WAD)} title={Send to ${emp.name}} onClick={()=>sendOneWA(emp,entry,calc,wd)}>??</button>
                      </td>
                    </tr>
                  ))}
                </tbody>
                <tfoot>
                  <tr>
                    <td colSpan={13} style={{...S.th,textAlign:"right",paddingRight:10}}>TOTAL ?</td>
                    <td style={S.th}>{Math.round(grand.net)}</td>
                    <td style={{...S.th,color:"#ffd700",fontSize:13}}>{grand.roundOff}</td>
                    <td style={S.th} className="no-print"></td>
                    <td style={S.th} className="no-print"></td>
                  </tr>
                </tfoot>
              </table>
            </div>
          </div>
        </>}

        {/* -- ATTENDANCE -- */}
        {tab==="attendance" && <>
          <div style={S.row}>
            <select style={S.sel} value={deptFilter} onChange={e=>setDeptFilter(e.target.value)}>
              {["All","Travels","Pharmaceutical"].map(d=><option key={d}>{d}</option>)}
            </select>
            <select style={S.sel} value={month} onChange={e=>setMonth(Number(e.target.value))}>
              {MONTHS.map((m,i)=><option key={i} value={i}>{m}</option>)}
            </select>
            <select style={S.sel} value={year} onChange={e=>setYear(Number(e.target.value))}>
              {[2024,2025,2026,2027,2028,2029,2030,2031,2032].map(y=><option key={y} value={y}>{y}</option>)}
            </select>
          </div>

          {/* Today quick mark */}
          <div style={{...S.card, marginBottom:12}}>
            <div style={{padding:"10px 14px", background:t.navy, color:"#fff", fontWeight:700, fontSize:12}}>
              ?? Today — {todayStr} (Quick Mark)
            </div>
            {filteredEmps.map((emp,i)=>(
              <div key={emp.id} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"9px 14px",borderBottom:1px solid ${t.border},background:i%2===0?t.card:t.rowAlt}}>
                <div style={{fontWeight:600,fontSize:12,color:t.text}}>{emp.name}</div>
                <div style={{display:"flex",gap:5,flexWrap:"wrap"}}>
                  {ATTENDANCE_STATUS.map(s=>(
                    <button key={s} style={{...S.btnSm(getAttendance(emp.id,today)===s?attColor(s):"#94a3b8"),fontSize:10,opacity:getAttendance(emp.id,today)===s?1:0.6}}
                      onClick={()=>markAttendance(emp.id,today,s)}>{s}</button>
                  ))}
                </div>
              </div>
            ))}
          </div>

          {/* Monthly summary */}
          <div style={S.card}>
            <div style={{padding:"10px 14px",background:t.navy,color:"#fff",fontWeight:700,fontSize:12}}>
              ?? {MONTHS[month]} {year} — Monthly Attendance Summary
            </div>
            {filteredEmps.map((emp,i)=>{
              const pc = getPresentCount(emp.id);
              const ac = Array.from({length:daysInMonth},(_,d)=>attendanceData[aKey(emp.id,d+1)]).filter(s=>s==="Absent").length;
              const lc = Array.from({length:daysInMonth},(_,d)=>attendanceData[aKey(emp.id,d+1)]).filter(s=>s==="Leave").length;
              return (
                <div key={emp.id} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"10px 14px",borderBottom:1px solid ${t.border},background:i%2===0?t.card:t.rowAlt}}>
                  <div style={{fontWeight:600,fontSize:12}}>{emp.name}</div>
                  <div style={{display:"flex",gap:10,fontSize:11}}>
                    <span style={{color:t.green,fontWeight:700}}>? {pc}</span>
                    <span style={{color:t.red,fontWeight:700}}>? {ac}</span>
                    <span style={{color:"#c05621",fontWeight:700}}>?? {lc}</span>
                  </div>
                </div>
              );
            })}
          </div>
        </>}

        {/* -- ADVANCES -- */}
        {tab==="advances" && <>
          <div style={S.row}>
            <button style={S.btn(t.navy)} onClick={()=>setAdvModal(true)}>+ {T.giveAdvance}</button>
          </div>
          <div style={S.card}>
            <div style={{padding:"10px 14px",background:t.navy,color:"#fff",fontWeight:700,fontSize:12}}>?? Advance History</div>
            {employees.map(emp=>{
              const advList = advanceData[adv-${emp.id}]||[];
              if(!advList.length) return null;
              const total = advList.reduce((a,b)=>a+Number(b.amount),0);
              return (
                <div key={emp.id} style={{borderBottom:1px solid ${t.border}}}>
                  <div style={{padding:"10px 14px",display:"flex",justifyContent:"space-between",background:t.rowAlt}}>
                    <div style={{fontWeight:700,fontSize:12,color:t.text}}>{emp.name} <span style={{fontSize:10,color:t.subtext}}>({emp.dept})</span></div>
                    <div style={{fontWeight:800,color:t.red}}>?{total}</div>
                  </div>
                  {advList.map(a=>(
                    <div key={a.id} style={{padding:"6px 24px",display:"flex",justifyContent:"space-between",fontSize:11,color:t.subtext,borderTop:1px solid ${t.border}}}>
                      <span>{a.date} — {a.reason||"Advance"}</span>
                      <span style={{fontWeight:700,color:t.navy}}>?{a.amount}</span>
                    </div>
                  ))}
                </div>
              );
            })}
            {Object.keys(advanceData).length===0 && <div style={{padding:20,textAlign:"center",color:t.subtext}}>No advances yet</div>}
          </div>
        </>}

        {/* -- REPORTS -- */}
        {tab==="reports" && <>
          <div style={S.row}>
            <select style={S.sel} value={deptFilter} onChange={e=>setDeptFilter(e.target.value)}>
              {["All","Travels","Pharmaceutical"].map(d=><option key={d}>{d}</option>)}
            </select>
            <select style={S.sel} value={month} onChange={e=>setMonth(Number(e.target.value))}>
              {MONTHS.map((m,i)=><option key={i} value={i}>{m}</option>)}
            </select>
            <select style={S.sel} value={year} onChange={e=>setYear(Number(e.target.value))}>
              {[2024,2025,2026,2027,2028,2029,2030,2031,2032].map(y=><option key={y} value={y}>{y}</option>)}
            </select>
          </div>

          {/* Summary cards */}
          <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fit,minmax(130px,1fr))",gap:12,marginBottom:16}}>
            {[
              [T.totalEmployees, filteredEmps.length, t.navy],
              ["Total Round OFF", "?"+grand.roundOff.toLocaleString(), t.green],
              ["Net Salary", "?"+Math.round(grand.net).toLocaleString(), t.blue],
              ["Deductions", "?"+Math.round(grand.ded).toLocaleString(), t.red],
            ].map(([l,v,c])=>(
              <div key={l} style={S.statCard(c)}>
                <div style={{fontSize:11,color:t.subtext,fontWeight:600}}>{l}</div>
                <div style={{fontSize:19,fontWeight:800,color:c,marginTop:4}}>{v}</div>
              </div>
            ))}
          </div>

          {/* Per employee */}
          <div style={S.card}>
            <div style={{padding:"10px 14px",background:t.navy,color:"#fff",fontWeight:700,fontSize:12}}>
              ?? {MONTHS[month]} {year} — Employee Report
            </div>
            {rows.map(({emp,entry,calc,wd},i)=>(
              <div key={emp.id} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"10px 14px",borderBottom:1px solid ${t.border},background:i%2===0?t.card:t.rowAlt}}>
                <div>
                  <div style={{fontWeight:600,fontSize:13}}>{emp.name}</div>
                  <div style={{fontSize:11,color:t.subtext}}>Gross: ?{calc.grossSalary} · Ded: ?{calc.totalDeduction}</div>
                  <span style={S.badge(deptColor(emp.dept))}>{emp.dept}</span>
                </div>
                <div style={{display:"flex",gap:8,alignItems:"center"}}>
                  <div style={{textAlign:"right"}}>
                    <div style={{fontWeight:800,fontSize:15,color:t.navy}}>?{calc.roundOff}</div>
                    <div style={{fontSize:10,color:t.subtext}}>Net Pay</div>
                  </div>
                  <button style={S.btnSm("#25d366")} onClick={()=>setShareModal({emp,entry,calc,wd})}>??</button>
                  <button style={S.btnSm(WAD)} onClick={()=>sendOneWA(emp,entry,calc,wd)}>??</button>
                  <button style={S.btnSm(t.blue)} onClick={()=>setSlipModal({emp,entry,calc,wd})}>??</button>
                </div>
              </div>
            ))}
          </div>
        </>}

        {/* -- WHATSAPP TAB -- */}
        {tab==="whatsapp" && <>
          <div style={{...S.card, marginBottom:14}}>
            <div style={{padding:"14px 16px", background:linear-gradient(135deg,${WAD},${WA}), color:"#fff"}}>
              <div style={{fontWeight:700,fontSize:16}}>?? WhatsApp Salary Sender</div>
              <div style={{fontSize:11,opacity:0.85,marginTop:3}}>Send individual or bulk salary slips directly via WhatsApp</div>
            </div>
            <div style={{padding:14}}>
              <div style={S.row}>
                <select style={S.sel} value={deptFilter} onChange={e=>setDeptFilter(e.target.value)}>{["All","Travels","Pharmaceutical"].map(d=><option key={d}>{d}</option>)}</select>
                <select style={S.sel} value={month} onChange={e=>setMonth(Number(e.target.value))}>{MONTHS.map((m,i)=><option key={i} value={i}>{m}</option>)}</select>
                <select style={S.sel} value={year} onChange={e=>setYear(Number(e.target.value))}>{[2024,2025,2026,2027,2028,2029,2030].map(y=><option key={y} value={y}>{y}</option>)}</select>
              </div>
              <div style={S.row}>
                <button style={S.btn(WA)} onClick={selectAllBulk}>?? {T.selectAll}</button>
                <button style={S.btn("#94a3b8")} onClick={clearBulkSelect}>?? {T.clear}</button>
                <button style={{...S.btn(WAD),padding:"8px 18px",fontSize:13}} disabled={bulkSelected.length===0} onClick={()=>setBulkModal(true)}>
                  ?? {T.sendAll} {bulkSelected.length>0?(${bulkSelected.length}):""}
                </button>
              </div>
              <div style={{fontSize:12,color:t.subtext}}>
                ? {bulkSelected.length} selected — {MONTHS[month]} {year}
              </div>
            </div>
          </div>

          <div style={S.card}>
            {rows.map(({emp,entry,calc,wd},i)=>{
              const selected = bulkSelected.includes(emp.id);
              const hasPhone = emp.mobile && emp.mobile.replace(/\D/g,"").length>=10;
              return (
                <div key={emp.id} style={{display:"flex",gap:12,alignItems:"center",padding:"12px 14px",borderBottom:1px solid ${t.border},background:selected?(dark?"#0f3460":"#e6fffa"):(i%2===0?t.card:t.rowAlt),cursor:"pointer"}}
                  onClick={()=>toggleBulkSelect(emp.id)}>
                  <div style={{width:22,height:22,borderRadius:6,border:2px solid ${selected?WA:t.border},background:selected?WA:"transparent",display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0}}>
                    {selected && <span style={{color:"#fff",fontSize:14,fontWeight:900}}>?</span>}
                  </div>
                  <div style={{width:36,height:36,borderRadius:"50%",background:deptColor(emp.dept),display:"flex",alignItems:"center",justifyContent:"center",color:"#fff",fontWeight:700,fontSize:13,flexShrink:0}}>{emp.name.charAt(0)}</div>
                  <div style={{flex:1,minWidth:0}}>
                    <div style={{fontWeight:700,fontSize:13}}>{emp.name}</div>
                    <div style={{fontSize:11,color:t.subtext}}>?? {emp.mobile||"—"} · {emp.dept}</div>
                    {!hasPhone && <div style={{fontSize:10,color:t.red,marginTop:2}}>?? {T.noMobile}</div>}
                  </div>
                  <div style={{textAlign:"right",flexShrink:0}}>
                    <div style={{fontWeight:800,fontSize:14,color:WA}}>?{calc.roundOff}</div>
                    <div style={{fontSize:10,color:t.subtext,marginBottom:5}}>Net Pay</div>
                    <button style={S.btnSm(WAD)} onClick={e=>{e.stopPropagation(); sendOneWA(emp,entry,calc,wd);}}>?? Send</button>
                  </div>
                </div>
              );
            })}
          </div>

          {bulkSelected.length>0 && (
            <div style={{position:"sticky",bottom:12,marginTop:14,textAlign:"center"}}>
              <button style={{...S.btn(WAD),padding:"12px 28px",fontSize:14,borderRadius:30,boxShadow:"0 4px 20px rgba(18,140,126,0.5)"}} onClick={()=>setBulkModal(true)}>
                ?? {T.sendAll} ({bulkSelected.length})
              </button>
            </div>
          )}
        </>}
      </div>

      {/* Bulk WhatsApp Confirm Modal */}
      {bulkModal && (
        <div style={S.modal}>
          <div style={{...S.mBox,maxWidth:400}}>
            <div style={{background:linear-gradient(135deg,${WAD},${WA}),color:"#fff",padding:"14px 16px",borderRadius:"8px 8px 0 0",margin:"-20px -20px 16px",fontWeight:700,textAlign:"center",fontSize:15}}>
              ?? WhatsApp Bulk Send
            </div>
            <div style={{fontSize:13,marginBottom:12}}>
              <b>{bulkSelected.length}</b> employees — <b>{MONTHS[month]} {year}</b> salary slips
            </div>
            <div style={{maxHeight:220,overflowY:"auto",marginBottom:12}}>
              {rows.filter(r=>bulkSelected.includes(r.emp.id)).map(({emp,calc})=>{
                const hasPhone = emp.mobile && emp.mobile.replace(/\D/g,"").length>=10;
                return (
                  <div key={emp.id} style={{display:"flex",justifyContent:"space-between",padding:"8px 10px",borderBottom:1px solid ${t.border},fontSize:12}}>
                    <div>
                      <span style={{fontWeight:600}}>{emp.name}</span>
                      <div style={{fontSize:10,color:hasPhone?t.subtext:t.red}}>?? {hasPhone?emp.mobile:"?? "+T.noMobile}</div>
                    </div>
                    <div style={{fontWeight:700,color:WA}}>?{calc.roundOff}</div>
                  </div>
                );
              })}
            </div>
            <div style={{background:t.rowAlt,borderRadius:8,padding:"10px 12px",fontSize:11,color:t.subtext,marginBottom:14}}>
              ?? Each WhatsApp chat opens one-by-one with a 2.5s gap. Allow pop-ups if blocked.
            </div>
            <div style={{display:"flex",gap:10,flexWrap:"wrap"}}>
              <button style={{...S.btn(WA),padding:"9px 20px",fontSize:13}} onClick={startBulkSend}>?? {T.startSending} ({bulkSelected.length})</button>
              <button style={S.btnOut} onClick={()=>setBulkModal(false)}>{T.cancel}</button>
            </div>
          </div>
        </div>
      )}

      {/* Bulk progress bar */}
      {bulkProgress && (
        <div style={S.progressBar}>
          <div style={{fontSize:20}}>??</div>
          <div style={{flex:1}}>
            <div style={{fontWeight:700,fontSize:13}}>Sending... {bulkProgress.current}/{bulkProgress.total}</div>
            <div style={{fontSize:11,opacity:0.85}}>{bulkProgress.name}</div>
            <div style={{marginTop:6,background:"rgba(255,255,255,0.3)",borderRadius:4,height:4}}>
              <div style={{background:"#fff",borderRadius:4,height:4,width:${(bulkProgress.current/bulkProgress.total)*100}%,transition:"width 0.4s"}}/>
            </div>
          </div>
        </div>
      )}
        <div style={S.modal}>
          <div style={S.mBox}>
            <h3 style={{margin:"0 0 14px",fontSize:15,color:t.navy}}>{editEmp??? ${T.editEmployee}:? ${T.addEmployee}}</h3>
            {[
              ["name",T.name,"text","Mr./Mrs. Name"],
              ["empId",T.empId,"text","RC001"],
              ["mobile",T.mobile,"tel","9876543210"],
              ["basicSalary",T.basicSalary,"number","15000"],
              ["workingDays",T.workingDays,"number","26"],
              ["joined",T.joined,"date",""],
            ].map(([field,label,type,ph])=>(
              <div key={field}>
                <label style={S.lbl}>{label}</label>
                <input style={S.mInp} type={type} placeholder={ph} value={empForm[field]||""} onChange={e=>setEmpForm(p=>({...p,[field]:e.target.value}))}/>
              </div>
            ))}
            {"dept",T.department,DEPTS],["designation",T.designation,DESIGNATIONS],["role",T.role,ROLES.map(([field,label,opts])=>(
              <div key={field}>
                <label style={S.lbl}>{label}</label>
                <select style={{...S.mInp,marginBottom:10}} value={empForm[field]||""} onChange={e=>setEmpForm(p=>({...p,[field]:e.target.value}))}>
                  {opts.map(o=><option key={o}>{o}</option>)}
                </select>
              </div>
            ))}
            <div style={{display:"flex",gap:10,marginTop:4}}>
              <button style={S.btn(t.navy)} onClick={saveEmp}>?? {T.save}</button>
              <button style={S.btnOut} onClick={()=>setEmpModal(false)}>{T.cancel}</button>
            </div>
          </div>
        </div>
      )}

      {/* Advance Modal */}
      {advModal && (
        <div style={S.modal}>
          <div style={S.mBox}>
            <h3 style={{margin:"0 0 14px",fontSize:15,color:t.navy}}>?? {T.giveAdvance}</h3>
            <label style={S.lbl}>Employee</label>
            <select style={{...S.mInp}} value={advForm.empId} onChange={e=>setAdvForm(p=>({...p,empId:e.target.value}))}>
              <option value="">Select Employee</option>
              {employees.map(e=><option key={e.id} value={e.id}>{e.name} ({e.dept})</option>)}
            </select>
            <label style={S.lbl}>{T.amount} (?)</label>
            <input style={S.mInp} type="number" value={advForm.amount} onChange={e=>setAdvForm(p=>({...p,amount:e.target.value}))} placeholder="Amount"/>
            <label style={S.lbl}>{T.reason}</label>
            <input style={S.mInp} value={advForm.reason} onChange={e=>setAdvForm(p=>({...p,reason:e.target.value}))} placeholder="Reason"/>
            <label style={S.lbl}>{T.date}</label>
            <input style={S.mInp} type="date" value={advForm.date} onChange={e=>setAdvForm(p=>({...p,date:e.target.value}))}/>
            <div style={{display:"flex",gap:10}}>
              <button style={S.btn(t.green)} onClick={giveAdvance}>? {T.save}</button>
              <button style={S.btnOut} onClick={()=>setAdvModal(false)}>{T.cancel}</button>
            </div>
          </div>
        </div>
      )}

      {/* Share Modal */}
      {shareModal && (
        <div style={S.modal}>
          <div style={{...S.mBox,maxWidth:380}}>
            <h3 style={{margin:"0 0 12px",fontSize:14,color:t.navy}}>?? {T.salarySlip} — {shareModal.emp.name}</h3>
            <div style={S.slipBox}>{slipText(shareModal.emp,shareModal.entry,shareModal.calc,shareModal.wd)}</div>
            <div style={{display:"flex",gap:10,flexWrap:"wrap"}}>
              <button style={S.btn("#25d366")} onClick={()=>doShare(shareModal.emp,shareModal.entry,shareModal.calc,shareModal.wd)}>
                {navigator.share?"?? WhatsApp / Share":"?? Copy"}
              </button>
              <button style={S.btnOut} onClick={()=>setShareModal(null)}>{T.close}</button>
            </div>
          </div>
        </div>
      )}

      {/* Salary Slip View Modal */}
      {slipModal && (
        <div style={S.modal}>
          <div style={{...S.mBox,maxWidth:360}}>
            <div style={{background:t.navy,color:"#fff",padding:"12px 16px",borderRadius:"8px 8px 0 0",margin:"-20px -20px 16px",fontWeight:700,textAlign:"center"}}>
              ?? SALARY SLIP
            </div>
            {[
              ["Employee",slipModal.emp.name],
              ["ID",slipModal.emp.empId],
              ["Department",slipModal.emp.dept],
              ["Designation",slipModal.emp.designation],
              ["Month",${MONTHS[month]} ${year}],
              ["Basic Salary",?${slipModal.emp.basicSalary}],
              ["Working Days",slipModal.wd],
              ["Present Days",slipModal.entry.presentDays||0],
              ["Salary for Month",?${slipModal.calc.salaryForMonth}],
              ["Collection",?${slipModal.entry.collection||0}],
              ["DB",?${slipModal.entry.db||0}],
              ["Gross Salary",?${slipModal.calc.grossSalary}],
              ["PF",?${slipModal.entry.pf||0}],
              ["GF",?${slipModal.entry.gf||0}],
              ["Advance",?${slipModal.entry.salaryAdvance||0}],
              ["Total Deduction",?${slipModal.calc.totalDeduction}],
            ].map(([k,v])=>(
              <div key={k} style={{display:"flex",justifyContent:"space-between",padding:"6px 0",borderBottom:1px solid ${t.border},fontSize:12}}>
                <span style={{color:t.subtext}}>{k}</span>
                <span style={{fontWeight:600,color:t.text}}>{v}</span>
              </div>
            ))}
            <div style={{display:"flex",justifyContent:"space-between",padding:"12px 0",fontSize:16,fontWeight:800,color:t.navy}}>
              <span>NET SALARY</span>
              <span>?{slipModal.calc.roundOff}</span>
            </div>
            <div style={{display:"flex",gap:10,marginTop:8}}>
              <button style={S.btn("#25d366")} onClick={()=>{setShareModal(slipModal);setSlipModal(null);}}>?? Share</button>
              <button style={S.btnOut} onClick={()=>setSlipModal(null)}>{T.close}</button>
            </div>
          </div>
        </div>
      )}

      {/* Toast */}
      {toast && <div style={S.toast}>{toast}</div>}
    </div>
  );
useEffect(() => {
  localStorage.setItem("rockcity_employees", JSON.stringify(employees));
}, [employees]);

useEffect(() => {
  localStorage.setItem("rockcity_salaryData", JSON.stringify(salaryData));
}, [salaryData]);

useEffect(() => {
  localStorage.setItem("rockcity_advanceData", JSON.stringify(advanceData));
}, [advanceData]);

useEffect(() => {
  localStorage.setItem("rockcity_attendanceData", JSON.stringify(attendanceData));
}, [attendanceData]);

useEffect(() => {
  const emp = localStorage.getItem("rockcity_employees");
  const sal = localStorage.getItem("rockcity_salaryData");
  const adv = localStorage.getItem("rockcity_advanceData");
  const att = localStorage.getItem("rockcity_attendanceData");

  if(emp) setEmployees(JSON.parse(emp));
  if(sal) setSalaryData(JSON.parse(sal));
  if(adv) setAdvanceData(JSON.parse(adv));
  if(att) setAttendanceData(JSON.parse(att));
}, []);useEffect(() => {
  const emp = localStorage.getItem("rockcity_employees");
  const sal = localStorage.getItem("rockcity_salaryData");
  const adv = localStorage.getItem("rockcity_advanceData");
  const att = localStorage.getItem("rockcity_attendanceData");

  if(emp) setEmployees(JSON.parse(emp));
  if(sal) setSalaryData(JSON.parse(sal));
  if(adv) setAdvanceData(JSON.parse(adv));
  if(att) setAttendanceData(JSON.parse(att));
}, []);
const sendAllEmployees = async () => {
  for (const { emp, entry, calc, wd } of rows) {
    if (emp.mobile) {
      const text = buildWASlip(
        emp,
        entry,
        calc,
        wd,
        month,
        year
      );

      openWA(emp.mobile, text);

      await new Promise(resolve =>
        setTimeout(resolve, 2500)
      );
    }
  }
};
