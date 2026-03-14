# Sunrise Multiflix Solutions — Website + Backend

This repository contains a complete starter website and backend for a construction company based in Alberta offering **drywall, painting, flooring, and roofing** services, with appointment booking backed by a SQLite database.

---

## Project structure

```
sunrise-multiflix-solutions/
├─ frontend/              # React + Tailwind frontend
│  ├─ package.json
│  └─ src/
│     └─ App.jsx
├─ backend/               # Express + sqlite3 backend API
│  ├─ package.json
│  └─ server.js
└─ README.md
```

---

## Features included

- Homepage with services (Drywall, Painting, Flooring, Roofing).
- Booking form (name, email, phone, address, service, preferred date/time, notes).
- Simple admin view to list appointments.
- Backend API built with Express and SQLite (persistent appointments database).
- CORS-enabled and ready to run locally or deploy to a small VPS.

---

## Frontend (React + Tailwind)

Create `frontend/package.json` and `frontend/src/App.jsx`.

### frontend/package.json

```json
{
  "name": "sunrise-frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "tailwindcss": "^3.0.0",
    "vite": "^4.0.0"
  },
  "scripts": {
    "dev": "vite"
  }
}
```

### frontend/src/App.jsx

```jsx
import React, { useState, useEffect } from 'react';

const SERVICES = [
  'Drywall',
  'Painting',
  'Flooring',
  'Roofing'
];

export default function App(){
  const [form, setForm] = useState({ name: '', email: '', phone: '', address: '', service: SERVICES[0], datetime: '', notes: '' });
  const [message, setMessage] = useState('');
  const [appointments, setAppointments] = useState([]);

  useEffect(()=>{
    // fetch appointments for admin list (read-only client display)
    fetch('/api/appointments').then(r=>r.json()).then(data=>{
      if(Array.isArray(data)) setAppointments(data);
    }).catch(()=>{});
  }, []);

  function handleChange(e){
    setForm({...form, [e.target.name]: e.target.value});
  }

  async function handleSubmit(e){
    e.preventDefault();
    setMessage('Submitting...');
    try{
      const res = await fetch('/api/appointments', {
        method: 'POST',
        headers: {'Content-Type':'application/json'},
        body: JSON.stringify(form)
      });
      const json = await res.json();
      if(res.ok){
        setMessage('Appointment requested — we will contact you soon!');
        setForm({ name: '', email: '', phone: '', address: '', service: SERVICES[0], datetime: '', notes: '' });
        setAppointments(prev=>[json, ...prev]);
      } else {
        setMessage(json.error || 'Failed to request appointment');
      }
    } catch(err){
      setMessage('Network error — please try again');
    }
  }

  return (
    <div className="min-h-screen bg-gray-50 p-6 font-sans">
      <header className="max-w-4xl mx-auto mb-8">
        <h1 className="text-3xl font-bold">Sunrise Multiflix Solutions</h1>
        <p className="text-sm text-gray-600">Drywall · Painting · Flooring · Roofing — Serving Alberta</p>
      </header>

      <main className="max-w-4xl mx-auto grid gap-8 md:grid-cols-2">
        <section className="bg-white p-6 rounded shadow">
          <h2 className="text-xl font-semibold mb-4">Our Services</h2>
          <ul className="space-y-3">
            {SERVICES.map(s=> (
              <li key={s} className="border-l-4 border-indigo-500 pl-3">
                <strong>{s}</strong>
                <p className="text-sm text-gray-600">Professional {s.toLowerCase()} services for residential and commercial projects.</p>
              </li>
            ))}
          </ul>
        </section>

        <section className="bg-white p-6 rounded shadow">
          <h2 className="text-xl font-semibold mb-4">Book an Appointment</h2>
          <form onSubmit={handleSubmit} className="space-y-3">
            <input name="name" value={form.name} onChange={handleChange} placeholder="Full name" required className="w-full p-2 border rounded" />
            <input name="email" value={form.email} onChange={handleChange} placeholder="Email" type="email" required className="w-full p-2 border rounded" />
            <input name="phone" value={form.phone} onChange={handleChange} placeholder="Phone" required className="w-full p-2 border rounded" />
            <input name="address" value={form.address} onChange={handleChange} placeholder="Address" className="w-full p-2 border rounded" />
            <select name="service" value={form.service} onChange={handleChange} className="w-full p-2 border rounded">
              {SERVICES.map(s => <option key={s} value={s}>{s}</option>)}
            </select>
            <input name="datetime" value={form.datetime} onChange={handleChange} type="datetime-local" className="w-full p-2 border rounded" />
            <textarea name="notes" value={form.notes} onChange={handleChange} placeholder="Notes" className="w-full p-2 border rounded" />
            <div className="flex items-center gap-3">
              <button type="submit" className="px-4 py-2 rounded bg-indigo-600 text-white">Request Appointment</button>
              <div className="text-sm text-gray-600">{message}</div>
            </div>
          </form>
        </section>

        <section className="md:col-span-2 bg-white p-6 rounded shadow">
          <h2 className="text-xl font-semibold mb-4">Recent Appointment Requests</h2>
          <ul className="space-y-3">
            {appointments.map(a => (
              <li key={a.id} className="p-3 border rounded">
                <div className="flex justify-between"><strong>{a.name}</strong><span className="text-sm text-gray-600">{a.service}</span></div>
                <div className="text-sm text-gray-600">{a.email} · {a.phone}</div>
                <div className="text-sm mt-1">{a.datetime ? new Date(a.datetime).toLocaleString() : 'No date chosen'}</div>
                <div className="text-sm text-gray-700 mt-2">{a.notes}</div>
              </li>
            ))}
            {appointments.length===0 && <li className="text-sm text-gray-600">No appointment requests yet.</li>}
          </ul>
        </section>
      </main>

      <footer className="max-w-4xl mx-auto mt-8 text-center text-sm text-gray-500">© Sunrise Multiflix Solutions — Alberta</footer>
    </div>
  );
}
```

