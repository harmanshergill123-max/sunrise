
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sunrise Multiflix Solutions</title>

<style>
body{
font-family:Arial;
margin:0;
background:#f5f5f5
}

header{
background:#222;
color:white;
padding:20px;
text-align:center
}

section{
padding:40px;
max-width:1000px;
margin:auto
}

.services{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
gap:20px
}

.card{
background:white;
padding:20px;
border-radius:8px;
box-shadow:0 2px 8px rgba(0,0,0,0.1)
}

form input,form select,form textarea{
width:100%;
padding:10px;
margin-top:10px
}

button{
margin-top:10px;
padding:10px 20px;
background:#ff9800;
border:none;
color:white;
cursor:pointer
}

.dashboard{
background:white;
padding:20px;
border-radius:8px
}

table{
width:100%;
border-collapse:collapse
}

th,td{
padding:10px;
border-bottom:1px solid #ddd
}

th{
background:#eee
}

footer{
text-align:center;
padding:20px;
background:#222;
color:white
}

</style>
</head>

<body>

<header>
<h1>Sunrise Multiflix Solutions</h1>
<p>Drywall • Painting • Flooring • Roofing | Serving Alberta</p>
</header>

<section>
<h2>Our Services</h2>

<div class="services">

<div class="card">
<h3>Drywall</h3>
<p>Professional drywall installation and repair for homes and businesses.</p>
</div>

<div class="card">
<h3>Painting</h3>
<p>Interior and exterior painting services with high quality finishes.</p>
</div>

<div class="card">
<h3>Flooring</h3>
<p>Laminate, vinyl, hardwood and tile flooring installation.</p>
</div>

<div class="card">
<h3>Roofing</h3>
<p>Reliable roofing repair and installation services.</p>
</div>

</div>
</section>

<section>
<h2>Book an Appointment</h2>

<form id="bookingForm">

<input type="text" placeholder="Full Name" id="name" required>

<input type="email" placeholder="Email" id="email" required>

<input type="text" placeholder="Phone" id="phone" required>

<select id="service">
<option>Drywall</option>
<option>Painting</option>
<option>Flooring</option>
<option>Roofing</option>
</select>

<input type="datetime-local" id="date">

<textarea id="notes" placeholder="Project details"></textarea>

<button type="submit">Book Appointment</button>

</form>

<p id="msg"></p>

</section>

<section>
<h2>Service Provider Dashboard</h2>

<div class="dashboard">

<table>

<thead>
<tr>
<th>Name</th>
<th>Service</th>
<th>Date</th>
<th>Status</th>
<th>Action</th>
</tr>
</thead>

<tbody id="appointments"></tbody>

</table>

</div>

</section>

<footer>
© Sunrise Multiflix Solutions - Alberta
</footer>

<script>

let appointments = JSON.parse(localStorage.getItem("appointments") || "[]")

function save(){
localStorage.setItem("appointments",JSON.stringify(appointments))
render()
}

function render(){

let table = document.getElementById("appointments")

 table.innerHTML=""

 appointments.forEach((a,i)=>{

 table.innerHTML += `

 <tr>
 <td>${a.name}</td>
 <td>${a.service}</td>
 <td>${a.date || ""}</td>
 <td>${a.status}</td>
 <td>
 <button onclick="confirmJob(${i})">Confirm</button>
 <button onclick="completeJob(${i})">Complete</button>
 <button onclick="removeJob(${i})">Delete</button>
 </td>
 </tr>

 `

 })

}

function confirmJob(i){
appointments[i].status="Confirmed"
save()
}

function completeJob(i){
appointments[i].status="Completed"
save()
}

function removeJob(i){
appointments.splice(i,1)
save()
}


document.getElementById("bookingForm").addEventListener("submit",function(e){

e.preventDefault()

let booking={
name:document.getElementById("name").value,
email:document.getElementById("email").value,
phone:document.getElementById("phone").value,
service:document.getElementById("service").value,
date:document.getElementById("date").value,
notes:document.getElementById("notes").value,
status:"Pending"
}

appointments.unshift(booking)

save()

this.reset()

 document.getElementById("msg").innerText="Appointment request submitted"

})

render()

</script>

</body>
</html>



