# project2
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>MediBook – Smart Hospital Booking</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<style>
:root{
  --primary:#4f46e5;
  --secondary:#22c55e;
  --bg1:#eef2ff;
  --bg2:#ecfeff;
  --glass:rgba(255,255,255,.65);
}

*{box-sizing:border-box;font-family:'Poppins',sans-serif;margin:0;padding:0}
body{min-height:100vh;background:linear-gradient(135deg,var(--bg1),var(--bg2));color:#1f2937}

header{
  background:linear-gradient(135deg,var(--primary),var(--secondary));
  color:#fff;padding:28px 20px;text-align:center;border-radius:0 0 35px 35px;
  box-shadow:0 20px 40px rgba(0,0,0,.15);
}
header h1{font-size:30px;font-weight:700}
header p{opacity:.9;font-size:14px}

.page{display:none;padding:22px}
.page.active{display:block;animation:slideUp .6s ease}

.card{
  background:var(--glass);backdrop-filter:blur(14px);
  border-radius:22px;padding:22px;margin-bottom:20px;
  box-shadow:0 25px 50px rgba(0,0,0,.12);
  transition:.4s;
}
.card:hover{transform:translateY(-6px)}

button{
  width:100%;padding:16px;border:none;border-radius:18px;
  background:linear-gradient(135deg,var(--primary),var(--secondary));
  color:#fff;font-size:16px;font-weight:600;cursor:pointer;
}

input,select,textarea{
  width:100%;padding:15px;margin:10px 0;border-radius:16px;border:none;
  background:rgba(255,255,255,.85);
}

.slot{
  padding:16px;border-radius:16px;margin:12px 0;text-align:center;
  background:linear-gradient(135deg,#f1f5f9,#e0f2fe);
  font-weight:600;cursor:pointer;
}
.slot:hover{background:linear-gradient(135deg,var(--primary),var(--secondary));color:#fff}

.loader{
  width:60px;height:60px;border:6px solid #ddd;border-top:6px solid var(--secondary);
  border-radius:50%;margin:30px auto;animation:spin 1s linear infinite;
}

@keyframes spin{to{transform:rotate(360deg)}}
@keyframes slideUp{from{opacity:0;transform:translateY(30px)}to{opacity:1;transform:translateY(0)}}
</style>
</head>

<body>

<header>
  <h1>🩺 MediBook</h1>
  <p>Smart • Fast • Trusted Doctor Booking</p>
</header>

<!-- HOME -->
<div class="page active" id="home">
  <div class="card">
    <h3>Book Doctor Appointments</h3>
    <p>Book hospital appointments easily from your mobile.</p>
  </div>
  <button onclick="go('patient')">Start Booking</button>
</div>

<!-- PATIENT -->
<div class="page" id="patient">
  <h3>Patient Details</h3>
  <input id="pname" placeholder="Patient Name">
  <input id="age" type="number" placeholder="Age">
  <select id="gender">
    <option value="">Select Gender</option>
    <option>Male</option>
    <option>Female</option>
  </select>
  <textarea id="problem" placeholder="Health Problem"></textarea>
  <button onclick="validatePatient()">Next</button>
</div>

<!-- HOSPITAL -->
<div class="page" id="hospital">
  <h3>Select Hospital</h3>
  <div class="card" onclick="selectHospital('City Care Hospital')">🏥 City Care Hospital</div>
  <div class="card" onclick="selectHospital('LifeLine Hospital')">🏥 LifeLine Hospital</div>
</div>

<!-- DOCTOR -->
<div class="page" id="doctor">
  <h3>Select Doctor</h3>

  <!-- READY EMPTY BLOCK (YOU CAN ADD MORE DOCTORS LATER) -->
  <div class="card" onclick="selectDoctor('Dr. Sanjay','Cardiologist','9392984011')">
    👨‍⚕️ <b>Dr. Sanjay</b><br>
    <small>Cardiologist • 10 Years</small>
  </div>

  <div class="card" onclick="selectDoctor('Dr. Abdul','Physician','9392984011')">
    👨‍⚕️ <b>Dr. Abdul</b><br>
    <small>Physician • 8 Years</small>
  </div>

</div>

<!-- SLOT -->
<div class="page" id="slot">
  <h3>Select Slot</h3>
  <div class="slot" onclick="bookSlot('10:00 AM')">10:00 AM</div>
  <div class="slot" onclick="bookSlot('12:00 PM')">12:00 PM</div>
  <div class="slot" onclick="bookSlot('4:00 PM')">4:00 PM</div>
</div>

<!-- PENDING -->
<div class="page" id="pending">
  <div class="card" style="text-align:center">
    <div class="loader"></div>
    <p>Waiting for doctor approval...</p>
  </div>
</div>

<!-- PAYMENT -->
<div class="page" id="payment">
  <div class="card">
    <h3>Appointment Approved ✅</h3>
    <p>Consultation Fee: ₹300</p>
    <button onclick="pay()">Pay via PhonePe</button>
  </div>
</div>

<!-- CONFIRM -->
<div class="page" id="confirm">
  <div class="card">
    <h3>🎉 Appointment Confirmed</h3>
    <p id="summary"></p>
  </div>
  <button onclick="go('home')">Book Again</button>
</div>

<script>
/* FIREBASE (ADD YOUR DETAILS LATER) */
firebase.initializeApp({
  apiKey:"PASTE_HERE",
  authDomain:"PASTE_HERE",
  projectId:"PASTE_HERE"
});
const db = firebase.firestore();

let data={};

function go(p){
  document.querySelectorAll('.page').forEach(x=>x.classList.remove('active'));
  document.getElementById(p).classList.add('active');
}

function validatePatient(){
  if(!pname.value||!age.value||!gender.value||!problem.value){
    alert("Fill all fields"); return;
  }
  data={name:pname.value,age:age.value,gender:gender.value,problem:problem.value};
  go('hospital');
}

function selectHospital(h){data.hospital=h;go('doctor')}
function selectDoctor(d,s,phone){data.doctor=d;data.phone=phone;go('slot')}

function bookSlot(t){
  data.slot=t;
  db.collection("appointments").add(data);

  const msg=`New Appointment\nPatient: ${data.name}\nSlot: ${t}`;
  window.open(`https://wa.me/${data.phone}?text=${encodeURIComponent(msg)}`,'_blank');

  go('pending');
  setTimeout(()=>go('payment'),4000);
}

/* PHONEPE PAYMENT */
function pay(){
  const hospitalUPI="hospital@upi";   // CHANGE LATER
  const amount=300;
  window.location.href=
    `upi://pay?pa=${hospitalUPI}&pn=MediBook Hospital&am=${amount}&cu=INR`;

  setTimeout(()=>{
    summary.innerHTML=
      `<b>Patient:</b> ${data.name}<br>
       <b>Hospital:</b> ${data.hospital}<br>
       <b>Doctor:</b> ${data.doctor}<br>
       <b>Slot:</b> ${data.slot}<br>
       <b>Status:</b> Paid`;
    go('confirm');
  },3000);
}
</script>

</body>
</html>  i want create a url for this code 