---

## Backend (Express + SQLite)

Create `backend/package.json` and `backend/server.js`.

### backend/package.json

```json
{
  "name": "sunrise-backend",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "sqlite3": "^5.1.6",
    "cors": "^2.8.5",
    "body-parser": "^1.20.1"
  },
  "scripts": {
    "start": "node server.js"
  }
}
```

### backend/server.js

```js
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const bodyParser = require('body-parser');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(bodyParser.json());

const db = new sqlite3.Database('./appointments.db');

// initialize table
db.serialize(() => {
  db.run(`CREATE TABLE IF NOT EXISTS appointments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    phone TEXT,
    address TEXT,
    service TEXT,
    datetime TEXT,
    notes TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )`);
});

app.get('/api/appointments', (req, res) => {
  db.all('SELECT * FROM appointments ORDER BY created_at DESC LIMIT 100', (err, rows) => {
    if(err) return res.status(500).json({ error: 'DB error' });
    res.json(rows);
  });
});

app.post('/api/appointments', (req, res) => {
  const { name, email, phone, address, service, datetime, notes } = req.body;
  if(!name || !email) return res.status(400).json({ error: 'name and email required' });
  const stmt = db.prepare('INSERT INTO appointments (name,email,phone,address,service,datetime,notes) VALUES (?,?,?,?,?,?,?)');
  stmt.run(name,email,phone,address,service,datetime,notes, function(err){
    if(err) return res.status(500).json({ error: 'DB insert failed' });
    const id = this.lastID;
    db.get('SELECT * FROM appointments WHERE id = ?', [id], (err,row)=>{
      if(err) return res.status(500).json({ error: 'DB fetch failed' });
      res.status(201).json(row);
    });
  });
  stmt.finalize();
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, ()=> console.log(`Server running on port ${PORT}`));
```

---

## Running locally

1. Clone the project folder.
2. Start the backend:

```bash
cd backend
npm install
node server.js
```

3. Start the frontend (from `/frontend`):

```bash
cd ../frontend
npm install
npm run dev
```

4. Edit and deploy: configure a reverse proxy (nginx) or use a platform like Render/Heroku. Ensure the frontend fetch calls point to the backend URL in production.

---

## Notes & next steps

- This is a starter template. Consider adding:
  - Email notifications on new bookings (e.g., nodemailer).
  - Authentication for admin views.
  - Validation & spam protection (recaptcha).
  - Photo gallery and portfolio pages inspired by an existing site.

---

## Service Provider Dashboard (Admin Panel)

A simple dashboard can allow the company team to view, manage, and update service appointments.

### Features

- Secure login for service providers
- View all bookings
- Update appointment status (Pending / Confirmed / Completed)
- Delete spam or cancelled bookings

---

### Backend additions (Admin + status field)

Update the database table to include status:

```js
ALTER TABLE appointments ADD COLUMN status TEXT DEFAULT 'Pending';
```

Add new routes in `server.js`:

```js
// Update appointment status
app.put('/api/appointments/:id', (req,res)=>{
  const { status } = req.body;

  db.run(
    'UPDATE appointments SET status=? WHERE id=?',
    [status, req.params.id],
    function(err){
      if(err) return res.status(500).json({error:'Update failed'});
      res.json({success:true});
    }
  );
});

// Delete appointment
app.delete('/api/appointments/:id',(req,res)=>{
  db.run('DELETE FROM appointments WHERE id=?',[req.params.id],function(err){
    if(err) return res.status(500).json({error:'Delete failed'});
    res.json({success:true});
  });
});
```

---

### Admin Dashboard React Component

Create `frontend/src/Dashboard.jsx`

```jsx
import React, {useEffect,useState} from 'react';

export default function Dashboard(){

 const [appointments,setAppointments]=useState([]);

 useEffect(()=>{
  load();
 },[]);

 async function load(){
  const res = await fetch('/api/appointments');
  const data = await res.json();
  setAppointments(data);
 }

 async function updateStatus(id,status){
  await fetch('/api/appointments/'+id,{
   method:'PUT',
   headers:{'Content-Type':'application/json'},
   body:JSON.stringify({status})
  });

  load();
 }

 async function remove(id){
  await fetch('/api/appointments/'+id,{method:'DELETE'});
  load();
 }

 return (
  <div className="p-10">
   <h1 className="text-2xl font-bold mb-6">Service Provider Dashboard</h1>

   <table className="w-full border">
    <thead>
     <tr className="bg-gray-100">
      <th>Name</th>
      <th>Service</th>
      <th>Date</th>
      <th>Status</th>
      <th>Actions</th>
     </tr>
    </thead>

    <tbody>

     {appointments.map(a=>(
      <tr key={a.id} className="border-t">

       <td>{a.name}</td>
       <td>{a.service}</td>
       <td>{a.datetime}</td>

       <td>{a.status}</td>

       <td className="space-x-2">

        <button onClick={()=>updateStatus(a.id,'Confirmed')}>
         Confirm
        </button>

        <button onClick={()=>updateStatus(a.id,'Completed')}>
         Complete
        </button>

        <button onClick={()=>remove(a.id)}>
         Delete
        </button>

       </td>

      </tr>
     ))}

    </tbody>

   </table>
  </div>
 );
}
```

---

### Dashboard Benefits

The service provider dashboard allows **Sunrise Multiflix Solutions** staff to:

- Manage drywall, painting, flooring, and roofing service requests
- Track customer bookings
- Confirm or complete jobs
- Remove spam bookings

---

Thank you — this template is ready to be customized for brand colors, copy, assets, and deployment for Sunrise Multiflix Solutions in Alberta.


